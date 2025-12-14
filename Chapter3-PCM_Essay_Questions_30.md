# Phase-Change Memory (PCM) Essay Questions
## Based on Pages 152-183: Write-friendly Arithmetic Coding & Drifting-tolerate Coding

---

## Part I: Write-friendly Arithmetic Coding (Questions 1-15)

### Question 1
**Explain the main drawbacks of Huffman Coding and why Arithmetic Coding is preferred in modern compression applications such as JPEG2000, H.264, and 7-Zip.**

**詳解：**

Huffman Coding 的主要缺點包括：

1. **需要預先知道符號頻率**：Huffman 編碼需要事先統計所有符號的出現頻率，這在串流資料或即時壓縮場景中不實用。

2. **只能分配整數位元長度**：Huffman 編碼只能給每個符號分配 1、2、3 位元等整數長度的編碼。如果理論上最佳編碼長度是 2.3 位元，Huffman 無法精確表示，導致壓縮效率略低於理論極限（熵）。

3. **固定模型**：靜態 Huffman 編碼使用固定的編碼表，無法適應資料特性的變化。

4. **符號頻率相近時無優勢**：當所有符號出現頻率相近時，所有編碼長度幾乎相同，壓縮效益極低。

5. **小資料集效率差**：對於小符號集或小資料量，Huffman 的效率不佳。

**Arithmetic Coding 的優勢**：
- 可以產生「分數位元」的平均編碼長度，更接近理論熵極限
- 適應性高，可以動態調整機率模型
- 因此被廣泛應用於 JPEG2000（影像）、H.264（視頻）、7-Zip（檔案壓縮）等現代壓縮標準

---

### Question 2
**Describe the encoding process of Arithmetic Coding using the example where P(a)=0.375, P(b)=0.375, P(c)=0.25. How is the string "abc" encoded?**

**詳解：**

Arithmetic Coding 的核心概念是將整個訊息編碼為一個位於 [0, 1) 區間內的浮點數。

**編碼過程**：

1. **初始狀態**：區間為 [0, 1)，範圍 Range = 1

2. **編碼 'a' (P=0.375)**：
   - 'a' 佔據區間 [0, 0.375)
   - 新區間：[0, 0.375)
   - 新 Range = 0.375

3. **編碼 'b' (P=0.375)**：
   - 在當前區間 [0, 0.375) 中，'b' 的位置是第二個符號
   - 'b' 的子區間起點 = 0 + 0.375 × 0.375 = 0.140625
   - 'b' 的子區間終點 = 0 + 0.375 × 0.75 = 0.28125
   - 新區間：[0.140625, 0.28125)

4. **編碼 'c' (P=0.25)**：
   - 在當前區間中，'c' 佔據最後 25%
   - 新區間起點 = 0.140625 + (0.28125 - 0.140625) × 0.75 = 0.246
   - 新區間終點 = 0.28125
   - 最終區間：[0.246, 0.281)

**結論**：任何位於 [0.246, 0.281) 區間內的浮點數都可以代表壓縮資料 "abc"。

---

### Question 3
**What is the concept of "Ignorable Bits" in Write-friendly Arithmetic Coding? Explain how it helps reduce write operations on PCM.**

**詳解：**

**Ignorable Bits（可忽略位元）的概念**：

在 IEEE 754 浮點數格式中，一個 64 位元的雙精度浮點數包含：
- 1 位元符號位（Sign）
- 11 位元指數（Exponent）
- 52 位元尾數（Mantissa）

**核心觀察**：
當 Arithmetic Coding 產生的編碼區間（如 [0.246, 0.281)）足夠寬時，尾數的某些低位元無論是 0 還是 1，最終的浮點數值都會落在有效區間內。這些位元就稱為「可忽略位元」。

**範例**：
```
編碼區間：[0.246, 0.281)
Write-friendly 編碼值 = 0.265625 = 
0 01111111101 0000111111111111111111111111111111111111111111111111
             └─────────────────────────────────────────────────┘
                              48 個可忽略位元
```

**對 PCM 的好處**：
1. **減少 RESET 操作**：可忽略位元不需要執行 RESET（寫入 0）
2. **減少 SET 操作**：可忽略位元不需要執行 SET（寫入 1）
3. **降低能耗**：PCM 的 RESET 耗能 19.2 pJ/bit，SET 耗能 13.5 pJ/bit，減少寫入直接降低能耗
4. **延長壽命**：減少位元翻轉次數可延長 PCM 的使用壽命

---

### Question 4
**Explain the "Upper Bits Preselecting" technique in Write-friendly Arithmetic Coding. Provide a step-by-step explanation using the encoding interval [0.246, 0.281).**

**詳解：**

**Upper Bits Preselecting（高位預選）技術**的目標是從實數角度預選高位位元，以產生盡可能多的可忽略位元。

**步驟說明（以區間 [0.246, 0.281) 為例）**：

**步驟 1：轉換上下界的指數位元為實數**
- 下界（LB）= 0.246 → 指數位元 = 01111111100 = 1020
  - 偏移量 = 1023 - 1020 = 3 → 表示為 0.001（二進制）= 0.125（十進制）
- 上界（UB）= 0.281 → 指數位元 = 01111111101 = 1021
  - 偏移量 = 1023 - 1021 = 2 → 表示為 0.01（二進制）= 0.25（十進制）

**步驟 2：選擇起始點**
- 選擇導致最少移位位元的邊界作為起始點
- 選擇 0.01 = 2^(-2) = 0.25 作為起始點

**步驟 3：驗證是否在區間內**
- 0.25 位於 [0.246, 0.281) 之間 ✓
- 完整表示：0 01111111101 0000000000000000000000000000000000000000000000000000

**結果**：透過選擇 0.25 作為編碼值，可以獲得 52 個可忽略位元（整個尾數都是 0）。

---

### Question 5
**When the encoding interval is narrow (e.g., [0.246, 0.24601)), explain how the "Upper Bits Preselecting" algorithm handles this case with rollback operations.**

**詳解：**

當編碼區間很窄時（如 [0.246, 0.24601)），簡單選擇一個 2 的冪次方可能無法落入區間，需要更精細的位元操作。

**演算法步驟**：

**步驟 1：選擇起始點**
- 起始點：0.001（二進制）= 2^(-3) = 0.125

