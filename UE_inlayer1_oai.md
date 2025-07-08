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

## coding
**openair1/PHY/NR_UE_TRANSPORT/nr_ulsch_coding.c**
實現 5G NR UE 的上行共享通道 (ULSCH) 編碼流程
- 資料準備：從輸入參數中提取TB的配置（如大小、調製方式、編碼率等）。
```
   int nr_ulsch_encoding(PHY_VARS_NR_UE *ue,
                      NR_UE_ULSCH_t *ulsch,
                      const uint32_t frame,
                      const uint8_t slot,
                      unsigned int *G,
                      int nb_ulsch,
                      uint8_t *ULSCH_ids)
- ue：指向 UE 在PHY的結構，包含 UE 的配置和狀態。
- ulsch：指向 ULSCH 結構，包含上行共享通道的配置（如 PUSCH PDU）。
- frame：在執行的幀編號。
- slot：在執行的時隙編號。
- G：編碼後的數據bits數 (碼塊大小)，每個 PUSCH 對應一個 G 值。
- nb_ulsch：需要處理的 PUSCH (Physical Uplink Shared Channel) 數量。
- ULSCH_ids：ULSCH 的 ID 陣列，用於搜尋不同的 ULSCH 實例。
```
```
unsigned int crc = 1;
    NR_UL_UE_HARQ_t *harq_process = &ue->ul_harq_processes[harq_pid];
    const nfapi_nr_ue_pusch_pdu_t *pusch_pdu = &ulsch->pusch_pdu;
    uint16_t nb_rb = pusch_pdu->rb_size;
    uint32_t A = pusch_pdu->pusch_data.tb_size << 3;
    uint8_t Qm = pusch_pdu->qam_mod_order;
    // target_code_rate is in 0.1 units
    float Coderate = (float)pusch_pdu->target_code_rate / 10240.0f;

    LOG_D(NR_PHY, "ulsch coding nb_rb %d, Nl = %d\n", nb_rb, pusch_pdu->nrOfLayers);
    LOG_D(NR_PHY, "ulsch coding A %d G %d mod_order %d Coderate %f\n", A, G[pusch_id], Qm, Coderate);
    LOG_D(NR_PHY, "harq_pid %d, pusch_data.new_data_indicator %d\n", harq_pid, pusch_pdu->pusch_data.new_data_indicator);
    從 ulsch 和 ue 中提取相關參數，如：
    - harq_pid：HARQ (Hybrid Automatic Repeat reQuest)  ID。
    - nb_rb：分配的資源塊 (Resource Blocks) 數量。
    - Qm：調製階數 (Modulation Order，如 QPSK、16QAM 等)。

```
- CRC 添加：為傳輸塊添加 CRC，用於錯誤檢測。
```
  if (A > NR_MAX_PDSCH_TBS) {
    crc = crc24a(harq_process->payload_AB, A) >> 8;
    harq_process->payload_AB[A >> 3] = ((uint8_t *)&crc)[2];
    harq_process->payload_AB[1 + (A >> 3)] = ((uint8_t *)&crc)[1];
    harq_process->payload_AB[2 + (A >> 3)] = ((uint8_t *)&crc)[0];
    B = A + 24;
} else {
    crc = crc16(harq_process->payload_AB, A) >> 16;
    harq_process->payload_AB[A >> 3] = ((uint8_t *)&crc)[1];
    harq_process->payload_AB[1 + (A >> 3)] = ((uint8_t *)&crc)[0];
    B = A + 16;
}
**大的加24bits,小的加16bits。
```

- 分段：將大傳輸塊分割成多個小塊，符合 LDPC 編碼需求。

```
TB_parameters->Kb = nr_segmentation(harq_process->payload_AB,
                                    harq_process->c,
                                    B,
                                    &harq_process->C,
                                    &harq_process->K,
                                    &harq_process->Z,
                                    &harq_process->F,
                                    harq_process->BG);
**Input：
payload_AB：包含 CRC 的資料。
B：總bits數（包括 CRC）。
BG：LDPC Base Graph，決定編碼結構。
**Output：
C：segment數量。
K：每個segment的bits數。
Z：LDPC segment的 Lifting Size。
F：填充bits數(資料bits不到K時，為保持對齊)。
Kb：BG中資料bits的列數。
**分段為使ldpc正確編碼。
 ```
                                   
- LDPC 編碼：對每個碼塊進行 LDPC 編碼，生成冗餘比特。

``` 
ue->nrLDPC_coding_interface.nrLDPC_coding_encoder(&slot_parameters);
** 程式內容包括:
交錯 (Interleaving)：重新排列比特以提高抗干擾能力。
LDPC 編碼：生成奇偶校驗位元，增加冗餘。
速率匹配：根據傳輸資源選擇部分編碼比特，適配通道容量。
```
**openair1/PHY/NR_UE_TRANSPORT/nr_ulsch_ue.c**
實現了 5G NR UE 的上行共享通道 (ULSCH) 的PHY傳輸流程
- 資料準備：
從 MAC 層接收傳輸塊，提取 PUSCH 配置（如資源分配、DMRS/PTRS 配置、調製方式等）。
- encoding與scrambling：
  CRC 、Segmentation、LDPC encoding、rate matching（nr_ulsch_encoding）。使用偽隨機序列加擾碼字（nr_pusch_codeword_scrambling）。
- modulation與layer mapping：
  將加擾後的資料調製為複數符號（nr_modulation）。分配到多個transport layer（nr_ue_layer_mapping）。
- 轉換預編碼（可選）：
  若啟用 DFT-s-OFDM，對資料進行 DFT 轉換（nr_dft），並使用低 PAPR DMRS 序列。
- 資源映射：
  將資料、DMRS 和 PTRS 映射到 PUSCH 資源網格（map_symbols, map_current_symbol）。處理 DMRS 類型（Type 1 或 Type 2）、PTRS 位置和 DC 載波的特殊情況。
- 預編碼與天線端口映射：
  應用預編碼矩陣，將層資料映射到天線端口（nr_layer_precoder）。輸出到頻域緩衝區 txdataF。
- OFDM 調製：
  應用頻域旋轉，執行 IFFT 和循環前綴添加，生成時域信號（nr_ue_pusch_common_procedures）。
輸出到 txdata。
## LDPC Encoding
[Read LDPC Encoder Code in OAI](https://hackmd.io/TOM6je4tQ9mjBCRt6VAyYQ#LDPC-Encoder-Code-in-OAI-5GNR)

## Modulation
[modulation](https://github.com/Karlyoo/LDPCinOAI/blob/main/modulation.md)
