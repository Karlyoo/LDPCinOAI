# CSI (channel state information)
## TS 38.214 
### 5.2 2 UE procedure for reporting channel state information (CSI)
#### 5.2.1.1 Reporting Settings
- 每個 CSI-ReportConfig 包含：
- 對應一個 DL BWP（由 BWP-Id 指定，設定一個 CSI reporting band
- 配置包含：
   - Codebook (Type I/II/enhanced) + subset restriction
   - 時域行為（`reportConfigType`）：`aperiodic`, `semiPersistentOnPUCCH`, `semiPersistentOnPUSCH`, `periodic`
   - 頻域粒度（`reportFreqConfiguration`）
   - 限制測量時間（timeRestrictionForChannel/Interference）
   - reportQuantity：報哪些項目（例如 CQI/PMI/RI）

#### 5.2.1.2 Resource Settings（CSI 資源設定）
- 每個 CSI-ResourceConfig 包含：
- 一組以上（S ≥ 1）的 resource sets：
  - NZP CSI-RS、CSI-IM、SS/PBCH block set
  - 所有資源屬於相同 DL BWP（BWP-Id）

- 時域行為（resourceType）：
  - aperiodic, periodic, semi-persistent
     - periodic 與 semi-persistent 限定 S = 1
**一致性原則**
- 若有多個設定包含同一個 NZP CSI-RS 或 CSI-IM 資源 ID，這些設定必須有相同的 resourceType
  
#### 5.2.1.4 Reporting Configurations（CSI 報告觸發與條件）
| CSI-RS 類型          | Periodic | Semi-Persistent        | Aperiodic (觸發型)         |
| ------------------ | -------- | ---------------------- | ----------------------- |
| Periodic CSI-RS    | o 無需觸發   | o DCI 觸發 (PUCCH/PUSCH) | o DCI 觸發 + subselection |
| Semi-Persistent RS | x        | o 同上                   | o 同上                    |
| Aperiodic CSI-RS   | x        | x                      | o 僅支援此類型                |

##### Periodicity & Slot Offset
- 使用 reportSlotConfig 設定 periodicity 和 offset（依 BWP numerology）
- Slot offset 選擇依據：
  - 若由 DCI format 0_2 觸發 → `reportSlotOffsetListForDCI-Format0-2`
  - 若由 DCI format 0_1 觸發 → `reportSlotOffsetListForDCI-Format0-1`
  - 否則 → `reportSlotOffsetList`
##### 頻域粒度設定（reportFreqConfiguration）
- 子頻帶大小（根據 PRB 數量）：
 
| PRBs in BWP | 可選 Subband Size |
| ----------- | --------------- |
| 24–72       | 4, 8            |
| 73–144      | 8, 16           |
| 145–275     | 16, 32          |

##### QCL (Quasi Co-Location) 假設
- NZP CSI-RS for channel 與 CSI-IM/NZP for interference 是 QCL-TypeD 同源
- 若有兩組 Resource Setting：一為 channel，另一為 interference，需逐一對應
##### CRI 報告條件
- repetition=off 時：需回報 CRI（由 TS 38.212 §6.3.1.1.2 定義）
- repetition=on 時：不回報 CRI
- 若使用 Type II codebook → 不支援 CRI 回報





