## Architecture
```
【 Uplink: UE TX (模擬傳送端)】

Input: MAC PDU (bytes)
        ↓
┌──────────────────────────────┐
│ CRC 附加 (crc_byte.c)         │
│ Output: bits + CRC           │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Physical-layer HARQ processing │
│ Output: encoded bits         │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Segmentation (nr_segmentation.c) │
│ Output: LDPC segments (bits) │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LDPC 編碼 (nr_dlsch_coding.c + CODING/) │
│ Output: encoded bits         │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Rate Matching + Interleaving │
│ Output: RM bits              │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Scrambling                   │
│ Output: RM bits              │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 調變 (Modulation: QPSK, 16QAM) │
│ Output: complex symbols (IQ) │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Resource Grid Mapping │
│ Output: complex symbols (IQ) │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Insert DMRS (UE uplink reference signal) │
│ Output: complex symbols (IQ) │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ DFT-s-OFDM IFFT + CP 加入 (ofdm_mod.c) │
│ Output: time-domain samples │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 模擬通道:  │
│ Input: TX samples            │
│ Output: RX samples           │
└────────────┬─────────────────┘
             ↓

                    【 Uplink: gNB RX (模擬接收端)】

┌──────────────────────────────┐
│ OFDM FFT + CP 去除 (ofdm_mod.c) │
│ Input: RX samples            │
│ Output: frequency-domain symbols │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 通道估計 / 等化(nr_ulsch_demodulation.c)│
│ Output: equalized symbols   │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LLR 計算 (nr_ulsch_llr_computation.c) │
│ Output: soft bits (LLRs)    │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Rate Dematching              │
│ Input: LLRs                  │
│ Output: code blocks          │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LDPC 解碼 (nr_ulsch_decoding.c) │
│ Output: decoded bits         │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ CRC 檢查 + Segment 合併       │
│ Output: MAC SDU (bytes)      │
└──────────────────────────────┘
```
| 子資料夾                           | 功能                                      | 重點說明                                              |
| ------------------------------ | --------------------------------------- | ------------------------------------------------- |
| `NR_TRANSPORT/`                | 5G NR Transport Channels（DLSCH、ULSCH 等） | LDPC 編碼、Rate Matching、HARQ                        |
| `NR_UE_TRANSPORT/`             | UE 對應的 Transport 功能                     | UE 上行的 LDPC、發射管理                                  |
| `NR_REFSIG/`                   | NR 的 reference signal 模組                | 包含 **DMRS, PTRS, PRACH, SSB** waveform 產生與插入      |
| `MODULATION/`                  | OFDM IFFT/FFT、調變、符號對映等                  |                   |
| `TOOLS/`                       | 通道估計、向量運算、FFT 工具、phase noise 等          |  |
| `INIT/`                        | Layer 1 變數初始化                           | 通常在 `phy_init_nr_ue()` 內呼叫                        |
| `CODING/`                      | LDPC、Polar 編碼與測試                        | 覆蓋 TS 38.212                                      |
| `defs.h`, `extern.h`, `vars.h` | 全域定義與變數引用                               | 模組間變數共用依賴這三個檔案架構                                  |

**openair1/PHY/NR_UE_TRANSPORT/** //NR UE transport channel procedures are here
| 檔案名稱             | 功能                                 |
| ---------------- | ---------------------------------- |
| `nr_dci.c`       | 處理 DCI 格式（如 format 1\_0/1\_1）與資訊打包 |
| `cic_filter_nr.c` | Cascaded Integrator-Comb and FIR filter 做抽樣，減少取樣率     | 
| `nr_dlsch_decoding.c` | UE接收下行資料後，ldpc解碼、HARQ更新、CRC驗證過程  | 
| `nr_dlsch_demodulation.c`  | UE接收下行資料後，解映射、通道補償、算LLR，接到解碼部分  | 
| `nr_initial_sync.c`                 | UE進行初始同步，找基地台   | 
| `nr_initial_sync_sl.c`              | SideLink模式下，UE進行初始同步             | 
| `nr_ulsch_decoding.c`        | gNB 解碼 PUSCH（LDPC 解碼與 CRC 驗證） | 
| `nr_ulsch_demodulation.c`    | 頻域通道估計、MMSE 等化與符號提取           | 
| `nr_prach.c`        | 產生 PRACH 前導碼與接收匹配過濾器 | 
| `nr_psbch_rx.c`        | UE接收、解碼（Physical Sidelink Broadcast Channel, PSBCH）訊號 | 
| `nr_psbch_tx.c` | UE發送（Physical Sidelink Broadcast Channel, PSBCH）訊號    |
| `nr_pbch.c` | MIB 封裝與 PBCH 導頻插入    | 
| `pss_nr.c`  | 偵測主同步信號 PSS（用於時間同步）  | 
| `sss_nr.c`  | 偵測次同步信號 SSS並解碼使用 | 
| `nr_ue_rf_helpers.c` | 配置射頻卡（RF Card）的頻率和增益。 | 
| `nr_ulsch_coding.c` | 實現ULSCH編碼過程，它包含了 CRC 、SEGEMENT、LDPC Encode和rate matching    | 
| `nr_ulsch_ue.c`             | 處理 UE 在ULSCH的所有相關工作，如編碼及調變 | 
| `nr_transport_proto_ue.h`        | UE實現PHY中與DLSCH、ULSCH、PUCCH）、PBCH、PRACH以及 PSBCH等相關的處理功能。                           | 
| `nr_transport_ue.h` | UE端和傳輸層相關的資料結構。用於在PHY 內部以及PHY與更高層 (MAC 層) 之間傳遞資料和配置資訊，HARQ 和 ULSCH/DLSCH相關的資訊。 | 
| `pucch_nr.c`       | 處理 UCI（如 SR, HARQ-ACK, CSI）                | 
| `pucch_nr.h`                  |   定義UE端和PUCCH相關的數據結構、函數，用於實現UCI的傳輸。              | 
| `srs_modulation_nr.c`                    | 實現UE端的 SRS（Sounding Reference Signal，探測參考信號)生成和處理功能 |
| `srs_modulation_nr.h`                    | 定義UE端和SRS相關的數據結構、函數   | 