**步驟 2：逐步探索**
```
0.0011 = 0.125 + 2^(-4) = 0.1875
0.00111 = 0.1875 + 2^(-5) = 0.21875
...
0.00111111 = 0.24609375  ← 超過上界 0.24601！
```

**步驟 3：回滾（Rollback）操作**
- 當累加值超過上界時，回滾到前一個累加值，跳過當前位元
```
0.0011111 = 0.2421875（跳過第 8 位）
0.001111101 = 0.2421875 + 2^(-9) = 0.244140625
...
0.0011111011111 = 0.2459716796875
0.00111110111111 = 0.24603271484375  ← 超過上界！
0.001111101111101 = 0.2459716796875 + 2^(-15) = 0.246002197265625
```

**結果**：
- 最終累加值 SUM = 0.246002197265625 位於 [0.246, 0.24601) 之間 ✓
- 預選的高位位元：001111101111101
- 可能的可忽略位元 = 40 位元

---

### Question 6
**Describe the "Ignorable Bits Determining" phase. How does it find the maximum number of ignorable bits based on the preselected upper bits?**

**詳解：**

**Ignorable Bits Determining（可忽略位元決定）階段**的目標是在預選高位位元的基礎上，找出能使編碼值仍落在區間內的最大 '1' 位元數量。

**演算法原理**：

以區間 [0.246, 0.24601) 和預選高位 001111101111101 為例：

**步驟 1：從最低有效位開始嘗試加入 '1'**
```
基礎值：0.0011111011111010000...0（52位尾數）
SUM = 0.246002197265625

嘗試加入低位 '1'：
+2^(-55) → 0.24600219726562502... ✓ 在區間內
+2^(-54) → 0.24600219726562508... ✓ 在區間內
...
+2^(-18) → 0.24600982666015622  ✓ 在區間內
+2^(-17) → 0.24601745605468747  ✗ 超過上界！
```

**步驟 2：確定可忽略位元**
```
最終編碼值 = 0.24600982666015622
= 0 01111111100 111101111101001111111111111111111111111111111111
                └──14位──┘└──────────38個可忽略位元──────────┘
                不可忽略     無論是0或1都不影響結果
```

**關鍵特性**：
- 可忽略位元是**連續的**，從尾數的某個位置開始到最低位
- 這種連續性使得實作簡單，可用位址遮罩實現
- 無論這 38 位元是 0 還是 1，編碼值都會落在 [0.246, 0.24601) 區間內

---

### Question 7
**What is the energy consumption model for PCM write operations? Calculate the energy savings when reducing 100 RESET and 50 SET operations.**

**詳解：**

**PCM 能耗模型**：
根據文獻 [9]（Lee et al., ISCA 2009）：
- RESET 操作：19.2 pJ/bit
- SET 操作：13.5 pJ/bit

**能耗計算**：

原始能耗（假設執行這些操作）：
```
RESET 能耗 = 100 × 19.2 pJ = 1,920 pJ
SET 能耗 = 50 × 13.5 pJ = 675 pJ
總能耗 = 1,920 + 675 = 2,595 pJ
```

**能耗節省**：
若透過 Write-friendly Arithmetic Coding 完全避免這些操作：
- 節省 2,595 pJ 的能耗

**實際效益**：
- 實驗顯示可減少 10-48% 的 RESET 操作
- 可減少 13-41% 的 SET 操作
- 這對大量資料壓縮場景（如物聯網感測器資料）特別有價值

---

### Question 8
**Compare Traditional Arithmetic Coding with Write-friendly Arithmetic Coding in terms of NVM performance. What are the experimental results?**

**詳解：**

**比較維度**：

| 指標 | Traditional AC | Write-friendly AC | 改善幅度 |
|------|---------------|-------------------|----------|
| RESET 操作數 | 基準 | 較少 | 減少 10-48% |
| SET 操作數 | 基準 | 較少 | 減少 13-41% |
| 資料準確性 | 100% | 100% | 無損失 |
| 可忽略位元 | 0% | 可變 | 顯著增加 |

**實驗結果（Large Text Compression Benchmark）**：

1. **enwik8 資料集**：
   - RESET 減少 47.9%
   - SET 減少 41.2%

2. **enwik9 資料集**：
   - RESET 減少 9.1%
   - SET 減少 12.5%

3. **其他資料集**：
   - RESET 減少 10-13.1%
   - SET 減少 13.3-14.2%

**關鍵發現**：
- Write-friendly AC 在不犧牲資料準確性的前提下顯著降低寫入操作
- 對不同資料集的效果有所差異，取決於編碼區間的寬度分布
- 編碼區間越寬，可忽略位元越多，節能效果越好

---

### Question 9
**Explain the relationship between the encoding interval width and the number of ignorable bits. Why does a wider interval lead to more ignorable bits?**

**詳解：**

**編碼區間寬度與可忽略位元的關係**：

**數學原理**：
- 編碼區間 [L, U) 的寬度 W = U - L
- 可忽略位元數量 ≈ log₂(W × 2^52)（對於雙精度浮點數）

**直觀解釋**：
```
寬區間 [0.25, 0.50)：
- 任何 0.25 ≤ x < 0.50 的值都有效
- 尾數有很大的選擇空間
- 可忽略位元多

窄區間 [0.246, 0.24601)：
- 只有 0.246 ≤ x < 0.24601 的值有效
- 尾數的選擇空間小
- 可忽略位元少
```

**具體範例**：

| 區間 | 寬度 | 可忽略位元 |
|------|------|-----------|
| [0.0, 0.5) | 0.5 | ~51 位元 |
| [0.246, 0.281) | 0.035 | ~48 位元 |
| [0.246, 0.24601) | 0.00001 | ~38 位元 |

**原因分析**：
1. 寬區間意味著更多的浮點數可以代表同一壓縮資料
2. 更多的選擇讓我們能找到「最省寫入」的編碼值
3. IEEE 754 格式的尾數低位對數值影響小，寬區間允許忽略更多低位

---

### Question 10
**What benchmarks were used to evaluate Write-friendly Arithmetic Coding? Why are these benchmarks appropriate for evaluating NVM compression techniques?**

**詳解：**

**使用的基準測試**：

1. **Large Text Compression Benchmark [10]**
   - 來源：Matt Mahoney 的文字壓縮基準
   - 包含 enwik8、enwik9 等大型文字資料集
   - 被 164 篇論文引用

