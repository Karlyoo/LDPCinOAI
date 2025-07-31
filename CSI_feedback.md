# CSI (channel state information)
## TS 38.214 
- CSI (Channel State Information): Information a UE (User Equipment, like a smartphone) sends to the base station (gNodeB) about the channel’s quality, such as signal strength or interference. This helps the base station optimize data transmission.
- CSI-RS (Channel State Information Reference Signal): A reference signal transmitted by the base station that the UE measures to estimate channel conditions.
- Types of CSI Reporting:
  - Aperiodic: Triggered on-demand by the base station via DCI (Downlink Control Information).
  - Periodic: Sent at regular intervals.
  - Semi-persistent: Activated/deactivated by the base station and sent periodically while active.
- Resource Settings: Configurations that define which CSI-RS resources are used for channel or interference measurements.
- Trigger States: Predefined configurations that link CSI reporting settings with CSI-RS resources. These are activated by DCI.
### **Simplified Explanation for Beginners**

Imagine you’re using a 5G phone, and the network (base station) needs to know how good or bad the signal is at your location to send data efficiently. This is where **CSI (Channel State Information)** comes in—it’s like a report card your phone sends to the network about the signal quality. To create this report, the network sends a special signal called **CSI-RS**, which your phone measures.

#### **How It Works**:
1. **Aperiodic CSI (On-Demand Reporting)**:
   - The network sends a command (via DCI, a kind of control message) saying, “Hey, measure the signal now and tell me how it is.”
   - This is called **aperiodic** because it happens only when the network asks, not on a schedule.
   - The phone measures the CSI-RS signal and sends a report back. The timing and settings (like which signal to measure) are carefully coordinated to avoid confusion.
   - If the phone is too busy or the signal is in a different frequency band (BWP), it might skip the measurement or report.

2. **Numerology (Subcarrier Spacing)**:
   - **Numerology** is like the rhythm of the signal—how fast or slow the data is sent (e.g., 15 kHz or 30 kHz spacing).
   - If the control message (PDCCH) and the CSI-RS use the same rhythm, the phone follows one set of rules. If they use different rhythms, the phone adjusts the timing to make sure it can process everything correctly.

3. **Semi-Persistent CSI**:
   - Instead of one-time requests, the network can say, “Keep sending reports every so often until I tell you to stop.”
   - This is activated by a special command and stops when the network deactivates it.
   - It’s like setting up a recurring meeting instead of a one-off call.

4. **Processing Limits**:
   - Your phone has a limited “brain” (CPUs) for calculating these reports. If the network asks for too many reports at once, the phone prioritizes the most important ones and skips the rest.
   - Think of it like juggling—you can only handle so many balls at once!

5. **QCL and Beams**:
   - The network uses beams to send signals to your phone, like shining a flashlight in a specific direction.
   - **QCL (Quasi Co-Location)** tells the phone which beam to expect the CSI-RS from, so it knows where to “look.”
   - If the timing is too tight (e.g., not enough time to switch beams), the phone might use a default beam setting instead.
## 5.2  UE procedure for reporting channel state information (CSI)
### 5.2.1 Organized Notes: Channel State Information (CSI) Framework 
- **Purpose**: The CSI framework allows the UE (User Equipment, e.g., a smartphone) to measure channel conditions and report them to the base station (gNodeB) to optimize data transmission.
- **CSI Components**: 
  - **CQI (Channel Quality Indicator)**: Indicates channel quality for data transmission.
  - **PMI (Precoding Matrix Indicator)**: Suggests the best precoding matrix for beamforming.
  - **CRI (CSI-RS Resource Indicator)**: Identifies the preferred CSI-RS resource.
  - **SSBRI (SS/PBCH Block Resource Indicator)**: Identifies the preferred SS/PBCH block.
  - **LI (Layer Indicator)**: Indicates the strongest layer in the precoding matrix.
  - **RI (Rank Indicator)**: Indicates the number of data streams (layers) the channel can support.
  - **L1-RSRP (Layer 1 Reference Signal Received Power)**: Measures signal strength.
  - **L1-SINR (Layer 1 Signal-to-Interference-plus-Noise Ratio)**: Measures signal quality relative to interference and noise.
- **Control by gNodeB**: The base station controls the time and frequency resources the UE uses for CSI reporting.
- **Triggering**: Aperiodic CSI reports are typically triggered by DCI (Downlink Control Information) formats 0_1 or 0_2, with the latter using the parameter `reportTriggerSize-ForDCIFormat0_2`.

