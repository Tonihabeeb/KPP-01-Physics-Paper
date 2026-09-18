# KPP-01 Buoyancy-Driven Pneumatic DEMO SIMULATOR — Governing Physics

**Author:** Omar (KPP-01 Physicist)  
**Document type:** Scientist-grade physics note for the live HMI/simulator  
**Plant / code:** `D:\PNEUMATIC project v1` (`hmi_demo`)  
**Live HMI:** `http://127.0.0.1:4173/`  
**Grounding snapshot:** `GET http://127.0.0.1:4173/api/grok/snapshot`  
**capturedAt:** `2026-09-18T19:23:45.567Z`  
**Fidelity:** `engineering` · **demoAssist live:** `0 N·m` · **Plant status:** `running`  


> **Grounding file (Dina):** `C:\Users\Nizar\AppData\Local\Temp\kpp_snap_out.txt` — outer `capturedAt` `2026-09-18T19:23:45.568Z`, snapshot `2026-09-18T19:23:45.567Z` (RUNNING, engineering, receiverSP=8, injReq=1.9, demoAssist=0).

> Every numeric claim below is taken from that snapshot’s live tags and/or `designBasis` / `designBasisAdvice` / `designBasisWarnings`. No invented plant constants.

---

## Abstract

KPP-01’s HMI simulator integrates open-cycle pneumatic injection, hydrostatic floater gas state, buoyant torque on an endless chain, rotational rigid-body dynamics, geared generation, and an energy ledger each tick. Design-basis net plant power is **−13.257 kW** (`compressorPowerKW = 15.843`, `designElectricalKW = 2.586`). The plant is **air-supply limited** near **2.59 kW** electrical — not the **8 kW** generator nameplate. With receiver cut-out **8 bara** and dock injection need **1.90 bara**, about **11.0 kW** is throttled at the regulator (~**69%** of design compressor shaft power). In engineering fidelity, **demoAssist ≡ 0** and must never be labelled buoyancy.

**Keywords:** buoyancy; open-cycle pneumatics; Boyle/polytropic floater gas; hydrostatic injection; throttle loss; energy balance; KPP-01 simulator

---


## 0. Grounding snapshot (authoritative)

Source file: `C:\Users\Nizar\AppData\Local\Temp\kpp_snap_out.txt`  
`snapshot.capturedAt = 2026-09-18T19:23:45.567Z` · status=`running` · fidelity=`engineering`

| Symbol / tag | Value | Role in equations |
|--------------|-------|-------------------|
| $P_{rec}$ `setpoints.receiverBara` | **8** bara | compressor cut-out |
| $P_{reg}$ `setpoints.regulatorBara` | **2.3** bara | header setpoint |
| $P_{dock}$ `injection.requiredBara` / `designBasis.injectionPressureBara` | **1.9** / **1.9** bara | hydrostatic fill head |
| $\tau_b$ `torqueNm.buoyancy` | **7856.1** N·m | buoyant drive |
| $\tau_{\mathrm{assist}}$ `torqueNm.demoAssist` | **0** N·m | **must stay 0 in eng.** |
| $n_{\mathrm{tw}}$ `towerRPM` | **2.755** rpm | $\omega=2\pi n/60$ |
| $P_{\mathrm{gen}}$ `deliveredKW` | **1.817** kW | live electrical |
| $P_{\mathrm{gen,des}}$ `designElectricalKW` | **2.586** kW | air-limited design |
| $P_{\mathrm{comp,des}}$ `compressorPowerKW` | **15.843** kW | design compression |
| $P_{\mathrm{net,des}}$ `netPlantPowerKW` | **-13.257** kW | $P_{gen}-P_{comp}$ |
| nameplate `ratedKW` | **8** kW | **not** delivered |
| gear `liveRatio` | **108.9** | $\omega_g \approx i\,\omega$ |
| air mass injected/vented | **127.81 / 126.237** kg | open-cycle balance |

Throttle fraction from designBasis warning (8 bar vs 1.90 bara):

$$
\frac{15.843-4.89}{15.843} \approx 0.691 \ (69\%)
$$


## 1. What the plant is

KPP-01 is an **open-cycle buoyancy pneumatic demonstration**:

