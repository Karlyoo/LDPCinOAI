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

# 5. Physical Layer 
## 5.1 Waveform, numerology and frame structure 
- DL waveform：CP-OFDM（Cyclic Prefix - OFDM）
- UL waveform：CP-OFDM,DFT-spread OFDM（DFT-S-OFDM）

| Feature     | OFDM                                                             | CP-OFDM (Cyclic Prefix OFDM)                                  |
| ----------- | ---------------------------------------------------------------- | ------------------------------------------------------------- |
| Full Name   | Orthogonal Frequency Division Multiplexing                       | Cyclic Prefix Orthogonal Frequency Division Multiplexing      |
| Structure   | Transmit time-domain symbols via IFFT                            | Adds a cyclic prefix (CP) before each OFDM symbol             |
| Purpose     | Improves spectral efficiency; handles frequency-selective fading | Resists multipath and ISI; preserves subcarrier orthogonality |
| Difference  | No prefix; sensitive to delay                                    | CP tolerates delay spread due to multipath                    |
| Symbol Time | T                                                                | $T + T_{cp}$ (slightly longer)                                |
| Application | Theoretical analysis, academic use                               | Used in practical systems (LTE, 5G NR DL/UL)                  |

**example**
Original OFDM Symbol:   [ a b c d e f g h ]
With Cyclic Prefix:     [ e f g h | a b c d e f g h ]

**Numerology**
- Numerology defines subcarrier spacing (SCS) and associated OFDM parameters.

| OFDM Parameter  | Impact                                                               |
| --------------- | -------------------------------------------------------------------- |
| Symbol duration | $T_{sym} \propto \frac{1}{\text{SCS}}$; larger SCS → shorter symbols |
| FFT size        | At fixed bandwidth, larger SCS → smaller FFT size                    |
| Slot length     | 14 symbols per slot; larger μ → shorter slot                         |
| CP length       | Related to symbol duration; larger SCS → shorter CP                  |
| TTI granularity | Larger μ → finer time granularity → faster system reaction           |


| μ | Subcarrier Spacing | Symbol Duration  | Slot Length         | Example Use         |
| - | ------------------ | ---------------- | ------------------- | ------------------- |
| 0 | 15 kHz             | Long (\~66.7 µs) | 1 ms                | eMBB, IoT           |
| 1 | 30 kHz             | Medium           | 0.5 ms              | Mid-band mainstream |
| 2 | 60 kHz             | Short            | 0.25 ms             | URLLC               |
| 3 | 120 kHz            | Very Short       | 0.125 ms            | mmWave              |
| 4 | 240 kHz            | Ultra Short      | -- (for PRACH only) | FR2 PRACH           |

- Each PRB (Physical Resource Block): 12 consecutive subcarriers
- Max 275 PRBs per carrier
- UE can configure multiple BWPs (Bandwidth Parts), but only one can be active at a time

**frame**

| Unit     | Duration                 | Description                               |
| -------- | ------------------------ | ----------------------------------------- |
| Frame    | 10 ms                    | Top-level unit                            |
| Subframe | 1 ms                     | 10 subframes per frame                    |
| Slot     | $\frac{1}{2^\mu}$ ms     | Number of slots per subframe depends on μ |
| Symbol   | ≈ $\frac{1}{\text{SCS}}$ | 14 symbols per slot (normal CP)           |

## 5.2 Downlink
- PDSCH（Physical Downlink Shared Channel）+ DMRS（Demodulation Reference Signal）
- DMRS (Demodulation Reference Signal)
  - Helps receiver (UE or gNB) estimate the channel for equalization and demodulation
  - Only reference signal in NR tightly coupled with data (PDSCH/PUSCH)
  - DMRS configuration depends on numerology (symbol spacing)
  - More DMRS symbols possible with shorter slots
| Task                       | Responsible Layer |
| -------------------------- | ----------------- |
| DMRS configuration         | RRC Layer         |
| Activation for PDSCH/PUSCH | MAC Layer         |
| Generation & Transmission  | PHY Layer         |

**Precoding and Precoder Matrix**
- Precoding applies spatial filtering to boost performance and reduce interference
- Done at transmitter side (gNB/UE)

| Function                 | Description                           |
| ------------------------ | ------------------------------------- |
| Spatial Multiplexing     | Supports multiple layers              |
| Beamforming              | Focus energy directionally            |
| Interference Suppression | Reduces inter-layer interference      |
| Reliability              | Improves SINR and spectral efficiency |


| Scenario     | Precoding Usage      |
| ------------ | -------------------- |
| gNB → PDSCH  | Yes (Beamforming)    |
| UE → PUSCH   | Yes, if UL MIMO used |
| SRS feedback | gNB selects PMI      |
| CSI-RS       | UE selects best PMI  |

