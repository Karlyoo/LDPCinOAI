# UE Tx and RX
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
│ LDPC 編碼 (nr_dlsch_coding.c ) │
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
| `defs.h`, `extern.h`, `vars.h` | 全域定義與變數引用                               | 模組間變數共用依賴這三個檔案架構                    |

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

## 對應3GPP規範及OAI函式

| 功能                       | 3GPP 規格章節                | OAI 對應函式                                                        |
| ------------------------ | ------------------------ | --------------------------------------------------------------- |
| CRC attachment            |  38.212 §5.1             |    crc24a()、crc24b().....                                            |
| Segmentation              |  38.212 §5.2             |    nr_segmentation()                                           |
| LDPC encoding             |  38.212 §5.2             |    LDPCencoder()                                             |
| Scrambling               | 38.211 §7.3.1.1          | `nr_pdsch_unscrambling()`                                       |
| Modulation               | 38.211 §7.3.2            | `nr_dlsch_llr_level()`、`nr_rx_pdsch()`                          |
| Layer Mapping            | 38.211 §7.3.1.2          | 內含於 `nr_rx_pdsch()` 處理中                                         |
| Resource Mapping         | 38.211 §7.3.3            | `nr_dlsch_extract_rbs()`、`nr_dlsch_channel_compensation()`      |
| Channel Estimation       | 38.211 §6.4.1.1.2 (DMRS) | `nr_pbch_channel_estimation()`、`dlsch_channel_estimation()` |
| LLR Calculate            | 38.212 attachment A      | `nr_dlsch_qpsk_llr()`, `nr_dlsch_16qam_llr()` ...                |

**openair1/PHY/NR_UE_TRANSPORT/nr_ulsch_ue.c**
實現 5G NR UE 的上行共享通道 (ULSCH) 的PHY傳輸流程
- 資料準備：
從 MAC 層接收傳輸塊，提取 PUSCH 配置（如資源分配、DMRS/PTRS 配置、調製方式等）。
- encoding與scrambling：
  CRC 、Segmentation、LDPC encoding、rate matching（nr_ulsch_encoding）。使用偽隨機序列打亂（nr_pusch_codeword_scrambling）。
  ```
  void nr_pusch_codeword_scrambling(uint8_t *in, uint32_t size, uint32_t Nid, uint32_t n_RNTI, bool uci_on_pusch, uint32_t* out)
  void nr_pusch_codeword_scrambling_uci(uint8_t *in, uint32_t size, uint32_t Nid, uint32_t n_RNTI, uint32_t* out)
  *上為scrambling函式，如果 uci_on_pusch is true（即 PUSCH 有 UCI，Uplink Control Information），則使用 nr_pusch_codeword_scrambling_uci；否則使用通用函數 nr_codeword_scrambling。而ldpc編碼部分則套用nr_ulsch_encoding。
  ```
- modulation與layer mapping：
  將加擾後的資料調製為複數符號（nr_modulation）。分配到多個transport layer（nr_ue_layer_mapping）。
- 轉換預編碼（可選）：
  若啟用 DFT-s-OFDM，對資料進行 DFT 轉換（nr_dft），並使用低 PAPR DMRS 序列。
- 資源映射：
  將資料、DMRS 和 PTRS 映射到 PUSCH 資源網格（map_symbols, map_current_symbol）。處理 DMRS 類型（Type 1 或 Type 2）、PTRS 位置和 DC 載波的特殊情況。
- precoding與天線映射：
  應用precoding matrix，將層資料映射到天線端（nr_layer_precoder）。輸出到頻域緩衝區 txdataF。
  
  ---
  以上函式定義包含在modulation.c程式內
- OFDM 調製：
  應用頻域旋轉，執行 IFFT 和循環前綴添加，生成時域信號（nr_ue_pusch_common_procedures）。輸出到 txdata。
## LDPC Encoding
[ldpc_encoding](https://github.com/Karlyoo/LDPCinOAI/blob/main/ldpc_encodeanddecode.md#ldpc-encoder-code-in-oai-5gnr)
## Modulation
[modulation](https://github.com/Karlyoo/LDPCinOAI/blob/main/modulation%26demodulation.md#modulation)
