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
| `nrLDPC_coding_segment_encoder.c`       | 經過分段處理的程式會送進此程式，執行LDPC encoding、Rate matching、Interleaving | `nrLDPC_coding_encoder()`|
| `nrLDPC_coding_segment_encoder.c` | 為每個 CB 建立編碼任務      | `nrLDPC_launch_TB_encoding()`  |
|  `nrLDPC_coding_segment_encoder.c`            | 進行單個 8-block 的編碼處理 | `ldpc8blocks()`                                 |
|`ldpc_encode_parity_check.c`|執行 parity 計算|`encode_parity_check_part_optim()`|

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