2. **IoT dataset - Heart Rate during Sleeping [11]**
   - 來源：心率監測研究
   - 代表物聯網感測器資料特性
   - 被 322 篇論文引用

**為何適合評估 NVM 壓縮技術**：

1. **Large Text Compression Benchmark**：
   - 資料量大，能體現壓縮演算法的整體效能
   - 文字資料具有可預測的統計特性
   - 是壓縮領域的標準測試集

2. **IoT Heart Rate 資料集**：
   - 代表邊緣運算和物聯網場景
   - 這些場景通常使用 NVM 作為儲存
   - 感測器資料具有連續性和局部性，適合測試壓縮效率

3. **綜合原因**：
   - 涵蓋不同資料類型（文字 vs 數值）
   - 代表實際應用場景
   - 有足夠的資料量來展示統計顯著性

---

### Question 11
**How does IEEE 754 double-precision floating-point format affect the design of Write-friendly Arithmetic Coding?**

**詳解：**

**IEEE 754 雙精度格式結構**：
```
總共 64 位元：
| Sign (1 bit) | Exponent (11 bits) | Mantissa (52 bits) |
```

**對 Write-friendly AC 設計的影響**：

**1. 指數位元的利用**：
- 指數決定數值的數量級
- Upper Bits Preselecting 首先確定指數位元
- 指數位元的選擇影響可達到的數值範圍

**2. 尾數位元的優化空間**：
- 52 位尾數提供了大量的優化空間
- 低位尾數對數值影響最小，最適合作為可忽略位元
- 演算法從低位開始尋找可忽略位元

**3. 正規化表示的考慮**：
```
實際值 = (-1)^sign × 1.mantissa × 2^(exponent-1023)

例：0.25 = 0 01111111101 0000...0
    = 1.0 × 2^(-2) = 0.25
```

**4. 精度與可忽略位元的權衡**：
- 雙精度提供約 15-17 位十進制精度
- 對於大多數壓縮應用，不需要完整精度
- 可忽略低位尾數而不影響解碼正確性

**5. 位元連續性的優勢**：
- IEEE 754 的尾數是連續排列的
- 可忽略位元也是連續的
- 便於用位元遮罩和位址計算實現

---

### Question 12
**Explain why the ignorable bits in Write-friendly Arithmetic Coding are continuous. What implementation advantages does this continuity provide?**

**詳解：**

**可忽略位元連續性的原因**：

**數學原理**：
設編碼區間為 [L, U)，預選的高位位元產生基礎值 B。

若尾數的第 k 位到第 52 位都是可忽略的，則：
```
B + 0 ≥ L  且  B + 2^(-k) + 2^(-k-1) + ... + 2^(-52) < U
```

這等價於：
```
B + (2^(-k+1) - 2^(-52)) < U
```

由於 2^(-k+1) - 2^(-52) ≈ 2^(-k+1)，一旦某位可忽略，其後的所有位（數值影響更小）自然也可忽略。

**實作優勢**：

1. **簡化位址計算**：
   ```
   可忽略區域 = [bit_start, 52]
   只需記錄一個起始位置，不需要位元遮罩表
   ```

2. **高效記憶體操作**：
   ```
   // 只需一次遮罩操作
   uint64_t mask = ~((1ULL << (52 - bit_start)) - 1);
   encoded_value &= mask;  // 清除可忽略位元
   ```

3. **減少元資料開銷**：
   - 只需儲存可忽略位元的起始位置（6 位元足夠）
   - 不需要為每個位元儲存獨立的忽略標記

4. **批次寫入優化**：
   - 可以一次性跳過連續的可忽略區域
   - 減少 PCM 控制器的操作次數

5. **與 PCM 寫入單元對齊**：
   - PCM 通常以固定大小的寫入單元操作
   - 連續可忽略位元更容易與寫入單元邊界對齊

---

### Question 13
**In the context of Write-friendly Arithmetic Coding, what is the trade-off between compression ratio and write reduction? How does the algorithm balance these factors?**

**詳解：**

**壓縮率與寫入減少的權衡**：

**核心觀察**：
- Arithmetic Coding 的壓縮率取決於最終編碼區間的位置
- Write-friendly AC 選擇區間內「寫入成本最低」的值
- 兩者的編碼區間相同，只是選擇不同的代表值

**權衡分析**：

| 因素 | Traditional AC | Write-friendly AC |
|------|---------------|-------------------|
| 壓縮率 | 最佳 | 相同（無損失）|
| 編碼區間 | [L, U) | [L, U)（相同）|
| 選擇的值 | 區間內任意值 | 最少寫入的值 |
| 解碼正確性 | 100% | 100% |

**平衡策略**：

1. **不犧牲壓縮率**：
   - 編碼區間完全由原始 Arithmetic Coding 決定
   - Write-friendly 只是在區間內選擇最佳值
   - 壓縮率不受影響

2. **最大化寫入減少**：
   ```
   Upper Bits Preselecting → 找到最少 '1' 的高位表示
   Ignorable Bits Determining → 最大化可忽略的低位
   ```

3. **計算開銷考慮**：
   - Upper Bits Preselecting：O(n) 複雜度，n 為位元數
   - 額外計算開銷相對於壓縮本身較小
   - 對於大資料集，寫入節省遠超計算開銷

**結論**：
Write-friendly AC 透過在編碼區間內選擇最佳代表值，在**不犧牲壓縮率**的前提下最大化寫入減少。這是一個「免費午餐」——獲得寫入節省而不付出壓縮率代價。

---

### Question 14
**Describe the experimental methodology used to evaluate Write-friendly Arithmetic Coding. What metrics were measured and why?**

**詳解：**

**實驗方法論**：

**1. 評估指標**：

**NVM 效能指標**：
- **總能耗（Total Energy Consumption）**：
  - 計算公式：E_total = N_RESET × 19.2 pJ + N_SET × 13.5 pJ
  - 意義：直接反映 PCM 的能源效率
  
- **總寫入次數（Total Number of Writes）**：
  - 分別統計 RESET 和 SET 操作次數
  - 意義：反映 PCM 壽命和效能影響

**Write-friendly AC 效能指標**：
- **可忽略位元比率（Ignorable Bits Ratio）**：
  - 計算公式：可忽略位元數 / 總尾數位元數 × 100%
  - 意義：衡量演算法找出可忽略位元的能力

**2. 基準測試選擇**：
- Large Text Compression Benchmark：代表通用壓縮場景
- IoT Heart Rate 資料：代表邊緣運算場景