1. Compressor fills a **receiver**.
2. **Regulator** drops pressure to a header setpoint.
3. At the **dock** (bottom), air is injected into floaters against hydrostatic head.
4. Air-filled floaters **ascend**; empty/vented floaters **descend**.
5. Net buoyant torque turns the tower → gearbox → generator.
6. At the **top**, air is **vented to atmosphere** (open cycle). Compression work in that air is largely rejected.

Simulator modules (code): `hmi_demo/js/utils/FloaterPhysics.js`, `hmi_demo/js/models/TowerDynamics.js`, `hmi_demo/js/utils/Config.js` (`pressureAtDepth`, designBasis builders), `hmi_demo/js/utils/PlantSnapshot.js` (API exposure).

---

## 2. Nomenclature

| Symbol | Meaning | SI / plant unit |
|--------|---------|-----------------|
| \(P_{\mathrm{atm}}\) | Surface absolute pressure | bara |
| \(h\) | Depth below free surface | m |
| \(\rho\) | Water density (`WATER_DENSITY`) | 1000 kg/m³ |
| \(g\) | Gravity | m/s² |
| \(P_{\mathrm{dock}}\) | Absolute injection pressure at dock | bara |
| \(V\) | Displaced / gas volume of a floater | m³ |
| \(m\) | Floater structure mass (plus entrained terms in design) | kg |
| \(F_b\) | Net buoyant force | N |
| \(r\) | Sprocket radius (`sprocketRadiusM`) | m |
| \(\tau\) | Torque | N·m |
| \(\omega\) | Angular velocity | rad/s |
| \(I\) | Tower rotational inertia | kg·m² |
| \(\dot{V}_{\mathrm{FAD}}\) | Compressor free-air delivery | m³/min |
| \(P_{\mathrm{gen}}, P_{\mathrm{comp}}, P_{\mathrm{net}}\) | Electrical out, compressor in, net | kW |
| \(\tau_{\mathrm{assist}}\) | demoAssist torque (**not buoyancy**) | N·m |

---

## 3. Hydrostatics at the dock

### 3.1 Governing equation

Fresh-water hydrostatic gradient in the simulator is **0.1 bar/m** (`BAR_PER_METRE = 0.1` in `FloaterPhysics.js`):

\[
P(h) = P_{\mathrm{atm}} + (0.1\,\mathrm{bar/m})\,h
\]

equivalently

\[
P_{\mathrm{dock}} = P_{\mathrm{atm}} + \rho g h
\quad\text{with}\quad
\frac{\rho g}{10^5}\approx 0.0981\,\mathrm{bar/m}\approx 0.1\,\mathrm{bar/m}.
\]

Code: `ambientPressureAt(depthM, atmBara)` and `pressureAtDepth(depthM)` in `Config.js`.

### 3.2 Snapshot / designBasis numbers

| Quantity | Value | Source |
|----------|-------|--------|
| `designBasis.injectionPressureBara` | **1.9** | designBasis |
| Live `pneumatic.injection.requiredBara` (RUNNING) | **1.9** (no active deep fill demand this frame) | live |
| Implied dock depth at 1.9 bara if \(P_{\mathrm{atm}}=1\) | \(h=(1.9-1)/0.1=9\) m | derived |

**Physics truth for running fill:** designBasis injection head is **1.90 bara**.

---

## 4. Boyle / polytropic free-air volume for floater fill

### 4.1 Isothermal (Boyle) mass–volume relation

Default gas law in `FloaterPhysics.js` is isothermal:

\[
m = \rho_1\, P\, V
\quad\Rightarrow\quad
V = \frac{m}{\rho_1 P}
\]

with \(P\) in **bara** and \(\rho_1\) = air density at 1 bara.

Free-air equivalent volume to fill geometric volume \(V_f\) at dock pressure:

\[
V_{\mathrm{free}} = V_f \cdot \frac{P_{\mathrm{dock}}}{P_{\mathrm{atm}}}
\]

Example: if \(V_f = 0.16\,\mathrm{m}^3\) and \(P_{\mathrm{dock}}=1.9\,\mathrm{bara}\), \(V_{\mathrm{free}} = 0.304\,\mathrm{m}^3\) per fill (illustrative geometry; live inventory uses mass tags).

### 4.2 Polytropic extension

For exponent \(n\in[1,1.4]\):