#### 5.2.1.1 Reporting Settings 
- Reporting settings define how and what the UE reports about the channel.
1. **Configuration**:
   - Each **CSI-ReportConfig** is linked to a single downlink Bandwidth Part (BWP) via `BWP-Id`.
   - Contains parameters for:
     - **Codebook configuration**: Defines the type of precoding (e.g., Type-I, Type-II, Enhanced Type-II) and restrictions.
     - **Time-domain behavior**: Aperiodic, periodic, semi-persistent on PUCCH, or semi-persistent on PUSCH.
     - **Frequency granularity**: Wideband (entire BWP) or sub-band (specific parts of the BWP).
     - **Measurement restrictions**: Time-domain restrictions for channel or interference measurements.
     - **Reported quantities**: CQI, PMI, CRI, SSBRI, LI, RI, L1-RSRP, or L1-SINR.

2. **Time-Domain Behavior**:
   - **Aperiodic**: Triggered on-demand by DCI.
   - **Periodic**: Reported at fixed intervals on PUCCH.
   - **Semi-persistent**: Activated/deactivated by DCI or MAC CE, reported periodically on PUCCH or PUSCH.
   - For periodic/semi-persistent reporting, periodicity and slot offset are set in the numerology (subcarrier spacing) of the uplink (UL) BWP.

3. **Frequency Granularity**:
   - **Wideband**: Reports cover the entire CSI reporting band.
   - **Sub-band**: Reports cover specific sub-bands within the BWP.
   - Configured via `reportFreqConfiguration`, which specifies:
     - The CSI reporting band (contiguous or non-contiguous sub-bands).
     - Whether CQI/PMI is wideband or sub-band (via `cqi-FormatIndicator` and `pmi-FormatIndicator`).
   - Restrictions:
     - Sub-bands must have sufficient CSI-RS density (ports per PRB).
     - For CSI-IM (Interference Measurement), all PRBs in a sub-band must have CSI-IM resource elements.

4. **Special Case for Shared Spectrum**:
   - In shared spectrum (unlicensed bands), the UE does not average CSI-RS measurements across different downlink transmission bursts to account for dynamic channel access.


#### 5.2.1.2 Resource Settings 
- Resource settings define the CSI-RS or SS/PBCH resources used for channel and interference measurements.
1. **Configuration**:
   - Each **CSI-ResourceConfig** includes a list of CSI Resource Sets (`csi-RS-ResourceSetList`) containing:
     - Non-Zero Power (NZP) CSI-RS sets.
     - SS/PBCH block sets.
     - CSI-IM (Interference Measurement) sets.
   - Linked to a specific DL BWP via `BWP-Id`.
   - All resource settings linked to a CSI report setting must use the same DL BWP.

2. **Time-Domain Behavior**:
   - **Periodic/Semi-persistent**: Limited to one resource set (`S=1`).
   - **Aperiodic**: Can have multiple resource sets.
   - Periodicity and slot offset for periodic/semi-persistent settings are in the numerology of the DL BWP.

3. **Consistency**:
   - If multiple resource settings use the same NZP CSI-RS or CSI-IM resource ID, they must have the same time-domain behavior (aperiodic, periodic, or semi-persistent).
   - All resource settings linked to a CSI report must have the same time-domain behavior.

4. **Measurement Types**:
   - **Channel Measurement**: Uses NZP CSI-RS or SS/PBCH blocks.
   - **Interference Measurement**: Uses CSI-IM or NZP CSI-RS.
   - **QCL (Quasi Co-Location)**: NZP CSI-RS for channel measurement and CSI-IM/NZP CSI-RS for interference measurement are assumed to be QCLed with respect to `QCL-TypeD` (spatial properties).

5. **L1-SINR Measurement**:
   - **One Resource Setting**: NZP CSI-RS (1 port, 3 REs/RB) for both channel and interference measurement.
   - **Two Resource Settings**:
     - First: Channel measurement on SSB or NZP CSI-RS.
     - Second: Interference measurement on CSI-IM or NZP CSI-RS (1 port, 3 REs/RB).
     - Each channel resource is paired with an interference resource.
   - **Three Resource Settings** (optional):
     - First: Channel measurement on SSB or NZP CSI-RS.
     - Second: Interference measurement on CSI-IM.
     - Third: Interference measurement on NZP CSI-RS (1 port, 3 REs/RB).
   - QCL: The SSB or QCL-TypeD RS for channel measurement determines the QCL assumption for interference measurement resources.
   - NZP CSI-RS resource sets for channel and interference measurement may have the `repetition` parameter set to ensure consistent beam assumptions.

