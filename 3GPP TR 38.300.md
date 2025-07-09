# NR and NG-RAN Overall description
---
## CH4 architecture
### 4.3 Network Interfaces
![image](https://github.com/user-attachments/assets/0cfb4e40-7f27-4e10-aa4e-41983730cc2d)

- Xn:gNB連接至gNB介面
- NG:5GC(5G Core network)連接至gNB介面
- AMF (Access and Mobility Management Function) by means of the NG-C(NG-Control plane)interface  
- UPF (User Plane Function) by means of the NG-U (NG-User plane)interface
![image](https://github.com/user-attachments/assets/ec3895f8-4e88-4036-a0ef-e6631e66ac3a)
![image](https://github.com/user-attachments/assets/311ce358-6650-4b32-903d-78f69c63819d)

### 4.4 Radio Protocol Architecture
![image](https://github.com/user-attachments/assets/7b71cbfd-e8ca-4ee1-b0f3-1a1342b615f4)

- PHY：physical signal modulation、encoding/decoding、mapping、transmit/receive。
- MAC：HARQ、邏輯通道到傳輸通道的封裝、排程回報等。
- RLC: 數據分段、重組、ARQ（在確認模式下）
  - 以上為DU
- PDCP:數據壓縮、加解密、完整性保護、數據重傳
- SDAP：負責 QoS 流映射與管理。
  - 以上為CU
- CU與DU會以F1連接
  
| 子介面                   | 功能                   |
| ------------------------ | ------------------------- |
| **F1-C** (Control Plane) | 控制信令，例如 UE 管理、資源配置、狀態控制等。 |
| **F1-U** (User Plane)    | 實際用戶資料的傳輸。使用 GTP-U 協定。    |

## CH5 Physical Layer
### 5.2 、5.3 Physical Channels and Signals(uplink and downlink )

Downlink Physical Channels：
|通道	|功能|
|-----|--------|
|PDSCH|	Physical Downlink Shared Channel → 傳送使用者數據|
|PDCCH|	Physical Downlink Control Channel → 傳送控制訊息（排程資訊）|
|PBCH	|Physical Broadcast Channel → 傳送系統資訊（SIB）|

Uplink Physical Channels：
|通道	|功能|
|-----|--------|
|PUSCH|	Physical Uplink Shared Channel → UE 上傳使用者數據|
|PUCCH	|Physical Uplink Control Channel → 上傳控制資訊（例如 HARQ 回報）|
|PRACH	|Physical Random Access Channel → 隨機接入程序（初始連線）|

Signals（not channel）：
|訊號	|功能|
|-----|--------|
|SSB	|Synchronization Signal Block（包括 PSS/SSS/PBCH） → UE 同步並識別|
|DMRS	|Demodulation Reference Signal → 解調參考信號，幫助通道估測|
|CSI-RS|	Channel State Information RS → 用來取得通道品質（MIMO feedback）|
|PTRS	|Phase Tracking RS → 協助相位追蹤（高階調變需用）|
```
The downlink physical-layer processing of transport channels consists of the following steps:
- Transport block CRC attachment;
- Code block segmentation and code block CRC attachment;
- Channel coding: LDPC coding;
- Physical-layer hybrid-ARQ processing;
- Rate matching;
- Scrambling;
- Modulation: QPSK, 16QAM, 64QAM and 256QAM;
- Layer mapping;
- Mapping to assigned resources and antenna ports.
```
```
The uplink physical-layer processing of transport channels consists of the following steps:
- Transport Block CRC attachment;
- Code block segmentation and Code Block CRC attachment;
- Channel coding: LDPC coding;
- Physical-layer hybrid-ARQ processing;
- Rate matching;
- Scrambling;
- Modulation: π/2 BPSK (with transform precoding only), QPSK, 16QAM, 64QAM and 256QAM;
- Layer mapping, ** transform precoding (enabled/disabled by configuration), and pre-coding; ** 
- Mapping to assigned resources and antenna ports.
```
### 5.5 Transport Channels
| Downlink Transport Channels | Uplink Transport Channels | Sidelink Transport Channels(UE-UE) |
|---------------------------|-----------------------------|------------------------|
| BCH：固定格式，廣播傳輸<br>DL-SCH：支援 HARQ、AMC、beamforming、DRX<br> PCH：用於 Paging，支援 DRX，動態使用資源|UL-SCH：支援 HARQ、AMC、beamforming<br>RACH：PRACH 隨機接入用，碰撞風險高|SL-BCH：固定格式廣播<br>SL-SCH：支援 unicast、groupcast、broadcast，支援 HARQ|
### 5.6 Shared Spectrum Access
- 使用 LBT（Listen Before Talk）感測裝置是否空閒
- UE 若偵測到連續 LBT 失敗，須回報 MAC CE 或發 SR
- 可切換 UL BWP 或觸發 RACH
- CAPC（Channel Access Priority Class）
  - 定義 MAC CE 與 RLC bearer 的優先順序
  - 依據 5QI 對應不同 CAPC（1~4），數字越小優先越高

