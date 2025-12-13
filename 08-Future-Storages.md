# 08. Future Storages: Glass Storage & DNA Storage 技術筆記

這份筆記整理自課程投影片，探討針對 **「冷資料 (Cold Data) 長期歸檔」** 的兩種未來儲存技術：玻璃儲存 (Glass Storage) 與 DNA 儲存 (DNA Storage)。這兩者旨在解決傳統硬碟 (HDD) 與磁帶 (Tape) 壽命短、維護成本高的問題。

---

## 1. Glass Storage (玻璃儲存) —— Project Silica

**核心概念**：利用飛秒雷射 (Femtosecond Laser) 在石英玻璃 (Fused Silica) 內部刻蝕出微小的 3D 奈米結構，稱為 **Voxel (體素)**。

### 1.1 寫入機制 (Writing)
* **工具**：使用極短脈衝的飛秒雷射 ($10^{-15}$秒)。
* **原理**：雷射改變玻璃內部的物理特性，創造出永久性的結構。
* **多維度編碼**：每個 Voxel 不只存 1 bit，而是透過改變雷射的 **偏振角度 (Orientation)** 和 **強度 (Strength)** 來儲存多個 bits (例如 5 bits/voxel)。

### 1.2 讀取機制 (Reading)
* **物理現象**：當偏振光穿過 Voxel 時，會產生 **雙折射 (Birefringence)** 現象，導致光線產生相位延遲 (Retardance) 和角度偏移。
* **解碼技術 (Decoding)**：
    * **傳統方法**：容易受雜訊影響，準確率低。
    * **機器學習 (ML)**：使用 **CNN (卷積神經網路)** 識別 Voxel 訊號，將解碼準確率從 75% 提升至 **99% 以上**。

### 1.3 讀取效能優化策略：GAIA
為了解決讀取頭機械移動 (X-Y軸) 緩慢且有慣性的問題，提出了 **GAIA** 演算法，結合以下三種策略，減少約 **82%** 的延遲：

1.  **Zigzag (之字形)**：規劃路徑以減少讀取頭轉向的次數。
2.  **SMTF (Seek Moving Time First)**：考慮機械臂的 **加減速 (Acceleration)** 時間，找出時間最短的路徑，而非單純距離最短。
3.  **Z-First (Z軸優先)**：利用 Z 軸對焦速度 ($15~m/s$) 遠快於 X 軸機械移動 ($0.8~m/s$) 的特性。在移動機械臂前，先把同一位置不同深度的資料全讀完。

---

## 2. DNA Storage (DNA 儲存)

**核心概念**：利用生物分子的 4 種鹼基 (A, T, C, G) 對應二進位資料 (0, 1)。這是目前已知密度最高的儲存介質。

### 2.1 運作流程 (The Pipeline)
1.  **Encoding (編碼)**：將數位檔案 (0101...) 轉換成鹼基序列 (ATCG...)。
2.  **Synthesis (合成/寫入)**：使用 DNA 合成儀將化學分子組合成實際的 DNA 短鏈 (Oligos)。
3.  **Storage (儲存)**：將 DNA 放入試管 (in vitro) 或細菌體內 (in vivo) 保存。
4.  **Retrieval (檢索)**：**關鍵技術！**
    * 利用 **PCR (聚合酶連鎖反應)**。
    * 每個檔案加上特定的「引物 (Primer)」當作 Key。
    * 加入對應 Primer 進行 PCR，即可 **選擇性放大 (Amplify)** 目標檔案的 DNA，實現 **隨機存取 (Random Access)**。
5.  **Sequencing (定序/讀取)**：使用定序儀 (Sequencer) 讀出鹼基序列。
6.  **Decoding (解碼)**：將 ATCG 轉回數位訊號。

### 2.2 優缺點分析
* **優點**：
    * **極高密度 (Dense)**：理論上幾公斤 DNA 能儲存全世界的資料。
    * **超長壽命 (Durable)**：保存數百年甚至千年不壞（化石中的 DNA 仍可讀取）。
* **缺點**：
    * **速度極慢 (High Latency)**：讀寫過程需要數小時甚至數天。
    * **成本高昂 (High Cost)**：目前的合成與定序成本極高。

---

## 3. 技術比較總結

| 特性 | Glass Storage (玻璃) | DNA Storage (DNA) | 傳統 HDD/Tape |
| :--- | :--- | :--- | :--- |
| **儲存媒介** | 石英玻璃 (Fused Silica) | 生物分子 (A/T/C/G) | 磁性物質 |
| **壽命** | **> 1000 年** (耐高溫水火) | **數百年** (需乾燥冷藏) | 5 - 30 年 |
| **寫入方式** | 飛秒雷射刻蝕 Voxel | 化學合成 DNA 鏈 | 磁