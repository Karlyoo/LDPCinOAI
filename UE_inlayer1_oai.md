# UE Tx and RX
## Architecture
```
【 Uplink: UE TX  】              【 Downlink: UE RX  】

Input: MAC PDU (bytes)                      ┌──────────────────────────────┐
        ↓                                   │ OFDM FFT + CP remove         │
┌──────────────────────────────┐            │ Input: RX samples            │
│ CRC attachment (crc_byte.c)  │            │ Output: freq-domain symbols  │
│ Output: bits + CRC           │            └────────────┬─────────────────┘
└────────────┬─────────────────┘                         ↓
             ↓                            ┌──────────────────────────────┐
┌──────────────────────────────┐         │ channel_estimation           │
│ Segmentation (nr_segmentation.c)│       │ Output: equalized symbols    │
│ Output: LDPC segments (bits)  │         └────────────┬─────────────────┘
└────────────┬─────────────────┘                         ↓
             ↓                            ┌──────────────────────────────┐
┌──────────────────────────────┐         │ Demodulation + LLR calculate │
│ LDPC encode (nr_dlsch_coding.c)│       │ Output: soft bits (LLRs)     │
│ Output: encoded bits          │         └────────────┬─────────────────┘
└────────────┬─────────────────┘                         ↓
             ↓                            ┌──────────────────────────────┐
┌──────────────────────────────┐         │ Rate Dematching              │
│ Rate Matching + Interleaving │         │ Input: LLRs                  │
│ Output: RM bits              │         │ Output: code blocks          │
└────────────┬─────────────────┘         └────────────┬─────────────────┘
             ↓                                                    ↓
┌──────────────────────────────┐         ┌──────────────────────────────┐
│ Scrambling                   │         │ LDPC decoding                │
│ Output: RM bits              │         │ (nr_dlsch_decoding.c)        │
└────────────┬─────────────────┘         │ Output: decoded bits         │
             ↓                           └────────────┬─────────────────┘
┌──────────────────────────────┐                      ↓
│ Modulation: QPSK, 16QAM       │       ┌──────────────────────────────┐
│ Output: complex symbols (IQ) │       │ CRC CHECK + merge segments    │
└────────────┬─────────────────┘       │ Output: MAC SDU (bytes)       │
             ↓                         └──────────────────────────────┘
┌──────────────────────────────┐
│ Layer Mapping                │
│ Output: complex symbols (IQ) │
└────────────┬─────────────────┘
             ↓
┌────────────────────────────────────────┐
│ Insert DMRS (UE uplink reference signal) │
│ Output: complex symbols (IQ)             │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ DFT-s-OFDM IFFT + CP (ofdm_mod.c)      │
│ Output: time-domain samples            │
└────────────┬────────────────────────────┘
             ↓
┌──────────────────────────────┐
│ Simulate channel             │
│ Input: TX samples            │
│ Output: RX samples           │
└──────────────────────────────┘

```

| Folder                         | Function                                                      | Key Point                                                                  |
| ------------------------------ | ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `NR_TRANSPORT/`                | 5G NR gNB Transport                                           | Functions related to gNB-side NR transport processing.                     |
| `NR_UE_TRANSPORT/`             | UE Transport Functions                                        | Handles uplink/downlink transport for UE.                                  |
| `NR_REFSIG/`                   | NR Reference Signal Module                                    | Includes **DMRS, PTRS, PRACH, and SSB** waveform generation and insertion. |
| `MODULATION/`                  | OFDM IFFT/FFT, Modulation, Mapping                            | Contains modulation and mapping functions for physical channels.           |
| `TOOLS/`                       | Channel Estimation, Vector Operations, FFT Tools, Phase Noise | Provides utility functions for PHY signal processing.                      |
| `INIT/`                        | Layer 1 Variable Initialization                               | Called within `phy_init_nr_ue()` to initialize PHY parameters.             |
| `CODING/`                      | LDPC/Polar Encoding & Decoding, CRC, Segmentation             | Implements channel coding and rate matching functions.                     |
| `defs.h`, `extern.h`, `vars.h` | Global Definitions and Variable References                    | These files define and share variables across PHY modules.                 |


  
| Function                           | Specification Reference                                               | Corresponding OAI Function / File                                              |
| ---------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Channel Estimation**             | TS 38.211 §7.4.1.1 (DM-RS)                                            | `nr_pbch_channel_estimation.c`<br>`nr_pdsch_channel_estimation.c`              |
| **Demodulation + LLR Calculation** | TS 38.211 §7.3.1.4 (Modulation)<br>TS 38.212 §7.1.4 (LLR Calculation) | `nr_dlsch_llr_computation.c`<br>`nr_qpsk_llr.c`, `nr_qam16_llr.c`              |
| **Rate Dematching**                | TS 38.212 §5.4.1 (Downlink)                                           | `nr_rate_matching_ldpc.c`<br>`nr_dlsch_decoding.c`                             |
| **LDPC Decoding**                  | TS 38.212 §5.3.2                                                      | `nrLDPC_decoder.c`<br>`ldpc_decode.c`<br>Called within `nr_dlsch_decoding.c`   |
| **CRC Check + Segment Merging**    | TS 38.212 §5.1 (CRC)<br>§5.2.2 (Code Block Segment Combination)       | `crc_byte.c`, `check_crc.c`<br>Combined and processed in `nr_dlsch_decoding.c` |