\[
V = V_{\mathrm{ref}}\left(\frac{P_{\mathrm{ref}}}{P}\right)^{1/n}
\]

\(n=1\) recovers Boyle; \(n>1\) reduces mean ascending volume (less buoyancy work per injected mass). Code: `expansionVolume(...)`.

### 4.3 Live air inventory (this snapshot)

| Tag | Value |
|-----|-------|
| `accounting` injected/vented | **127.81 / 126.237 kg** |
| `pneumatic.accounting.injectedKg` | **2.156 kg** |
| `ventedKg` / `recoveredKg` | **0 / 0** (plant ready; no active lap vent this frame) |

---

## 5. Buoyant force, torque, and shaft power

### 5.1 Force on one floater

\[
F_b = \big(\rho V - m\big) g
\]

Ascending (air volume \(V\)): typically \(F_b > 0\) (up).  
Descending (vented / water-filled): buoyancy shrinks; weight dominates.

Design builder in `Config.js` uses the same idea for mean ascending force:

\[
F_{\mathrm{asc,mean}} = \bar{V}_{\mathrm{air}}\,\rho\,g - W_{\mathrm{structure}}
\]

### 5.2 Torque sum on the sprocket

\[
\tau_{\mathrm{buoyancy}} = \sum_i F_{b,i}\, r_i
\qquad
(r \approx \texttt{sprocketRadiusM})
\]

**Critical fidelity rule:**

\[
\tau_{\mathrm{driving}} = \tau_{\mathrm{buoyancy}} + \tau_{\mathrm{assist}}
\]

with **\(\tau_{\mathrm{assist}}=\texttt{demoAssist}\)** — **not buoyancy**.

| Quantity | Value | Source |
|----------|-------|--------|
| Live `torqueNm.buoyancy` | **7856.1 N·m** | snapshot |
| Live `torqueNm.demoAssist` | **0 N·m** | snapshot |
| `designBasis.designTorqueNm` | **10796.809 N·m** | designBasis |
| `designBasis.demoAssistTorqueNm` | **17841.538 N·m** | **what-if only** — not live eng torque |

### 5.3 Mechanical power

\[
P_{\mathrm{mech}} = \tau\,\omega,\qquad \omega = 2\pi n/60
\]

Design check:

\[
P_{\mathrm{mech,design}} = 10796.809 \times \frac{2\pi\times 2.755}{60} \approx 3.114\,\mathrm{kW}
\]

matches `designBasis.designMechanicalKW = 3.114`.

Live (ready/braked): `towerRPM = 0`, so \(P_{\mathrm{mech,live}}=0\) this frame; buoyancy torque is held against brake (`brake = 14500 N·m`).

### 5.4 Rotational dynamics each tick (`TowerDynamics.updateDynamics`)

Newton’s second law for rotation:

\[
I\,\alpha = \tau_{\mathrm{net}},\qquad
\alpha = \frac{\tau_{\mathrm{net}}}{I}
\]

\[
\omega \leftarrow \omega + \alpha\,\Delta t
\]

Resistive torques (friction, water drag, brake, generator load) **oppose motion**; they stop the shaft but do not reverse-drive it. Code zero-cross clamps prevent chatter.

Chain speed:

\[
v_{\mathrm{chain}} = \omega \cdot r_{\mathrm{sprocket}}
\]

---

## 6. Pneumatic chain: receiver, regulator, FAD vs demand

### 6.1 Snapshot setpoints & sensors

| Quantity | Value | Source |
|----------|-------|--------|
| `setpoints.receiverBara` | **8** | live |
| `setpoints.regulatorBara` | **2.3** | live |
| `sensors.ptTankBara` | **6.936** | live |
| `sensors.ptHeaderBara` | **2.095** | live |
| Compressor FAD | **3.2 m³/min** | live + designBasis |
| `designBasis.airDemandM3Min` | **3.2** | designBasis |
| `designBasis.compressorPowerKW` | **15.843** | designBasis |

### 6.2 Why 8 bara vs ~1.9 dock wastes ~69%

Idealized isothermal compressor work to pressure \(P\) scales as \(P\ln(P/P_0)\) (or catalogue curve at cut-out). DesignBasis warning states explicitly:

