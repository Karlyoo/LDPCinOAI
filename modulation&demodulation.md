## Modulation
**openair1/PHY/MODULATION** 
| 檔案名稱             | 功能                                 |
| ---------------- | ---------------------------------- |
| `beamforming.c`       | 在eNB端實現波束賦形 (Beamforming) 的功能 |
| `compute_bf_weights.c` | 目前程式裡沒有資料，可能是嘗試從 (ULCSI) 來估計 (DLCSI)來計算波束賦形的權重 (beam_weights)     | 
| `modulation_UE.h` | 定義UE在實體層接收端調變/解調變的前端處理的核心函數介面  | 
| `modulation_eNB.h`  | 定義 基地台在實體層中與調變和波束賦形相關的關鍵函數介面。 | 
| `nr_modulation.c`                 | 用於 調變 (Modulation)、層映射 (Layer Mapping)、DFT (離散傅立葉轉換) 和 預編碼 (Precoding)    | 
| `modulation_common.h` | NR UE 中使用的共通調變常數 |
| `modulation_extern.h` |  調變模組中使用的 extern 函式與變數宣告 |

##  OFDM 與前端處理模組

| 檔案名稱 | 功能分類 | 描述 |
|----------|----------|------|
| `ofdm_mod.c` | OFDM modulation |  FFT/IFFT、CP insert（NR/LTE 通用） |
| `slot_fep.c` | Slot FEP | LTE slot 前端處理（接收端用，做 FFT 等） |
| `slot_fep_mbsfn.c` | Slot FEP | MBSFN 專用 slot 前處理 |
| `slot_fep_nr.c` | Slot FEP (NR) | 5G NR slot 的接收前處理（FFT、SC-FDMA 處理等） |
| `slot_fep_ul.c` | Slot FEP (UL) | 上行 LTE slot 處理邏輯 |

```
  A[Encoded Codewords] --> B[nr_modulation.c (QAM Mapping)]
  B --> C[Resource Mapping]
  C --> D[OFDM IFFT + CP insert (ofdm_mod.c)]
  D --> E[Slot packing (slot_fep_nr.c)]
  E --> F[ RF or channel simulator]
```

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

- Layer mapping (nr_layer_mapping, nr_ue_layer_mapping)：將調變後的符號分配到多個傳輸層 (layers)，支援 MIMO (多輸入多輸出)。
  - 參數：
       - nbCodes：編碼單元數（Codewords），通常為 1 或 2，代表獨立的data flow。
       - encoded_len：每個編碼單元的符號長度。
       - mod_symbs：調變後的複數符號，格式為 c16_t（16 位元整數表示實部和虛部，Q15 固定點數），為 nbCodes 個編碼單元，每個包含 encoded_len 個符號。
       - n_layers：傳輸層數（1 到 4），對應 MIMO 配置。
       - layerSz：每層的符號數量。
       - n_symbs：總符號數量，表示要處理的資料量。
       - tx_layers：輸出層資料，格式為 c16_t 陣列，每層儲存 layerSz 個符號
   - 以4個layer為例，資料位元每4個各自到不同的4層，從這些層最終會映射到不同的天線，實現 MIMO 傳輸。而依照裝置有分:
     - 基站 (gNB)：使用 nr_layer_mapping 處理下行傳輸（如 PDSCH），支援 1 到 4 層的 MIMO，並利用 SIMD 指令（AVX512、AVX2）優化，高資料速率和多天線分配。
     - UE：使用 nr_ue_layer_mapping 處理上行傳輸（如 PUSCH），支援簡單的映射。通常層數較少（1 或 2 層），因為 UE 的天線數量和功率受限。