- UE doesn’t need to know the precoding matrix to decode PDSCH
- PRBs can be grouped into PRG (Precoding Resource Group)

**Physical-layer processing for physical downlink shared channel**
- Transport Block CRC attachment
- Segmentation與 CBG CRC attachment
-  LDPC Encoding
- Hybrid-ARQ processing（HARQ）
- Rate Matching
- Scrambling
- Modulation
- Layer Mapping
- Mapping to assigned resources and antenna ports. 

### PDCCH
**Generation Process:**
```
MAC Layer → Configures parameters via RRC (e.g., CORESET, SearchSpace)
         ↓
MAC Layer → Determines DCI content (e.g., scheduling, MCS, HARQ, resource allocation)
         ↓
PHY Layer → Encodes DCI into PDCCH data (Polar encoding + QPSK modulation)
         ↓
PHY Layer → Transmits PDCCH over CORESET resources (according to Search Space configuration)

```
- Core Purpose of PDCCH:
  - PDCCH carries DCI (Downlink Control Information), which is encapsulated within PDCCH and transmitted to the UE.
  - Downlink scheduling:
    - Modulation and Coding Scheme (MCS)
    - Resource allocation (PRBs)
    - Slot/timing information
    - HARQ-related parameters (e.g., Redundancy Version)
  - Uplink scheduling grant:
    - MCS
    - Resource allocation
    - HARQ retransmission parameters

**Downlink Control Information**
- DCI (Downlink Control Information) is a control message sent by the gNB to inform the UE how to receive or transmit data.
- What Information is Included in DCI? DCI content depends on the format,

| Category                       | Purpose                                            | 
| ------------------------------ | -------------------------------------------------- | 
|  **Time domain**             | Slot index, start symbol, number of OFDM symbols   | 
|  **Frequency domain**        | PRB allocation (resource blocks)                   | 
|  **Modulation & Coding**     | MCS index, modulation order (QPSK/16QAM/etc.)      |                               
|  **HARQ Control**            | NDI flag, HARQ Process ID, RV (Redundancy Version) | 
|  **Precoding / Beamforming** | PMI, transmission rank, antenna port               | 
|  **Power Control**           | Transmit Power Control command (TPC)               | 
|  **UL/DL Scheduling**        | Downlink or uplink scheduling                      | 
|  **PTRS / SRS / CSI**       | Time/frequency density and activation              | 



**Example: DCI Format 1_0 (Uplink Grant)**

| Field                       | Function                      |
| --------------------------- | ----------------------------- |
| Frequency domain assignment | PRB allocation                |
| Time domain assignment      | OFDM symbols                  |
| Frequency hopping           | Enable/disable hopping        |
| MCS                         | Modulation and coding scheme  |
| NDI                         | New Data Indicator            |
| RV                          | Redundancy version (for HARQ) |
| HARQ ID                     | Identifies HARQ process       |
| TPC for PUSCH               | Uplink power control          |
| CSI request                 | Request CSI feedback          |

**PDCCH Monitoring and Search Space**
- The UE monitors PDCCH within configured Search Spaces on specific CORESETs.
- Each CORESET may correspond to multiple Search Spaces, each defining its monitoring occasions.

| Item                      | Description                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| **CORESET**               | Control Resource Set, defines RE regions where PDCCH can exist      |
| **Slot offset**           | Indicates which slots to search in                                  |
| **Symbols**               | Which OFDM symbols in a slot carry the PDCCH                        |
| **CCE Aggregation Level** | Number of CCEs (Control Channel Elements) forming a candidate PDCCH |
| **DCI Format**            | Expected DCI format by UE, e.g., Format 0\_1 / 1\_0                 |


| Element                   | Description                                                                    |
| ------------------------- | ------------------------------------------------------------------------------ |
| **CORESET**               | Composed of PRBs, lasting 1–3 OFDM symbols                                     |
| **REG**                   | Resource Element Group; the smallest unit composed of REs                      |
| **CCE**                   | Control Channel Element; consists of REGs and used to form PDCCH               |
| **CCE Aggregation Level** | Indicates how many CCEs are used to transmit control messages (1, 2, 4, 8, 16) |
| **Mapping**               | CCEs are mapped to REGs in Interleaved or Non-interleaved modes                |