> Receiver set to 8 bar but injection only needs 1.90 bara: **15.84 kW** of compression where **4.89 kW** would do. The surplus is throttled away at the regulator.

\[
\eta_{\mathrm{throttle\,waste}} = \frac{15.843 - 4.89}{15.843} \approx 0.691 \;(69\%)
\]

`designBasisAdvice`: *“About 11.0 kW is throttled away at the regulator.”* Suggestion: cut-out **2.3 bar** (dock + 0.4 bar).

Regulator drop:

\[
\Delta P_{\mathrm{throttle}} = P_{\mathrm{receiver}} - P_{\mathrm{header}}
\]

is **irreversible** for plant electrical output: shaft power spent raising pressure above dock need does not increase buoyant \(V\) at the dock.

---

## 7. Open-cycle vent mass and energy

### 7.1 Mass balance tags

\[
m_{\mathrm{injected}} = m_{\mathrm{vented}} + m_{\mathrm{held}} + m_{\mathrm{spilled}} - m_{\mathrm{recovered}} + \varepsilon
\]

This snapshot (ready): injected 2.156 kg, vented 0, recovered 0, `balanceErrorKg = 0`.

During a running lap, design/intent is **vent at top** → open cycle (`recoveredKg` remains 0 unless a recovery model is enabled).

### 7.2 Energy ledger (`energyKWh`)

| Channel | kWh (this snapshot window) |
|---------|----------------------------|
| in | 0.0753 |
| out | 0 |
| compressionLoss | 0.0226 |
| throttle | 0 |
| spillVent | 0.0022 |
| assist | **0** |
| unexplainedFrac | 0.5178 (short/idle window — not a steady run) |

On a long engineering run, throttle and compression dominate `in − out`, consistent with designBasis.

---

## 8. Generator path: air-limited ~2.6 kW vs 8 kW nameplate

| Quantity | Value | Source |
|----------|-------|--------|
| `electrical.generator.ratedKW` | **8** | live |
| `designBasis.designElectricalKW` | **2.586** | designBasis |
| `electrical.generator.loadRequestKW` | **2.6** | live |
| Live delivered (RUNNING) | **1.817 kW** | tower stopped |

Advice: *“An 8 kW generator at this torque needs about 9.90 m³/min FAD… selected compressor delivers 3.20 m³/min, which supports 2.59 kW.”*

\[
P_{\mathrm{elec,air\text{-}limit}} \approx 2.59\,\mathrm{kW} \ll 8\,\mathrm{kW}_{\mathrm{nameplate}}
\]

Gear: `fittedGearRatio = 108.9` (design recommended 108.911) — matches gen ~300 rpm at tower ~2.755 rpm when running.

---

## 9. Net plant power

\[
P_{\mathrm{net}} = P_{\mathrm{gen}} - P_{\mathrm{comp}}
\]

| Basis | \(P_{\mathrm{gen}}\) | \(P_{\mathrm{comp}}\) | \(P_{\mathrm{net}}\) |
|-------|----------------------|------------------------|------------------------|
| designBasis | 2.586 kW | 15.843 kW | **−13.257 kW** |
| live plantPower (this frame; compressor coasting) | 1.817 | 0 kW | **1.817 kW** |

designBasisWarnings: *“Design-basis net plant power is -13.26 kW (2.59 kW generated vs 15.84 kW of compression).”*

**Interpretation:** negative net is **expected open-cycle physics** with over-pressured receiver — not a sensor bug.

---

## 10. demoAssist vs buoyancy (engineering fidelity)

| | Live engineering | designBasis field |
|--|------------------|-------------------|
| Buoyancy torque | 10779.7 N·m | designTorqueNm 10796.809 |
| demoAssist | **0 N·m** | demoAssistTorqueNm 17841.538 (**counterfactual**) |
| energyKWh.assist | **0** | — |
| fidelityMode | **engineering** | — |

**Rule:** Never call demoAssist buoyancy. Engineering acceptance requires \(\tau_{\mathrm{assist}}\equiv 0\).

---

## 11. What the simulator is solving each tick

State vector (conceptual; tags in `State.js`):

\[
\mathbf{x} = \big\{
\omega,\;\theta,\;
\{m_i,V_i,\text{segment}_i\}_{i=1}^{N},\;
P_{\mathrm{tank}},P_{\mathrm{header}},\;
\text{valve/compressor flags},\;
E_{\mathrm{ledger}},\;
\text{gen load}
\big\}
\]

