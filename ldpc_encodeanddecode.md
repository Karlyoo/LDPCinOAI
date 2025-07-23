LDPC Encoder Code in OAI 5GNR
===


**Introduction**
---
* Before user's data is sent from UE to gNB, it
goes through  **physical layer**  processing, including:

1.CRC attachment
  
2.Code block segmentation

**3.LDPC encoding** 

4.Rate matching

5.Modulation (e.g., QPSK, 16QAM)

6.Layer mapping, precoding

7.OFDM modulation

8.Transmission over the air

* **What is LDPC?**
  * Low-Density Parity-Check Code(LDPC),it uses a mathematical matrix to check for and correct errors. technique called **iterative message passing**,
  * LDPC codes are defined by a parity-check matrix H, which consists of variable nodes and check nodes . The variable nodes send their  LLR (log-likelihood ratio) messages to the check nodes. The check nodes perform XOR operations to verify the parity constraints. This iterative message passing continues for about 10 to 50 iterations until the codeword satisfies all parity checks.
  * In 5G, LDPC codes have a specially structured parity-check matrix (PCM), which allows for efficient decoding algorithms. [38.212](https://www.etsi.org/deliver/etsi_ts/138200_138299/138212/17.01.00_60/ts_138212v170100p.pdf) page.11 provides a deeper understanding of the decoding process.

LDPC File list
---
| 檔案                  | 角色             | 模組層級                    | 實際內容                                          | 主要用途             |
| ------------------- | -------------- | ----------------------- | --------------------------------------------- | ----------------------------- |
| `nr_dlsch_coding.c` | **LDPC 編碼主邏輯** | PHY Layer               | segmentation, CRC, LDPC encode, rate matching | 負責將 TB → bitstream     |
| `nr_dlsch.c`        | **下行調變/傳送邏輯**  | PHY Layer      | 調變、PDSCH 映射、DMRS/PTRS 資料插入       | 把 encoded bits 變成 OFDM symbols |
| `dlschsim.c` | **功能測試用例**     | Simulation Tool  | 建立簡化 gNB/UE 結構，模擬 `nr_dlsch_encoding()` 整體流程  | 驗證功能與除錯               |
| `dlsim.c`  | **大型模擬框架**  | System-level Simulation | 整合 MAC/PHY/調變/通道環境，建立全鏈路模擬   | 驗證整體系統通聯能力與參數效能        |



 
| Directory Path         | Description |函式|
|------------------------|-------------|----|
| `ldpc_encoder.c`            | Main controller of the LDPC encoding process. |`LDPCencoder()`|
| `ldpc_encode_parity_check.c`   | Uses mod-2 operations, bit shifts, and cyclic shifts to construct the final codeword. | |
| `nr_rate_matching.c`     | 碼率匹配               | `nr_rate_matching_ldpc()`              |
| `nr_interleaving.c`      | 比特交錯               | `nr_interleaving_ldpc()`               |
| `nrLDPC_coding_segment_encoder.c`       | 經過分段處理的程式會送進此程式，執行LDPC encoding | `nrLDPC_coding_encoder()`|
| `nrLDPC_coding_segment_encoder.c` | 為每個 CB 建立編碼任務      | `nrLDPC_launch_TB_encoding()`  |
|  `nrLDPC_coding_segment_encoder.c`            | 進行單個 8-block 的編碼處理 | `ldpc8blocks()`                                 |
|`ldpc_encode_parity_check.c`|執行 parity 計算|`encode_parity_check_part_optim()`|


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








**PHY/CODING/nr_segmentation.c(由nr_dlsch_coding.c/nr_dlsch_encoding()呼叫)**
-  分段（Segmentation）
-  CRC 附加（CRC Attachment）呼叫crc_byte.c裡的函式
-  Z、K 等參數計算
-  F 補零數量（若有）
   
ldpc_encoder.c
---

![image](https://hackmd.io/_uploads/SJgF9UVEgg.png)

FROM https://www.mdpi.com/1424-8220/21/18/6266

```
#include <stdlib.h>
#include <math.h>
#include <stdio.h>
#include <string.h>
#include "defs.h"
#include "assertions.h"
#include "openair1/PHY/CODING/nrLDPC_defs.h"
#include "openair1/PHY/CODING/nrLDPC_extern.h"
#include "ldpc_generate_coefficient.c"
```
1. `nrLDPC_defs.h` includes all the necessary parameters for the encoding process. eg.block length
2. `ldpc_generate_coefficient.c` 
```
int LDPCencoder(unsigned char **inputArray, unsigned char *outputArray, encoder_implemparams_t *impp)
{
  const unsigned char *input = inputArray[0];
  // channel input is the output of this function!
  unsigned char *output = outputArray;
  const int Zc = impp->Zc;
  const int Kb = impp->Kb;
  const short block_length = impp->K;
  const short BG = impp->BG;
  const uint8_t gen_code = impp->gen_code;
  uint8_t c[22*384]; //padded input, unpacked, max size
  uint8_t d[68 * 384]; // coded output, unpacked, max size
```
**Encoding parameters:**
1. Kb: The number of information bits,the original information bits that you want to transmit before encoding.
2. K=Kb+padding. If the Kb does not match the LDPC structure,we have to put 0s to match it.

  
  ```
const short *Gen_shift_values = choose_generator_matrix(BG, Zc);
  if (Gen_shift_values==NULL) {
    printf("ldpc_encoder_orig: could not find generator matrix\n");
    return(-1);
  }
```
To choose the 'Generator Matrix',we need to know the BG and Zc. BG=Base Gragh,often in 1 or 2. 
we will get a short type pointer matrix. It means Shift values table(構造 LDPC parity-check matrix H 的位移值).

The front one is BG1,and the following one is BG2.

 KEY parameters in ldpc encode 
---
參數	|說明|
-----|------
BG, Zc, Kb|	定義 parity-check matrix 結構
K, F	|定義實際 LDPC encoder 的輸入長度與填 0 數量
C|	多少個 segment（Code Block）
d[]|	編碼輸出的中繼 buffer，大小是 68*384（最大可能編碼位元）
c[]|	每個 segment 的輸出指標，會寫回去最終 output


**Spilt to code word.**
Convert the byte-based input data (input[]) into a bit-level array (c[]) to enable bit-wise LDPC encoding.
```
for (i=0; i<block_length; i++)
  {
    // c[i] = input[i/8]<<(i%8);
    // c[i]=c[i]>>7&1;
    c[i] = (input[i / 8] & (128 >> (i & 7))) >> (7 - (i & 7));
  }
```
**Read cycle shift**
use 4 integer
i1: row index of parity-check matrix
i2: Bit index within Zc (cyclic shift index)
i3: column index of parity-check matrix
i4: Multiple shift values per matrix cell
```
for (i1=0; i1 < nrows-no_punctured_columns; i1++)
      {
        unsigned char channel_temp = 0;

        for (i3 = 0; i3 < Kb; i3++) {
          temp_prime = i1 * ncols + i3;

          for (i4 = 0; i4 < no_shift_values[temp_prime]; i4++) {
            channel_temp = channel_temp ^ c[i3 * Zc + Gen_shift_values[pointer_shift_values[temp_prime] + i4]];
          }
        }
```
**Obtain the Parity Code Block 1 / 2**
- gen_code == 0 → scalar XOR shift ：
- gen_code >= 1 →  SIMD C code（AVX512, AVX2...）
```
else if (gen_code == 1 && (Zc&31)==0) {
      shift=5; // AVX2 - 256-bit SIMD
      mask=31;
      strcpy(data_type,"simde__m256i");
      strcpy(xor_command,"simde_mm256_xor_si256");
    }
```
**Connect Code word**
Combine output and parity bits.
 ```
memcpy(&output[0], &c[2 * Zc], block_length - 2 * Zc);
  memcpy(&output[block_length - 2 * Zc], &d[0], (nrows - no_punctured_columns) * Zc - removed_bit);
  // memcpy(output,c,Kb*Zc*sizeof(unsigned char));
  return block_length - 2 * Zc + (nrows - no_punctured_columns) * Zc - removed_bit;
```

**ldpc_encode_parity_check.c**
---

|步驟 |	內容 |
|-----|----------|
|(1) Initialize c[]	|Build a double copied data array,in using of  SIMD process.|
|(2) 將系統位元 cc 重複複製進 c[]	|為 SIMD 資料對齊。|
|(3) 多次擴展 c[]	|擴展為多通道形式供 SIMD 平行處理。|
|(4)Call the corresponding hardcoded parity function(ldpcXX_byte.c	)|執行真正的 parity 計算，並將結果寫到 d。|

Creating a double-length c[] buffer is essential for supporting SIMD alignment, data expansion, and error avoidance. It forms the foundation for accelerating the parity calculation.
SIMD (Single Instruction, Multiple Data) is a CPU vectorization acceleration technique that can process many data elements simultaneously.

To use SIMD, two important conditions must be met:
- Data must be contiguous
- Data must be aligned in memory (64-byte aligned)
Therefore, OAI expands the original cc[] buffer by a factor of two, duplicating each Zc block twice. In LDPC parity calculation, data needs to be shifted by certain offsets for XOR operations. If the data is not contiguous in memory, it may cause errors or partial reads.

**nrLDPC_coding_segment_encoder.c**
---
- 主要負責實作 整體 LDPC 編碼流程的頂層整合，屬於 5G NR 下行/上行傳輸的實體層編碼總控模組
- nrLDPC_coding_encoder()<br>
         └── nrLDPC_launch_TB_encoding()<br>
  _________                └── ldpc8blocks()   

- **nrLDPC_coding_encoder** 對一個 slot 中所有 Transport Blocks (TBs) 執行 LDPC 編碼，並將編碼後的輸出寫入對應 buffer
- **nrLDPC_launch_TB_encoding** 一個完整傳輸區塊 (TB) 的 LDPC 編碼任務流程控制器，創建並派送對應的執行緒任務給 ldpc8blocks() 處理每 8 個 segments 的編碼
- **ldpc8blocks()**  在一個執行緒中處理 最多 8 個 LDPC segments 
   - 把一整個 Slot 中的所有 Transport Blocks (TB)，依照 5G NR 標準進行：
      - LDPC Encoding:`LDPCencoder(c, d, impp)`
      - Rate Matching:`nr_rate_matching_ldpc()`
      - Bit Interleaving (交錯處理):`nr_interleaving_ldpc()`
      - Output result 
    - 每 8 個 segments 為一個任務


# decode
- 程式入口為```nr_dlsch_decoding.c```
### nr_dlsch_decoding.c
```
void nr_ue_dlsch_init(NR_UE_DLSCH_t *dlsch_list, int num_dlsch, uint8_t max_ldpc_iterations) {
  for (int i=0; i < num_dlsch; i++) {
    NR_UE_DLSCH_t *dlsch = dlsch_list + i;
    memset(dlsch, 0, sizeof(NR_UE_DLSCH_t));
    dlsch->max_ldpc_iterations = max_ldpc_iterations;
  }
}
```
- 初始化 DLSCH 通道結構
  - 迴圈處理：遍歷所有需要初始化的 DLSCH 結構。
  - 清空記憶體：使用 memset 將每個結構的記憶體全部設為 0，確保沒有殘留的舊資料。
  - 設定參數：將傳入的 max_ldpc_iterations 參數寫入結構中。這是 LDPC 解碼演算法的一個重要設定，會影響解碼效能和運算複雜度。

```
void nr_dlsch_unscrambling(int16_t *llr, uint32_t size, uint8_t q, uint32_t Nid, uint32_t n_RNTI)
{
  nr_codeword_unscrambling(llr, size, q, Nid, n_RNTI);
}
```
- 做還原scramble的動作
- llr：從解調器（Demodulator）輸出的軟位元（Log-Likelihood Ratios, LLRs），代表每個位元是 0 還是 1 的可能性。
- size：LLR 陣列的大小。
- q：碼字（Codeword）的索引，因為一個傳輸塊（Transport Block）可能對應到多個碼字。
- Nid 和 n_RNTI：用於產生和發送端完全相同的擾碼序列的必要參數，分別是細胞 ID 和使用者臨時識別碼。
```
void nr_dlsch_decoding(PHY_VARS_NR_UE *phy_vars_ue,
                       const UE_nr_rxtx_proc_t *proc,
                       NR_UE_DLSCH_t *dlsch,
                       int16_t **dlsch_llr,
                       uint8_t **b,
                       int *G,
                       int nb_dlsch,
                       uint8_t *DLSCH_ids)
```
- 執行 LDPC 解碼：對輸入的 LLR 進行 LDPC 解碼，嘗試修正錯誤。
- 速率恢復（Rate Dematching）：還原傳輸前的速率匹配操作。
- CRC 校驗：解碼完成後，會計算資料塊的循環冗餘校驗碼（CRC）。如果計算出的 CRC 與資料中包含的 CRC 相符，則表示解碼成功；反之則表示解碼失敗。

<img width="388" height="803" alt="image" src="https://github.com/user-attachments/assets/9fa47ab7-b330-4b46-8d48-b4fb31d5da2d" />

```
| 流程步驟                      | 說明                               |                                         
| ------------------------- | -------------------------------- | 
| Start Decoder             | 啟動解碼器流程                          |
| Load Parameters / Buffers | 設定 C、K、Z、E、F、LLR 等資訊             | 
| BN → CN 初始處理              | Variable nodes 傳遞資訊給 Check nodes | 
| CN → BN 初始處理              | Check nodes 計算校驗並回傳給變數節點         |
| First Parity Check        | 嘗試解出正確 codeword                  | 
| Output Bits               | 若成功，輸出 b\[], 否則重傳                | 
| Iterative Loop            | 進行最多 `max_ldpc_iterations` 次反覆計算 | 
| CN Processing             | 更新所有 parity constraints          | 
| CN→BN 回傳                  | 傳遞回 variable node                | 
| BN Processing             | 更新變數節點訊息                         | 
```

- 主入口函式為: nrLDPC_decoder_core in nrLDPC_decoder.c
- Decoder Setup
   - 載入 LUT（lookup tables）：供後續 CN/BN 運算使用
   - 初始化緩衝區：分別為 CN、BN 與 LLRs 分配記憶體空間
   - 資料對齊與 SIMD 準備：便於後續使用 AVX 加速
**LUT**
- LUT 是一種預先建立好資料的「表格」，讓演算法可以用索引快速查出對應值，而不是每次都重複計算
- 為什麼使用 LUT？:
   - 速度快、節省運算資源、適合定義明確的映射關係
 
**SIMD**
- SIMD 是一種 CPU 向量運算技術，允許一條指令同時操作多筆資料
- 在 LDPC 解碼中如何用到 SIMD
```
| 階段          | SIMD 加速的用途                               |
| ----------- | ---------------------------------------- |
| LLR 輸入處理    | 加速整批 bit 或 symbol 的 LLR 計算與存取            |
| BN→CN 訊息傳遞  | 同時處理多個 variable node 的 message           |
| CN→BN 訊息傳遞  | 比對最小值（min-sum），適合用 SIMD 做 parallel min() |
| CRC 計算      | 多 bit 並行 XOR                             |
| LLR 解調（調變器） | 輸入 IQ 值轉換為 LLR 時用 SIMD 加速 inner product  |
```
- First Pass Processing
   - BN Pre-processing with nrLDPC_bnProcPc_* (rate & BG-specific)
   - CN Pre-processing via nrLDPC_bn2cnProcBuf_*
   - First CN processing: nrLDPC_cnProc_*
   - Back-propagate to BN via nrLDPC_cn2bnProcBuf_*
 
- Iterative Decoding Loop
**執行條件**
  - numIter < numMaxIter
  - Parity check (or CRC) failed
 
- Steps per iteration
   - CN 運算
   - 將 CN 傳回 BN 的訊息
   - BN 運算（更新 soft LLR）
   - 準備下一輪 CN 輸入
   - 進行 Parity Check 或 CRC 驗證
 
- Early Termination
   - If check_crc is enabled
     - 解碼後會使用 CRC 驗證硬判定資料
     - 若通過驗證，提前結束迴圈
   - 若未啟用 CRC：
     - 每次都執行 parity-check 決定是否提早終止

- Output Formatting
- Output mode: LLRINT8, BIT, or BITINT8