### Synchronization signal and PBCH block
**SSB（Synchronization Signal Block，同步訊號區塊**
- The first physical signal block UE detects when powering on, used for synchronization and acquiring MIB.
- Composed of:
  - PSS (Primary Sync Signal): Coarse frequency and timing sync
  - SSS (Secondary Sync Signal): Cell ID, frame timing
  - PBCH (Physical Broadcast Channel): Carries MIB
 ```
┌──────────────────────────────┐
│ SSB (Synchronization Block)  │
├──────────┬──────────┬────────┤
│   PSS    │   SSS    │  PBCH  │
├──────────┴──────────┴────────┤
│  Sync, Cell ID, MIB decoding │
└──────────────────────────────┘

```
| Item                | Description                                                            |
| ------------------- | ---------------------------------------------------------------------- |
| Time Duration       | Typically 4 OFDM symbols (symbol length depends on μ)                  |
| Frequency Width     | 240 subcarriers (20 RBs)                                               |
| Transmission Period | Default 20ms (can be configured as 5ms, 10ms, 20ms, 40ms, 80ms, 160ms) |
| Max per Burst       | **Up to 64 beams (= 64 SSBs)**, varies by FR1/FR2                      |


- Functional Procedure
  - UE powers on → scans time-frequency space to detect SSB
  - Uses PSS + SSS for sync and Cell ID detection
  - Decodes PBCH → obtains MIB (includes how to access SIB1)
  - Uses pdcch-ConfigSIB1 in MIB to decode SIB1

- SSB is periodically sent and not scheduled by MAC (unlike PDSCH/PDCCH)

**PBCH**
- Used for broadcasting key system info (MIB) to all UEs
- First control channel UE receives before RRC or scheduling
- Functions:

| Function            | Description                                                             |
| ------------------- | ----------------------------------------------------------------------- |
| System Info         | Carries **MIB (Master Information Block)** — entry point to the network |
| Always-on           | PBCH is always present; UE only needs to detect SSB to receive it       |
| Cell Identification | Contains Cell ID, subcarrier spacing, and timing parameters             |
| Channel Setup       | UE needs MIB to configure SIB1 and PDCCH reception                      |


### Physical layer procedures 
**Link adaptation**
- Adjusts modulation and coding rate based on channel conditions (a.k.a. AMC).
- For a given TTI and MIMO codeword, all allocated RBs use the same MCS.

**CSI-RS**
CSI-RS (Channel State Information Reference Signal) is a downlink signal from gNB to let UE estimate channel conditions and generate CSI (CQI, PMI, RI, CRI) for reporting.

| Function              | Description                                                         |
| --------------------- | ------------------------------------------------------------------- |
| Channel Estimation    | UE estimates channel matrix H from CSI-RS                           |
| Beam Selection        | UE uses CSI-RS to pick optimal beam direction (CRI)                 |
| CSI Basis             | Used to compute CQI, PMI, RI, etc. for reporting                    |
| Frequency Selectivity | Supports frequency-domain channel quality assessment for better AMC |


| Type       | Sender | Receiver | Main Purpose                           |
| ---------- | ------ | -------- | -------------------------------------- |
| **DMRS**   | gNB/UE | Receiver | Demodulation & channel estimation      |
| **SRS**    | UE     | gNB      | Uplink channel estimation              |
| **CSI-RS** | gNB    | UE       | **Channel quality measurement/report** |

- Procedure
  - gNB configures CSI-RS via RRC
  - gNB periodically transmits CSI-RS
  - UE estimates channel and generates CSI feedback 
  - gNB uses CSI for:
    - Spectrum allocation
    - MCS selection
    - Beamforming
    - Resource scheduling

| CSI Type                             | Description                                |
| ------------------------------------ | ------------------------------------------ |
| **CQI** (Channel Quality Indicator)  | UE's max supportable MCS                   |
| **PMI** (Precoding Matrix Indicator) | Best precoding/beam direction              |
| **RI** (Rank Indicator)              | Best spatial stream count                  |
| **CSI-RSRP/RSRQ/SINR**               | Assist beam/cell selection                 |
| **CRI** (CSI Resource Indicator)     | Best CSI-RS resource index (for beam mgmt) |


| Function               | Description                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| **UE measures CSI-RS** | UE derives SNR, RI, CQI, PMI from CSI-RS                                |
| **UE reports CSI**     | UE reports the CSI to gNB                                               |
| **gNB selects MCS**    | gNB determines modulation/coding (e.g., QPSK + 1/3 rate or 64QAM + 5/6) |


**Cell search**
- UE synchronizes time and frequency and detects Cell ID using:
  - PSS（Primary Synchronization Signal）
  - SSS（Secondary Synchronization Signal）
  - PBCH DMRS
 
| Function       | Signal    | Description                                |
| -------------- | --------- | ------------------------------------------ |
| Frequency Sync | PSS       | Detect subcarrier spacing                  |
| Timing Sync    | SSS       | Determine timing and frame structure       |
| Cell ID        | PSS + SSS | Combine to form Physical Cell ID (0–1007)  |
| Beam ID        | SSB index | Indicates beam direction                   |
| Info Decoding  | PBCH      | Contains MIB                               |
| Channel Est.   | PBCH DMRS | Used to estimate channel for PBCH decoding |


