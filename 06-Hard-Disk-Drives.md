# 06. Hard Disk Drives & SMR Technology 技術筆記

這份筆記整理自課程投影片，探討硬碟 (HDD) 的物理限制如何催生了 SMR 技術，以及不同層級（Firmware, Driver, File System）的 SMR 管理方案。

---

## 1. 硬碟基礎與瓶頸 (HDD Basics & Bottleneck)

### 1.1 物理結構
* **Platter (碟盤)**：儲存資料的圓盤。
* **Track (磁軌)**：同心圓軌道。
* **Sector (磁區)**：磁軌上的最小讀寫單位（通常 512 bytes 或 4KB）。
* **特性**：隨機 I/O 需要機械移動（Seek + Rotation），速度慢；循序 I/O 速度快。

### 1.2 磁記錄技術演進
* **LMR (水平磁記錄)**：磁性粒子平躺，密度低。
* **PMR (垂直磁記錄)**：磁性粒子站立，密度較高，是目前的標準技術。
* **瓶頸**：**超順磁效應 (Superparamagnetic Effect)**。當磁性顆粒太小時，熱能會導致磁性翻轉，造成資料遺失。這限制了 PMR 的密度上限（約 1 TB/in²）。

---

## 2. SMR (疊瓦式磁記錄) 核心概念

為了突破密度瓶頸，**SMR (Shingled Magnetic Recording)** 被發明出來。

### 2.1 運作原理
* **讀寫頭差異**：物理上，**寫入頭 (Write Head)** 必須比 **讀取頭 (Read Head)** 寬（為了產生足夠磁場）。
* **疊瓦堆疊**：傳統硬碟軌距由「寫入頭」寬度決定。SMR 像屋頂瓦片一樣，讓新磁軌**覆蓋 (Overlap)** 掉上一條磁軌的一部份，只留下讀取頭所需的窄寬度。
* **優點**：密度提升 25% 以上。

### 2.2 核心挑戰
* **禁止原地更新 (No In-place Update)**：如果你想修改 Track N 的資料，因為寫入頭很寬，你會不小心覆蓋到 Track N+1 的資料。
* **寫入限制**：必須像 Log 一樣，依序 **循序寫入 (Sequential Write)**。
* **Zone (區域)**：硬碟被切分成多個 Zones，每個 Zone 有一個 **Write Pointer (WP)**，寫入只能從 WP 開始追加。

---

## 3. SMR 的三種模型 (Drive Models)

根據「誰來負責管理 WP 和垃圾回收」，SMR 分為三種模型：

1.  **Drive-Managed (DM-SMR)**：
    * **管理者**：硬碟 Firmware。
    * **特性**：對主機透明 (Transparent)，像傳統硬碟一樣用。
    * **缺點**：效能不可預測 (Unpredictable)，內部 GC 會導致卡頓。
2.  **Host-Aware (HA-SMR)**：
    * **管理者**：主機與硬碟合作。
    * **特性**：支援隨機寫入（硬碟會救），但建議主機循序寫入以獲得高效能。
3.  **Host-Managed (HM-SMR)**：
    * **管理者**：主機軟體 (OS/FS)。
    * **特性**：**強制**循序寫入。若違反規則，I/O 會報錯。效能最穩定，但需修改軟體堆疊。

---

## 4. 四大解決方案技術詳解 (Solutions Comparison)

針對 SMR 的限制，不同層級提出了不同的解決方案。

### 4.1 SMaRT (針對 DM-SMR 的 Firmware 優化)
* **核心策略**：**Track-based Mapping (以磁軌為單位的映射)**。放棄 Sector 粒度，改用 Track 粒度管理，平衡表的大小與靈活性。
* **關鍵技術**：
    * **Safety Gap (Lazy Update)**：如果隔壁磁軌是空的或無效的，允許直接原地更新，不用搬移資料。
    * **Auto Cold Data Progression**：在 GC 過程中，將熱資料寫到 Zone 的前端（寫入點），冷資料自然沉積在 Zone 的後端，減少搬運冷資料的次數。

### 4.2 VPC (針對 HA-SMR 的 Driver 層優化)
* **問題**：HA-SMR 內部的 Persistent Cache 容易被隨機寫入塞滿，導致長尾延遲。
* **核心策略**：**Virtual Persistent Cache (虛擬持久快取)**。
* **關鍵技術**：
    * **虛擬快取層**：在 SMR 區域挖一塊空間模擬成 Cache。
    * **雙層過濾**：隨機寫入先進入 Virtual Cache，只有**極熱**的資料才准進入硬碟真正的物理 Cache。這保護了物理 Cache 不被塞爆。

### 4.3 HiSMRfs (針對 HM-SMR 的檔案系統)
* **問題**：HM-SMR 禁止隨機寫，且 Metadata 更新頻繁。
* **核心策略**：**Data/Metadata Separation (資料與元數據分離)**。
* **關鍵技術**：
    * **異質儲存**：Metadata 存放在 **SSD** 或 **CMR** 分區（支援隨機寫）；檔案內容存放在 **SMR** 分區。
    * **Log-structured Writing**：將所有檔案寫入轉為 Append-only。
    * **L2P Mapping (類 Page Table)**：因為資料位置一直變，Metadata 紀錄了邏輯位址到物理 SMR Zone 位址的映射表。

### 4.4 ZoneAlloy (Hybrid SMR 解決方案)
* **概念**：硬碟格式不是固定的，可以在 CMR 和 SMR 之間轉換。
* **核心策略**：**Elastic Data Space (彈性空間)**。
* **關鍵技術**：
    * **兩階段生命週期**：剛開始全是 CMR (效能好)，空間不夠時動態轉為 SMR (容量大)。
    * **Quantized Migration**：轉換格式很慢，所以切分成小塊 (Quantum) 進行，利用 I/O 空檔做，避免卡死系統。
    * **H-Buffer**：在 CMR 區保留緩衝區，累積隨機寫入後再一次性寫入 SMR。

---

## 5. 綜合比較表

| 特性 | **SMaRT** (DM-SMR) | **VPC** (HA-SMR) | **HiSMRfs** (HM-SMR) | **ZoneAlloy** (Hybrid) |
| :--- | :--- | :--- | :--- | :--- |
| **實作層級** | **Firmware**<br>(硬碟韌體) | **Block Driver**<br>(驅動程式) | **File System**<br>(檔案系統) | **Driver / Firmware**<br>(混合層) |
| **核心哲學** | **欺騙 (Transparency)**<br>硬碟自己扛 | **分擔 (Offloading)**<br>Host 幫忙做快取 | **重構 (Redesign)**<br>Host 完全掌控 | **彈性 (Elasticity)**<br>動態改變物理格式 |
| **隨機寫入策略** | **Safety Gap**<br>利用空隙原地更新 | **Virtual Cache**<br>寫入 SMR 模擬的快取 | **Log-structured**<br>全部轉為循序追加 | **CMR First**<br>優先寫在 CMR 區 |
| **空間利用率** | 固定 (SMR 高密度) | 固定 (犧牲部分作快取) | 固定 (SMR 高密度) | **動態** (隨需求轉換) |
| **優點** | 相容性最高，隨插即用 | 解決長尾延遲，保護硬碟 | 效能最穩，無額外開銷 | 前期效能強(CMR)，後期容量大 |
| **缺點** | 負載高時效能不可預測 | 需佔用 Host 資源 | 軟體相容性低 (需專用 FS) | 格式轉換成本高 |