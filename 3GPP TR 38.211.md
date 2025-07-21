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

#### 6.3.1.5  Precoding 
- Precoding is applied to the modulated symbols before transmission across multiple antenna ports.
- It transforms a block of modulation symbols for each layer into a new set of symbols for each antenna port using a precoding matrix W.
- Let:
  - M_symb_layer be the number of modulation symbols per layer
  - M_symb_ap be the number of modulation symbols per antenna port
  - rho_p be the number of antenna ports
  - W be the precoding matrix
```    
[z_0(i), ..., z_(rho_p - 1)(i)]^T = W * [y_0(i), ..., y_(number_of_layers - 1)(i)]^T
```
  -  y_l(i) is the symbol for layer l at symbol index i
  -  z_p(i) is the precoded output symbol for antenna port p
##### Precoding Matrix Tables
- Precoding matrices depend on:
  - Number of layers
  - Number of antenna ports
  - Transform precoding status (enabled/disabled)
  - TPMI index
- Single-layer transmission
  - 2 antenna ports: Table 6.3.1.5-1
  - 4 antenna ports:
    - Transform precoding enabled: Table 6.3.1.5-2
    - Transform precoding disabled: Table 6.3.1.5-3
- Two-layer transmission
  - 2 antenna ports: Table 6.3.1.5-4
  - 4 antenna ports (no transform precoding): Table 6.3.1.5-5
- Three-layer transmission
  - 4 antenna ports (no transform precoding): Table 6.3.1.5-6
- Four-layer transmission
  - 4 antenna ports (no transform precoding): Table 6.3.1.5-7

##### Precoding Matrix Selection
- Matrix W  is selected based on:
  - Number of antenna ports
  - Number of layers
  - TPMI index
  - Transform precoding enable/disable
  - Configurations in TS 38.214

### 6.3.2 Physical Uplink Control Channel (PUCCH)
#### 6.3.2.1 General Description
- The PUCCH supports multiple formats as listed in the table below.

| Format | Number of OFDM Symbols (Nsymb\_PUCCH) | Number of Bits |
| ------ | ------------------------------------- | -------------- |
| 0      | 1 to 2                                | ≤ 2            |
| 1      | 4 to 14                               | ≤ 2            |
| 2      | 1 to 2                                | > 2            |
| 3      | 4 to 14                               | > 2            |
| 4      | 4 to 14                               | > 2            |

#### 6.3.2.2 Sequence and Cyclic Shift Hopping
- Formats using sequence hopping:
  - Formats 0, 1, 3, and 4.
  - Sequences follow clause 5.2.2.
  - Group and sequence hopping are determined by higher-layer parameter ``` pucch-GroupHopping.```

##### 6.3.2.2.1 Group and Sequence Hopping
- Let n_ID be:
  - If hoppingId is configured: n_ID = hoppingId
  - Otherwise: n_ID = physicalCellId
- Let v be the sequence group number.
- Group Hopping Modes:
  - GroupHopping = 'neither'
    - v = 0
    - Group index = 0 mod 30
  - GroupHopping = 'enable'
    - v = (2^7 * c(0) + 2^6 * c(1) + ... + c(6)) mod 30
    - Sequence c(i) is pseudo-random, defined in clause 5.2.1
    - Initialized at the beginning of each radio frame using n_ID mod 30
  - GroupHopping = 'disable'
    - v = (2 * f + c(f + m0)) mod 30
    - Sequence c(i) is the same as above
    - Initialization: init = floor(n_ID / 30) + (n_ID mod 30)
- Frequency Hopping Index:
  - hop = 0 if intra-slot frequency hopping is disabled
  - hop = 0 for first hop, hop = 1 for second hop if enabled

##### 6.3.2.2.2 Cyclic Shift Hopping
The cyclic shift value n_cs is calculated per symbol and slot.

```
n_cs = (2 * m0 + cs + interlace_offset + c_cs(sym, slot)) mod N_sc_RB
```
- slot is the slot number
- sym is the OFDM symbol index (starting at 0)
- sym' is the index of the first OFDM symbol in the slot (defined in TS 38.213)
- m_cs = 0, except for Format 0 (depends on the content being transmitted)
- interlace_offset:
  - = 5 * IRB if interlaced mapping is enabled by higher-layer parameters
  - = 0 otherwise