**One \(\Delta t\) step (engineering fidelity):**

1. **Hydrostatics / gas state** — for each floater, \(P_{\mathrm{amb}}(h)=P_{\mathrm{atm}}+0.1\,h\); update mass/volume via Boyle/polytropic (`FloaterPhysics`).
2. **Dock injection** — if docked and header above `requiredBara`, add air mass; demand accumulates toward FAD limit.
3. **Vent / spill** — at top segment, open-cycle mass leave; accounting tags update.
4. **Forces → torques** — \(F_{b,i}=(\rho V_i-m_i)g\); \(\tau_{\mathrm{buoy}}=\sum F_{b,i} r\); add **demoAssist only if enabled** (0 in eng.).
5. **Resistive torques** — friction, bearing, water drag, brake, generator load.
6. **Rigid-body update** — \(\alpha=\tau_{\mathrm{net}}/I\); integrate \(\omega,\theta\); derive RPM & chain speed (`TowerDynamics.updateDynamics`).
7. **Pneumatic plant** — compressor toward receiver setpoint; regulator toward header setpoint; throttle loss enters energy ledger.
8. **Electrical** — gearbox ratio maps \(\omega\to\omega_g\); delivered kW ≤ air-limited / torque-limited available power; never silently equal to 8 kW nameplate.
9. **Energy ledger** — integrate compressor in, gen out, compression/throttle/vent/drag/friction/brake/gearbox/converter/assist channels.
10. **Snapshot API** — `PlantSnapshot` publishes tags + recomputed `designBasis` block for `GET /api/grok/snapshot`.

Loop geometry this basis: `loopLengthM = 21.705`, `floaterPitchM = 0.748`, `N = 29` floaters, `lapTimeSec = 86.989` at design RPM, `fillsPerMin = 20.003`.

---

## 12. Equation index (quick)

1. \(P_{\mathrm{dock}}=P_{\mathrm{atm}}+\rho g h\) (0.1 bar/m in code)  
2. \(V_{\mathrm{free}}=V_f P_{\mathrm{dock}}/P_{\mathrm{atm}}\) (Boyle fill)  
3. \(F_b=(\rho V-m)g\) ; \(\tau=\sum F_b r\) ; \(P=\tau\omega\)  
4. \(I\alpha=\tau_{\mathrm{net}}\) ; \(\omega\leftarrow\omega+\alpha\Delta t\)  
5. Throttle waste \(\approx(15.843-4.89)/15.843\approx 69\%\) at receiver 8 vs dock 1.9  
6. \(P_{\mathrm{net}}=P_{\mathrm{gen}}-P_{\mathrm{comp}}\) → designBasis **−13.257 kW**  
7. \(\tau_{\mathrm{assist}}\equiv 0\) in engineering (≠ buoyancy)

---

## 13. Conclusions

1. The simulator is a coupled **hydrostatic–pneumatic–mechanical–electrical** integrator, not a cartoon.  
2. Design-basis net **−13.257 kW** with air-limited electrical **2.586 kW** is the honest stock plant.  
3. **~69%** of design compressor power is regulator throttle from receiver **8 bara** vs dock **1.9 bara**.  
4. **8 kW** is nameplate ceiling, not delivered physics.  
5. **demoAssist = 0** in engineering; designBasis assist torque is a what-if only.  
6. Net ≥ 0 requires pressure matching **and** vent recovery / closed cycle — not a bigger generator sticker.

---

## 14. Data citation

- Snapshot: `capturedAt = 2026-09-18T19:23:45.567Z` via `GET http://127.0.0.1:4173/api/grok/snapshot`  
- Blocks used: `plant`, `pneumatic`, `mechanical.torqueNm`, `electrical`, `energyKWh`, `designBasis`, `designBasisAdvice`, `designBasisWarnings`  
- Code references: `hmi_demo/js/utils/FloaterPhysics.js`, `Config.js` (`pressureAtDepth`, `WATER_DENSITY`), `models/TowerDynamics.js` (`updateDynamics`)  
- Author actions: **read-only**; no plant START/STOP/setpoint writes for this document.

*— End —*
