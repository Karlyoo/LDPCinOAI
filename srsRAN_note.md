## Architecture
srsRAN_Project/<br>
├── apps/ <br>
│   ├── gnb/       ← CU+DU  <br>
│   ├── cu/        ← 控制面 CU app   <br>
│   └── du/        ← 使用面 DU app   <br>
├── configs/       ← YAML 配置範例（頻段、天線、split 類型）<br>
├── lib/           ← 共用函式庫 (實作 PHY/MAC/RLC/L1~L3) <br>
├── utils/         ← 工具程式（如 trx_srsran, 介接 Amarisoft）<br>
└── CMakeLists.txt build scripts...<br>
| 資料夾 / 檔案                    | 功能描述                                            | 備註                 |
| --------------------------- | ----------------------------------------------- | ------------------ |
| `apps/gnb/`                 | gNB 應用，整合 CU + DU。適合簡易測試與模擬環境使用。            | 主程式入口（main）     |
| `apps/cu/`                  | control unit（CU）應用。可進一步拆為 CU‑CP（控制平面）與 CU‑UP（使用者平面）。     | 適用於分離部署            |
| `apps/du/`                  | deploy unit（DU）應用。處理 PHY、MAC、RLC，與 RU 溝通（如 Split‑7.2）。  | 與 RU 互動            |
| `configs/`                  | 儲存 YAML 配置檔，設定頻段、MIMO、split 模式、test mode 等。     | 有多組範例可參考           |
| `lib/`                      | 核心protocal函式庫（PHY, MAC, RLC, PDCP, RRC 等）。           | 最重要的協定邏輯           |
| `lib/phy/`                  | Layer 1 實作：包含調變/解調、編碼、同步、FFT、LDPC、Polar 等處理。    | 使用 SIMD 加速         |
| `lib/mac/`                  | MAC 層實作：調度、HARQ、BWP 處理、上/下行排程。                  | 支援 F1 interface    |
| `lib/rlc/`, `pdcp/`, `rrc/` | 各層協定的處理，如分段重組、加密、狀態維護、連線管理。                     | 對應 38.322 / 38.323 |
| `lib/interfaces/`           | 定義 CU–DU–RU 間的 interface（如 F1AP、E1AP、NGAP、OFH）。 | 用於模組通信             |
| `utils/`                    | 各種工具程式。包含 TRX driver、模擬 RF、log 解析工具等。           | 如對接 Amarisoft UE   |
| `utils/trx_srsran/`         | 支援模擬 RF 或真實 USRP 裝置的 TX/RX 驅動。                  | Virtual RF 用       |
| `cmake/`                    | CMake 建構模組與選項，控制是否啟用特定元件（如 DPDK、ZMQ）。           | 編譯組態控制             |
| `docker/`                   | Dockerfile 與相關設定。便於容器化部署 gNB / CU / DU。         | 適用於雲端測試            |
| `scripts/`                  | CI/CD、自動部署或測試腳本集合。                              | 如一鍵跑起系統            |
| `tests/`                    | 單元測試與整合測試，包括 PHY pipeline、Split7 模擬等。           | 可用 `ctest` 執行      |
| `docs/`                     | 說明文件、介面定義與開發者指南。                                | 可產出 Doxygen 文件     |
| `CMakeLists.txt`            | 頂層建構說明，定義所有模組與相依關係。                             | 建構入口點              |
| `.gitlab-ci.yml`            | GitLab CI/CD 自動建構與測試流程設定。                       | 開發自動化用途            |

Input: MAC PDU (bytes)                                      
        ↓
┌────────────────────────────────────────┐
│ CRC 附加 (crc_byte.c)                    │    
│ Output: bits + CRC                      │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ HARQ 處理（儲存/重傳控制）               │
│ Output: 處理後的 TB 位元                 │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ 分段處理 (Segmentation, nr_segmentation.c) │
│ Output: CB 區塊（Code Blocks）           │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ LDPC 編碼（nr_dlsch_coding.c + coding/） │
│ Output: encoded bits                   │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ Rate Matching + Interleaving（38.212） │
│ Output: 處理後的位元流                  │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ Scrambling（nr_ulsch_scrambling.c）     │
│ Output: scrambled bits                 │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ 調變（QPSK / 16QAM / 64QAM / 256QAM）    │
│ Output: IQ symbols (複數值)              │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ 子載波映射（subcarrier mapping）         │
│ Output: Resource grid 中的 symbols     │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ 插入 DMRS（uplink 參考訊號）             │
│ Output: 加上 DMRS 的 grid               │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ DFT-s-OFDM 調變（sc-FDMA）               │
│ 包含 DFT, IFFT, CP 加入（ofdm_mod.c）   │
│ Output: 時域 TX 樣本                    │
└────────────┬────────────────────────────┘
             ↓
┌────────────────────────────────────────┐
│ RF 模組（USRP / 模擬通道 / 虛擬 RF）     │
│ Output: 發送至 DU 或 gNB                │
└────────────────────────────────────────┘