- DFT (nr_dft)：對上行資料執行 DFT 轉換（適用於 DFT-s-OFDM，SC-FDMA）。
  - 負責將層映射後的時域調變符號（例如 QPSK 或 16-QAM 符號）轉換為頻域資料。生成類似單載波的信號，降低 PAPR。為子載波映射提供頻域資料，確保 DFT-s-OFDM 波形的正確生成。
  - 參數：
     - d：輸入時域資料，格式為 int32_t（通常表示 c16_t，即 16 位元整數表示實部和虛部的複數符號，Q15 固定點數格式）。
           這些是層映射後的調變符號（例如 QPSK 或 16-QAM），來自 nr_ue_layer_mapping 的 tx_layers[l]。
     - z：輸出頻域資料，格式同樣為 int32_t（c16_t），儲存 DFT 轉換後的結果。
     - Msc_PUSCH：DFT 大小，等於分配的子載波數量，通常為N_RB × 12（每個RB包含 12 個子載波）。
- 符號旋轉 (perform_symbol_rotation, init_symbol_rotation, init_timeshift_rotation)：校正頻域或時域的相位偏移。
  - perform_symbol_rotation()：
    為每個 OFDM symbol 計算一個 複數旋轉因子（ phase compensation）=e^{j2πf0t}，對應到 TX/RX 的頻率偏移。
    存成整數格式的複數到 symbol_rotation[],是第 l 個 OFDM symbol 的旋轉補償值，會在 TX 或 RX 處乘進每個 symbol。
  - init_symbol_rotation():
    對 DL / UL 都各呼叫一次 perform_symbol_rotation()，
    來計算整個 frame（或 subframe）裡每個 symbol 的旋轉值。
  - init_timeshift_rotation():
    接收時，FFT 的起點與實際 symbol 開始不完全對齊，會造成時域偏移對頻域符號造成的相位偏移。
- 預編碼 (nr_layer_precoder, nr_layer_precoder_cm, nr_layer_precoder_simd)：應用預編碼矩陣以適應多天線傳輸。
  - Layer → Antenna 的變換（linear precoding），依據 PMI (Precoding Matrix Indicator) 所對應的預編碼矩陣，把每個 Layer 的資料加權相加後分配到每根天線。
  - c16_t nr_layer_precoder():
    使用簡單字符矩陣（如 '1', '-1', 'j', '-j'）對每個 Layer 資料進行加權合成，用於簡化的 precoding，如 2x1 MIMO 對 layer 0 用 '1'，對 layer 1 用 'j'
  - c16_t nr_layer_precoder_cm():
    根據完整的複數權重（from PMI PDU）進行乘法累加，每個 layer 對應一組 weight (Re + jIm)對每個 layer 的符號與其權重做複數乘法，最後累加所有 layer 結果
  - void nr_layer_precoder_simd():
    使用 SIMD 向量化指令加速 Layer-to-Antenna 的 Precoding

## nr_dlsch_demodulation.c  
```
nr_rx_pdsch()
│
├─ nr_dlsch_channel_level()           ← 只在整個 PDSCH 首個 symbol 時執行，用來計算每個 layer+antenna 的平均通道強度，用於後續 scaling 與合併。
│
├─ nr_dlsch_channel_compensation()    ← Equalization 核心：頻域符號乘上通道共軛值 + magnitude 計算 (|H|²) 並分為 QAM scaling →
|                                        output 是 rxdataF_comp[]：補償後的頻域資料，供後續處理使用。
│
├─ nr_dlsch_detection_mrc()           ← 若使用 MRC，這邊進行多天線合併（非 spatial multiplexing）
│
├─ modulation LLR 計算：根據 modulation方式 呼叫
│   ├─ nr_dlsch_qpsk_llr()
│   ├─ nr_dlsch_qam16_llr()
│   └─ nr_dlsch_qam64_llr()
│    (根據調變類型（QPSK、16QAM、64QAM、256QAM）計算出軟決 LLR 給 Transport layer buffer 使用。)
└─ LLR 輸出 → softbuffer → LDPC 解碼

```