- IRB is the interlace Resource Block index
- Function c_cs(sym, slot):
  
```
c_cs(sym, slot) = sum over k=0 to sym of [2^k * c(k + 8 * slot)]
```
#### 6.3.2.3 PUCCH Format 0
##### 6.3.2.3.1 Sequence Generation
- The sequence x(n) is generated as follows:
  - For single-symbol PUCCH: only one symbol is generated.
  - For double-symbol PUCCH: a two-symbol sequence is generated.
    
 ```
  x(n) = r_alpha(l, n_RBsc * n + delta_alpha)
 ```
  - r_alpha(...) is the sequence defined in clause 6.3.2.2
  - n = 0 to N_sc_RB - 1 (N_sc_RB = 12 subcarriers)
  - l is the OFDM symbol index
  - m_cs is determined by clause 9.2 in TS 38.213, depending on UCI content
##### 6.3.2.3.2 Mapping to Physical Resources
- The sequence x(n) is:
  - Scaled by a power factor β_PUCCH,0 (for transmit power normalization)
  - Mapped to REs (Resource Elements) starting from x(0)
- Mapping rules:
  - Order: increasing subcarrier index k → then OFDM symbol index l on antenna port p = 2000
  - For interlaced transmission, the mapping is done per resource block (RB) in the interlace and bandwidth part.

#### 6.3.2.4 PUCCH Format 1
##### 6.3.2.4.1 Sequence Modulation
- The UCI bits b(0)...b(Mbit-1) are modulated:
  - BPSK for 1 bit
  - QPSK for 2 bits
  - Result: one complex-valued symbol d(0)
```    
y(n) = d(0) * r_alpha(l, n)
```
  - r_alpha(...) is from clause 6.3.2.2
  - Length: N_sc_RB (12 subcarriers)