**3. 比較方法**：
- 對照組：Traditional Arithmetic Coding
- 實驗組：Write-friendly Arithmetic Coding
- 相同輸入資料，比較輸出的寫入需求

**4. 為何選擇這些指標**：
- **能耗**：PCM 最關鍵的問題之一是寫入能耗高
- **寫入次數**：直接影響 PCM 壽命（10^8 次寫入限制）
- **可忽略位元比率**：反映演算法核心機制的效能

---

### Question 15
**How would Write-friendly Arithmetic Coding perform with different types of data (e.g., highly compressible vs. incompressible data)? Analyze the expected behavior.**

**詳解：**

**不同資料類型的預期行為分析**：

**1. 高度可壓縮資料（如重複文字、低熵資料）**：

特性：
- 符號頻率分布不均勻
- Arithmetic Coding 產生的編碼區間較寬
- 每次編碼後區間收縮較慢

Write-friendly AC 效果：
```
預期：優異
原因：
- 寬區間 → 更多可忽略位元
- 如實驗中 enwik8：RESET 減少 47.9%，SET 減少 41.2%
```

**2. 低壓縮率資料（如隨機資料、高熵資料）**：

特性：
- 符號頻率接近均勻分布
- 編碼區間收縮快
- 最終區間較窄

Write-friendly AC 效果：
```
預期：中等
原因：
- 窄區間 → 可忽略位元較少
- 但仍有一定改善空間
- 如實驗中 enwik9：RESET 減少 9.1%，SET 減少 12.5%
```

**3. 二進制/已壓縮資料**：

特性：
- 熵接近最大值
- 幾乎無法進一步壓縮
- 區間極窄

Write-friendly AC 效果：
```
預期：較差
原因：
- 區間寬度極小
- 可忽略位元數量有限
- 但不會比 Traditional AC 更差
```

**4. 串流資料（如感測器資料）**：

特性：
- 資料具有時間局部性
- 相鄰值差異小

Write-friendly AC 效果：
```
預期：良好
原因：
- 預測模型效果好
- 編碼區間相對寬
- 適合 IoT 應用場景
```

---

## Part II: Drifting-tolerate Coding (Questions 16-30)

### Question 16
**Explain the concept of Multi-Level Cell (MLC) PCM and why it introduces the problem of asymmetric writes. What are the energy and latency differences among MLC states?**

**詳解：**

**MLC PCM 概念**：

MLC（Multi-Level Cell）PCM 透過在單一 cell 中儲存多個位元來提高資料密度。一般的 MLC PCM 儲存 2 bits/cell，使用 4 種電阻狀態：

```
狀態    電阻值      代表
00      最高       Amorphous
01      中高       Partial crystalline
10      中低       Partial crystalline
11      最低       Crystalline
```

**不對稱寫入問題**：

不同狀態轉換需要不同的能耗和延遲：

| 狀態轉換 | 能耗 | 原因 |
|---------|------|------|
| → 00 | 30 pJ | 需要 RESET 操作 |
| → 01 | 66.36 pJ | 精確控制部分結晶 |
| → 10 | 52.44 pJ | 精確控制部分結晶 |
| → 11 | 36 pJ | 需要 SET 操作 |

**能耗差異分析**：
- 最高：01 狀態（66.36 pJ）
- 最低：00 狀態（30 pJ）
- 差異：高達 2.2 倍

**延遲差異**：
- 最高和最低延遲差異可達 5 倍
- 原因：中間狀態（01, 10）需要精確的電流控制和驗證

**問題根源**：
1. PCM 的 SET 操作（結晶化）本質上較慢
2. 中間狀態需要精確的部分結晶
3. 需要 iterative programming（反覆寫入-驗證）

---

### Question 17
**What is "Resistance Drift" in MLC PCM? Explain the physical mechanism and why it is more severe in MLC than in SLC PCM.**

**詳解：**

**Resistance Drift（電阻漂移）定義**：

電阻漂移是 PCM 的一種物理現象，指 cell 的電阻值會隨時間自發性增加。這是由 PCM 材料（通常是 GST：Ge₂Sb₂Te₅）的固有特性造成的。

**物理機制**：

1. **結構弛豫（Structural Relaxation）**：
   - 非晶態（amorphous）的原子排列不穩定
   - 原子會緩慢重新排列以降低能量
   - 重排導致電阻增加

2. **漂移公式**：
   ```
   R(t) = R₀ × (t/t₀)^ν
   
   其中：
   R₀ = 初始電阻
   t₀ = 參考時間
   ν = 漂移係數（材料相關，通常 0.05-0.1）
   ```

3. **溫度影響**：
   - 較高溫度加速結構弛豫
   - 漂移率與溫度正相關

**為何 MLC 比 SLC 更嚴重**：

**SLC PCM**：
```
只有兩種狀態：High R (1) 和 Low R (0)
電阻間隔大，漂移不易造成讀取錯誤
```

**MLC PCM**：
```
四種狀態：00, 01, 10, 11
電阻間隔小，漂移容易造成狀態重疊

[圖示]
時間 t=0:    |---00---|---01---|---10---|---11---|
時間 t=T:    |----00----|---01---|--10--|--11--|
                     ↑ 重疊區域（讀取錯誤）
```

**具體影響**：
- 01 可能漂移到 00 的範圍
- 10 可能漂移到 01 的範圍
- 需要定期刷新（refresh）來維持資料正確性

---

### Question 18
**Describe the Partial-SET technique. What are its advantages and disadvantages in MLC PCM?**

**詳解：**

**Partial-SET 技術**：

Partial-SET 是一種折衷的寫入策略，透過縮短 SET 操作的脈衝時間來加速寫入，但代價是較短的資料保持時間。

**傳統 Full-SET vs Partial-SET**：

| 特性 | Full-SET | Partial-SET |
|------|----------|-------------|
| 脈衝時間 | 長（完整結晶化）| 短（部分結晶化）|
| 能耗 | 高 | 低 |
| 寫入延遲 | 長 | 短 |
| 資料保持時間 | 長（>10年）| 短（秒~分鐘）|
| 電阻穩定性 | 高 | 低（易漂移）|

**優點**：

1. **顯著降低能耗**：
   - 脈衝時間短，消耗電流少
   - 可減少 50% 以上的寫入能耗

