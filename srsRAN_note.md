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