#### 5.2.1.4 Reporting Configurations 
- Defines how CSI parameters are calculated and reported, including dependencies and supported combinations.
1. **Parameter Dependencies**:
   - **LI**: Calculated based on reported CQI, PMI, RI, and CRI.
   - **CQI**: Calculated based on reported PMI, RI, and CRI.
   - **PMI**: Calculated based on reported RI and CRI.
   - **RI**: Calculated based on reported CRI.

2. **Supported Configurations** (Table 5.2.1.4-1):
   - **Periodic CSI-RS**:
     - Periodic reporting: No dynamic triggering, configured by higher layers.
     - Semi-persistent reporting: Activated by MAC CE (PUCCH) or DCI (PUSCH).
     - Aperiodic reporting: Triggered by DCI, with possible subselection.
   - **Semi-persistent CSI-RS**:
     - No periodic reporting.
     - Semi-persistent reporting: Activated by MAC CE (PUCCH) or DCI (PUSCH).
     - Aperiodic reporting: Triggered by DCI.
   - **Aperiodic CSI-RS**:
     - Only supports aperiodic reporting, triggered by DCI.

3. **CRI Reporting**:
   - When `repetition` is `off` in an NZP CSI-RS resource set, the UE reports a CRI to select the best resource.
   - When `repetition` is `on`, CRI is not reported (beam repetition assumed).
   - CRI reporting is not supported for Type-II or Type-II Port Selection codebooks.

4. **Periodicity for Periodic/Semi-persistent Reporting**:
   - **PUCCH**: Periodicity (`T_CSI`) and slot offset (`offset`) are configured by `reportSlotConfig`. Reports are transmitted in slots satisfying:
     \[
     \left( N_{\text{slot}}^{\text{frame},\mu} \cdot n_f + n_{s,f} - \text{offset} \right) \mod T_{\text{CSI}} = 0
     \]
   - **PUSCH (Semi-persistent)**: Similar formula, but relative to the initial PUSCH transmission slot.

5. **Slot Offsets for Aperiodic/Semi-persistent PUSCH**:
   - Configured by `reportSlotOffsetList`, `reportSlotOffsetListForDCI-Format0-1`, or `reportSlotOffsetListForDCI-Format0-2`, selected by the triggering DCI.

6. **Subband Sizes** (Table 5.2.1.4-2):
   - Depend on BWP size:
     - 24–72 PRBs: Subband size = 4 or 8 PRBs.
     - 73–144 PRBs: Subband size = 8 or 16 PRBs.
     - 145–275 PRBs: Subband size = 16 or 32 PRBs.
   - First and last subband sizes may differ due to non-integer division of PRBs.

7. **Frequency Granularity**:
   - **Wideband**: Applies to entire reporting band.
     - Used for `cri-RI-PMI-CQI`, `cri-RI-LI-PMI-CQI`, `cri-RI-i1`, `cri-RI-CQI`, `cri-RI-i1-CQI` (with `widebandCQI`), `cri-RSRP`, `ssb-Index-RSRP`, `cri-SINR`, or `ssb-Index-SINR`.
   - **Sub-band**: Reports per sub-band.
   - BWPs < 24 PRBs must use wideband granularity with Type-I Single Panel codebook.

##### 5.2.1.4.1 Resource Setting Configurations 
- Specifies how resource settings are linked to CSI reports.
1. **Aperiodic CSI**:
   - **One Resource Setting**: For channel measurement (L1-RSRP or L1-SINR).
   - **Two Resource Settings**: First for channel measurement, second for interference measurement (CSI-IM or NZP CSI-RS).
   - **Three Resource Settings**: First for channel measurement, second for CSI-IM interference, third for NZP CSI-RS interference.

2. **Periodic/Semi-persistent CSI**:
   - **One Resource Setting**: For channel measurement (L1-RSRP or L1-SINR).
   - **Two Resource Settings**: First for channel measurement, second for interference measurement (CSI-IM or NZP CSI-RS).

