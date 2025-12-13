# Advanced Storage Technologies & Data Structures 完整圖解與比較筆記 (全集)

這份筆記是針對現代與未來儲存技術的終極指南。內容包含核心概念、運作機制、深層設計哲學、圖解輔助，以及**各章節的詳細技術比較表**。

---

## 目錄 (Table of Contents)

1.  [Chapter 05: Racetrack Memory (賽道記憶體)](#1-chapter-05-racetrack-memory)
2.  [Chapter 06: Hard Disk Drives & SMR (硬碟與疊瓦式磁記錄)](#2-chapter-06-hard-disk-drives--smr)
3.  [Chapter 07: SSD & ZNS (固態硬碟與分區命名空間)](#3-chapter-07-ssd--zns)
4.  [Chapter 08: Future Storages (未來儲存：玻璃與 DNA)](#4-chapter-08-future-storages)
5.  [Chapter 09: Data Index Structures (1): LSM-tree 家族](#5-chapter-09-data-index-structures-1-lsm-tree-family)
6.  [Chapter 10: Data Index Structures (2): Hash & Learned Indexes](#6-chapter-10-data-index-structures-2-hash--learned-indexes)

---

## 1. Chapter 05: Racetrack Memory (賽道記憶體)

### 1.1 技術概述
Racetrack Memory (RM) 是一種利用 **自旋電子學 (Spintronics)** 原理的次世代非揮發性記憶體。它的目標是結合 **硬碟 (HDD) 的高容量** 與 **快閃記憶體 (Flash/DRAM) 的高速度**。



* [cite_start]**核心概念**：想像一條奈米線（賽道），資料（磁性區域，Domain）像車子一樣在上面跑。讀寫頭是固定不動的，透過注入電流產生自旋轉移力矩 (Spin-Transfer Torque, STT)，推動整個磁性序列移動 (**Shift**) 到讀寫頭下方進行存取 [cite: 3213-3224]。
* **3D 堆疊**：RM 的賽道可以垂直豎立在晶片上，形成極高密度的 3D 儲存陣列。

### 1.2 兩大技術類型 (Two Types of RM)
根據儲存資料的物理單元不同，分為兩類：

1.  [cite_start]**Domain-Wall Racetrack Memory (DW-RM)** [cite: 3226-3236]
    * **原理**：利用 **磁壁 (Domain Wall)** 來區隔不同的磁性方向（代表 0 或 1）。
    * **缺點**：移動磁壁需要較大的電流密度 ($10^{11}-10^{12} A/m^2$)，容易發熱且穩定性較差。
2.  [cite_start]**Skyrmion Racetrack Memory (SK-RM)** [cite: 3389-3406]
    * **原理**：利用 **斯格明子 (Skyrmion)**，一種具有拓樸保護性質的粒子狀磁性結構。
    * **優勢**：超低電流（省電，約 $10^{6} A/m^2$）、高穩定性（不易受雜質干擾）、尺寸極小。



### 1.3 架構設計：Micro-cell vs. Macro-cell
[cite_start]為了在「存取速度」與「儲存密度」之間取得平衡，設計了兩種單元架構 [cite: 3237-3273]：

* **Micro-cell DWM**：賽道短，存少量資料，**低延遲**，但周邊電路佔比高導致密度較低 (約 $40F^2$)。定位類 SRAM。
* **Macro-cell DWM**：賽道長，存大量資料，**極高密度** (約 $2F^2$)，但移位時間長導致**高延遲**。定位類 DRAM/SCM。

### 📊 技術比較表：RM 類型與架構

| 特性 | DW-RM (磁壁式) | SK-RM (斯格明子) | Micro-cell 架構 | Macro-cell 架構 |
| :--- | :--- | :--- | :--- | :--- |
| **物理單元** | 磁壁 (Domain Wall) | 斯格明子 (Skyrmion) | 短賽道 | 長賽道 |
| **驅動電流** | 高 ($10^{11} A/m^2$) | **極低** ($10^{6} A/m^2$) | - | - |
| **穩定性** | 較差 (易受干擾) | **極佳** (拓樸保護) | - | - |
| **儲存密度** | 高 | **極高** (尺寸更小) | 低 ($40F^2$) | **極高** ($2F^2$) |
| **存取延遲** | - | - | **低** (移位快) | 高 (移位久) |
| **定位應用** | 早期原型 | 未來主流 | Cache / SRAM | Main Memory / Storage |

### 1.4 關鍵挑戰與解法
* **Shift Latency (移位延遲)**：資料越遠讀越久。解法是資料配置優化。
* [cite_start]**Position Error (位置錯誤)**：移位滑過頭。解法是 **Pinning (釘扎)**，在賽道刻凹槽固定位置 [cite: 3564]。
* [cite_start]**寫入能耗**：產生 Skyrmion 很耗電。解法是 **Permutation-Write (PW)**，重用舊的 Skyrmion 排列來形成新資料，減少注入操作 [cite: 4482-4508]。

---

### 👨‍🏫 老師的深度原理與延伸解析：打破記憶體牆 (Memory Wall)

**1. 物理學的勝利：為什麼是 Skyrmion？**
DW-RM 的磁壁就像是用蠻力推箱子，摩擦力大、耗電、容易卡住。而 Skyrmion 因為其特殊的**拓樸性質 (Topological Protection)**，就像箱子底下裝了輪子（或者是說它本身就像一個滑順的渦旋），非常「滑」，只需要極微小的電流就能推動。這讓 RM 從理論走向實用的關鍵一步。

**2. 記憶體的「磁帶化」與 Shift 的代價**
RM 最反直覺的地方在於它是**循序存取 (Sequential Access)** 的記憶體。
* **傳統 DRAM**：隨機存取，任何地址讀取時間一樣 (O(1))。
* **RM**：存取時間取決於資料距離讀寫頭多遠 (O(N))。
這意味著作業系統或控制器必須非常聰明，能夠**預測**資料存取模式，提前把資料移到讀寫頭附近（Pre-shifting）。

---

## 2. Chapter 06: Hard Disk Drives & SMR (硬碟與疊瓦式磁記錄)

### 2.1 硬碟基礎與 SMR 核心機制
* **瓶頸**：**超順磁效應** 限制了傳統 PMR 硬碟的密度。
* [cite_start]**SMR 原理**：利用**寫入頭 > 讀取頭**的特性，讓磁軌像瓦片般**疊加 (Overlap)**，提升密度 25%+ [cite: 501-508]。
* **代價**：禁止原地更新 (No In-place Update)，必須在 Zone 內**循序寫入**。



### 2.2 SMR 的三種模型 (Drive Models)
* **DM-SMR**：硬碟自己管，效能不穩。
* **HA-SMR**：硬碟與主機合作，支援隨機寫但建議循序。
* [cite_start]**HM-SMR**：主機強制管理，效能最穩 [cite: 761-767]。

### 2.3 四大技術解決方案
1.  [cite_start]**SMaRT (Firmware)**：利用 **Safety Gap** 原地更新，利用 GC 自動冷熱分離 [cite: 790-800]。
2.  [cite_start]**VPC (Driver)**：主機端建立 **Virtual Cache**，過濾隨機寫入保護硬碟 [cite: 957-979]。
3.  [cite_start]**HiSMRfs (FS)**：**資料/元數據分離**，全盤 Log-structured 寫入 [cite: 1188-1218]。
4.  [cite_start]**ZoneAlloy (Hybrid)**：動態 **CMR/SMR 格式轉換**，利用 Quantized Migration 避免卡頓 [cite: 1379-1419]。

### 📊 技術比較表：SMR 解決方案

| 特性 | **SMaRT** (DM-SMR) | **VPC** (HA-SMR) | **HiSMRfs** (HM-SMR) | **ZoneAlloy** (Hybrid) |
| :--- | :--- | :--- | :--- | :--- |
| **實作層級** | **Firmware** (硬碟內部) | **Block Driver** (驅動層) | **File System** (檔案系統) | **Driver / Firmware** (混合) |
| **核心策略** | **Track-based Mapping**<br>(利用 Safety Gap) | **Virtual Cache**<br>(雙層快取過濾) | **Data/Metadata Separation**<br>(全盤 Log 化) | **Elastic Data Space**<br>(動態格式轉換) |
| **隨機寫入處理** | 原地更新 (若有空隙) | 寫入虛擬快取 (SMR區) | 轉為循序追加 (Append) | 優先寫入 CMR 區 |
| **空間利用率** | 固定 (高) | 固定 (犧牲部分作快取) | 固定 (高) | **動態** (隨需求調整) |
| **優點** | 相容性最高 (隨插即用) | 解決長尾延遲，保護硬碟 | 效能最穩，無額外開銷 | 前期效能強(CMR)，後期容量大 |
| **缺點** | 效能不可預測 (GC 卡頓) | 需佔用 Host 資源 | 軟體相容性低 (需專用 FS) | 格式轉換成本高 |

---

### 👨‍🏫 老師的深度原理與延伸解析：HDD 的生存保衛戰

**1. 為什麼一定要用 SMR？（經濟學視角）**
這是一場經濟學的戰役。SSD 價格每年都在跌，HDD 的速度已經輸了，如果連「容量價格比 (Cost per GB)」優勢都守不住，HDD 就會被淘汰。SMR 是物理極限下的妥協，用效能換取容量。

**2. 解決方案的演化邏輯 (Who cleans the mess?)**
這四個技術其實是在問：「SMR 不能隨機寫入這個『麻煩』，該由誰來處理？」
* **SMaRT** 說：「硬碟自己偷偷處理。」(方便但效能差)
* **VPC** 說：「作業系統幫忙騙一下硬碟。」(利用 Host 資源換取效能)
* **HiSMRfs** 說：「檔案系統直接重寫，根本不產生隨機寫入。」(最徹底但門檻高)
* **ZoneAlloy** 說：「不要永遠都是 SMR，重要時刻切回 CMR。」(動態平衡)

---

## 3. Chapter 07: SSD & ZNS (固態硬碟與分區命名空間)

### 3.1 SSD 架構演進
從傳統 SSD (FTL 在裝置端) 到 OCSSD (FTL 在主機端)，最後演化到 **ZNS (Zoned Namespace)**。



### 3.2 ZNS 核心機制
* **Zone**：儲存空間切分為多個區域。
* **規則**：每個 Zone 必須從頭開始**循序寫入**，寫滿需 Reset。
* [cite_start]**ZRWA**：可選的隨機寫入緩衝區 [cite: 5020-5054]。
* [cite_start]**Zone Append**：**無名寫入**指令，主機只指定 Zone，由 SSD 決定位置，解決多執行緒鎖競爭問題 [cite: 5091-5101]。



### 📊 技術比較表：SSD 架構演進

| 特性 | **Conventional SSD** | **Open-Channel SSD (OCSSD)** | **ZNS SSD** |
| :--- | :--- | :--- | :--- |
| **類型** | **Black-box** (黑盒子) | **White-box** (白盒子) | **Grey-box** (灰盒子) |
| **FTL 位置** | Device (裝置端) | Host (主機端) | **混合** (Host管配置, Device管物理) |
| **介面** | LBA (Block Device) | Physical Chunk / PU | Zoned Namespace (Zone) |
| **寫入限制** | 無 (可隨機寫入) | 需配合物理特性 | **Zone 內強制循序寫入** |
| **DRAM 需求** | **高** (存 L2P Table) | 低 (Host 負責) | **低** (Table 變極小) |
| **效能隔離** | 差 (內部 GC 干擾) | 佳 (Host 完全控制) | **佳** (透過 Zone 隔離 I/O) |
| **寫入放大 (WA)**| 高 (內部 GC 搬運) | Host 決定 | **接近 1** (無內部 GC) |

---

### 👨‍🏫 老師的深度原理與延伸解析：ZNS 的去中心化革命

**1. 傳統 SSD 的原罪：FTL 與 GC**
傳統 SSD 為了假裝自己像硬碟一樣可以隨便寫，內部維護了巨大的 **L2P 映射表** (需要昂貴的 DRAM) 和複雜的 **GC 機制**。這導致了 **寫入放大 (Write Amplification)** ——你寫 1GB，SSD 內部因為搬運垃圾，實際磨損了 3GB 的壽命。

**2. ZNS 的哲學：攤牌**
ZNS 直接告訴 OS：「我其實不能隨便覆蓋寫，我只能循序寫。」
這帶來了巨大的好處：
1.  **消滅 GC**：既然都是循序追加，就沒有碎片，SSD 內部**完全不需要做 GC**。
2.  **消滅寫入放大**：寫多少就是多少 (WA $\approx$ 1)，大幅延長壽命。
3.  **降低成本**：不需要原本 SSD 內部昂貴的 DRAM 來存巨大的 L2P 表。

---

## 4. Chapter 08: Future Storages (未來儲存)

### 4.1 Glass Storage (玻璃儲存)
* [cite_start]**核心**：飛秒雷射在石英玻璃刻蝕 **Voxel**。每個 Voxel 可存多 bits [cite: 1555-1584]。
* [cite_start]**讀取優化 (GAIA)**：**Z-First** (先讀深度)、**SMTF** (考慮慣性)、**Zigzag** (減少轉彎) [cite: 1883-1885]。


### 4.2 DNA Storage (DNA 儲存)
* **核心**：利用 ATCG 鹼基編碼。
* [cite_start]**檢索**：利用 **PCR** 生化反應，透過引物 (Primer) 選擇性放大目標檔案，實現隨機存取 [cite: 2085-2125]。


### 📊 技術比較表：未來儲存技術

| 特性 | **Glass Storage (玻璃)** | **DNA Storage (DNA)** | **傳統 HDD/Tape** |
| :--- | :--- | :--- | :--- |
| **儲存媒介** | 石英玻璃 (Fused Silica) | 生物分子 (A/T/C/G) | 磁性物質 |
| **壽命** | **> 1000 年** (耐高溫水火) | **數百年** (需乾燥冷藏) | 5-30 年 |
| **寫入方式** | 飛秒雷射刻蝕 Voxel | 化學合成 DNA 鏈 | 磁頭磁化 |
| **讀取方式** | 偏振光顯微鏡 + ML 解碼 | DNA 定序儀 (Sequencer) | 磁頭感應 |
| **存取機制** | WORM (寫一次讀多次) | PCR 實現隨機存取 | 隨機 (HDD) / 循序 (Tape) |
| **密度** | 高 (3D 堆疊) | **極高** (分子級) | 中/高 |
| **主要用途** | 雲端資料中心冷資料歸檔 | 極致冷資料、極高密度需求 | 溫/熱資料 |

---

### 👨‍🏫 老師的深度原理與延伸解析：為數據永生而設計

**1. 極限冷儲存的應用場景**
對於核電廠數據、國家歷史檔案、頂級科研數據，我們需要一種「丟進去一萬年都不用管」的媒體。HDD 和磁帶每 10 年就要維護更換，成本太高且有資料遺失風險。

**2. DNA 的搜索魔法**
DNA 儲存最反直覺的地方在於它的「搜索」。想像在幾億本書的圖書館，你只要喊一聲書名 (加入 Primer)，那本書就會自動複製幾億份浮出來 (PCR)。這是一種生物化學層級的 **Content-Addressable Memory (內容可定址記憶體)**。

---

## 5. Chapter 09: Data Index Structures (1): LSM-tree 家族

### 5.1 基礎對決：B-tree vs. LSM-tree
* **B-tree**：**原地更新**。讀取快，隨機寫入慢。
* [cite_start]**LSM-tree**：**異地追加**。寫入快 (變循序)，讀取慢 (搜多層)。適合寫多讀少 [cite: 2353-2360]。



### 5.2 LSM-tree 家族演進
1.  [cite_start]**LevelDB**：標準 LSM。L0 讀取慢，Compaction 造成寫入放大 [cite: 2444-2471]。
2.  [cite_start]**WiscKey**：**鍵值分離**。Value 拆去 vLog，LSM 只存 Key。大幅降低寫入放大，但範圍搜尋變慢 [cite: 2671-2745]。
3.  [cite_start]**SLM-DB**：**PM 優化**。利用 PM 存全域索引，硬碟只剩**單一層**。極致降低寫入放大 [cite: 2794-2843]。
4.  [cite_start]**Be-tree**：**緩衝樹**。B-tree 節點加 Buffer，結合查詢優勢與批次寫入優勢 [cite: 2961-3184]。



### 📊 技術比較表：LSM-tree 家族演進

| 特性 | **LevelDB** (標準 LSM) | **WiscKey** (SSD 優化) | **SLM-DB** (PM 優化) | **Be-tree** ($B^{\epsilon}$-tree) |
| :--- | :--- | :--- | :--- | :--- |
| **核心哲學** | **分層合併**<br>(記憶體緩衝 + 多層硬碟) | **鍵值分離**<br>(LSM 存 Key, vLog 存 Value) | **單層儲存**<br>(PM 存索引 + Disk 單層) | **緩衝樹**<br>(B-tree + 節點緩衝區) |
| **硬體對象** | HDD / 通用 SSD | **SSD** (利用平行讀取) | **Persistent Memory** | 通用 / HDD |
| **寫入策略** | Append, 多層 Compaction | Append, Key 做 Compaction | Append, 同層 Self-Merging | 寫入 Buffer, 滿了 Flush |
| **寫入放大 (WA)**| **高** (資料搬運多次) | **低** (只搬運 Key) | **極低** (資料不換層) | **中低** (優於 B-tree) |
| **讀取策略** | 查 Mem → L0 → L1... | 查 LSM 拿地址 → 讀 vLog | 查 PM B+ Tree → 直讀 Disk | 沿樹搜尋 (查路徑 Buffer) |
| **範圍搜尋** | **優秀** (資料大致有序) | **較差** (隨機讀, 需 SSD 補救) | **優秀** (全域索引) | **優秀** (本質是 B-tree) |

---

### 👨‍🏫 老師的深度原理與延伸解析：演算法為硬體服務

**1. 一切都是為了適應硬體特性**
這章節完美展示了「軟體如何適應硬體」：
* **B-tree** 為 **HDD** 設計 (減少 Seek)。
* **LSM-tree** 為解決 **HDD 寫入瓶頸** 設計 (轉隨機為循序)。
* **WiscKey** 為 **SSD** 設計 (利用隨機讀取快，拆分 KV 救壽命)。
* **SLM-DB** 為 **PM** 設計 (利用不掉電特性，省去 WAL 和分層)。

**2. 寫入放大 (WA) 是萬惡之源**
在儲存系統設計中，我們做的一切努力，本質上都是在對抗寫入放大。因為 WA 不僅拖慢速度，更是直接殺死 Flash 壽命的元兇。

---

## 6. Chapter 10: Data Index Structures (2): Hash & Learned Indexes

### 6.1 傳統 Hash 在 PM 上的挑戰
傳統 Hash Table 在 Persistent Memory (PM) 上有兩個問題：
1.  **一致性 (Consistency)**：PM 寫入不是原子的。若擴展 (Resize) 到一半斷電，結構會損壞。
2.  **寫入放大**：擴展時需要 Rehash 所有資料，對 PM 壽命和效能傷害大。

### 6.2 針對 PM 優化的雜湊結構
#### [cite_start]1. CCEH (Cacheline-Conscious Extendible Hashing) [cite: 3650-3700]
* **原理**：利用「可擴展雜湊」的目錄機制。
* **結構**：**Directory** (存指標) -> **Segment** -> **Bucket** (存資料)。
* **擴展機制**：當 Bucket 滿了，**只分裂該 Bucket**，並利用 **Directory Doubling** (複製目錄指標) 來指向新 Bucket，無需搬移其他資料。利用 PM 寫入小的特性，大幅減少寫入量。



#### [cite_start]2. Level Hashing [cite: 3750-3800]
* **原理**：**寫入優化** 的雜湊結構。
* **結構**：**兩層式 (Top & Bottom Levels)**。Top 層是主力，Bottom 層是備用。
* **Sharing (共享)**：每個 Key 有兩個候選位置 (Hash1, Hash2)。
* **Movement (搬移)**：如果兩個位置都滿了，嘗試把這兩個位置裡原有的 Key 搬到它的「另一個家」，騰出空間給新 Key。這保證了極高的負載率 (>90%)。
* **In-place Resizing**：當 Top Level 滿了，Bottom Level 直接變成新的 Top Level (的 1/2)，以此類推。無需一次性大規模搬運。

### [cite_start]6.3 Learned Index (學習型索引) [cite: 3900-4000]
這是一個顛覆性的概念：**「索引就是模型 (Index is a Model)」**。

* **核心哲學**：
    * 傳統 B-tree：透過比較 (Comparison) 找到 Key 的位置。
    * Learned Index：透過 **回歸預測 (Regression)** 算出 Key 的位置。
    * 假設資料是已排序的陣列，位置 $Pos \approx F(Key)$。如果我們能學出函數 $F$（例如線性回歸 $y=ax+b$），我們就能直接算出位置，而不需要遍歷 B-tree。
* **RMI (Recursive Model Index)**：
    * 單一模型很難精準。所以採用 **多層級模型 (Hierarchy)**。
    * 上層模型負責「分流」，下層模型負責「精確預測」。
* **優勢**：模型只需要存幾個參數 (Weights)，比 B-tree 的指標省 **100倍** 空間。且數學運算比記憶體存取快。
* **劣勢**：只適用於唯讀或唯寫 (Read-only) 場景。一旦資料更新，模型就要重新訓練。



### 📊 技術比較表：索引結構大亂鬥

| 特性 | **Traditional Hash** | **CCEH (PM Hash)** | **Level Hashing (PM Hash)** | **Learned Index** |
| :--- | :--- | :--- | :--- | :--- |
| **基本原理** | Key $\rightarrow$ Bucket Address | 3-Level + Dynamic Hashing | 2-Level + Sharing | **Key $\rightarrow$ Position (Regression)** |
| **適用場景** | DRAM, 通用 | **Persistent Memory** | **Persistent Memory** | Read-Heavy, 已知分布 |
| **擴展成本** | 高 (Rehash all) | **低** (Directory Doubling) | **低** (In-place Resizing) | **極高** (需 Retrain 模型) |
| **空間效率** | 普通 (指針佔空間) | 普通 | 高 (高負載率) | **極高** (只存參數) |
| **查詢速度** | $O(1)$ | $O(1)$ | $O(1)$ (最多移動一次) | 快 (數學運算) |
| **範圍搜尋** | 不支援 | 不支援 | 不支援 | **支援** (基於排序) |

---

### 👨‍🏫 老師的深度原理與延伸解析：從結構到模型的典範轉移

**1. 為什麼 Hash 在 PM 上這麼難做？**
因為 PM 是「位元組定址 (Byte-addressable)」且「非揮發」的。這意味著 CPU 的 Cache Line Flush 指令 (CLFLUSH) 變成了資料一致性的關鍵。CCEH 和 Level Hashing 的設計核心，都在於 **「如何減少 CLFLUSH 的次數」** 以及 **「如何保證指針更新的原子性」**。

**2. Learned Index 的啟示：算力換空間**
Learned Index 告訴我們：隨著 CPU 算力越來越便宜，而記憶體頻寬 (Memory Bandwidth) 越來越珍貴，我們應該**「多算一點，少讀一點」**。用 CPU 算出位置，比去記憶體裡撈 B-tree 的 Node 更快、更省空間。這是資料庫領域近十年來最大的思維突破。