2. **大幅減少寫入延遲**：
   - SET 操作是效能瓶頸
   - Partial-SET 可加速 2-3 倍

3. **減少熱累積**：
   - 較短的加熱時間
   - 降低對相鄰 cell 的熱干擾

**缺點**：

1. **嚴重的電阻漂移**：
   - 部分結晶狀態不穩定
   - 漂移速度比 Full-SET 快很多

2. **需要定期刷新**：
   - 資料可能在短時間內失效
   - 刷新增加額外的寫入開銷

3. **較高的位元錯誤率**：
   - Partial-SET 後 4-14 秒內錯誤率急劇上升
   - 需要額外的錯誤更正機制

---

### Question 19
**Explain the core idea of Drifting-tolerate (DT) Coding. How does it remap 3-bit datawords into 4-bit DT codes?**

**詳解：**

**DT Coding 核心概念**：

Drifting-tolerate Coding 的核心思想是：**設計一種編碼方式，使得即使 MLC PCM cell 發生電阻漂移，漂移後的狀態仍能正確解碼為原始資料**。

**編碼映射表**：

| 原始資料 (3-bit) | DT Code (4-bit) | 漂移後 Code | 漂移後仍可解碼 |
|-----------------|-----------------|-------------|---------------|
| 000 | 0001 | 0000 | ✓ |
| 001 | 0101 | 0100 | ✓ |
| 010 | 1001 | 1000 | ✓ |
| 011 | 1101 | 1100 | ✓ |
| 100 | 1111 | 1111 | ✓ (Full-SET) |
| 101 | 1110 | 1110 | ✓ (Full-SET) |
| 110 | 0110 | 0010 | ✓ |
| 111 | 0111 | 0011 | ✓ |

**設計原理**：

1. **利用 01→00 漂移特性**：
   - Partial-SET 產生的 "01" 狀態會漂移到 "00"
   - 設計編碼使 01→00 不影響解碼結果

2. **Partial-SET 狀態覆蓋率**：
   - DT Code 中 "01" 的比例：6/16 = 37.5%
   - 這些 "01" 可以使用 Partial-SET，節省能耗

3. **Full-SET 用於穩定狀態**：
   - 資料 100 和 101 使用 Full-SET（1111, 1110）
   - 這些狀態不會漂移

**編碼過程範例**：
```
原始資料：000 010 100
         ↓
DT 編碼：0001 1001 1111
         ↓ (時間流逝，01→00)
漂移後：  0000 1000 1111
         ↓
解碼：    000  010  100 ✓ 正確
```

---

### Question 20
**What is "Write Process Segmentation"? Explain how it improves the write performance of DT Coding.**

**詳解：**

**Write Process Segmentation（寫入過程分段）**：

這是一種將 MLC PCM 寫入過程分為「快速寫入單元」和「慢速寫入單元」的技術，以最大化並行度和效能。

**傳統寫入方式的問題**：

```
傳統方式：混合快慢狀態
Cache Line (64 bytes = 512 bits)
00 01 10 01 11 11 ... 01 10 11 01

Write Unit 0~7：混合 00, 01, 10, 11
    WU0  WU1  WU2  WU3  WU4  WU5  WU6  WU7
    ─────────────────────────────────────→ Time
    
問題：快狀態 "01" 必須等待慢狀態完成
```

**Write Process Segmentation 方案**：

```
分段方式：
Fast Write Unit (FWU)：只包含 "01" 狀態
Slow Write Unit (SWU)：包含 "00", "10", "11" 狀態

Cache Line 重組：
01 01 01 01 01 01 ... → FWU 0, 1, 2 (快速完成)
00 11 10 11 ...       → SWU 0, 1, 2, 3 (較慢)

    FWU0 FWU1 FWU2    SWU0  SWU1  SWU2  SWU3
    ──────────────────────────────────────→ Time
    └─快速完成─┘      └───較慢但並行───┘
```

**效能改善原因**：

1. **消除等待時間**：
   - 快狀態 "01" 不需要等待慢狀態
   - 可提前釋放寫入資源

2. **提高並行度**：
   - FWU 的功耗低，可以同時寫入更多 cell
   - 受功率限制 [17, 18]，混合狀態無法高並行

3. **功率預算優化**：
   ```
   傳統：每 WU 混合快慢 → 功率波動大
   分段：FWU 功率低，SWU 功率高 → 可優化調度
   ```

---

### Question 21
**Explain the "Redundant Write Elimination" technique in DT Coding. How does it further reduce energy consumption?**

**詳解：**

**Redundant Write Elimination（冗餘寫入消除）**：

這是一種透過「讀取後比較」來避免不必要寫入的技術。

**核心觀察**：
在 MLC PCM 中，讀取操作的能耗和延遲遠低於寫入操作（約 1%）。因此，先讀取再決定是否寫入是划算的。

**技術原理**：

```
原始資料（PCM 中）：0001 1000 1000 1110 0010 1111 0010 1000

要寫入的 DT Code：  ？？？？ ？？？？ ...

步驟 1：分離 FWU 和 SWU
FWU（目標）：01 01 01 01 01 01 01
SWU（目標）：00 10 00 11 11 10 11 11 10

步驟 2：讀取原始資料
FWU（原始）：00 10 00 11 11 10 11 10
SWU（原始）：01 00 00 10 00 11 00 00

步驟 3：比較並決定是否寫入
```

**FWU 的特殊優化**：

根據 DT Coding 表，"01" 和 "00" 代表相同的 3-bit 資料（因為漂移容忍設計）：
```
如果原始值是 "01" 或 "00" → 不需要更新
只有原始值是 "10" 或 "11" 時才需要寫入 "01"
```

**SWU 的優化**：
```
標準比較：如果原始值 ≠ 目標值 → 寫入
```

**效益分析**：

| 操作 | 能耗 | 備註 |
|------|------|------|
| 讀取 | 1% 的寫入能耗 | 很低 |
| 條件寫入 | 節省 30-50% | 避免冗餘 |

**為何對 DT Coding 特別有效**：
- DT Coding 最大化了 "01" 和 "00" 的使用（37.5%）
- 這兩個狀態在 FWU 中等效
- 大量減少 FWU 的實際寫入次數

---

### Question 22
**Compare DT Coding with other coding schemes: Baseline, TriState-SET, and Enhanced WOM. What are the energy consumption improvements?**

**詳解：**

**比較方法概述**：

