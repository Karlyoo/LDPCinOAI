# 4 Frame structure and physical resources 
## 4.1 General
- Usually,Time units are defined as:
  - Tc = 1 / (Δf_max × Nf),
  - where Δf_max = 480,000 Hz and Nf = 4096.
- Another commonly used time unit is:
  - Ts = Tc × (Nf_ref / Δf_ref),
  - where Δf_ref = 15,000 Hz and Nf_ref = 2048.
## 4.2 Numerologies
- The subcarrier spacing is defined as:
  - Δf = 2^μ × 15 kHz,  where μ is the numerology index.
  - Supported configurations:
    
    | μ | Subcarrier spacing (kHz) | Cyclic Prefix      |
    | - | ------------------------ | ------------------ |
    | 0 | 15                       | Normal             |
    | 1 | 30                       | Normal             |
    | 2 | 60                       | Normal or Extended |
    | 3 | 120                      | Normal             |
    | 4 | 240                      | Normal             |

## 4.3 Frame Structure
### 4.3.1 Frames and Subframes
- One frame is 10 milliseconds long and consists of 10 subframes (each 1 ms).
- Each frame is split into two half-frames:
  - Half-frame 0: subframes 0 to 4
  - Half-frame 1: subframes 5 to 9
- Downlink (DL) and uplink (UL) have separate frame structures on a carrier.
- The UL frame number i starts earlier than the corresponding DL frame by:
  - TA = (N_TA + N_TA_offset) × Tc
### 4.3.2 Slots
- For each numerology μ:
  - Slots are numbered within subframes and frames.
  - Each slot contains a fixed number of OFDM symbols depending on the cyclic prefix.
 
| μ | Symbols per slot | Slots per frame | Slots per subframe |
| - | ---------------- | --------------- | ------------------ |
| 0 | 14               | 10              | 1                  |
| 1 | 14               | 20              | 2                  |
| 2 | 14               | 40              | 4                  |
| 3 | 14               | 80              | 8                  |
| 4 | 14               | 160             | 16                 |

- OFDM symbols in a slot are categorized as:
  - Downlink
  - Uplink
  - Flexible
- In a downlink slot, the UE should expect transmissions only in downlink or flexible symbols.
- In an uplink slot, the UE may only transmit in uplink or flexible symbols.
#### Timing Requirements for Half-Duplex UEs
- A UE that does not support full-duplex or simultaneous transmission and reception must observe certain timing gaps.

| Transition | FR1 (Tc units) | FR2 (Tc units) |
| ---------- | -------------- | -------------- |
| Tx to Rx   | 25600          | 13792          |
| Rx to Tx   | 25600          | Not specified  |

- Such a UE must:
  - Wait Rx-to-Tx time before transmitting after receiving.
  - Wait Tx-to-Rx time before receiving after transmitting.
 
# 5 Generic functions
## 5.1 Modulation mapper
- The modulation mapper takes binary digits, 0 or 1, as input and produces complex-valued modulation symbols as
output.
<img width="938" height="165" alt="image" src="https://github.com/user-attachments/assets/d100f01d-4867-48f8-af10-da2d8392f7e0" />
<img width="537" height="574" alt="image" src="https://github.com/user-attachments/assets/d40729a3-46c5-4636-b43b-cb33ba702f65" />
### 5.3.1 Baseband Signal for All Channels Except PRACH and RIM-RS
#### Time-domain Baseband Signal Definition
- The time-domain signal on antenna port p, symbol index l, and subcarrier spacing configuration mu is:
   - s_l^(mu)(t, p) = s̄_l^(mu)(t, p), if t_start,l^(mu) ≤ t < t_start,l^(mu) + T_symb,l^(mu)
   - s_l^(mu)(t, p) = 0, otherwise
- OFDM Signal Formula
   - s̄_l^(mu)(t, p) = sum from k = k_min to k_max of [a_k,l^(mu)(p) * exp(j * 2 * pi * k * Delta_f * (t - T_CP,l^(mu)))]
- Symbol Start Time
  - t_start,l^(mu) =0, if l = 0
  - t_start,l-1^(mu) + T_symb,l-1^(mu), if l > 0
- Cyclic Prefix Extension (First Symbol)
  - For PUSCH, SRS, or PUCCH with extended CP:
     - s_ext,0^(mu)(t, p) = s̄_0^(mu)(t, p), for t in [t_start,0^(mu) - T_ext, t_start,0^(mu))

### 5.3.2 Baseband Signal Generation for PRACH
- Time-domain PRACH Signal
   - s_RA^(mu)(t, p) = sum from m = 0 to L_RA - 1 of [x_RA(m) * exp(j * 2 * pi * m * Delta_f_RA * t)]
   - x_RA(m): Preamble sample
   - Delta_f_RA: PRACH subcarrier spacing (e.g., 1.25, 5 kHz)
   - L_RA: PRACH sequence length
-  Time and Frequency Mapping for PRACH
   - t_start,RA = 0, if l = 0
   - t_start,l-1^(mu) + T_symb,l-1^(mu), if l > 0
Notes:
   - mu = 0 assumed for 1.25 or 5 kHz PRACH
   - Higher mu used for 15/30/60/120 kHz PRACH
   - Parameters:
     - start_NBWP,i: Lowest RB index of the uplink BWP
     - f_RA_start: PRACH frequency offset
     - n_RA: Frequency-domain occasion index
     - N_RB_RA: Number of RBs allocated for PRACH
     - L_RA: Preamble length
     - T_CP_RA: CP duration for PRACH