3. **Restrictions**:
   - For Type-II codebooks, only one CSI-RS resource is allowed per resource set for channel measurement.
   - Maximum 64 NZP CSI-RS/SSB resources for `reportQuantity` = `none`, `cri-RI-CQI`, `cri-RSRP`, `ssb-Index-RSRP`, `cri-SINR`, or `ssb-Index-SINR`.
   - For CSI-IM interference measurement, each CSI-RS resource is paired with a CSI-IM resource.
   - For NZP CSI-RS interference measurement (non-L1-SINR), only one NZP CSI-RS resource is allowed, with up to 18 ports.

4. **Assumptions**:
   - For non-L1-SINR:
     - Each NZP CSI-RS port for interference measurement represents an interference layer.
     - Other interference signals are assumed on CSI-RS/CSI-IM resources.
   - For L1-SINR:
     - Dedicated NZP CSI-RS/CSI-IM resources represent total interference and noise.

##### 5.2.1.4.2 Report Quantity Configurations 
- Defines what CSI quantities the UE reports.
1. **Possible `reportQuantity` Values**:
   - `none`, `cri-RI-PMI-CQI`, `cri-RI-i1`, `cri-RI-i1-CQI`, `cri-RI-CQI`, `cri-RSRP`, `cri-SINR`, `ssb-Index-RSRP`, `ssb-Index-SINR`, `cri-RI-LI-PMI-CQI`.

2. **Behavior**:
   - **None**: No report is sent.
   - **cri-RI-PMI-CQI, cri-RI-LI-PMI-CQI**: Report precoder matrix (wideband or sub-band).
   - **cri-RI-i1, cri-RI-i1-CQI**: Report wideband PMI (`i1`) with Type-I Single Panel codebook.
   - **cri-RI-CQI**:
     - Uses port indices from `non-PMI-PortIndication` (if configured) or assumes default port ordering.
     - CQI assumes an identity matrix precoder scaled by \( \frac{1}{\sqrt{\nu}} \).
   - **cri-RSRP, ssb-Index-RSRP, cri-SINR, ssb-Index-SINR**: Report signal strength or quality for selected resources.

3. **CRI/SSBRI Derivation**:
   - If multiple resources (\( N_S > 1 \)) are configured, CRI/SSBRI selects the \((k+1)\)-th resource, and other parameters are conditioned on the reported CRI/SSBRI.
   - Maximum ports per CSI-RS resource: 16 (for \( N_S = 2 \)), 8 (for \( 2 < N_S \leq 8 \)).

##### 5.2.1.4.3 L1-RSRP Reporting 
- Measures signal strength for beam selection.
1. **Resources**: CSI-RS, SS/PBCH blocks, or both, QCLed with `QCL-TypeC` (Doppler) and `QCL-TypeD` (spatial).
2. **Configuration**: Up to 16 CSI-RS resource sets, each with up to 64 resources (total ≤ 128 resources).
3. **Reporting**:
   - **Single resource (`nrofReportedRS = 1`)**: 7-bit value, [-140, -44] dBm, 1 dB step.
   - **Multiple resources or group-based reporting**: Largest RSRP is 7-bit, others are 4-bit differential (2 dB step).
     
##### 5.2.1.4.4 L1-SINR Reporting 
- Measures signal quality relative to interference and noise.
1. **Resources**:
   - Channel: NZP CSI-RS or SS/PBCH blocks.
   - Interference: NZP CSI-RS or CSI-IM.
2. **Configuration**: Up to 16 resource sets, with up to 64 CSI-RS or SSB resources.
3. **Reporting**:
   - **Single resource (`nrofReportedRSForSINR = 1`)**: 7-bit value, [-23, 40] dB, 0.5 dB step.
   - **Multiple resources or group-based reporting**: Largest SINR is 7-bit, others are 4-bit differential (1 dB step).
   - Reported SINR values are not adjusted for power offsets.


#### 5.2.1.5 Triggering/activation of CSI Reports and CSI-RS 
##### 5.2.1.5.1 Aperiodic CSI Reporting and CSI-RS 
- Aperiodic CSI reporting is triggered dynamically by the base station using DCI when it needs immediate channel information.
- This section applies when the triggering PDCCH (Physical Downlink Control Channel) and the CSI-RS use the **same numerology** (subcarrier spacing, SCS).

1. **Configuration**:
   - A higher-layer parameter, `CSI-AperiodicTriggerStateList`, defines trigger states for aperiodic CSI reports and CSI-RS resources.
   - Each trigger state can be associated with any downlink (DL) Bandwidth Part (BWP).
   - CSI-RS resources can be configured as aperiodic, periodic, or semi-persistent via the `resourceType` parameter.

