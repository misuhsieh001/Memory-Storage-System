# 07. SSD vs OCSSD vs ZNS SSD 技術筆記

這份筆記整理自課程投影片，探討固態硬碟（SSD）架構如何從傳統的黑盒子設計，演變為讓主機端（Host）擁有更多控制權的開放架構，重點在於 **ZNS (Zoned Namespace)** 技術。

---

## 1. SSD 架構的演進 (Evolution of SSDs)

SSD 的發展光譜在於「Flash 管理工作（FTL）」究竟該由 **裝置端 (Device)** 負責，還是 **主機端 (Host)** 負責。

### 1.1 Conventional SSD (傳統 SSD) - "Black-box"
* [cite_start]**特性**：黑盒子設計。所有的 Flash 管理（FTL、GC、磨損平衡）都在 SSD 內部控制器完成 [cite: 6-7]。
* **介面**：模擬成傳統的 Block Device，主機只看得到 LBA (Logical Block Address)。
* **缺點**：主機無法控制內部的 GC 時機，容易產生延遲不可預測的問題（Latency Spikes）。

### 1.2 Open-Channel SSD (OCSSD) - "White-box"
* [cite_start]**特性**：白盒子設計。將 Flash 的物理細節（Chunks, Parallel Units）完全暴露給主機 [cite: 8, 65]。
* [cite_start]**管理**：FTL 幾乎全移到主機端 (Host-side FTL)。主機負責資料配置、磨損平衡和錯誤處理 [cite: 15, 66]。
* **優點**：極致的效能隔離 (Performance Isolation)。
* **缺點**：主機端軟體開發負擔過重，且不同廠商規格可能不統一。

### 1.3 Zoned Namespaces SSD (ZNS SSD) - "Grey-box"
* [cite_start]**特性**：灰盒子設計。這是目前的甜蜜點，達成主機與裝置的協同合作 [cite: 10, 48]。
* **分工**：
    * [cite_start]**主機 (Host)**：負責高層決策，如資料配置 (Data Placement)、I/O 排程 [cite: 11]。
    * [cite_start]**裝置 (Device)**：負責底層物理管理，如讀寫錯誤處理、Flash 可靠性維護 [cite: 11]。

---

## 2. ZNS SSD 核心機制 (ZNS Mechanics)

[cite_start]ZNS 遵循 **NVMe** 規範，將儲存空間劃分為多個等長的 **Zones (區域)** [cite: 87, 97]。

### 2.1 Zone 的寫入規則
* **循序寫入 (Sequential Write Only)**：每個 Zone 必須從頭開始循序寫入。
* [cite_start]**Write Pointer (WP)**：SSD 內部維護一個寫入指針，指向該 Zone 下一個可寫入的 LBA 位置 [cite: 99]。
* [cite_start]**Reset**：Zone 寫滿後必須重設 (Reset) 才能再次寫入（相當於 Erase Block）[cite: 100]。

### 2.2 Zone 的狀態機 (Zone States)
[cite_start]ZNS 定義了嚴格的 Zone 生命週期管理 [cite: 164, 185-201]：

* **ZSE (Empty)**：空的，WP 在開頭。
* **ZSIO / ZSEO (Implicitly/Explicitly Opened)**：正在被寫入的狀態。
    * [cite_start]**MOR (Max Open Resources)**：同時能處於 Open 狀態的 Zone 數量有限制 [cite: 179]。
* **ZSC (Closed)**：暫時關閉，釋放 Open 資源，但資料還在。
* **ZSF (Full)**：寫滿了。
* **ZSRO (Read Only)**：唯讀（通常是壽命快到了）。
* **ZSO (Offline)**：壞損不可用。

### 2.3 隨機寫入緩衝區：ZRWA (Zone Random Write Area)
[cite_start]為了解決 ZNS 強制循序寫入的限制，規範定義了可選的 **ZRWA** [cite: 204]。
* **概念**：掛在 Zone 旁邊的一塊暫存區（通常在 SSD 內部的 Buffer）。
* **功能**：允許在 ZRWA 內進行 **隨機寫入 (Random Write)** 或 **原地更新 (In-place Update)**。
* [cite_start]**Flush**：當資料在 ZRWA 整理好後，透過 Flush 指令，將其轉移到真正的 Zone 儲存區（變為循序資料）[cite: 220-234]。


---

## 3. ZNS 的軟體存取與寫入模式

### 3.1 軟體堆疊 (Software Stack)
[cite_start]要使用 ZNS，作業系統和應用程式需要支援 [cite: 247-263]：
* **檔案系統**：F2FS, Btrfs (支援 ZBD - Zoned Block Device)。
* **中介層**：`dm-zoned` (為了相容傳統檔案系統)。
* **應用程式**：RocksDB, Ceph (透過 `libzbd` 直接控制)。

### 3.2 兩種寫入指令
1.  **Classical ZNS Write**：
    * 主機指定 LBA 寫入。
    * [cite_start]**缺點**：多個執行緒同時寫入同一個 Zone 時，必須鎖定 (Lock) 以確保順序，限制了併發效能 [cite: 280-282]。
2.  **Zone Append** (推薦)：
    * [cite_start]**無名寫入 (Nameless Write)**：主機只指定「寫入哪個 Zone」，不指定 LBA [cite: 278]。
    * SSD 接收資料後，決定寫在該 Zone 的哪個位置，並回傳寫入的 LBA。
    * [cite_start]**優點**：解決了併發寫入時的鎖競爭問題，大幅提升平行度 [cite: 283]。

---

## 4. 總結比較表

| 特性 | Conventional SSD | OCSSD | ZNS SSD |
| :--- | :--- | :--- | :--- |
| **類型** | **Black-box** (黑盒子) | **White-box** (白盒子) | **Grey-box** (灰盒子) |
| **FTL 位置** | Device | Host | 混合 (Host管配置, Device管物理) |
| **介面** | Block Device (LBA) | Physical Chunk / PU | Zoned Namespace (Zone) |
| **寫入限制** | 無 (可隨機寫入) | 需配合物理特性 | **Zone 內強制循序寫入** |
| **DRAM 需求** | 高 (存 L2P Table) | 低 (Host 負責) | [cite_start]**低** (Table 變小) [cite: 239] |
| **效能隔離** | 差 (內部 GC 干擾) | 佳 (Host 完全控制) | [cite_start]**佳** (透過 Zone 隔離 I/O) [cite: 32] |
| **OP (預留空間)**| 需要大量 | Host 決定 | [cite_start]**減少需求** [cite: 241] |

**ZNS 的核心價值**：它移除了 SSD 內部龐大的 L2P (Logical-to-Physical) 映射表需求，降低了 DRAM 成本，同時消除了 SSD 內部的寫入放大 (Write Amplification)，因為循序寫入不需要像傳統 SSD 那樣頻繁做 GC 搬運資料。