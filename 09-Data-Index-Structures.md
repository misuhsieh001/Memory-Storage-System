# 09. Data Index Structures: LSM-tree, LevelDB, WiscKey, SLM-DB, Be-tree 重點整理

這份筆記整理自課程投影片，涵蓋了現代 Key-Value Store 儲存結構的演進與核心原理。

---

## 1. Log-Structured Merge-Tree (LSM-tree)
這是所有後續變體的始祖，核心思想是「用空間換取寫入時間」。

* **核心概念**：為了優化寫入效能，放棄了 B-tree 的「原地更新 (In-place update)」，改採 **「異地追加 (Append-only)」**。所有的寫入都視為順序寫入 (Sequential Write)。
* **架構組成**：
    * **C0 (Memory)**：駐留在記憶體中的排序樹（如 AVL 或 SkipList）。所有新的寫入都先到這裡。
    * **C1 ... Ck (Disk)**：駐留在硬碟上的排序樹（通常是 B-tree 結構，但唯讀）。
* **運作流程**：
    1.  資料寫入 C0。
    2.  當 C0 滿了，與硬碟上的 C1 進行 **Merge Sort (合併排序)**，寫成新的 C1。
    3.  當 Ci 滿了，再往下與 Ci+1 合併。
* **優點**：寫入吞吐量極高（循序寫入）。
* **缺點**：讀取效能較差（可能要查多層）、寫入放大 (Write Amplification) 嚴重。

---

## 2. LevelDB (Google 的 LSM 實作)
LSM-tree 的工業級標準實作，定義了現代 KV Store 的分層架構。

* **架構組成**：
    * **MemTable & Immutable MemTable**：記憶體中的 SkipList。Immutable 是為了防止寫入停頓 (Write Stall) 而設計的緩衝。
    * **Level 0 (L0)**：直接由記憶體 Flush 下來的 SSTable 檔案。**特點是 Key 會重疊 (Overlapping)**，讀取時可能要搜尋多個檔案。
    * **Level 1 ~ Level 6**：經過 Compaction 整理的層級。**特點是 Key 絕對不重疊**，且容量隨層級指數增長 (10x)。
* **Compaction 機制**：
    * **Minor Compaction**：MemTable → L0。
    * **Major Compaction**：Li 的檔案與 Li+1 重疊的檔案進行合併排序，消除無效數據。
* **讀取路徑**：MemTable → Immutable → L0 → L1... → L6。L0 是效能瓶頸。

---

## 3. WiscKey (針對 SSD 的鍵值分離)
為了解決 LSM-tree 頻繁搬運資料導致的「寫入放大」問題，特別針對 SSD 特性設計。

* **核心痛點**：LevelDB 在 Compaction 時，會不斷搬運 `<Key, Value>`。如果 Value 很大，這是在浪費頻寬，因為我們只需要排序 Key。
* **解決方案：Key-Value Separation (鍵值分離)**
    * **LSM-tree 只存 `<Key, Address>`**：讓 LSM-tree 變得很小，Compaction 飛快。
    * **vLog (Value Log) 存 `<Key, Value>`**：真正的資料存在一個巨大的 Log 檔中，只做 Append。
* **挑戰與對策**：
    * **範圍搜尋 (Range Query) 變慢**：因為 Value 在 vLog 裡是亂序的。解法是利用 SSD 的 **平行讀取 (Parallel Random Reads)** 特性來加速。
    * **垃圾回收 (GC)**：需要額外機制清理 vLog 中的無效資料（讀取舊資料 → 檢查是否有效 → 搬運到 Head）。

---

## 4. SLM-DB (Single-Level Merge with PM)
針對 **Persistent Memory (PM, 持久記憶體)** 硬體特性設計的極簡架構。

* **核心痛點**：傳統多層級結構是為了掩蓋硬碟隨機寫入慢的缺點，但在 PM 時代，這造成了不必要的寫入放大。
* **架構創新**：
    * **No WAL**：利用 PM 的持久性，不需要 Write-Ahead Log。
    * **Global B+ Tree (on PM)**：在 PM 上維護一個全域索引，直接指向硬碟上的資料位置。
    * **Single Level Disk (L0 only)**：硬碟上只有一層。資料寫入後不再層層搬運。
* **合併機制 (Self-Merging)**：
    * 不再是上下層合併，而是同一層挑選幾個檔案合併，整理完放回同一層。
* **優點**：極低的寫入放大，讀寫效能都大幅超越 LevelDB。

---

## 5. Be-tree (B-epsilon Tree)
B-tree 的改良版，試圖在 B-tree 的讀取優勢和 LSM-tree 的寫入優勢間取得平衡。

* **核心結構**：
    * 在每個 B-tree 的內部節點 (Internal Node) 增加一個巨大的 **Buffer (緩衝區)**。
* **運作流程 (Message Passing)**：
    * 寫入（Insert/Delete）被視為一條 **Message**，先暫存在 Root Buffer。
    * 當 Buffer 滿了，訊息會像瀑布一樣 **Flush** 到子節點的 Buffer。
    * 直到推送到 Leaf Node 才真正寫入資料。
* **優點**：比傳統 B-tree 寫入快很多（因為是批次寫入），且比 LSM-tree 的讀取更穩定（保留了樹狀結構的搜尋優勢）。

---

## 6. 綜合比較表

| 特性 | LevelDB (LSM-tree) | WiscKey | SLM-DB | Be-tree |
| :--- | :--- | :--- | :--- | :--- |
| **核心哲學** | **分層合併**<br>(記憶體緩衝 + 多層硬碟) | **鍵值分離**<br>(LSM 存 Key, vLog 存 Value) | **單層儲存**<br>(PM 存索引 + Disk 單層) | **緩衝樹**<br>(B-tree + 節點緩衝區) |
| **硬體優化對象** | HDD / 通用 SSD | **SSD** (利用平行讀取) | **Persistent Memory (PM)** | 通用 / HDD |
| **寫入策略** | Append-only, 多層 Compaction | Append-only, Key 做 Compaction | Append-only, 同層 Self-Merging | 寫入 Buffer, 滿了往下 Flush |
| **寫入放大 (WA)** | **高** (資料搬運多次) | **低** (只搬運 Key) | **極低** (資料不換層) | **中低** (優於 B-tree) |
| **讀取策略** | 查 Mem → L0 → L1...<br>(需 Bloom Filter) | 查 LSM 拿地址 → 讀 vLog<br>(隨機讀) | 查 PM B+ Tree → 直讀 Disk | 沿樹搜尋<br>(需檢查路徑上的 Buffer) |
| **範圍搜尋** | **優秀** (資料大致有序) | **較差** (需依賴 SSD 平行讀取補救) | **優秀** (有全局 B+ Tree) | **優秀** (本質是 B-tree) |
| **主要缺點** | L0 瓶頸、Compaction 耗資源 | GC 複雜、範圍搜尋依賴硬體 | 需要昂貴的 PM 硬體 | 實作複雜度高 |
| **適用場景** | 寫多讀少、通用 KV Store | Value 較大、使用 SSD 的場景 | 高效能需求、有 PM 設備 | 讀寫混合、需穩定效能 |