2. **Triggering Mechanism**:
   - The base station uses the **CSI request field** in DCI to trigger a CSI report.
   - If the CSI request field is all zeros, no CSI report is requested.
   - The number of bits in the CSI request field (`NTS`, configured by `reportTriggerSize`) determines how many trigger states can be addressed:
     - If the number of configured trigger states is more than `2^(NTS) - 1`, a subselection mechanism maps trigger states to DCI codepoints.
     - If fewer or equal, the DCI directly indicates the trigger state.
   - The mapping or subselection is applied after a delay, starting from the slot after the UE sends HARQ-ACK for the PDSCH carrying the subselection indication.

3. **Quasi Co-Location (QCL)**:
   - QCL defines how CSI-RS resources are spatially related to other reference signals (e.g., SS/PBCH block or other CSI-RS).
   - Each aperiodic CSI-RS resource is associated with a TCI-State (Transmission Configuration Indication) via `qcl-info`, which specifies QCL sources (e.g., for beamforming).
   - If a TCI-State includes `QCL-TypeD` (spatial relation), the reference signal can be an SS/PBCH block or a periodic/semi-persistent CSI-RS in the same or different component carrier (CC)/BWP.

4. **Scheduling Offset and Beam Switching**:
   - The **scheduling offset** is the time between the last symbol of the PDCCH (carrying the DCI trigger) and the first symbol of the CSI-RS.
   - If the offset is smaller than the UE’s reported `beamSwitchTiming` (e.g., 14, 28, or 48 symbols), the UE uses alternative QCL assumptions:
     - If another DL signal (e.g., PDSCH or another CSI-RS) is in the same symbols, the UE applies that signal’s QCL assumption.
     - If no other DL signal exists but a CORESET is configured, the UE uses the QCL of the CORESET with the lowest ID.
     - If no CORESET is configured but `enableDefaultBeamForCCS` is set, the UE uses the QCL of the lowest-ID active TCI-State for PDSCH.
   - If the offset is sufficient (≥ `beamSwitchTiming`), the UE applies the QCL assumptions from the TCI-State indicated in the DCI.

5. **Restrictions**:
   - A UE can’t receive more than one DCI with a non-zero CSI request per slot.
   - A UE can’t be triggered to report CSI for a non-active BWP unless it supports `CSItriggerStateContainingNonactiveBWP`.
   - If triggered for a non-active BWP, the UE skips the CSI report or measurement.
   - Aperiodic CSI-RS must not be transmitted before the DCI that triggers it.

6. **CSI-RS Triggering Offset**:
   - Configured per resource set via `aperiodicTriggeringOffset` or `aperiodicTriggeringOffsetExt-r16`.
   - Possible values: {0, 1, 2, ..., 15, 16, 24} slots.
   - If no minimum scheduling offset is configured and QCL-TypeD is not used, the offset is fixed at 0.
   - The offset for CSI-IM (Interference Measurement) follows the offset of the associated NZP (Non-Zero Power) CSI-RS.

7. **BWP Switching**:
   - If the active DL BWP changes between the DCI trigger and CSI-RS reception, the DCI for BWP switching must not end later than the DCI for CSI triggering.
   - No additional BWP switching is allowed between the CSI trigger and the CSI-RS.

##### 5.2.1.5.1a Aperiodic CSI Reporting with Different Numerologies 
- This section extends the rules for aperiodic CSI reporting when the PDCCH and CSI-RS use **different numerologies** (different subcarrier spacings, e.g., 15 kHz vs. 30 kHz).
1. **Beam Switching Timing**:
   - The scheduling offset is measured in **CSI-RS symbols** instead of slots.
   - If the PDCCH has a smaller SCS (µ_PDCCH < µ_CSIRS), an additional delay `d` (from Table 5.2.1.5.1a-1) is added to the `beamSwitchTiming`:
     - µ_PDCCH = 0: d = 8 PDCCH symbols
     - µ_PDCCH = 1: d = 8 PDCCH symbols
     - µ_PDCCH = 2: d = 14 PDCCH symbols
   - If the offset is less than `beamSwitchTiming + d · 2^(µ_PDCCH/µ_CSIRS)`, the UE uses alternative QCL assumptions (same as Section 5.2.1.5.1).
   - If the offset is sufficient, the UE applies the TCI-State’s QCL assumptions.

