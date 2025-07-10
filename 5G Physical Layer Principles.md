# Chapter 2
## 2.1 ARCHITECTURE
![image](https://github.com/user-attachments/assets/c80f5e6b-7886-431a-adf2-82d7e0de1d80)
![image](https://github.com/user-attachments/assets/44943f7d-c789-441e-b071-03e841b23f21)

| 協定層級     | 全名                               | 主要功能                                              |
| -------- | -------------------------------- | ------------------------------------------------- |
| **SDAP** | Service Data Adaptation Protocol | - mapping between QoS flow ↔ Radio bearers <br>-  IP packet 照QoS映射到Radio bearers |
| **PDCP** | Packet Data Convergence Protocol | - IP header 壓縮／解壓<br>- 加密與完整性保護<br>- 重排序、重複資料檢測   |
| **RLC**  | Radio Link Control               | - ARQ 錯誤修正<br>- 資料分段與重組<br>- 按序資料傳遞               |
| **MAC**  | Medium Access Control            | - HARQ 錯誤修正<br>- 排程與資源分配            |
| **PHY**  | Physical Layer                   | - 編碼／調變處理<br>- （MIMO）天線發射與接收          |


## 2.2
### 2.2.1 MODULATION
| 類別             | 調變方式                                 | 備註                    |
| -------------- | ------------------------------------ | --------------------- |
|  下行（Downlink） | QPSK, 16QAM, 64QAM, 256QAM           | 與 LTE 相同              |
|  上行（Uplink）   | QPSK, 16QAM, 64QAM, 256QAM, π/2-BPSK | π/2-BPSK 提升Low data rates時的效率，<br>適合 mMTC(massive Machine Type Communications，大規模機器型通訊) |
[MODULATION](https://github.com/Karlyoo/LDPCinOAI/blob/main/modulation.md#modulation)
### 2.2.2 WAVEFORM
| 技術      | 下行波形    | 上行波形                        | 備註               |
| ------- | ------- | --------------------------- | ---------------- |
| **NR**  | CP-OFDM | CP-OFDM（主）<br>DFTS-OFDM（可選） | 上下行統一主波形，可根據場景選擇 |
-  在上、下行使用相同波形有助於簡化整體系統設計
-  DFTS-OFDM 適用情境:覆蓋範圍受限、單流上行傳輸、無空間多工，特別對於硬體受限的UE
### 2.2.3 MULTIPLE ANTENNAS
  
| 技術   For low frequencies            |                  |
| ---------------- | ------------------- |
| LTE 發展延伸         | 建立在既有多天線功能上         |
| 頻譜效率需求高          | 面對頻譜擁擠，需提升容量與速率     |
| MU-MIMO 支援增強 | 同時服務多使用者，提升空間重複使用效率<br>控制大量天線，提升空間解析度與效率 |
| CSI 架構更新     | 靈活參考訊號傳輸、更高解析度、易擴展  |

| 技術  For high frequencies               |                             |
| ------------------ | ----------------------------- |
| 挑戰重點           | 涵蓋不足（非效率）                     |
| 波束成形是        | 特別適用於 LoS 場景                  |
| 類比 beamforming | 適合現有硬體限制                      |
| 支援初始接入與廣播      | NR 首次支援 beamforming 應用於非資料面傳輸 |

### 2.2.4 CHANNEL CODING
-  使用的通道編碼類型

| 應用類型        | 使用的通道編碼            | 備註                  |
| ----------- | ------------------ | ------------------- |
| 數據傳輸（MBB）   | LDPC（低密度同位檢查碼）   | 支援高資料率與 HARQ        |  <----目前5G NR使用
| 控制訊號（如 RRC） | Polar Code（極化碼）  | 適合短封包，支援 SC-List 解碼 |
| 最小控制負載      |  Reed–Muller Code | 僅用於極短封包控制場景         |

## 2.3 PHYSICAL TIME-FREQUENCY RESOURCES

| 單位                                | 定義                       | 備註                  |
| --------------------------------- | ------------------------ | ------------------- |
| **Resource Element (RE)**         | 1 子載波 × 1 OFDM symbol        | 最小資源單位              |
| **PRB (Physical Resource Block)** | 12 子載波 × N OFDM symbols  | NR 中最基本的調度單位        |
| **Slot**                          | 14 個 OFDM symbols        | 可根據 numerology 調整長度 |
| **Mini-slot**                     | 2 / 4 / 7 個 OFDM symbols | 可選擇適用於突發或低延遲場景         |
| **Subframe**                      | 1 ms                     | 包含一個或多個 slots       |
| **Frame**                         | 10 ms                    | 包含 10 個 subframes   |
 * OFDM（正交分頻多工，Orthogonal Frequency Division Multiplexing）:將資料分散到許多子載波上，正交意旨互不干擾，每個子載波傳送一小部分資料，使整體能不受干擾的傳送。
## 2.4 PHYSICAL CHANNELS
-  **下行downlink**

| 通道                                           | 功能                                                 |
| -------------------------------------------- | ------------------------------------------------------ |
| **PDSCH**（Physical Downlink Shared Channel）  | 用於下行資料傳輸                                               |
| **PDCCH**（Physical Downlink Control Channel） | 傳送下行控制資訊<br>ex:‣ PDSCH 的排程<br>‣ 授權 UE 傳送上行資料（PUSCH） |
| **PBCH**（Physical Broadcast Channel）         | 廣播系統資訊，協助 UE 尋找並連線至網路                                     |

-  1.UE 監控 PDCCH：通常每 slot 至少一次。
-  2.若偵測到有效 PDCCH，UE 接收對應 PDSCH 上的資料（稱為 transport block）。
-  3.UE 回傳 HARQ 回應：表示資料是否正確解碼，若錯誤則 gNB 安排重傳。


-  **上行uplink**

| 通道                                         | 功能說明                           |
| ------------------------------------------ | ------------------------------ |
| **PUSCH**（Physical Uplink Shared Channel）  | UE 上傳數據的通道                     |
| **PUCCH**（Physical Uplink Control Channel） | UE 傳送控制訊息（如 ACK/NACK、排程請求(SR)、CSI） |
| **PRACH**（Physical Random Access Channel）  | UE 發送隨機接入請求，用於建立連線或初始連接        |

-  1.UE 透過 PUCCH 傳送排程請求（SR） 至 gNB，。
-  2.gNB 透過PDCCH 排程授權（scheduling grant），授權 UE 使用某些 PUSCH 資源。
-  3.UE 透過 PUSCH 傳送資料，並由 gNB 接收與解碼。
-  4.gNB 傳送 HARQ ACK/NACK：指示資料是否解碼成功。

## 2.5 PHYSICAL SIGNALS

| 信號      | 全名                                       | 傳輸方向 | 功能用途                |
| ------- | ---------------------------------------- | ---- | ------------------- |
| DM-RS   | Demodulation Reference Signal            | 上／下行 | 通道估計                |
| PT-RS   | Phase Tracking Reference Signal          | 上／下行 | 相位雜訊補償              |
| CSI-RS  | Channel State Information RS             | 下行   | CSI 獲取、同步|
| PSS/SSS | Primary/Secondary Synchronization Signal | 下行   | 初始接入與同步             |
| SRS     | Sounding Reference Signal                | 上行   | 測量與 beamforming   |

```
- DM-RS（解調參考信號）
  -  用於估計解調所需的通道資訊（channel estimation）
  -  為 UE 專屬（UE-specific）
  -  可搭配 beamforming 使用
  -  支援 slot 前置（front-loaded）配置
  -  在低速場景：使用低密度
   在高速場景：高密度（增加 OFDM 符號內的 RE 數量）
- PT-RS（相位追蹤參考信號）
  -  補償傳送過程引起的相位雜訊（phase noise）
  -  特別適用於高頻傳輸
  -  疏頻配置於頻域、高密度配置於時域
  -  PT-RS 的分布依據：震盪器品質、頻率、子載波間距、調變方式等

- CSI-RS（通道狀態資訊參考信號）
   -  用途：
     - 獲取CSI（計算 CQI、PMI、RI 等）
     - beam management / RSRP 量測 
     - 時頻追蹤 / 頻偏補償

- SRS（Sounding Reference Signal）
   -  上行傳輸，用於：
       - CSI 測量、上行 link adaptation
       - UE 回報用於 precoding 的通道資訊（Reciprocity-based beamforming），始能順利接至天線
```
# Chapter 8
## 8.1 Fundamental Limits of FEC
### 8.1.1 Binary AWGN Channel
- binary-input AWGN (bi-AWGN) channel：
  - yk = √ρ * xk + wk, k = 1, ..., n
  - xk ∈ {−1, 1} 是 BPSK 調變符號（來自bit ck）
  - wk ∼ N(0,1) 為獨立同分佈的高斯雜訊
  - ρ : SNR（Signal-to-Noise Ratio）
  - n 是一個packet使用的總 channel uses（即 blocklength）
### 8.1.2 Coding Schemes for bi-AWGN Channel
- 傳輸資料透過編碼器轉換為碼字，並以 BPSK 映射：
  - xk = 2 * ck − 1
  - Def：一個 (n, M, ε) 的 coding scheme 包括：
  - 
|Encoder：| Decoder：|
|---------|----------|
|f : {1, ..., M} → F2^n<br>將訊息編碼為 binary codeword|g : R^n → {1, ..., M}<br>根據接收到的 y ∈ R^n 預測訊息、回報錯誤|

   - 錯誤率限制：平均錯誤率 ≤ ε
- 傳輸速率：R = log2(M) / n = k / n

註：a code 是bits的集合；a coding scheme 則包含編碼器和解碼器

### 8.1.3 Performance Metrics
- 最大可達速率 R(n, ε)： R*(n, ε) = sup { log2(M)/n : 存在 (n, M, ε) coding scheme }
- 反映 blocklength 與錯誤率 ε 的 trade-off
- Shannon 理論：
   - 若 R < C，則隨著 n → ∞，可以做到錯誤率趨近於 0
   - 若 R > C，則錯誤率趨近於 1
   - 對 bi-AWGN channel，容量 C 由以下積分公式表示：
```
C = ∫ (1 / √(2π)) * e^(−z²/2) * [1 − log2(1 + e^(−2ρ + 2z√ρ))] dz
實際上，Shannon 的 capacity 是 n → ∞ 的極限值，對 5G 等低延遲應用來說並不適用
```
#### Finite Blocklength Theory:
- 使用 finite-blocklength 工具，可對 R*(n, ε) 給出 上下界。
- 對 bi-AWGN channel 而言，存在：
  - Converse bound：上界 on R*(n, ε)、下界 on ε*(n, R)
  - Achievability bound（RCUs）：下界 on R*(n, ε)、上界 on ε*(n, R)
- Normal approximation（正態近似）：
  ```
  - R*(n, ε) ≈ C − √(V/n) * Q⁻¹(ε) + (1/(2n)) * log2(n)
  - Q⁻¹ 是 Gaussian Q 函數的反函數，V 為 channel dispersion，表示為：
  - V = ∫ (1 / √(2π)) * e^(−z²/2) * [1 − log2(1 + e^(−2ρ + 2z√ρ)) − C]² dz
  ```
#### Minimum Packet Error Probability
- 對固定的 n, R，定義：
  - ε*(n, R) = min { ε : 存在 (n, 2^(nR), ε) coding scheme }
- 對應的近似式為：
  - ε*(n, R) ≈ Q((C − R + (1/(2n)) * log2(n)) / √(V/n))
  - 可進一步將能量效率表示為：
    - Eb/N0 = ρ / (2R)
#### Normalized Rate Rnorm
- 定義 normalized rate 以衡量實際與理論極限之間的差距：
  - Rnorm = R / R*(n, ε)
  - Rnorm 越接近 1，表示編碼 scheme 越接近最佳表現

### 8.2.2  Definitions
#### 線性區塊碼 Linear Block Code
- 一個 (n, k) 線性區塊碼是 F2^n 中的一個 k 維度子空間。
- 共有 2^k 個碼字，且每個碼字可以寫成 k 個線性無關碼字的線性組合。
#### 產生矩陣 Generator Matrix G
- 記碼字的基底為 {g1, g2, ..., gk}，則產生矩陣為：
- G = [ g1
      g2
      ...
      gk ]
- 編碼公式：c = b * G
  - b: 為 k-bit 的資料（行向量）
  - c: 編碼後的碼字（長度為 n）

#### 同質空間與同質碼 Parity Check Matrix H
- C 的正交補空間（dual space）定義為：
  - C⊥ = { ĉ ∈ F2^n : ĉ · cᵀ = 0, ∀ c ∈ C }
  - 選擇 {h1, h2, ..., h_{n−k}} 為 C⊥ 的一組基底，則 同質矩陣 (H) 為：
  - H = [ h1
      h2
      ...
      h_{n−k} ]

c * Hᵀ = 0 ⇨ 表示 c ∈ C
註：G 和 H 可以唯一地描述一個線性碼，換言之，兩者一定要正交，表示資料無誤。

#### 最佳解碼：最大似然解碼 (ML(maximum-likelihood) decoding)
- ML原理:在觀察到接收端的訊號 y 之後，找出最有可能被傳送的原始碼字 x(m)
- 假設所有訊息等機率選取，則 ML 解碼規則 為：
  - j = arg max_{m ∈ {1, ..., 2^k}} p(y | x(m))
  - 對 bi-AWGN channel，有：
    - p(y | x) = (1 / √(2π)) * exp(−‖y − √ρ x‖² / 2)
  - ML 解碼可寫為：
    - j = arg min_{m ∈ {1, ..., 2^k}} ‖y − √ρ x(m)‖²
    - ML 解碼器會選擇在 Euclidean 空間中與接收到的 y 最近的碼字 x(m)
    - x(m) 是可能的 BPSK 編碼碼字（例如：[-1, +1, -1, ...]）
    - y 是傳送 x(m) 並加上高斯雜訊後的實數向量
    - √ρ 是 SNR 對碼字振幅的影響

#### 錯誤率與距離有關
- 若傳送的是 x(1)，錯選 x(m≠1) 的機率為：
  - P(j = m | j = 1) = Q(√ρ * ‖x(m) − x(1)‖ / 2)
  - 若 Kd 為平均上與碼字距離為 d 的碼字數量，則由聯集界 (union bound) 有：
    - ε ≤ ∑_d Kd * Q(√ρ * d / 2)
  - 上界通常由最小距離支配：
    - ε ≈ K_{dmin} * Q(√ρ * dmin / 2)
    - dmin：最小歐幾里得距離，等於 非零碼字中最小 Hamming weight 的兩倍

### 8.2.3  LDPC Codes
#### Definition
- LDPC (Low-Density Parity-Check) codes 是一種線性區塊碼，其特徵為：
  - Parity Check Matrix (PCM) 是一個稀疏矩陣（大多為 0）
  - 最早由 Gallager 提出（1960s），1990s 再被重新發現與推廣
  - 接近 Shannon capacity，且已被廣泛應用於多個標準（如 Wi-Fi、WiMAX、DVB-S2）
- Tanner Graph 表示法
  -LDPC 可使用 Tanner Graph（雙部圖）表示
  - 節點分為兩類：
    - Variable nodes (VNs)：每個對應一個碼位，共 n 個
    - Check nodes (CNs)：每個對應一個同質條件，共 m 行（m ≥ n − k）
  - PCM 中若 H[i,j] = 1，則 Tanner Graph 中 CN_i 與 VN_j 連接
#### 規則與非規則 LDPC
- Regular LDPC：所有 VNs 與 CNs 具有相同度數（連線數）
- Irregular LDPC：VNs 或 CNs 的度數不同 ➜ 通常效能更好
- 用 degree distributions 表示（每種度數的邊數比例）
#### 解碼原理：Belief Propagation (BP)
- 解碼流程為迭代式傳遞 log-likelihood ratios (LLRs)：
- VN phase：每個 VN 接收來自通道與 CNs 的 LLR，並產生輸出給鄰近 CNs
- CN phase：每個 CN 收到 VNs 的 LLR，計算出更新後的值，傳回給 VNs
- 重複直到： 成功找到滿足 Hcᵀ = 0 的碼字，或達到最大迭代次數

#### LDPC 編碼設計方法

|隨機法（pseudorandom with cycle avoidance）：|結構性設計法：Protograph-based LDPC：|
|---|----|
|可達到好效能<br>但結構性不足 ➜ 編碼與實作複雜度高，不適用於實務|先設計一個小的 base matrix (Hb)，然後進行 lifting：<br>每個元素替換為 Q × Q 的子矩陣（如循環置換矩陣）<br>可生成 quasi-cyclic LDPC code<br>有利於簡化編碼與實作，效能損失極小|
(OAI實作為QC-LDPC)
####  The LDPC-Code Solution Chosen for 5G NR
- 使用 quasi-cyclic LDPC codes
- 為支援 HARQ，有 rate-compatible 結構
  -定義了 兩組 base matrices 以支援不同訊息長度與速率需求
  - Base Matrix 設計（如圖 Fig. 8.4）
    
  |Base Matrix 編號	|用途	|尺寸	|系統碼元數	|支援速率	|最大 k 值|
  |-----------------|-----|-----|-----------|---------|---------|
  |Base Matrix #1|	長封包、高速率	|46 × 68|	22|	1/3 ~ 8/9|	8448 bits|
  |Base Matrix #2|	短封包、低速率|	42 × 52|	10	|1/5 ~ 2/3|	3840 bits|
  
- 特點：
  - 前兩欄（灰色）為 punctured bits：實際不傳送，但提升效能（改善閾值）
  - 核心區（藍色）：dual-diagonal 結構，有助於快速編碼
  - 低速率傳輸：可依需求擴展更多 parity bits（使用粉紅色區塊）
  - 外部行設計為近似正交 ➜ 提高解碼並行性
![image](https://github.com/user-attachments/assets/e4bdf8ca-9edb-4077-8a65-c15886d4f271)

**規範可參照 3GPP TR 38.212 LDPC CODING**

## 8.3 CODING SCHEMES FOR FADING CHANNELS
### 8.3.1 The SISO Case
- 使用 OFDM 傳輸，每個碼字跨越多個 Resource Blocks (RBs)：
  - 每個 RB 位於不同頻率，但同一時間送出
  - 每個 RB 含有 d 個 OFDM symbols，u 個子載波
  - 總符號數：n_c = d × u
#### Fading Channel 模型
- Block-memoryless fading 假設：
  - 每個 coherence bandwidth 內，通道保持不變
  - 不同 RB 視為互相獨立的 fading 分支
  - 定義參數：
     - B：整體頻寬
     - B_c：coherence bandwidth
     - L_max = ⌊B / B_c⌋：最多可利用的頻率多樣性數 L_max
     - L ≤ L_max：實際使用的 diversity branches
  - 通道輸入輸出關係（每個 RB）：
     - y_l = √ρ × h_l × x_l + w_l,   l = 1,...,L
#### Pilot-assisted 傳輸與解碼
- 每個 RB 有 n_p 個 pilot symbols（已知符號），其餘為資料符號
- x_l = [x_l^(p), x_l^(d)]
- 假設調變為QPSK, 單位能量，接收端使用 ML channel estimation：
  - ĥ_l = y_l^(p) × (x_l^(p))ᴴ / (n_p × √ρ)
  - 解碼器採用 mismatch decoding（假設 ĥ_l 為真通道）：
  -ĵ = argmin_{m ∈ {1, ..., 2^k}} ∑_{l=1}^{L} ‖y_l^(d) − ĥ_l × x_l^(d)(m)‖²
![image](https://github.com/user-attachments/assets/2852764f-9304-4c3e-af62-9779b2618e5c)
- 圖 8.11：與理論界限比較，設定：L = 7、k = 81、n = 186、np = 2 或 4
- 使用 (324, 81) tail-biting convolutional code（記憶長 m = 14）
- 經過：
   - Pseudorandom interleaver
   - Puncturing：去掉一些符號，空出 pilot 空間
   - QPSK 調變，解碼採用 OSD (Ordered Statistics Decoding), t = 3
- 結果：
   - 與理論極限差距約為 1 dB
**np 設太小或太大都會損失效能 ➜ 需最適化** 

![image](https://github.com/user-attachments/assets/24793a26-8d33-4e5e-b540-51c392305b28)
- 圖 8.12：設定：k = 30, n = 288, Rayleigh fading
- 比較：(A) L = 4、(B) L = 12
**顯示更多頻率多樣性能明顯降低封包錯誤率（PER）**

### 8.3.2 The MIMO Case
- 使用多天線架構（MIMO） 提供空間多樣性 (spatial diversity)
- 發射端若無通道資訊，需透過空間-頻率碼 (space-frequency codes) 尋找資訊
- 在圖 8.12 中，與SISO比較:
  
| 模式                 | (A) L = 4                     | (B) L = 12              |
| ------------------ | ----------------------------- | ----------------------- |
| **SISO**           | 最差表現                          | 最差表現                    |
| **1 × 2 SIMO**     | 明顯改善                          | 表現仍受限                   |
| **2 × 2 Alamouti** | 表現與 1×2 接近                    | 受通道估計誤差影響，甚至 **劣於 1×2** |
| **1 × 4 SIMO**     | **唯一達成 PER < 10⁻⁵**（URLLC 標準） | 同樣是最佳                   |

- Alamouti 雖然理論上提供等同 1x4 的多樣性，但對通道估計誤差更敏感：
  - 特別是在 n_c 小時（如 B 圖） ➜ Pilot overhead 高，估計品質差