| 方法 | 核心概念 | 目標 |
|------|---------|------|
| Baseline | 無特殊編碼 | 基準 |
| TriState-SET [21] | 使用三狀態編碼 | 減少 SET 操作 |
| Enhanced WOM [22] | 改進的 Write-Once Memory 碼 | 減少位元翻轉 |
| DT Coding | 漂移容忍編碼 | 容忍漂移 + 節能 |

**能耗實驗結果**：

| Benchmark | Baseline | TriState-SET | Enhanced WOM | DT Coding |
|-----------|----------|--------------|--------------|-----------|
| basicmath | 基準 | 改善 | 改善 | -5.89% |
| bitcount | 基準 | 改善 | 改善 | -7.27% |
| qsort | 基準 | 改善 | 改善 | -5.69% |
| susan | 基準 | 改善 | 改善 | -10.15% |
| dijkstra | 基準 | 改善 | 改善 | -10.08% |
| patricia | 基準 | 改善 | 改善 | -10.88% |

**DT Coding 的優勢分析**：

1. **不需要 PreSET 開銷**：
   - TriState-SET 和 Enhanced WOM 需要 PreSET 操作
   - PreSET 本身消耗能量
   - DT Coding 透過編碼設計避免此開銷

2. **漂移容忍**：
   - 其他方法仍受漂移影響，需要刷新
   - DT Coding 天然容忍 01→00 漂移
   - 節省刷新能耗

3. **Partial-SET 利用率**：
   - DT Coding 有 21.9-43.9% 的 Partial-SET 覆蓋率
   - 最大化節能效益

---

### Question 23
**Analyze the write latency improvements of DT Coding. What factors contribute to the 9.26-18.24% latency reduction?**

**詳解：**

**寫入延遲改善分析**：

**實驗結果**：

| Benchmark | 延遲減少 |
|-----------|---------|
| basicmath | -9.26% |
| bitcount | -10.58% |
| qsort | -10.59% |
| susan | -15.12% |
| dijkstra | -15.48% |
| patricia | -18.24% |

**貢獻因素分析**：

**1. Write Process Segmentation 的貢獻**：

```
傳統：WU0 → WU1 → ... → WU7（序列化，受最慢狀態限制）
總延遲 = max(狀態延遲) × 8

分段：FWU0~2（並行）→ SWU0~3（並行）
總延遲 = FWU延遲 + SWU延遲 < 傳統延遲
```

改善幅度：約 5-10%

**2. Partial-SET 的貢獻**：

```
Full-SET 延遲：~150ns
Partial-SET 延遲：~50ns（減少約 67%）

覆蓋率 21.9-43.9% → 整體改善 ~3-8%
```

**3. Redundant Write Elimination 的貢獻**：

```
避免不必要的寫入 → 直接節省時間
對於 FWU："01"/"00" 等效，減少寫入
改善幅度：約 2-5%
```

**4. 不同 Benchmark 差異的原因**：

- **patricia（-18.24%）**：資料模式有利於 DT Coding
  - 較高的 Partial-SET 覆蓋率（43.9%）
  - 較多的冗餘寫入可消除

- **basicmath（-9.26%）**：資料模式較不利
  - 較低的 Partial-SET 覆蓋率（21.9%）
  - 較少的優化空間

---

### Question 24
**What is "Partial-SET State Coverage"? How does DT Coding achieve 21.9-43.9% coverage in the experiments?**

**詳解：**

**Partial-SET State Coverage（部分設定狀態覆蓋率）定義**：

指在 DT 編碼後的資料中，可以使用 Partial-SET 操作寫入的 cell 比例。

**計算公式**：
```
Coverage = (使用 Partial-SET 的 cell 數) / (總 cell 數) × 100%
```

**DT Coding 如何達成高覆蓋率**：

**1. 編碼表設計優化**：

```
DT 編碼表中 "01" 的分布：
Data  DT Code  含有 "01" 的位置
000   0001     位置 1-2
001   0101     位置 0-1, 2-3
010   1001     位置 1-2
011   1101     位置 1-2
100   1111     無（Full-SET）
101   1110     無（Full-SET）
110   0110     位置 0-1
111   0111     位置 0-1

"01" 出現次數：6/16 = 37.5%（每 4-bit code 中）
```

**2. 資料分布影響**：

不同 benchmark 的資料特性不同：

| Benchmark | 原始資料分布 | Partial-SET 覆蓋率 |
|-----------|-------------|-------------------|
| basicmath | 較均勻 | 21.9% |
| bitcount | 偏向某些模式 | 28.5% |
| patricia | 有利模式較多 | 43.9% |

**3. 實現機制**：

```
Cache Line → DT 編碼 → 分離 FWU/SWU
                    ↓
              FWU：全部使用 Partial-SET
              SWU：使用 Full-SET/RESET
```

**4. 為何範圍是 21.9-43.9%**：

- **下限（21.9%）**：資料模式不利，"01" 出現較少
- **上限（43.9%）**：資料模式有利，接近理論最大值
- **理論最大值**：37.5%（隨機資料的期望值）

---

### Question 25
**Explain how the DT Coding table is designed to tolerate the drift from "01" to "00". Provide a detailed analysis of the encoding/decoding process.**

**詳解：**

**DT Coding 表設計原理**：

**設計目標**：確保 "01" 漂移到 "00" 後，解碼結果不變。

**設計約束**：
```
對於任意 3-bit 資料 D：
DT_Code(D) 漂移後的結果必須能唯一解碼回 D
```

**編碼表分析**：

| 資料 | DT Code | 漂移後 | 解碼 | 驗證 |
|-----|---------|-------|------|------|
| 000 | 0001 | 0000 | 000 | ✓ |
| 001 | 0101 | 0100 | 001 | ✓ |
| 010 | 1001 | 1000 | 010 | ✓ |
| 011 | 1101 | 1100 | 011 | ✓ |
| 100 | 1111 | 1111 | 100 | ✓ (無漂移) |
| 101 | 1110 | 1110 | 101 | ✓ (無漂移) |
| 110 | 0110 | 0010 | 110 | ✓ |
| 111 | 0111 | 0011 | 111 | ✓ |

**解碼表（包含漂移後狀態）**：