2. **CSI-RS Triggering Offset**:
   - The offset is configured as {0, 1, ..., 31} slots if µ_PDCCH < µ_CSIRS, or {0, 1, ..., 15, 16, 24} slots if µ_PDCCH > µ_CSIRS.
   - The CSI-RS is transmitted in a slot calculated based on the numerology of both PDCCH and CSI-RS, accounting for carrier aggregation slot offsets (`ca-SlotOffset`).

3. **Measurement Timing**:
   - If µ_PDCCH < µ_CSIRS, the CSI-RS must start at least `Ncsirs` PDCCH symbols after the end of the triggering PDCCH (values from Table 5.2.1.5.1a):
     - µ_PDCCH = 0: Ncsirs = 4 symbols
     - µ_PDCCH = 1: Ncsirs = 5 symbols
     - µ_PDCCH = 2: Ncsirs = 10 symbols
     - µ_PDCCH = 3: Ncsirs = 14 symbols
   - If µ_PDCCH > µ_CSIRS, the same rule applies but without the additional delay `d`.

##### 5.2.1.5.2 Semi-Persistent CSI and CSI-RS
- Semi-persistent CSI reporting or CSI-RS is activated/deactivated by the base station and operates periodically until deactivated.
1. **Semi-Persistent Reporting on PUSCH**:
   - Configured via `CSI-SemiPersistentOnPUSCH-TriggerStateList`.
   - Activated by DCI scrambled with SP-CSI-RNTI (Semi-Persistent CSI Radio Network Temporary Identifier).
   - A UE can’t have two semi-persistent CSI reports with the same `CSI-ReportConfigId` active simultaneously.

2. **Semi-Persistent Reporting on PUCCH**:
   - Configured via `reportConfigType`.
   - Activated by a MAC CE (Medium Access Control Control Element) as per TS 38.321.
   - The reporting starts after a delay from the slot where the UE sends HARQ-ACK for the PDSCH carrying the activation command.

3. **Semi-Persistent CSI-RS**:
   - Activated/deactivated via MAC CE.
   - QCL assumptions are provided via TCI-States, similar to aperiodic CSI-RS.
   - Active when the associated DL BWP is active; otherwise, suspended.

4. **Validation of Activation/Deactivation**:
   - DCI for activation/deactivation must use SP-CSI-RNTI and meet specific field settings (e.g., HARQ process number set to all 0s).
   - Deactivation DCI also sets modulation and coding scheme to all 1s and resource block assignment based on resource allocation type.

5. **Carrier Deactivation**:
   - If a carrier is deactivated, all semi-persistent configurations (CSI-RS, CSI reporting, SRS, ZP CSI-RS) are deactivated and need re-activation.

#### 5.2.1.6 CSI Processing Criteria 
- The UE has limited processing capabilities for CSI calculations, defined by the number of **CSI Processing Units (CPUs)** it supports.
1. **CSI Processing Units (CPUs)**:
   - A UE supports `N` simultaneous CSI calculations (CPUs).
   - If `L` CPUs are occupied, `N - L` are available.
   - If more CSI reports are triggered than available CPUs, the UE prioritizes reports based on Clause 5.2.5, dropping the lowest-priority reports.

2. **CPU Occupancy**:
   - Depends on the type of CSI report:
     - **ReportQuantity = 'none' with trs-Info**: 0 CPUs.
     - **ReportQuantity = 'cri-RSRP', 'ssb-Index-RSRP', 'cri-SINR', 'ssb-Index-SINR', or 'none' (no trs-Info)**: 1 CPU.
     - **Complex reports (e.g., cri-RI-PMI-CQI)**:
       - If simple (wideband, ≤4 CSI-RS ports, Type I codebook), occupies `N` CPUs.
       - Otherwise, occupies as many CPUs as the number of CSI-RS resources in the resource set.
   - **Duration of CPU Occupancy**:
     - **Periodic/Semi-persistent reports**: From the earliest CSI-RS/CSI-IM/SSB occasion to the last symbol of the PUSCH/PUCCH carrying the report.
     - **Aperiodic reports**: From the first symbol after the triggering PDCCH to the last symbol of the PUSCH carrying the report.
     - **L1-RSRP reports**: Specific durations based on measurement and reporting timing.

3. **Resource Limits**:
   - A UE can’t have more active CSI-RS ports or resources than its reported capability.
   - Aperiodic CSI-RS is active from the end of the triggering PDCCH to the end of the PUSCH carrying the report.
   - Semi-persistent CSI-RS is active from activation to deactivation.
   - Periodic CSI-RS is active from configuration to release.





