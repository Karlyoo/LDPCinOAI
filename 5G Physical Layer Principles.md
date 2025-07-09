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
# CH8

8.1.1 二元輸入加性高斯白噪聲通道（The Binary AWGN Channel）
通道模型

定義：bi-AWGN通道是一個無記憶離散時間通道，數學模型為：[y_k = \sqrt{\rho} x_k + w_k, \quad k = 1, \dots, n]其中：

( y_k )：接收信號。
( x_k )：發送信號，屬於二元字母表 ({ -1, 1 })（對應BPSK調製）。
( w_k )：加性高斯白噪聲，獨立同分佈（i.i.d.），均值為0，方差為1的正態分佈隨機變量。
( \rho )：信噪比（SNR），表示信號功率與噪聲功率的比值。
( n )：通道使用的次數，等於傳輸一個信息包的區塊長度（blocklength）。


特點：

無記憶性：每次通道使用獨立，無時間相關性。
二元輸入：限制輸入符號為 ({ -1, 1 })，對應二元相位移鍵控（BPSK）。
該模型簡化了分析，但適用於5G中許多實際場景的理論研究。




8.1.2 bi-AWGN通道的編碼方案（Coding Schemes for the Binary-AWGN Channels）
編碼方案的定義

編碼方案結構：一個 ((n, M, \epsilon)) 的二元編碼方案包括：

編碼器：
映射函數：( f: {1, 2, \dots, M} \to \mathbb{F}_2^n )。
將信息消息 ( j \in {1, 2, \dots, M} )（共 ( M = 2^k ) 個消息，( k ) 為信息比特數）映射到碼字集合 ({ c_1, \dots, c_M })，其中 ( c_m \in \mathbb{F}_2^n )。
碼字集合稱為通道碼或碼本（codebook）。
每個二元碼字 ( c_k ) 通過映射 ( x_k = 2c_k - 1 ) 轉換為BPSK符號 ({ -1, 1 })。


解碼器：
映射函數：( g: \mathbb{R}^n \to {1, 2, \dots, M} )。
將接收序列 ( y \in \mathbb{R}^n ) 解碼為消息 ( \hat{j} )，或宣佈錯誤。
滿足平均包錯誤概率約束：[P(\hat{j} \neq j) \leq \epsilon]


碼率（Rate）：
定義為：( R = \frac{\log_2(M)}{n} = \frac{k}{n} )（單位：比特/通道使用）。
表示每個通道使用傳輸的信息比特數。




術語澄清：

碼（Code）：僅指碼字集合。
編碼方案（Coding Scheme）：包括碼字、編碼器和解碼器。
不同解碼算法（例如最大似然解碼或信念傳播解碼）可能導致不同複雜度的編碼方案，即使使用相同的碼字集合。



實現挑戰

編碼方案需要平衡以下因素：
碼率 ( R )：希望盡可能高以提高數據傳輸效率。
區塊長度 ( n )：受延遲約束影響，特別是在5G URLLC場景中，( n ) 可能較小。
錯誤概率 ( \epsilon )：需要足夠低以確保可靠性。


在有限區塊長度下，實現高碼率和低錯誤概率是一大挑戰。


8.1.3 性能指標（Performance Metrics）
最大碼率 ( R^*(n, \epsilon) )

定義：[R^*(n, \epsilon) = \sup \left{ \frac{\log_2(M)}{n} : \exists (n, M, \epsilon) \text{ coding scheme} \right}]

表示在給定區塊長度 ( n ) 和錯誤概率 ( \epsilon ) 的約束下，可實現的最大碼率。
描述了碼率、區塊長度和錯誤概率之間的基礎權衡。


Shannon的理論貢獻：

在1948年，Shannon通過隨機編碼論證，證明當 ( n \to \infty ) 時，最大碼率 ( R^*(n, \epsilon) ) 趨向於通道容量 ( C )。
通道容量 ( C ) 是通道的固有屬性，對於bi-AWGN通道，容量公式為：[C = \frac{1}{\sqrt{2\pi}} \int e^{-\frac{z^2}{2}} \left[ 1 - \log_2 \left( 1 + e^{-2\rho + 2z\sqrt{\rho}} \right) \right] dz]
結論：
若 ( R < C )，存在一系列編碼方案，使得錯誤概率隨 ( n \to \infty ) 趨於0。
若 ( R > C )，對於大多數實際通道（包括bi-AWGN），錯誤概率趨向於1。




挑戰：

Shannon的證明基於隨機編碼，屬於非構造性方法，無法直接提供實際的編碼方案。
接近通道容量的實際編碼方案花費了約50年才實現（如LDPC碼、Turbo碼）。



非漸進分析（Nonasymptotic Analysis）

背景：

通道容量 ( C ) 是漸進性能指標，適用於 ( n \to \infty )。
在5G場景（如URLLC）中，延遲約束導致 ( n ) 較小，通道容量無法準確描述性能。
因此，需要研究有限區塊長度下的 ( R^*(n, \epsilon) )。


方法：

通過有限區塊長度信息論工具（如文獻[29]），可計算 ( R^*(n, \epsilon) ) 的上下界：
上界（Converse Bound）：基於最小最大對話（minimax converse，文獻[29, Thm. 27]）。
下界（Achievability Bound）：基於隨機編碼聯合界（Random-Coding Union, RCU，文獻[29, Thm. 16]）的放寬版本（RCUs，文獻[23, Thm. 1]）。