**Blockwise Spreading with Orthogonal Sequences**
- Spreading is applied depending on slot configuration and hopping.
- Resulting spread sequence z(n) is:
```
z(n + m′ * N_sc_RB) = y(n) * w_m(i)
```
  - w_m(i) is an orthogonal sequence (from Table 6.3.2.4.1-2)
  - i is determined from higher-layer parameters (TS 38.213 clause 9.2.1)
  - m′ is 0 or 1 depending on whether intra-slot frequency hopping is used
  - The total number of sequences (PUCCH_1_N_SF,m') depends on PUCCH symbol length

| PUCCH Length | No hopping (m′ = 0) | Intra-slot hopping (m′ = 0, 1) |
| ------------ | ------------------- | ------------------------------ |
| 4            | 2                   | 1, 1                           |
| 5            | 2                   | 1, 1                           |
| 6            | 3                   | 1, 2                           |
| 7            | 3                   | 1, 2                           |
| 8            | 4                   | 2, 2                           |
| 9            | 4                   | 2, 2                           |
| 10           | 5                   | 2, 3                           |
| 11           | 5                   | 2, 3                           |
| 12           | 6                   | 3, 3                           |
| 13           | 6                   | 3, 3                           |
| 14           | 7                   | 3, 4                           |

##### 6.3.2.4.2 Mapping to Physical Resources
- The final spread sequence z(n) is:
  - Scaled by β_PUCCH,1
  - Mapped to REs not used by associated DM-RS
- Mapping rule:
  - First by subcarrier index k in the assigned RB
  - Then by OFDM symbol index l on antenna port p = 2000
- For interlaced transmission, mapping is repeated per RB in interlace region.
- Sequence r_alpha(...) remains RB-dependent (as in clause 6.3.2.2)

#### 6.3.2.5 PUCCH format 2 
- Scrambling (§6.3.2.5.1):
  - Scrambling with scrambled_bit = bit XOR c(i).
  - Init = RNTI * 2^16 + nID (dataScramblingIdentityPUSCH or cell ID).
- Modulation (§6.3.2.5.2):
  - QPSK modulation.
- Spreading (§6.3.2.5.2A):
  - Optional spreading if OCC-Length-r16 is configured.
  - Spreading factor ∈ {2,4}, orthogonal sequence depends on OCC-Index-r16.
- Mapping (§6.3.2.5.3):
  - Multiply with β<sub>PUCCH,2</sub>.
  - Map to RBs assigned and not used by DMRS.

#### 6.3.2.6 PUCCH Formats 3 and 4
- Support more bits than format 2; difference: format 4 uses intra/inter-slot frequency hopping with transform precoding.
- Scrambling (§6.3.2.6.1):
  - Same method as format 2.
- Modulation (§6.3.2.6.2):
  - QPSK (bit/2 symbols) or π/2-BPSK (1 bit = 1 symbol).
- Block-wise Spreading (§6.3.2.6.3):
  - Required for interlaced format 3 and all format 4.
  - Spreading factors from {2,4}, using orthogonal sequences (Tables 6.3.2.6.3-1 and 6.3.2.6.3-2).
- Transform Precoding (§6.3.2.6.4):
  - Applied after spreading to create a DFT-precoded symbol block.
- Mapping (§6.3.2.6.5):
  - Multiply with β<sub>PUCCH,s</sub>.
  - Map to assigned RBs (exclude DMRS).
- Intra-slot frequency hopping: split symbols evenly between hops.

### 6.3.3 Physical random-access channel
#### 6.3.3.1 PRACH Sequence Generation
**Zadoff-Chu Sequence Generation**
- Time-domain base sequence:
```
x_u,v(n) = x_u((n + Cv + L_RA * i) mod L_RA)
```
  - x_u(n): Zadoff-Chu sequence of length L_RA with root index u
  - Cv: cyclic shift
  - i: repetition index (if needed)
- Frequency-domain representation:
  - Y_u,v(n) = sum over m = 0 to L_RA - 1 of [x_u,v(m) * exp(-j * 2 * pi * m * n / L_RA)]
 
| Format       | L\_RA | Number of Root Sequences | Max Preambles Per Root |
| ------------ | ----- | ------------------------ | ---------------------- |
| Format 0–3   | 839   | 1151                     | Depends on N\_CS       |
| Format B4–B5 | 139   | 571                      | Depends on N\_CS       |

**Cyclic Shift Calculation (Cv)**
- Cv depends on the restricted set type:
- Unrestricted Set
  - Simple and direct:
    - Cv = v * N_CS , where v = 0, 1, ..., floor(L_RA / N_CS) - 1
- Restricted Set Type A
  - Used for longer sequences (L_RA = 839)
  - If d_u < 2: No shift allowed
  - If 3 <= d_u <= L_RA - 1:
    - Compute various shift-related parameters:
  - N_CS_group, N_shift, N_start, etc.
  - Use table-defined formulas for Cv
- Depends on the range of d_u (split into subcases)
- Restricted Set Type B
  - Used for shorter sequences (L_RA = 139)
  - If d_u < 2: No shift allowed
  - Otherwise:
    - Multiple d_u ranges each with their own formulas
    - Includes special handling when d_u >= 72
**Key Parameters**
- L_RA: Length of ZC sequence (839 or 139 depending on preamble format)
- N_CS: Cyclic shift granularity (determines how many shifts can be made)
- u: Root sequence index (logical root)
- v: Preamble index (0 to 63)
- d_u: Derived from L_RA and root index u
- restrictedSetConfig: Tells whether unrestricted, type A, or type B is used
- Tables 6.3.3.1-1 to 6.3.3.1-7 define all needed constants (like N_CS values)

#### 6.3.3.2 Mapping to Physical Resources
- After generating the PRACH preamble sequence, it is mapped to physical resource elements for transmission.
- The sequence y_{u,v}(k) is scaled and mapped as:
```
a_p(k) = β_PRACH × y_{u,v}(k)
```
- β_PRACH: Amplitude scaling factor (for transmit power control as defined in TS 38.213).
- p = 4000: Fixed antenna port for PRACH.
- Mapping follows the baseband signal generation procedure in clause 5.3.
**Frequency-Domain Mapping**
- Index k used for mapping depends on the PRACH format.
- The mapping location in frequency depends on:
- The configured PRACH frequency domain occasion (FDO)
- The starting frequency index
- The msg1-FDM parameter (if used)
**Time-Domain Resource Mapping**
- PRACH preambles are only transmitted in specific time resources defined by:
  - Tables 6.3.3.2-2 to 6.3.3.2-4, depending on:
     - Frequency Range (FR1 or FR2)
     - Spectrum type (paired or unpaired)
     - Subcarrier spacing (SCS)
- Time-domain resources are determined by the PRACH configuration index, which is configured via one of:
  - prach-ConfigurationIndex
  - prach-ConfigurationIndexNew
  - msgA-PRACH-ConfigurationIndex-r16
**Subcarrier Spacing Assumptions for PRACH**
- FR1: 15 kHz SCS
- FR2: 60 kHz SCS
- Used for interpreting PRACH configuration tables (slot numbering).

**Timing Assumptions for Handover**
- For handover in paired/unpaired spectrum (with N_frame_max = 4):
   - UE can assume max time offset < 153600 × Ts (Ts: base time unit)
- For inter-frequency handover to a cell in unpaired spectrum (N_frame_max = 8):
   - UE may assume max time offset < 7680 × Ts

##  6.4 Physical signals
### 6.4.1 Reference signals 
#### 6.4.1.3 PUCCH Demodulation Reference Signal (DM-RS)
**PUCCH Format 1**
- Sequence Generation:
- The DM-RS sequence for PUCCH format 1 is defined as:

```
r̄ₘₙ = z_{m'N_SF,PUCCH1}(n) · wₘ^{(i)}  for n = 0,...,N_RB^PUCCH,1 · N_sc^RB - 1
```
  - z_{m'N_SF,PUCCH1}(n) is defined in Clause 5.2.2 (Zadoff-Chu-like sequence).
  - m' is the DM-RS symbol index defined by PUCCH length and hopping setting (Table 6.4.1.3.1.1-1).
  - wₘ^{(i)} is the orthogonal cover code sequence from Table 6.3.2.4.1-2.

- Resource Mapping:
  - The sequence is scaled by β_PUCCH,1.
  - Mapped to antenna port 2000 across assigned RBs.
**PUCCH Format 2**
- Sequence Generation:
- QPSK symbols based on pseudo-random sequence c(i):
```
r̄ₙ = 1/√2 · [1 - 2·c(2n)] + j·1/√2 · [1 - 2·c(2n+1)]
```
  - c(i) is defined in Clause 5.2.
  - Initialization of the PRBS depends on scramblingID0 or n_ID_cell.
- Resource Mapping:
  - Multiplied by β_PUCCH,2 and mapped to antenna port 2000.
  - Subcarrier index k is relative to CRB0 (common resource block 0).

**PUCCH Formats 3 and 4**
- Sequence Generation:
  - Based on r̄ₘ(n) = r_{l_uv}^{(α, δ)}(n), where the base sequence comes from:
    - Clause 5.2.3 if transform precoding is used and π/2-BPSK applies.
    -Clause 6.3.2.2 otherwise.
- Cyclic shift varies with:
  - Symbol index
  - Intra-slot frequency hopping (from Table 6.4.1.3.3.1-1)
- Resource Mapping:
  - The sequence is scaled by β_PUCCH,s (s ∈ {3,4}).
  - Mapped to port 2000, k is relative to lowest-numbered PUCCH RB.
- Symbol positions depend on hopping and additional DM-RS (Table 6.4.1.3.3.2-1).

#### 6.4.1.4 SRS: Sounding Reference Signal
- An SRS resource includes:
- 1, 2, or 4 antenna ports (p_i = 1000 + i for codebook-based; otherwise follows TS 38.214)
- 1, 2, 4, 8, or 12 OFDM symbols (given by nrofSymbols)
- Time-domain position l₀ = N_symb,slot - 1 - offset, where offset ∈ [0,13]
- Frequency-domain starting position k₀
- The sequence is defined as:
```
r_i(n, l') = r_{u,v}^{(α_i)}(n) for each symbol l' ∈ [0, N_symb,SRS - 1]
```
- Cyclic shift α_i is given by:
```
α_i = 2π · (n_cs_i mod N_cs_max) / N_cs_max
```
- where N_cs_max depends on transmissionComb (see Table 6.4.1.4.2-1).
- Sequence hopping controlled by groupOrSequenceHopping:
  - "neither": v = 0
  - "groupHopping": hopping group changes per slot based on PRBS
  - "sequenceHopping": hopping applied if certain conditions are met

**Resource Mapping**
- Sequence is scaled by β_SRS.
- Mapped to k and l positions for each antenna port.
- Mapping formula:
```
a_p(k, l') = β_SRS · r_i(n, l')
```
- Total subcarriers:
```
N_sc,SRS = m_SRS,b · N_sc^RB / comb
```
- comb = {2, 4, 8}
- m_SRS,b and N_b are taken from Table 6.4.1.4.3-1
- b_SRS is selected via higher-layer parameter freqHopping (or default = 0)

**SRS Slot Configuration**
- For periodic or semi-persistent SRS:
- Periodicity T_SRS and offset Toffset configured by periodicityAndOffset-p or -sp
- Slot selection condition:
```
(slot_index - Toffset) mod T_SRS == 0
```

# 7. Downlink
### 7.1.1 Physical Channels
**Downlink physical channels carry higher layer information and include:**
- Physical Downlink Shared Channel (PDSCH)
- Physical Broadcast Channel (PBCH)
- Physical Downlink Control Channel (PDCCH)
### 7.1.2 Physical Signals
- Downlink physical signals are used by the physical layer but do not carry higher-layer information:
  - Demodulation Reference Signal (DM-RS)
  - Phase Tracking Reference Signal (PT-RS)
  - Positioning Reference Signal (PRS)
  - Channel State Information Reference Signal (CSI-RS)
  - Primary Synchronization Signal (PSS)
  - Secondary Synchronization Signal (SSS)
## 7.2 Physical Resources
- Downlink antenna ports:
  - Ports starting with 1000: PDSCH
  - Ports starting with 2000: PDCCH
  - Ports starting with 3000: CSI-RS
  - Ports starting with 4000: SS/PBCH block
  - Ports starting with 5000: PRS
- UE must not assume two antenna ports are quasi co-located (QCL) unless explicitly specified.

## 7.3 Physical Channels
### 7.3.1 PDSCH (Physical Downlink Shared Channel)
#### 7.3.1.1 Scrambling
- Up to two codewords (q ∈ {0,1}) may be transmitted.
- For each codeword, bits are scrambled before modulation:
```
b_scrambled(q)(j) = b(q)(j) + c(q)(j) mod 2
```
- The scrambling sequence c(q)(j) is initialized using:
```
c_init = RNTI * 2^15 + q * 2^14 + nID
```
#### 7.3.1.2 Modulation
Scrambled bits are modulated using one of the following schemes:

| Modulation | Order (Qm) |
| ---------- | ---------- |
| QPSK       | 2          |
| 16QAM      | 4          |
| 64QAM      | 6          |
| 256QAM     | 8          |

#### 7.3.1.3 Layer Mapping
- Scrambled modulation symbols are mapped to transmission layers based on the number of codewords and layers.
- Mapping follows specific rules (see Table 7.3.1.3-1 in the spec) depending on:
  - Number of layers (1–8)
- Whether single or dual codewords are transmitted

#### 7.3.1.4 Antenna Port Mapping
- Each layer is mapped to a corresponding antenna port (e.g., ports 1000+ for PDSCH).
- The number of antenna ports used equals the number of layers.

#### 7.3.1.5 Mapping to Virtual Resource Blocks
- The mapping of modulation symbols to Resource Elements (REs) follows these rules:
  - Only mapped to REs in assigned virtual resource blocks (VRBs).
  - REs must not:
     - Overlap with associated DM-RS or other UEs' DM-RS
     - Be used by non-zero-power CSI-RS (with some exceptions)
     - Be used by PT-RS
     - Be declared unavailable for PDSCH (per TS 38.214 clause 5.1.4)
  - Symbols are mapped in increasing order by:
     - Subcarrier index k' within VRBs
     - Symbol index l