| File Name                 | Function Description                                                                                     |
| ------------------------- | -------------------------------------------------------------------------------------------------------- |
| `nr_dci.c`                | Handles DCI formats (e.g., Format 1_0 / 1_1) and DCI information packing.                                |
| `cic_filter_nr.c`         | Implements Cascaded Integrator-Comb (CIC) and FIR filtering for decimation and sample rate reduction.    |
| `nr_dlsch_decoding.c`     | Performs LDPC decoding, HARQ updating, and CRC verification after UE receives downlink data.             |
| `nr_dlsch_demodulation.c` | Handles demapping, channel compensation, and LLR computation for received downlink data before decoding. |
| `nr_initial_sync.c`       | Performs UE initial synchronization to detect and synchronize with the gNB.                              |
| `nr_initial_sync_sl.c`    | Performs initial synchronization in Sidelink mode (UE-to-UE).                                            |
| `nr_prach.c`              | Generates PRACH preambles and implements the corresponding matched filtering for detection.              |
| `nr_psbch_rx.c`           | Receives and decodes the Physical Sidelink Broadcast Channel (PSBCH) signal.                             |
| `nr_psbch_tx.c`           | Transmits the Physical Sidelink Broadcast Channel (PSBCH) signal.                                        |
| `nr_pbch.c`               | Packages the MIB and inserts PBCH reference signals.                                                     |
| `pss_nr.c`                | Detects the Primary Synchronization Signal (PSS) used for time synchronization.                          |
| `sss_nr.c`                | Detects and decodes the Secondary Synchronization Signal (SSS).                                          |
| `nr_ue_rf_helpers.c`      | Configures RF card frequency and gain parameters.                                                        |
| `nr_ulsch_coding.c`       | Implements ULSCH encoding, including CRC attachment, segmentation, LDPC encoding, and rate matching.     |
| `nr_ulsch_ue.c`           | Handles all UE-side ULSCH processing such as encoding and modulation.                                    |
| `nr_transport_proto_ue.h` | Declares multiple PHY-layer functions related to DLSCH, ULSCH, PUCCH, PBCH, PRACH, and PSBCH processing. |
| `nr_transport_ue.h`       | Defines UE-side data structures related to the transport layer.                                          |
| `pucch_nr.c`              | Handles UCI transmission (e.g., SR, HARQ-ACK, CSI) on the PUCCH.                                         |
| `pucch_nr.h`              | Defines UE-side data structures and functions related to PUCCH and UCI transmission.                     |
| `srs_modulation_nr.c`     | Implements generation and processing of the Sounding Reference Signal (SRS) for the UE.                  |
| `srs_modulation_nr.h`     | Defines UE-side data structures and functions related to SRS.                                            |

## UPLINK
### Corresponding 3GPP Specifications and OAI Functions
| Function               | 3GPP Specification Section | Corresponding OAI Function(s)                       |
| ---------------------- | -------------------------- | --------------------------------------------------- |
| **CRC Attachment**     | 38.212 §5.1                | `crc24a()`, `crc24b()`, etc.                        |
| **Segmentation**       | 38.212 §5.2                | `nr_segmentation()`                                 |
| **LDPC Encoding**      | 38.212 §5.2                | `LDPCencoder()`, `nr_ulsch_encoding()`              |
| **Scrambling**         | 38.211 §6.3.1.1            | `nr_pusch_codeword_scrambling()`                    |
| **Modulation**         | 38.211 §6.3.2              | `nr_modulation()`                                   |
| **Layer Mapping**      | 38.211 §6.3.1.2            | `nr_ue_layer_mapping()`                             |
| **Channel Estimation** | 38.211 §6.4.1.1.2 (DMRS)   | `nr_pbch_channel_estimation()`                      |
| **LLR Calculation**    | 38.212 Annex A             | `nr_dlsch_qpsk_llr()`, `nr_dlsch_16qam_llr()`, etc. |


##### openairinterface5g/openair1/PHY/CODING
**crc_byte.c**
  <img width="889" height="253" alt="image" src="https://github.com/user-attachments/assets/184d8388-9041-4d15-8cf6-1f072b9850fe" />
  
- Defines functions for inserting and verifying CRC codes
  
  <img width="749" height="345" alt="image" src="https://github.com/user-attachments/assets/f59d8d42-349b-402c-aae0-2f8081bc944f" />

