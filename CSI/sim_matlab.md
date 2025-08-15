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

- Initialize the channel estimation matrix H_est.
- For each subcarrier, use the LS (Least Squares) method to estimate the channel: X is the transmitted CSI-RS, Y is the received signal, and the generalized inverse pinv is used to calculate H = X† Y.
- Background: Channel estimation is the core of CSI feedback, measured using CSI-RS (TS 38.214 Section 5.2.2.3).

<img width="512" height="450" alt="image" src="https://github.com/user-attachments/assets/591be3be-debc-4e8c-8775-da0ecaf346cf" />

- Channel estimates are averaged to calculate the overall CSI.
- RI: Channel rank (number of significant singular values) is calculated using SVD.
- PMI: Precoding matrix is extracted from the V matrix obtained by SVD.
- CQI: Calculated based on SINR, with one value per layer, mapped to the range [0, 15].

## type 1 codebook function
```
function [W] = type1_codebook(nTxAnts, ri, i1, i2, N1, N2, O1, O2)
    isDualPolarized = (nTxAnts == 2*N1*N2);
    nPortsBase = N1 * N2; 
    
    if nTxAnts ~= N1*N2 && ~isDualPolarized
        % warning('nTxAnts (%d) does not match N1*N2 (%d) or 2*N1*N2 (%d).', nTxAnts, N1*N2, 2*N1*N2);
        W = [];
        return;
    end
    
    L = 2; % 每個碼字由兩個 DFT 波束組合而成
    
    % i1 決定了兩個 DFT 波束的選擇
    i1_1 = mod(floor(i1 / (O2 * N2)), O1 * N1); 
    i1_2 = mod(i1, O2 * N2);                    
    
    % *** 核心修正：使用 ' 將向量轉置為直向向量 ***
    u1_1 = exp(-1j * 2 * pi * (0:N1-1)' * i1_1 / (O1 * N1)) / sqrt(N1);
    u2_1 = exp(-1j * 2 * pi * (0:N2-1)' * i1_2 / (O2 * N2)) / sqrt(N2);
    W1_base_1 = kron(u2_1, u1_1); % N1*N2 x 1 向量
    
    % 第二個波束 (簡化為相鄰索引)
    i1_1_2 = mod(i1_1 + 1, O1 * N1);
    u1_2 = exp(-1j * 2 * pi * (0:N1-1)' * i1_1_2 / (O1 * N1)) / sqrt(N1);
    u2_2 = u2_1;
    W1_base_2 = kron(u2_2, u1_2);
    
    % W1: 波束選擇矩陣 (nTxAnts x L)
    W1_base = [W1_base_1, W1_base_2]; % nPortsBase x L (e.g., 4x2)
    
    if isDualPolarized
        W1 = kron(eye(2), W1_base); % 簡化的雙極化結構
    else
        W1 = W1_base;
    end
    
    % W2: 波束組合係數 (L x ri)
    if ri == 1
        phi = exp(1j * pi * i2 / 2); % 修正相位以匹配標準
        W2 = [1; phi] / sqrt(2); % 2x1 矩陣
    elseif ri == 2
        if i2 == 0
            W2 = [1 0; 0 1]/sqrt(2);
        elseif i2 == 1
            W2 = [1 1; 1 -1]/sqrt(4);
        elseif i2 == 2
            W2 = [1 1; 1j -1j]/sqrt(4);
        else % i2 == 3
            W2 = []; % ri=2, i2=3 in single-panel is reserved
        end
    else
        W2 = [];
        % error('RI > 2 not implemented for this codebook type');
    end

    if isempty(W2)
        W = [];
        return;
    end
    
    % 最終預編碼矩陣
    W = W1 * W2;
end
```

- nTxAnts：發射天線總數
- ri：Rank Indicator，表示傳輸層數（1 或 2）
- i1：第一層索引，決定波束方向
- i2：第二層索引，決定波束組合方式
- N1:水平方向的天線數量（單極化時）
- N2：垂直方向的天線數量（單極化時）
- O1：水平方向 oversampling 因子
- O2：垂直方向 oversampling 因子
  
```
i1 → [DFT 水平索引, DFT 垂直索引] → 生成 W1_base_1
                 | 
                 |  (水平索引+1)
                 ↓
           生成 W1_base_2
                 
W1_base_1 & W1_base_2 → W1 (依雙/單極化)
RI, i2 → 選擇 W2（組合係數）
W1 × W2 → 最終預編碼矩陣 W
```
