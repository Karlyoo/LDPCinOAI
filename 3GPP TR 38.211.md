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