```
4-bit Code → 3-bit Data
0000 → 000（漂移後）
0001 → 000（原始）
0010 → 110（漂移後）
0011 → 111（漂移後）
0100 → 001（漂移後）
0101 → 001（原始）
0110 → 110（原始）
0111 → 111（原始）
1000 → 010（漂移後）
1001 → 010（原始）
1100 → 011（漂移後）
1101 → 011（原始）
1110 → 101（原始，無漂移）
1111 → 100（原始，無漂移）
```

**編碼/解碼過程範例**：

```
原始資料：010 111 100

步驟 1：編碼
010 → 1001
111 → 0111
100 → 1111
編碼結果：1001 0111 1111

步驟 2：存入 MLC PCM（使用 Partial-SET）
"01" 使用 Partial-SET，其他使用 Full-SET/RESET

步驟 3：時間流逝，漂移發生
1001 → 1000（01→00）
0111 → 0011（01→00）
1111 → 1111（無漂移）
漂移後：1000 0011 1111

步驟 4：解碼
1000 → 010 ✓
0011 → 111 ✓
1111 → 100 ✓
```

---

### Question 26
**Why are the states "100" and "101" encoded using Full-SET (1111 and 1110) in DT Coding? What would happen if Partial-SET were used for these states?**

**詳解：**

**為何 100 和 101 使用 Full-SET**：

**編碼表中的特殊處理**：
```
Data 100 → DT Code 1111 (全為 "11"，使用 Full-SET)
Data 101 → DT Code 1110 (含 "11" 和 "10"，無 "01")
```

**原因分析**：

**1. 編碼空間限制**：

```
4-bit DT Code 有 16 種可能值
需要映射 8 種 3-bit 資料
每種資料需要 2 個編碼（原始 + 漂移後）

已使用的編碼對：
000: 0001, 0000
001: 0101, 0100
010: 1001, 1000
011: 1101, 1100
110: 0110, 0010
111: 0111, 0011

剩餘編碼：1111, 1110, 1011, 1010
```

**2. 若 100 使用 Partial-SET**：

假設 100 → XX01（含 "01"）：
```
漂移後 → XX00
必須確保 XX00 不與其他漂移後編碼衝突
```

問題：
- 0001 的漂移後是 0000，若 100 也漂移到 0000 會衝突
- 編碼空間不足以為所有資料提供唯一的漂移後編碼

**3. Full-SET 的優勢**：

```
1111 和 1110：
- 不含 "01"，不會漂移
- 解碼穩定，不需要額外處理
- 犧牲這兩個資料的 Partial-SET 優化，換取整體正確性
```

**若強制使用 Partial-SET 會發生什麼**：

```
假設 100 → 0101（使用 Partial-SET）
漂移後 → 0100

但 0100 已經是 001 的漂移後編碼！
解碼時無法區分 001 和 100
→ 資料錯誤
```

**結論**：為了保證解碼的唯一性，必須犧牲部分資料（100, 101）的 Partial-SET 優化。

---

### Question 27
**Describe the power constraints that limit concurrent writes in MLC PCM. How does Write Process Segmentation address this limitation?**

**詳解：**

**MLC PCM 的功率限制**：

**物理約束**：
根據文獻 [17, 18]，MLC PCM 晶片有嚴格的功率預算：
- 每個寫入操作需要電流來加熱 cell
- 同時寫入太多 cell 會超過功率限制
- 超過限制可能導致電源不穩定或晶片損壞

**各狀態的功率需求**：

| 狀態 | 能耗 | 電流需求 |
|------|------|---------|
| 00 | 30 pJ | 中 |
| 01 | 66.36 pJ | 高 |
| 10 | 52.44 pJ | 高 |
| 11 | 36 pJ | 中 |

**傳統寫入的問題**：

```
Write Unit (WU) 包含混合狀態：
WU0: 00 01 10 01 11 11 ...

功率計算：
假設 WU 有 16 個 cell
混合狀態的平均功率 ≈ (30+66.36+52.44+36)/4 = 46.2 pJ/cell

功率限制允許同時寫入 N_max 個 cell
N_max = 功率預算 / 平均功率
```

**Write Process Segmentation 的解決方案**：

**1. 分離快慢狀態**：
```
FWU：只包含 "01" 狀態（Partial-SET）
     功率需求：低（Partial-SET 電流小）
     
SWU：包含 "00", "10", "11" 狀態
     功率需求：依狀態而定
```

**2. 提高並行度**：
```
FWU 的優勢：
- Partial-SET 功率低
- 可同時寫入更多 cell
- 假設功率限制允許同時寫 N_full 個 full-SET
- FWU 可同時寫 2N_full 個 Partial-SET

例：
傳統：每次 16 個 cell（混合）
FWU：每次 32 個 cell（全 Partial-SET）
```

**3. 調度優化**：
```
時間線：
t=0    t=T1        t=T2
|--FWU--|
        |----SWU----|

FWU 快速完成，釋放功率預算給 SWU
整體完成時間 < 傳統序列化方式
```

---

### Question 28
**Compare the read and write energy/latency in MLC PCM. Why is "read before write" (used in Redundant Write Elimination) an effective strategy?**

**詳解：**

**MLC PCM 讀寫能耗/延遲比較**：

| 操作 | 能耗 | 延遲 | 備註 |
|------|------|------|------|
| 讀取 | ~0.2 pJ/bit | ~50 ns | 非破壞性，低功率 |
| 寫入（平均）| ~40-60 pJ/bit | ~150-500 ns | 需加熱，高功率 |
| 比率 | ~1% | ~20-30% | 讀取遠低於寫入 |

**"Read Before Write" 策略分析**：

**基本概念**：
```
傳統寫入：直接覆寫所有 cell
Read-before-Write：
1. 讀取原始值
2. 比較原始值與目標值
3. 只寫入不同的 cell
```

**為何有效**：

**1. 能耗節省**：
```
假設有 100 個 cell 需要更新
其中 30% 的值已經正確

傳統：100 × 50 pJ = 5000 pJ（全寫入）
Read-before-Write：
  讀取：100 × 0.5 pJ = 50 pJ
  寫入：70 × 50 pJ = 3500 pJ
  總計：3550 pJ
節省：(5000-3550)/5000 = 29%
```

**2. 延遲節省**：
```
假設寫入可並行，讀取也可並行
傳統：寫入 100 cell 的延遲
Read-before-Write：
  讀取延遲（很短）+ 寫入 70 cell 的延遲
整體延遲減少
```

**3. 對 DT Coding 特別有效的原因**：