**HARQ**
- Supports Asynchronous Incremental Redundancy HARQ:
  - Asynchronous: No fixed timing between HARQ rounds. gNB schedules retransmissions   flexibly.
  - Incremental Redundancy: Retransmits different RVs to accumulate redundancy and improve decoding.
- HARQ-ACK:
  - UE must report ACK/NACK to gNB:
  - Timing specified via DCI
  - RRC configuration
  - PUCCH feedback
  - For SPS mode, HARQ-ACK may bypass PDCCH
- HARQ-ACK Transmission
  - Dynamic Codebook
  - One-shot response
- Timing Configuration
  - Semi-static via RRC
  - Dynamic via DCI


 
**PUSCH Scheduling**
- PUSCH can be scheduled in two ways:  
  -  Via DCI on PDCCH
  -  Via RRC-configured semi-static grant
  -  For semi-static grant, two operation modes exist:
    -  The first PUSCH is triggered by DCI, with subsequent transmissions following RRC and DCI configuration
    -  Or the PUSCH is triggered by data arrival in the UE's transmit buffer and follows the RRC configuration
 
**Physical uplink control channel**
- PUCCH has five formats, depending on the duration and UCI payload size:

| Format    | Length                 | UCI Payload              | UE Multiplexing                                   | Description                                    |
| --------- | ---------------------- | ------------------------ | ------------------------------------------------- | ---------------------------------------------- |
| Format #0 | 1 or 2 symbols (short) | Small (up to 2 bits)     | Up to 6 UEs (with 1-bit payload)                  | For very short control feedback                |
| Format #1 | 4–14 symbols (long)    | Small (up to 2 bits)     | Up to 84 UEs (no hopping) / 36 UEs (with hopping) | Provides better reliability and coverage       |
| Format #2 | 1 or 2 symbols (short) | Large (more than 2 bits) | **No** multiplexing supported                     | For short-duration but large control messages  |
| Format #3 | 4–14 symbols (long)    | Large (more than 2 bits) | **No** multiplexing supported                     | For high-capacity control signaling            |
| Format #4 | 4–14 symbols (long)    | Moderate                 | Multiplexing up to 4 UEs                          | Balanced between payload size and multiplexing |

**Difference Between Short and Long PUCCH**
- Short PUCCH (≤ 2 bits):Uses sequence selection
- Short PUCCH (> 2 bits):Uses frequency multiplexing of UCI and DMRS
- Long PUCCH:
  - Uses time multiplexing of UCI and DMRS
  - May support repetition across multiple slots or sub-slots (configured via DCI or semi-static RRC)
 
**PUCCH Priority Index**
- A UE can have up to two PUCCH configurations in one group:
  - One for priority index 0 (low)
  - One for priority index 1 (high)
 
**UCI Multiplexing Conditions**
- UCI Multiplexing in PUCCH:
  - When multiple UCIs are sent simultaneously with the same priority (high/low)
  - When HARQ-ACK (low priority) and UCI (high priority) coincide in time with matching priorities
 
- UCI Multiplexing in PUSCH:
  - When PUSCH and UCI coincide in time due to:
    - UL-SCH transport block transmission
    - A-CSI triggered transmission without UL-SCH TB
  - HARQ-ACK and UCI of different priorities may still be multiplexed when coinciding in time

**UCI May Contain**
- CSI (Channel State Information)
- ACK/NACK
- Scheduling Request


**Channel Coding for Uplink Control Information**
| UCI size including CRC | Channel code type |
| ---------------------- | ----------------- |
| 1 bit                  | Repetition code   |
| 2 bits                 | Simplex code      |
| 3–11 bits              | Reed Muller code  |
| >11 bits               | Polar code        |

**Random access**
- Four supported PRACH sequence lengths:

| Length | Supported Subcarrier Spacings | Restrictions                                                  |
| ------ | ----------------------------- | ------------------------------------------------------------- |
| 839    | 1.25 kHz, 5 kHz               | Restricted to Type A/B sets, licensed spectrum only           |
| 139    | 15–960 kHz                    | Unrestricted sets, for both licensed and shared spectrum      |
| 571    | 30 kHz, 120 kHz               | Unrestricted sets only                                        |
| 1151   | 15–480 kHz                    | Unrestricted sets, for FR1: shared only; FR2: licensed/shared |

- PRACH Configuration:
  - Multiple PRACH preamble formats are defined, based on:
    - Number of OFDM symbols
    - Cyclic prefix (CP) and guard time
  - PRACH configuration is provided to the UE via system information (e.g., SIB1)

- UE Behavior:
  - The UE determines PRACH transmit power for retransmission based on:
    - Estimated path loss
    - Power ramping counter
  - System Information provides rules for:
    - Associating SSB with RACH resources
    - RSRP threshold for SSB selection, configurable by the network


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

