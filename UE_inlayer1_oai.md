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
【 Downlink: UE RX (模擬接收端)】

┌──────────────────────────────┐
│ OFDM FFT + CP 去除 (ofdm_mod.c) │
│ Input: RX samples            │
│ Output: frequency-domain symbols │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 同步 + 識別 (initial_sync.c) │
│ 識別 SS/PBCH/DMRS             │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 通道估計  (nr_dlsch_channel_estimation.c) │
│ 使用下行 DMRS                 │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ PDCCH 解碼 (控制訊息解碼)     │
│ 解 DCI → 知道 PDSCH 位置       │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ PDSCH 解碼                   │
│ Demodulation + LLR 計算     │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Rate Dematching              │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LDPC 解碼                    │
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

## LDPC Encoding
---
[Read LDPC Encoder Code in OAI](https://hackmd.io/TOM6je4tQ9mjBCRt6VAyYQ#LDPC-Encoder-Code-in-OAI-5GNR)

## Modulation
**openair1/PHY/MODULATION** 
| 檔案名稱             | 功能                                 |
| ---------------- | ---------------------------------- |
| `beamforming.c`       | 在eNB端實現波束賦形 (Beamforming) 的功能 |
| `compute_bf_weights.c` | 目前程式裡沒有資料，可能是嘗試從 (ULCSI) 來估計 (DLCSI)來計算波束賦形的權重 (beam_weights)     | 
| `modulation_UE.h` | 定義UE在實體層接收端調變/解調變的前端處理的核心函數介面  | 
| `modulation_eNB.h`  | 定義 基地台在實體層中與調變和波束賦形相關的關鍵函數介面。 | 
| `nr_modulation.c`                 | 用於 調變 (Modulation)、層映射 (Layer Mapping)、DFT (離散傅立葉轉換) 和 預編碼 (Precoding)    | 
| `ofdm_mod.c`              | 5G NR 系統的 OFDM 調變流程，用於下行通道             | 
| `slot_fep_nr.c`        | 針對Downlink和Uplink的 OFDM 信號解調 | 
### nr_modulation.c
- 調變 (nr_modulation)：將編碼後的bitstream映射到 QPSK、16-QAM、64-QAM 或 256-QAM 的複數符號。
  - 輸入參數:
    - in：輸入位元流（uint32_t 格式）。
    - length：位元數量。
    - mod_order：調變階數（2 表示 QPSK，4 表示 16-QAM，6 表示 64-QAM，8 表示 256-QAM）。
    - out：輸出複數符號（int16_t 格式，表示實部和虛部的 Q15 固定點數）。
  - 工作原理:
    將輸入的位元流分組，查詢定義的調變表(3GPP TS 38.211) ，如 nr_qpsk_mod_table、nr_16qam_mod_table 等，並生成對應的複數符號。
  
```
  #if defined(__SSE2__)
case 2:
  nr_mod_table128 = (simde__m128i *)nr_qpsk_byte_mod_table;
  out128 = (simde__m128i *)out;
  for (i = 0; i < length / 8; i++)
    out128[i] = nr_mod_table128[in_bytes[i]];
  i = i * 8 / 2;
  nr_mod_table32 = (int32_t *)nr_qpsk_mod_table;
  while (i < length / 2) {
    const int idx = ((in_bytes[(i * 2) / 8] >> ((i * 2) & 0x7)) & mask);
    out32[i] = nr_mod_table32[idx];
    i++;
  }
  return;
#else
case 2:
  nr_mod_table32 = (int32_t *)nr_qpsk_mod_table;
  for (i = 0; i < length / mod_order; i++) {
    const int idx = ((in[i * 2 / 32] >> ((i * 2) & 0x1f)) & mask);
    out32[i] = nr_mod_table32[idx];
  }
  return;
#endif

*QPSK 每 2 位元映射到一個複數符號（例如 {1+j, 1-j, -1+j, -1-j}），並使用SSE2加速，每次迴圈處理 8 位元，生成 4 個複數符號，存入 out
```

```
case 6:
  if (length > (3 * 64))
    for (i = 0; i < length - 3 * 64; i += 3 * 64) {
      uint64_t x = *in64++;
      uint64_t x1 = x & 0xfff;
      *out64++ = nr_64qam_mod_table[x1];
      x1 = (x >> 12) & 0xfff;
      *out64++ = nr_64qam_mod_table[x1];
      // ... (處理多個 12 位元組)
    }
  while (i + 24 <= length) {
    uint32_t xx = 0;
    memcpy(&xx, in_bytes + i / 8, 3);
    uint64_t x1 = xx & 0xfff;
    *out64++ = nr_64qam_mod_table[x1];
    x1 = (xx >> 12) & 0xfff;
    *out64++ = nr_64qam_mod_table[x1];
    i += 24;
  }
  if (i != length) {
    uint32_t xx = 0;
    memcpy(&xx, in_bytes + i / 8, 2);
    uint64_t x1 = xx & 0xfff;
    *out64++ = nr_64qam_mod_table[x1];
  }
  return;
*以64-QAM為例，每 6 位元映射到一個複數符號，每次處理 64 位元（uint64_t），包含多個 6 位元組，生成多個 64-QAM 符號。
使用位移運算（>>）和&mask 提取 12 位元（2 個 6 位元符號），查詢 nr_64qam_mod_table，最後每個 64-QAM 符號由兩個 int16_t 表示，存入 out。
```

- 層映射 (nr_layer_mapping, nr_ue_layer_mapping)：將調變後的符號分配到多個傳輸層 (layers)，支援 MIMO (多輸入多輸出)。
  - 參數：
       - nbCodes：編碼單元數（Codewords），通常為 1 或 2，代表獨立的data flow。
       - encoded_len：每個編碼單元的符號長度。
       - mod_symbs：調變後的複數符號，格式為 c16_t（16 位元整數表示實部和虛部，Q15 固定點數），為 nbCodes 個編碼單元，每個包含 encoded_len 個符號。
       - n_layers：傳輸層數（1 到 4），對應 MIMO 配置。
       - layerSz：每層的符號數量。
       - n_symbs：總符號數量，表示要處理的資料量。
       - tx_layers：輸出層資料，格式為 c16_t 陣列，每層儲存 layerSz 個符號
   - 以4個layer為例，資料位元每4個各自到不同的4層，從這些層最終會映射到不同的天線端口，實現 MIMO 傳輸。而依照裝置有分:
     - 基站 (gNB)：使用 nr_layer_mapping 處理下行傳輸（如 PDSCH），支援 1 到 4 層的 MIMO，並利用 SIMD 指令（AVX512、AVX2）優化，高資料速率和多天線配置。
     - UE：使用 nr_ue_layer_mapping 處理上行傳輸（如 PUSCH），支援簡單的層映射和幅度縮放。通常層數較少（1 或 2 層），因為 UE 的天線數量和功率受限。
- DFT (nr_dft)：對上行資料執行 DFT 轉換（適用於 DFT-s-OFDM，SC-FDMA）。
  - 負責將層映射後的時域調變符號（例如 QPSK 或 16-QAM 符號）轉換為頻域資料。生成類似單載波的信號，降低 PAPR。為子載波映射提供頻域資料，確保 DFT-s-OFDM 波形的正確生成。
  - 參數：
     - d：輸入時域資料，格式為 int32_t（通常表示 c16_t，即 16 位元整數表示實部和虛部的複數符號，Q15 固定點數格式）。
           這些是層映射後的調變符號（例如 QPSK 或 16-QAM），來自 nr_ue_layer_mapping 的 tx_layers[l]。
     - z：輸出頻域資料，格式同樣為 int32_t（c16_t），儲存 DFT 轉換後的結果。
     - Msc_PUSCH：DFT 大小，等於分配的子載波數量，通常為資源塊數 (N_RB) × 12（每個資源塊包含 12 個子載波）。
- 符號旋轉 (perform_symbol_rotation, init_symbol_rotation, init_timeshift_rotation)：校正頻域或時域的相位偏移。
  - perform_symbol_rotation()：
    為每個 OFDM symbol 計算一個 複數旋轉因子（ phase compensation）=e^{j2πf0t}，對應到 TX/RX 的頻率偏移。
    存成整數格式的複數到 symbol_rotation[],是第 l 個 OFDM symbol 的旋轉補償值，會在 TX 或 RX 處乘進每個 symbol。
  - init_symbol_rotation():
    對 DL / UL 都各呼叫一次 perform_symbol_rotation()，
    來計算整個 frame（或 subframe）裡每個 symbol 的旋轉值。
  - init_timeshift_rotation():
    接收時，FFT 的起點與實際 symbol 開始不完全對齊，會造成時域偏移對頻域符號造成的相位偏移。
    在此函數做補償。
- 預編碼 (nr_layer_precoder, nr_layer_precoder_cm, nr_layer_precoder_simd)：應用預編碼矩陣以適應多天線傳輸。
  - Layer → Antenna 的變換（linear precoding），依據 PMI (Precoding Matrix Indicator) 所對應的預編碼矩陣，把每個 Layer 的資料加權相加後分配到每根天線。
  - c16_t nr_layer_precoder():
    使用簡單字符矩陣（如 '1', '-1', 'j', '-j'）對每個 Layer 資料進行加權合成，用於簡化的 precoding，如 2x1 MIMO 對 layer 0 用 '1'，對 layer 1 用 'j'
  - c16_t nr_layer_precoder_cm():
    根據完整的複數權重（from PMI PDU）進行乘法累加，每個 layer 對應一組 weight (Re + jIm)對每個 layer 的符號與其權重做複數乘法，最後累加所有 layer 結果
  - void nr_layer_precoder_simd():
    使用 SIMD 向量化指令加速 Layer-to-Antenna 的 Precoding
