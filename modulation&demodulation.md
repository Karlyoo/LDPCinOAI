## Modulation
**openair1/PHY/MODULATION** 
| File Name              | Function                                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `beamforming.c`        | Implements beamforming functionality on the eNB (base station) side.                                                           |
| `compute_bf_weights.c` | Currently empty; possibly intended for deriving downlink CSI (DLCSI) from uplink CSI (ULCSI) to calculate beamforming weights. |
| `modulation_UE.h`      | Defines the core function interfaces for UE-side modulation/demodulation front-end processing.                                 |
| `modulation_eNB.h`     | Defines key PHY-layer function interfaces related to modulation and beamforming for the base station.                          |
| `nr_modulation.c`      | Implements modulation, layer mapping, DFT (Discrete Fourier Transform), and precoding.                                         |
| `modulation_common.h`  | Contains common modulation constants used in NR UE.                                                                            |
| `modulation_extern.h`  | Declares extern variables and functions used in the modulation module.                                                         |


##  OFDM and Front-End Processing Modules

| File Name          | Category        | Description                                                                  |
| ------------------ | --------------- | ---------------------------------------------------------------------------- |
| `ofdm_mod.c`       | OFDM Modulation | Implements FFT/IFFT and cyclic prefix (CP) insertion (shared by NR and LTE). |
| `slot_fep.c`       | Slot FEP        | LTE slot front-end processing (receiver side FFT, etc.).                     |
| `slot_fep_mbsfn.c` | Slot FEP        | MBSFN-specific slot front-end processing.                                    |
| `slot_fep_nr.c`    | Slot FEP (NR)   | 5G NR slot front-end processing (FFT, SC-FDMA handling, etc.).               |
| `slot_fep_ul.c`    | Slot FEP (UL)   | Uplink LTE slot processing logic.                                            |


```
  A[Encoded Codewords] --> B[nr_modulation.c (QAM Mapping)]
  B --> C[Resource Mapping]
  C --> D[OFDM IFFT + CP insert (ofdm_mod.c)]
  D --> E[Slot packing (slot_fep_nr.c)]
  E --> F[ RF or channel simulator]
```

### nr_modulation.c
- Modulation (nr_modulation): Maps encoded bitstreams into complex modulation symbols (QPSK, 16-QAM, 64-QAM, or 256-QAM).
- Inputs:
  - in: input bitstream (in uint32_t format).
  - length: number of bits.
  - mod_order: modulation order (2 → QPSK, 4 → 16QAM, 6 → 64QAM, 8 → 256QAM).
- out: output complex symbols (int16_t fixed-point Q15 format for I/Q).
- Operation: Groups input bits according to modulation order and maps them using predefined modulation tables (e.g. nr_qpsk_mod_table, nr_16qam_mod_table from 3GPP TS 38.211), producing complex-valued symbols.
  
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