##### openairinterface5g/openair1/PHY/CODING
**nr_segmentation.c**
- Implements data segmentation according to 3GPP TS 38.212 §5.2, preparing the input blocks for LDPC encoding.

##### LDPC Encoding
[ldpc_encoding](https://github.com/Karlyoo/LDPCinOAI/blob/main/ldpc_encodeanddecode.md#ldpc-encoder-code-in-oai-5gnr)

##### Modulation
[modulation](https://github.com/Karlyoo/LDPCinOAI/blob/main/modulation%26demodulation.md#modulation)

  
**openair1/PHY/NR_UE_TRANSPORT/nr_ulsch_ue.c**
Implements the PHY transmission procedure for the 5G NR Uplink Shared Channel (ULSCH) at the UE side.
- data preparation：
   - Receives transport blocks from the MAC layer.
   - Extracts PUSCH configurations, including resource allocation, DMRS/PTRS setup, and modulation type.
- encoding and scrambling：
  CRC 、Segmentation、LDPC encoding、rate matching（nr_ulsch_encoding）。Applies a pseudo-random scrambling sequence using:（nr_pusch_codeword_scrambling）。
  ```
  void nr_pusch_codeword_scrambling(uint8_t *in, uint32_t size, uint32_t Nid, uint32_t n_RNTI, bool uci_on_pusch, uint32_t* out)
  void nr_pusch_codeword_scrambling_uci(uint8_t *in, uint32_t size, uint32_t Nid, uint32_t n_RNTI, uint32_t* out)
  ```
- Modulation and Layer Mapping
  - Modulates scrambled bits into complex symbols (nr_modulation).
  - Distributes symbols across transmission layers (nr_ue_layer_mapping).
- Transform Precoding (optional)
  - If DFT-s-OFDM is enabled, performs DFT precoding (nr_dft)
  - and uses low-PAPR DMRS sequences.
- Resource Mapping
  - Maps data, DMRS, and PTRS onto the PUSCH resource grid (map_symbols, map_current_symbol).
  - Handles: DMRS Type 1 / Type 2,PTRS placement, Special cases around the DC subcarrier.

- Precoding and Antenna Mapping
  - Applies the precoding matrix, mapping layer data to physical antennas (nr_layer_precoder).
- Outputs frequency-domain data to txdataF.
  - (All these mapping and precoding functions are defined in modulation.c.)
- OFDM Modulation
  - Performs frequency-domain rotation, IFFT, and cyclic prefix addition to generate the time-domain waveform (nr_ue_pusch_common_procedures).
  - Final output is written to txdata.



### 3GPP TS 38.211/212 程序與 OAI 函式對照表

### OAI 物理層端到端流程對照表

| 流程方塊 | 3GPP 規範 | OAI 函式 | 檔案位置 |
| :--- | :--- | :--- | :--- |
| **CRC 附加** (Tx) | TS 38.212 §5.1 | `crc_byte` / `crc24c` | `crc_byte.c` |
| **LDPC 編碼** (Tx) | TS 38.212 §5.3.2 | `nrLDPC_encoder` | `nrLDPC_encoder.c` |
| **調變** (Tx) | TS 38.211 §6.3.2 | `nr_modulation` | `nr_modulation.c` |
| **層映射與預編碼** (Tx) | TS 38.211 §6.3.1 | `nr_layer_mapping`<br>`nr_layer_precoder` | `nr_layer_mapping.c`<br>`nr_layer_precoder.c` |
| **資源映射** (Tx) | TS 38.211 §7.4.1 | `nr_generate_pdsch` | `nr_dlsch.c` |
| **OFDM 調變** (Tx) | TS 38.211 §5.3 | `nr_ofdm_mod` | `nr_ofdm_mod.c` |
| **通道模擬** | N/A | `new_channel_desc_scm` | `random_channel.c` |
| **OFDM 解調** (Rx) | TS 38.211 §5.3 | `nr_ofdm_demod` | `nr_ofdm_demod.c` |
| **資源解映射與估測** (Rx) | TS 38.211 §6.3/6.4 | `nr_ulsch_extract_rbs`<br>`nr_pusch_channel_estimation` | `nr_ulsch_demodulation.c`<br>`nr_pusch_channel_estimation.c` |
| **等化與層解映射** (Rx) | N/A | `nr_ulsch_channel_compensation` | `nr_ulsch_demodulation.c` |
| **LLR 計算** (Rx) | TS 38.211 §7.3.1 | `nr_ulsch_compute_llr` | `nr_ulsch_demodulation.c` |
| **LDPC 解碼** (Rx) | TS 38.212 §5.3.2 | `nr_ulsch_decoding`<br>`nrLDPC_coding_decoder` | `nr_ulsch_decoding.c` |
| **CRC 檢查** (Rx) | TS 38.212 §5.1 | `crc_parity_check` | `crc_byte.c` |