| 階段                   | 輸入資料                                 | 輸出資料                                 |
| -------------------- | ------------------------------------ | ------------------------------------ |
| 提取 RBs & 通道估計        | `rxdataF`, `dl_ch_estimates`         | `rxdataF_ext`, `dl_ch_estimates_ext` |
| Channel scaling      | `dl_ch_estimates_ext`                | scale 過後的 channel estimate           |
| Channel compensation | `rxdataF_ext`, `dl_ch_estimates_ext` | `rxdataF_comp`, `dl_ch_mag*`         |
| MRC/MMSE             | `rxdataF_comp`, `dl_ch_mag*`, `rho`  | 解調後流、MMSE 參數                         |
| LLR 計算               | `rxdataF_comp`、`dl_ch_mag*`          | 每 symbol LLR buffer (`layer_llr`)    |
| Layer mapping &輸出    | `layer_llr`, TB config               | `llr[CW0]`, `llr[CW1]`               |

```
NR_DL_UE_HARQ_t *dlsch0_harq, *dlsch1_harq;
dlsch0_harq = &ue->dl_harq_processes[0][harq_pid];
```
- 根據 HARQ PID 判斷目前哪個 TB（Transport Block）活躍：
  - 如果兩個 HARQ 都 active，會設 codeword_TB0/1
  - 否則就只處理一個 TB
```
nr_dlsch_extract_rbs()
```
- 這裡會從接收的 rxdataF 中提取對應 PDSCH RB 的樣本，並對應延伸（ext）到：
- rxdataF_ext[]：接收到的符號樣本
- dl_ch_estimates_ext[]：延伸後的 channel estimation
  
```
nr_dlsch_channel_level()
```
- 計算每條通道（每對 tx-rx antenna）的能量值
- 計算結果會給：
  - MRC 合併
  - LLR 正規化
  - log2_maxh：最大 channel gain，用來調整量化位元深度
```
nr_dlsch_channel_compensation()
```
- 把接收到的信號（rxdataF_ext）
  - 用 channel estimate 補償
  - 存成 rxdataF_comp：補償後資料，並傳至LLR做計算
    
```
static void nr_dlsch_llr(uint32_t rx_size_symbol,
                         int nbRx,
                         uint sz,
                         int16_t layer_llr[][sz],
                         int32_t rxdataF_comp[][nbRx][rx_size_symbol * NR_SYMBOLS_PER_SLOT],
                         int32_t dl_ch_mag[rx_size_symbol],
                         int32_t dl_ch_magb[rx_size_symbol],
                         int32_t dl_ch_magr[rx_size_symbol],
                         NR_DL_UE_HARQ_t *dlsch0_harq,
                         NR_DL_UE_HARQ_t *dlsch1_harq,
                         unsigned char symbol,
                         uint32_t len,
                         NR_UE_DLSCH_t dlsch[2],
                         uint32_t llr_offset_symbol)
{
  switch (dlsch[0].dlsch_config.qamModOrder) {
    case 2 :
      for(int l = 0; l < dlsch[0].Nl; l++)
        nr_qpsk_llr(&rxdataF_comp[l][0][symbol * rx_size_symbol], layer_llr[l] + llr_offset_symbol, len);
      break;

    case 4 :
      for(int l = 0; l < dlsch[0].Nl; l++)
        nr_16qam_llr(&rxdataF_comp[l][0][symbol * rx_size_symbol], dl_ch_mag, layer_llr[l] + llr_offset_symbol, len);
      break;

    case 6 :
      for(int l=0; l < dlsch[0].Nl; l++)
        nr_64qam_llr(&rxdataF_comp[l][0][symbol * rx_size_symbol], dl_ch_mag, dl_ch_magb, layer_llr[l] + llr_offset_symbol, len);
      break;

    case 8:
      for(int l=0; l < dlsch[0].Nl; l++)
        nr_256qam_llr(&rxdataF_comp[l][0][symbol * rx_size_symbol], dl_ch_mag, dl_ch_magb, dl_ch_magr, layer_llr[l] + llr_offset_symbol, len);
      break;

    default:
      AssertFatal(false, "Unknown mod_order!!!!\n");
      break;
  }
```
- 照接收到的數字選擇解llr的方式
- nr_256qam_llr等程式存放於nr_phy_common.c