這些界限在實際參數範圍內（例如 ( \epsilon = 10^{-4}, \rho = 0 , \text{dB} )）非常緊密。


圖表分析（圖8.1）：

圖8.1A：顯示 ( R^*(n, \epsilon) ) 隨區塊長度 ( n ) 的變化，參數為 ( \epsilon = 10^{-4}, \rho = 0 , \text{dB} )。

上界和下界顯示最大碼率隨 ( n ) 增加逐漸接近通道容量 ( C )。
正常近似（Normal Approximation）：[R^*(n, \epsilon) \approx C - \sqrt{\frac{V}{n}} Q^{-1}(\epsilon) + \frac{1}{2n} \log_2 n]其中：
( V )：通道分散（Channel Dispersion），表示通道隨機性的度量：[V = \frac{1}{\sqrt{2\pi}} \int e^{-\frac{z^2}{2}} \left[ 1 - \log_2 \left( 1 + e^{-2\rho + 2z\sqrt{\rho}} \right) - C \right]^2 dz]
( Q^{-1}(\cdot) )：高斯Q函數的逆函數。
正常近似在高SNR或中等錯誤概率下準確，但在低SNR和低錯誤概率下可能失效（需使用基於鞍點方法的近似，文獻[23]）。




圖8.1B：顯示最小錯誤概率 ( \epsilon^*(n, R) ) 隨比特能量與噪聲功率譜密度比 ( \frac{E_b}{N_0} ) 的變化，參數為 ( R = 0.5, n = 512 )。

( \epsilon^(n, R) ) 定義為：[\epsilon^(n, R) = \min \left{ \epsilon : \exists (n, \lfloor 2^{nR} \rfloor, \epsilon) \text{ coding scheme} \right}]
( \frac{E_b}{N_0} ) 與SNR的關係：[\frac{E_b}{N_0} = \frac{\rho}{2R}]
正常近似：[\epsilon^*(n, R) \approx Q \left( \frac{C - R + (2n)^{-1} \log_2 n}{\sqrt{V/n}} \right)]
圖表顯示上下界隨 ( \frac{E_b}{N_0} ) 變化的趨勢，表明錯誤概率隨著能量增加而降低。





歸一化碼率（Normalized Rate）

定義：[R_{\text{norm}} = \frac{R}{R^*(n, \epsilon)}]

其中，( R ) 為實際編碼方案的碼率，( R^*(n, \epsilon) ) 為理論最大碼率（可通過正常近似計算）。
( R_{\text{norm}} ) 允許比較不同區塊長度和碼率的編碼方案性能，值越大表示編碼方案越接近理論極限。


計算方法：

固定碼的區塊長度 ( n ) 和碼率 ( R )，計算實現目標錯誤概率 ( \epsilon ) 所需的最小SNR ( \rho_{\min}(\epsilon) )。
使用 ( \rho_{\min}(\epsilon) ) 計算 ( R^*(n, \epsilon) )（如通過正常近似公式8.7）。
計算 ( R_{\text{norm}} )，評估編碼方案的相對性能。


圖表分析（圖8.2）：

圖8.2展示了不同編碼方案在bi-AWGN通道上的 ( R_{\text{norm}} )，參數為 ( \epsilon = 10^{-4} )。
根據區塊長度 ( n ) 分為三個區域：
大區塊長度（( n \geq 1000 )）：
最佳編碼方案：基於信念傳播解碼的多邊型LDPC碼和Turbo碼。
這些碼在長區塊下接近通道容量，性能優異。


中等區塊長度（( 400 \leq n \leq 1000 )）：
最佳編碼方案：極化碼（Polar Codes）結合大列表大小的連續消除解碼（Successive-Cancellation Decoding）及外層循環冗餘校驗（CRC）。
提供良好的性能-複雜度權衡。


短區塊長度（( n \leq 400 )）：
最佳編碼方案：短代數碼或基於尾咬卷積碼的線性區塊碼，使用近最大似然解碼（如有序統計解碼，OSD）。
高階有限域上的LDPC碼也表現出潛力。




這些見解已被3GPP標準化活動採用：
LDPC碼用於新無線（NR）eMBB數據通道。
極化碼用於NR eMBB控制通道。






總結與實際意義

理論意義：

本節通過bi-AWGN通道的分析，闡述了FEC的基礎限制，特別是在有限區塊長度下的性能權衡。
Shannon的通道容量理論提供了漸進性能的基準，但對於5G低延遲場景，需依賴非漸進分析工具。
正常近似和上下界提供了實際可計算的性能指標，幫助評估編碼方案的效率。


實際應用：

非漸進分析為5G編碼方案的設計和選擇提供了具體指導。
不同區塊長度下的最佳編碼方案選擇（LDPC、極化碼、短代數碼等）直接影響5G系統的性能。
這些理論工具可用於優化多天線衰落通道、先導輔助傳輸等場景（詳見第8.3節）。


未來研究方向：

改進非漸進界的計算效率，特別是在低SNR和低錯誤概率場景下。
開發更高效的短區塊長度編碼方案，以滿足5G URLLC的需求。
探索基於機器學習的編碼和解碼算法，以進一步接近理論極限。





