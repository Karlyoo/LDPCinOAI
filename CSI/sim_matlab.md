# Simulate CSI-RS feedback using MATLAB
<img width="528" height="355" alt="image" src="https://github.com/user-attachments/assets/62888735-3a45-4eae-8351-55130cf8f8df" />

- Set basic simulation parameters, including the carrier structure (based on 5G NR OFDM parameters, such as 15 kHz subcarrier spacing and 52 PRBs), MIMO antenna configuration (4x4 MIMO), and SNR.
- Define channel model parameters (such as delay spread and Doppler shift) to simulate multipath fading channels.
- CSI-related parameters define the number of CSI-RS ports and density (a density of 1 indicates one RE per PRB is used for CSI-RS).
  
<img width="856" height="287" alt="image" src="https://github.com/user-attachments/assets/df440601-4925-4aba-ad84-2c8870de173f" />

- Create a 3D resource grid (resourceGrid) with dimensions equal to the number of subcarriers x the number of symbols x the number of transmit antennas to place the CSI-RS signals.
- Generate CSI-RS location indices (csiRsIndices). In this example, place one resource element (RE) for every four subcarriers and symbols, meeting the density parameter.
- Generate random QPSK symbols as the CSI-RS signals and fill them into specific locations in the resource grid.
- Output: Resource Grid resourceGrid contains the CSI-RS signal.
<img width="762" height="402" alt="image" src="https://github.com/user-attachments/assets/34735fa3-1d7d-4522-a404-c8471ce308e8" />

<img width="588" height="224" alt="image" src="https://github.com/user-attachments/assets/46cc6661-873e-4202-a371-46259297594b" />

<img width="811" height="312" alt="image" src="https://github.com/user-attachments/assets/8ab9e44c-9079-45bf-8a4e-57aa989331ee" />

<img width="512" height="450" alt="image" src="https://github.com/user-attachments/assets/591be3be-debc-4e8c-8775-da0ecaf346cf" />