*In QPSK, every 2 bits map to one complex symbol (e.g. {1+j, 1−j, −1+j, −1−j}).
The SSE2 version processes 8 bits per iteration, generating 4 complex symbols for high throughput.
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
      // ... 
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
*In 64-QAM, every 6 bits map to one complex symbol.
The function extracts 12-bit chunks (two 6-bit symbols) and looks them up in nr_64qam_mod_table.
Each symbol is stored as two int16_t values (I/Q) in the output buffer.
```

- Layer mapping (nr_layer_mapping, nr_ue_layer_mapping)：Distributes modulated symbols across multiple transmission layers for MIMO.
  - Parameters:
    - nbCodes: number of codewords (1 or 2).
    - encoded_len: symbol length per codeword.
    - mod_symbs: modulated complex symbols (c16_t).
    - n_layers: number of layers (1–4).
    - tx_layers: output arrays for each layer.
  - Example: For 4 layers, every group of symbols is assigned cyclically to each layer.
    - gNB: uses nr_layer_mapping for downlink (supports 1–4 layers, optimized with AVX512/AVX2).
    - UE: uses nr_ue_layer_mapping for uplink (usually 1–2 layers, due to antenna limits).
- DFT (nr_dft)：Performs the Discrete Fourier Transform on uplink data (for DFT-s-OFDM, i.e. SC-FDMA).
  - Converts time-domain modulated symbols to frequency domain to reduce PAPR.
  - Parameters:
    - d: input time-domain symbols (c16_t, Q15 format).
    - z: output frequency-domain symbols.
    - Msc_PUSCH: DFT size = number of allocated subcarriers (typically N_RB × 12).
- Symbol Rotation (perform_symbol_rotation, init_symbol_rotation, init_timeshift_rotation)
  - Applies complex phase correction to compensate for carrier frequency or timing offset.
  - perform_symbol_rotation():
    - Computes a complex rotation factor e^{j2πf₀t} for each OFDM symbol and stores it in symbol_rotation[].
  - init_symbol_rotation():
    - Initializes the rotation table for DL/UL frames.
  - init_timeshift_rotation():
    - Handles additional phase shift due to FFT start misalignment in the receiver.
- Precoding (nr_layer_precoder, nr_layer_precoder_cm, nr_layer_precoder_simd)
  - Applies precoding matrices for multi-antenna transmission (Layer → Antenna mapping).
  - nr_layer_precoder():
    - Simplified character-based weighting (e.g., ‘1’, ‘-1’, ‘j’, ‘-j’) for small MIMO.
  - nr_layer_precoder_cm():
    - Uses complex precoding weights (PMI-defined) and performs complex multiply-accumulate per layer.
  - nr_layer_precoder_simd():
    - Vectorized version using SIMD instructions for acceleration.
## nr_dlsch_demodulation.c  
```
nr_rx_pdsch()
│
├─ nr_dlsch_channel_level()        → compute average per-layer/antenna channel power
│
├─ nr_dlsch_channel_compensation() → equalization: multiply Y by conjugate(H)
│
├─ nr_dlsch_detection_mrc()        → MRC combining (if applicable)
│
├─ LLR computation based on modulation:
│   ├─ nr_dlsch_qpsk_llr()
│   ├─ nr_dlsch_qam16_llr()
│   ├─ nr_dlsch_qam64_llr()
│   └─ nr_dlsch_qam256_llr()
│
└─ Output LLR → soft buffer → LDPC decoder


```

| Stage                              | Input                                | Output                               |
| ---------------------------------- | ------------------------------------ | ------------------------------------ |
| RB extraction & channel estimation | `rxdataF`, `dl_ch_estimates`         | `rxdataF_ext`, `dl_ch_estimates_ext` |
| Channel scaling                    | `dl_ch_estimates_ext`                | Scaled channel estimates             |
| Channel compensation               | `rxdataF_ext`, `dl_ch_estimates_ext` | `rxdataF_comp`, `dl_ch_mag*`         |
| MRC/MMSE combining                 | `rxdataF_comp`, `dl_ch_mag*`         | Equalized data streams               |
| LLR computation                    | `rxdataF_comp`, `dl_ch_mag*`         | Symbol-level LLRs                    |
| Layer mapping/output               | `layer_llr`, TB config               | `llr[CW0]`, `llr[CW1]`               |


```
NR_DL_UE_HARQ_t *dlsch0_harq, *dlsch1_harq;
dlsch0_harq = &ue->dl_harq_processes[0][harq_pid];
```
- Determines which Transport Block (TB) is currently active based on the HARQ PID:
  - If both HARQs are active, codeword_TB0 and codeword_TB1 are set.
  - Otherwise, only one TB is processed.
```
nr_dlsch_extract_rbs()
```
- Extracts the corresponding PDSCH RB samples from the received rxdataF, and expands (extends) them into:
- rxdataF_ext[]: the received symbol samples
- dl_ch_estimates_ext[]: the extended channel estimation
  
```
nr_dlsch_channel_level()
```
- Computes the power level (energy) of each channel for every transmit–receive antenna pair.
- The results are used for:
  - MRC combining
  - LLR normalization
  - log2_maxh: the maximum channel gain used for adjusting quantization bit depth
```
nr_dlsch_channel_compensation()
```
- Uses the channel estimates to compensate the received signal (rxdataF_ext).
- The compensated results are stored in rxdataF_comp, which are then passed to the LLR computation stage.
    
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

