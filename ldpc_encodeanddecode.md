# LDPC  in OAI 5GNR
## Introduction
  * Low-Density Parity-Check Code(LDPC),it uses a mathematical matrix to check for and correct errors. technique called **iterative message passing**,
  * LDPC codes are defined by a parity-check matrix H, which consists of variable nodes and check nodes . The variable nodes send their  LLR (log-likelihood ratio) messages to the check nodes. The check nodes perform XOR operations to verify the parity constraints. This iterative message passing continues for about 10 to 50 iterations until the codeword satisfies all parity checks.
  * In 5G, LDPC codes have a specially structured parity-check matrix (PCM), which allows for efficient decoding algorithms. [38.212](https://www.etsi.org/deliver/etsi_ts/138200_138299/138212/17.01.00_60/ts_138212v170100p.pdf) page.11 provides a deeper understanding of the decoding process.

LDPC File list
---
| File Name           | Role                                      | Module Level | Content                                         |
| ------------------- | ----------------------------------------- | ------------ | ----------------------------------------------- |
| `nr_ulsch_coding.c` | **Main LDPC encoding routine (uplink)**   | PHY Layer    | segmentation, CRC, LDPC encoding, rate matching |
| `nr_dlsch_coding.c` | **Main LDPC encoding routine (downlink)** | PHY Layer    | segmentation, CRC, LDPC encoding, rate matching |


| Directory Path                    | Description                                                                          | Function                           |
| --------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------- |
| `ldpc_encoder.c`                  | Main controller for LDPC encoding                                                    | `LDPCencoder()`                    |
| `ldpc_encode_parity_check.c`      | Uses mod-2 operations, bit shifts, and cyclic shifts to construct the final codeword |                                    |
| `nr_rate_matching.c`              | Rate Matching                                                                        | `nr_rate_matching_ldpc()`          |
| `nr_interleaving.c`               | Bit Interleaving                                                                     | `nr_interleaving_ldpc()`           |
| `nrLDPC_coding_segment_encoder.c` | Handles segment-based LDPC encoding                                                  | `nrLDPC_coding_encoder()`          |
| `nrLDPC_coding_segment_encoder.c` | Creates encoding tasks for each Code Block (CB)                                      | `nrLDPC_launch_TB_encoding()`      |
| `nrLDPC_coding_segment_encoder.c` | Encodes one 8-block segment                                                          | `ldpc8blocks()`                    |
| `ldpc_encode_parity_check.c`      | Executes parity computation                                                          | `encode_parity_check_part_optim()` |



## coding
**openair1/PHY/NR_UE_TRANSPORT/nr_ulsch_coding.c**
This file implements the ULSCH (Uplink Shared Channel) encoding process for 5G NR UE.
- data：Extracts the transport block (TB) configuration from input parameters (e.g., size, modulation type, coding rate, etc.):
```
   int nr_ulsch_encoding(PHY_VARS_NR_UE *ue,
                      NR_UE_ULSCH_t *ulsch,
                      const uint32_t frame,
                      const uint8_t slot,
                      unsigned int *G,
                      int nb_ulsch,
                      uint8_t *ULSCH_ids)
- ue: Pointer to the UE PHY structure, containing configuration and state.
- ulsch: Pointer to the ULSCH structure (e.g., PUSCH PDU).
- frame: Frame index.
- slot: Slot index.
- G: Total number of coded bits (per PUSCH).
- nb_ulsch: Number of PUSCH instances.
- ULSCH_ids: Array of ULSCH identifiers.
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
   - From ulsch and ue, the following are extracted:
      - harq_pid: HARQ process ID
      - nb_rb: Number of allocated Resource Blocks  
      - Qm: Modulation order (e.g., QPSK, 16QAM, etc.)

```
- CRC Attachment：Adds CRC for error detection.
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
```

- Segmentation: Splits a large TB into smaller code blocks for LDPC encoding.

```
TB_parameters->Kb = nr_segmentation(harq_process->payload_AB,
                                    harq_process->c,
                                    B,
                                    &harq_process->C,
                                    &harq_process->K,
                                    &harq_process->Z,
                                    &harq_process->F,
                                    harq_process->BG);
- Inputs:
  - payload_AB: Data with CRC
  - B: Total bits (with CRC)
  - BG: LDPC Base Graph
- Outputs:
  - C: Number of segments
  - K: Bits per segment
  - Z: Lifting size
  - F: Number of filler bits
  - Kb: Number of data columns in BG
Segmentation ensures LDPC operates on correctly sized code blocks.
 ```


#### openair1/PHY/CODING/nrLDPC_coding
**nrLDPC_coding_segment**
- nrLDPC_coding_segment_encoder.c
- nr_rate_matching.c
- Called by nr_dlsch_coding.c / nr_dlsch_encoding()
  - Segmentation
  - CRC Attachment
  - Z, K, F computation
  - Zero padding (if needed)
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
we will get a short type pointer matrix. It means Shift values table.

The front one is BG1,and the following one is BG2.

 KEY parameters in ldpc encode 
---
| Parameter  | Description                                    |
| ---------- | ---------------------------------------------- |
| `Kb`       | Number of information bits                     |
| `K`        | Actual encoder input length (includes padding) |
| `BG`, `Zc` | Define the parity-check matrix structure       |
| `C`        | Number of segments (Code Blocks)               |
| `d[]`      | Encoded output buffer (up to 68×384 bits)      |
| `c[]`      | Intermediate bit array (input segment)         |



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

| Step                                       | Description                                           |
| ------------------------------------------ | ----------------------------------------------------- |
| (1) Initialize `c[]`                       | Build double-sized buffer for SIMD alignment          |
| (2) Duplicate system bits                  | Ensure data alignment for vector operations           |
| (3) Expand `c[]`                           | Prepare multi-channel structure for SIMD              |
| (4) Call parity function (`ldpcXX_byte.c`) | Execute parity computation and write results to `d[]` |


Creating a double-length c[] buffer is essential for supporting SIMD alignment, data expansion, and error avoidance. It forms the foundation for accelerating the parity calculation.
SIMD (Single Instruction, Multiple Data) is a CPU vectorization acceleration technique that can process many data elements simultaneously.

To use SIMD, two important conditions must be met:
- Data must be contiguous
- Data must be aligned in memory (64-byte aligned)
Therefore, OAI expands the original cc[] buffer by a factor of two, duplicating each Zc block twice. In LDPC parity calculation, data needs to be shifted by certain offsets for XOR operations. If the data is not contiguous in memory, it may cause errors or partial reads.

**nrLDPC_coding_segment_encoder.c**
- Integrates the overall LDPC encoding pipeline for both uplink/downlink PHY layer.
- Function hierarchy:

nrLDPC_coding_encoder()
└── nrLDPC_launch_TB_encoding()
└── ldpc8blocks()

- Descriptions:
  - nrLDPC_coding_encoder: Encodes all TBs in a slot.
  - nrLDPC_launch_TB_encoding: Manages each TB encoding task, dispatching threads to ldpc8blocks().
  - ldpc8blocks(): Encodes up to 8 LDPC segments per thread.
     - LDPC encoding: LDPCencoder(c, d, impp)
     - Rate matching: nr_rate_matching_ldpc()
     - Bit interleaving: nr_interleaving_ldpc()

```mermaid
graph LR
    %% 定義樣式
    classDef main fill:#f9f,stroke:#333,stroke-width:2px;
    classDef sub fill:#e1f5fe,stroke:#333,stroke-width:1px;
    
    Start((MAC 資料輸入<br>TB Payload)):::main --> Prep

    subgraph Prep [第一階段：資料準備]
        direction TB
        A[參數提取 & CRC 附加<br>Func: nr_ulsch_encoding] --> B[分段 Segmentation<br>Func: nr_segmentation]
        B -->|計算參數| C[準備編碼任務]
        
        note1[計算 BG 與 Zc]
        B -.- note1
    end

    Prep --> Core

    subgraph Core [第二階段：分段編碼 Loop]
        direction TB
        LoopStart{對每個 Segment<br>多執行緒並行} --> D[LDPC 核心編碼<br>Func: LDPCencoder]
        
        D --> E[奇偶校驗位計算<br>Func: ldpc_encode_parity_check]
    end

    Core --> Post

    subgraph Post [第三階段：後處理]
        direction TB
        F[速率匹配<br>Func: nr_rate_matching_ldpc] --> G[位元交錯<br>Func: nr_interleaving_ldpc]
        G --> H[串接所有 Segments]
    end

    Post --> EndNode((輸出編碼位元<br>To Modulation)):::main
 ```