| Feature            | All Channels (excl. PRACH/RIM-RS) | PRACH          |
| ------------------ | --------------------------------- | -------------- |
| IFFT-based         | Yes                               | No             |
| Data Type          | Modulated data symbols            | Preamble       |
| CP Handling        | Standard or Extended              | Special rule   |
| Subcarrier spacing | 15 \~ 120 kHz                     | 1.25 \~ 60 kHz |
| Application        | Data/Control channels             | Random Access  |

# 6 Uplink
## 6.3 Physical channels
### 6.3.1 Physical uplink shared channel
#### 6.3.1.1 Scrambling – Physical Uplink Shared Channel (PUSCH)
- Before modulation, the bit sequence in the PUSCH codeword (q = 0) is scrambled to improve the robustness of transmission against interference and to separate UEs.
- Scrambling Logic (Pseudocode)
- Let:
  - b_q(i): i-th input bit of codeword q
  - b̃_q(i): i-th scrambled output bit
  - c_q(i): i-th scrambling sequence bit
```
    i = 0
while i < N_q^bit:
    if x(i) == q:                // UCI placeholder bits
        b̃_q(i) = b_q(i)
    else if y(i) == q:           // UCI placeholder bits
        b̃_q(i) = 1 - b_q(i)
    else:
        b̃_q(i) = (b_q(i) + c_q(i)) mod 2
    i = i + 1
```
- N_q^bit is the number of bits in codeword q
**Scrambling Sequence Initialization**
 ``` 
  c_init = {
   RNTI × 2^15 + RAPID × 2^14 + ID         // if it's msgA on PUSCH
   RNTI × 2^16 + ID                        // otherwise
}
```
| Parameter | Meaning                                                                                    |
| --------- | ------------------------------------------------------------------------------------------ |
| **RNTI**  | Radio Network Temporary Identifier associated with the PUSCH (e.g., C-RNTI, RA-RNTI, etc.) |
| **RAPID** | Random Access Preamble Index (used for msgA triggered by random access)                    |
| **ID**    | Depends on context:                                                                        |
|           | - `msgA-dataScramblingIdentity` (if configured and msgA is used)                           |
|           | - `dataScramblingIdentityPUSCH` (if configured and RNTI is C-RNTI / SP-CSI-RNTI / etc.)    |
|           | - Else: `ID = N_ID_cell`, the physical cell ID                                             |

### 6.3.1 Physical Uplink Shared Channel (PUSCH)
#### 6.3.1.2 Modulation
- Input: Scrambled bit sequence for codeword q = 0
- Output: Modulated complex-valued symbols d_q(0), ..., d_q(N_symb(q) - 1)
| Transform Precoding | Modulation Scheme | Modulation Order (Q<sub>m</sub>) |
| ------------------- | ----------------- | -------------------------------- |
| **Disabled**        | π/2-BPSK          | 1                                |
|                     | QPSK              | 2                                |
|                     | 16QAM             | 4                                |
|                     | 64QAM             | 6                                |
|                     | 256QAM            | 8                                |
| **Enabled**         | QPSK              | 2                                |
|                     | 16QAM             | 4                                |
|                     | 64QAM             | 6                                |
|                     | 256QAM            | 8                                |

#### 6.3.1.3 Layer Mapping
- Input: Complex-valued modulation symbols d_q(0)...d_q(N_symb(q)-1)
- Output: Symbols mapped onto up to 4 transmission layers
- Each symbol is assigned to layer λ, where:
  - λ ∈ {0, 1, ..., ν−1} with ν being the number of layers
- Symbols are mapped to:
  - x_λ = [x_λ(0), ..., x_λ(N_symb_layer - 1)]^T
- (Mapping follows Table 7.3.1.3-1 in TS 38.211.)
#### 6.3.1.4 Transform Precoding (DFT-spread OFDM)
- Condition for Use:
  - Enabled or Disabled according to TS 38.214 Section 6.1.3
  - If enabled, only 1 layer (ν = 1) is allowed.
**Transform Precoding Process:**
- If transform precoding is disabled:
   - y_λ(i) = x_λ(i) for all λ
- If enabled and PT-RS is NOT used:
   - The input x_0(i) is divided per OFDM symbol (based on number of subcarriers).
- Transform precoding is applied directly.
- If PT-RS IS used:
   - Symbols are divided into sets by OFDM symbol index l
   - Set l contains:
      - Nsc_PUSCH - ε_l * Msamp_group
      - ε_l = 1 if the symbol contains PT-RS samples
      - Msamp_group: number of PT-RS samples per group
   - The remaining (non-PT-RS) symbols are input to the DFT.
**DFT-based Precoding Equation**
- The DFT (transform precoding) applied to each set:
  ```
  y_0(l ⋅ Nsc_PUSCH + k) = 
      ∑_{i=0}^{Nsc_PUSCH - 1} x_0(l ⋅ Nsc_PUSCH + i) ⋅ 
      exp(-j ⋅ 2π ⋅ i ⋅ k / Nsc_PUSCH)
  ```
- Result: ỹ_0(0), ..., ỹ_0(N_symb_layer - 1)
- Nsc_PUSCH = N_RB_PUSCH × 12 (Number of subcarriers = number of RBs × 12)
- N_RB_PUSCH: number of allocated resource blocks for PUSCH