```
DT Coding 的 FWU 特性：
- "01" 和 "00" 代表相同資料
- 只需比較：原始值 ∈ {01, 00}？
- 若是，則跳過寫入

期望跳過率：
- 隨機資料中 "01" 和 "00" 各佔 25%
- 預期跳過 50% 的 FWU 寫入
```

**4. 壽命改善**：
```
減少寫入次數 → 減少位元翻轉 → 延長 PCM 壽命
這對有限耐久度（~10^8 次）的 PCM 很重要
```

---

### Question 29
**What benchmarks (MiBench) were used to evaluate DT Coding? Why are these benchmarks representative for embedded and IoT applications?**

**詳解：**

**MiBench Benchmark Suite [20]**：

MiBench 是一套專為嵌入式系統設計的基準測試程式，由密西根大學開發。

**使用的 Benchmark**：

| Benchmark | 類別 | 應用場景 |
|-----------|------|---------|
| basicmath | 數學運算 | 數值計算 |
| bitcount | 位元操作 | 加密、編碼 |
| qsort | 排序演算法 | 資料處理 |
| susan | 影像處理 | 邊緣偵測 |
| dijkstra | 圖形演算法 | 路徑規劃 |
| patricia | 資料結構 | 路由表查找 |

**為何具代表性**：

**1. 嵌入式系統特性**：
```
特點：
- 記憶體受限
- 能源敏感
- 即時性要求

MiBench 模擬這些特點：
- 程式碼緊湊
- 資料存取模式真實
- 計算/記憶體操作比例合理
```

**2. IoT 應用相關性**：

```
basicmath：感測器資料處理（溫度、壓力計算）
bitcount：低功耗加密
qsort：資料整理
susan：影像監控（邊緣運算）
dijkstra：智慧導航、網路路由
patricia：網路封包處理
```

**3. 記憶體存取模式多樣性**：

```
不同 Benchmark 的存取模式：
- basicmath：序列存取為主
- patricia：隨機存取為主
- susan：區塊存取（影像處理）

這種多樣性確保 DT Coding 在不同場景下都被測試
```

**4. 業界認可**：
- 被超過 1000 篇論文引用
- 是嵌入式系統研究的事實標準
- 結果具有可比性和可重現性

---

### Question 30
**Summarize the key contributions of both Write-friendly Arithmetic Coding and Drifting-tolerate Coding. How do they collectively improve PCM-based storage systems?**

**詳解：**

**Write-friendly Arithmetic Coding 關鍵貢獻**：

**1. 可忽略位元概念**：
- 發現壓縮編碼區間內存在可忽略的尾數位元
- 這些位元無論是 0 或 1 都不影響解碼正確性

**2. 兩階段演算法**：
- Upper Bits Preselecting：找出最少位元表示的高位
- Ignorable Bits Determining：最大化可忽略的低位

**3. 實驗效果**：
- RESET 操作減少 10-48%
- SET 操作減少 13-41%
- 完全無損失壓縮

---

**Drifting-tolerate Coding 關鍵貢獻**：

**1. 漂移容忍編碼設計**：
- 3-bit 資料映射到 4-bit DT Code
- 確保 01→00 漂移後仍能正確解碼

**2. Write Process Segmentation**：
- 分離快速（FWU）和慢速（SWU）寫入單元
- 提高並行度，減少總延遲

**3. Redundant Write Elimination**：
- 讀取後比較，避免不必要寫入
- 利用 DT Code 的 "01"/"00" 等效特性

**4. 實驗效果**：
- 能耗減少 5.69-10.88%
- 延遲減少 9.26-18.24%
- 21.9-43.9% Partial-SET 覆蓋率

---

**協同改善 PCM 系統**：

**1. 針對不同層級的優化**：
```
應用層：Write-friendly AC（資料壓縮）
         ↓ 減少需要儲存的資料量
儲存層：DT Coding（資料編碼）
         ↓ 優化寫入方式
硬體層：MLC PCM
```

**2. 解決 PCM 的核心挑戰**：

| 挑戰 | Write-friendly AC 貢獻 | DT Coding 貢獻 |
|------|------------------------|----------------|
| 高寫入能耗 | 減少寫入量 | 使用 Partial-SET |
| 長寫入延遲 | 減少寫入量 | Write Process Segmentation |
| 電阻漂移 | 不直接解決 | 漂移容忍設計 |
| 有限耐久度 | 減少位元翻轉 | 減少寫入次數 |

**3. 互補性**：
```
Write-friendly AC：最佳化「壓縮後的資料」
DT Coding：最佳化「儲存資料的方式」

兩者可以串聯使用：
原始資料 → Write-friendly AC 壓縮 → DT Coding 編碼 → 寫入 PCM
```

**4. 整體效益**：
- 延長 PCM 使用壽命
- 降低系統能耗
- 提高資料可靠性
- 改善系統效能

---

## 參考文獻

[4] Huffman, David A. "A method for the construction of minimum-redundancy codes." Proceedings of the IRE 40.9 (1952): 1098-1101.

[5] Witten, Ian H., Radford M. Neal, and John G. Cleary. "Arithmetic coding for data compression." Communications of the ACM 30.6 (1987): 520-540.

[9] Lee, Benjamin C., et al. "Architecting phase change memory as a scalable dram alternative." ISCA 2009.

[10] Mahoney, M. (2011). "Large text compression benchmark".

[11] Peng, C-K., et al. "Exaggerated heart rate oscillations during two meditation techniques." International journal of cardiology 70.2 (1999): 101-107.

[12] Ielmini, Daniele, et al. "Reliability impact of chalcogenide-structure relaxation in phase-change memory (PCM) cells—Part I: Experimental study." IEEE TED 2009.

[14] Li, Bing, et al. "Partial-SET: Write speedup of PCM main memory." DATE 2014.

[16] Joshi, Madhura, Wangyuan Zhang, and Tao Li. "Mercury: A fast and energy-efficient multi-level cell based phase change memory system." HPCA 2011.

[20] Guthaus, Matthew R., et al. "MiBench: A free, commercially representative embedded benchmark suite." WWC-4, 2001.

[21] Zhang, Xianwei, Youtao Zhang, and Jun Yang. "Tristate-set: Proactive set for improved performance of mlc phase change memories." ICCD 2015.

[22] Luo, Huizhang, et al. "Energy, latency, and lifetime improvements in MLC NVM with enhanced WOM code." ASP-DAC 2018.
