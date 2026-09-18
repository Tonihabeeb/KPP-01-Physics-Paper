Energy Balance and Hydrostatics of an Open-Cycle Buoyancy-Driven
Pneumatic Power Demonstration Plant (KPP-01)
Omar - KPP-01 Physicist / Scientific Lead
Project: D:\PNEUMATIC project v1 | Live HMI: http://127.0.0.1:4173/
Manuscript UTC date: 2026-09-18 | Grounding snapshot: 2026-09-18T19:08:26.372Z

Abstract
KPP-01 is an open-cycle, buoyancy-driven pneumatic demonstration plant: compressed air is injected into submerged
floaters at dock depth, producing buoyant torque on an endless chain that drives a geared generator. Using the plant
design basis and a live engineering-fidelity snapshot (2026-09-18T19:08:26.372Z), design-basis net plant power is
-13.26 kW (compressor 15.84 kW versus electrical 2.59 kW). The machine is air-supply limited near 2.6 kW electrical,
far below the 8 kW nameplate. About 69% of design compressor shaft power is attributable to receiver cut-out at 8 bara
while dock injection needs only 1.9 bara; the surplus is irreversibly throttled. Live demo-assist torque is 0 N*m in
engineering fidelity and must not be called buoyancy. Net-positive electricity on this architecture requires pressure
matching and vent recovery or closed-cycle reuse - not a larger nameplate generator.
Keywords: buoyancy power; open-cycle pneumatics; hydrostatic injection; throttle loss; energy balance; KPP-01

1. Introduction
Buoyancy-driven pneumatic schemes store work in compressed air, displace water from submerged vessels (floaters),
and convert buoyant force into shaft work on ascent. KPP-01 implements this as a submerged endless-chain tower with
dock injection and free venting at the surface. The scientific question is where the energy goes, why net electrical power
is negative under honest accounting, and which losses are fundamental versus operational.
Every quantitative claim below is grounded in HMI designBasis and live snapshot (status=running,
fidelityMode=engineering). No plant setpoints were written for this study.

2. System description

2.1 Mechanical arrangement
The tower carries 29 rigid floaters on a closed loop of length 21.705 m with pitch 0.748 m. Design lap time at air-limited
speed is 87.0 s (20.0 fills/min). A fixed gearbox (live ratio 108.9:1) matches tower speed (~2.75 rpm) to generator target
near 300 rpm (live 300.1 rpm on GDG-1100).

2.2 Pneumatic train
Compressor FAD 3.2 m3/min feeds a receiver with cut-out 8 bara. Regulator setpoint 2.3 bara supplies the dock header.
Injection head required at depth is 1.9 bara (designBasis 1.9 bara). Snapshot injection was continuous
(pulseEnabled=False).

2.3 Electrical path
Generator nameplate 8 kW. Design-basis electrical output at the air-limited point is 2.586 kW. Live delivered power was
1.83 kW at load request 2.6 kW. Nameplate must not be treated as delivered power.

3. Hydrostatics and buoyancy torque

3.1 Dock hydrostatic head
Absolute injection pressure at depth follows
  P_dock ~= P_atm + rho*g*h        =>    ~1.0 bara + 0.1 bar/m * 9 m ~ 1.9 bara

This matches live requiredBara=1.9 and designBasis injectionPressureBara=1.9. Free-air volume scales by Boyle:
V_free ~= V_floater * P_dock / P_atm.

3.2 Buoyant torque
Net buoyant force on an air-filled floater is (rho*V - m_structure)*g upward; empty/vented floaters add descending
weight. Torque on the sprocket is force times sprocket radius, summed over the instantaneous population of
ascending-full, descending-empty, and filling floaters.
Live: buoyancy 7910.2 N*m; demoAssist 0.0 N*m; ascending_full=13, descending_empty=16; floater air inventory 1.597
kg. Design-basis torque 10796.8 N*m. Shaft power P = tau*omega:
  P_mech,design = 10796.8 N*m * (2.755 rpm * 2*pi/60) ~= 3.11 kW                          (designMechanicalKW)

Live mechanical estimate from the generator path: 2.20 kW at tower 2.755 rpm.

4. Open-cycle air mass and venting
KPP-01 vents floater air to atmosphere at the top of the loop. Compression work in that air is largely rejected rather than
expanded through a machine. Live accounting: injected 73.29 kg, vented 71.70 kg, recovered 0.0 kg. Run-ledger
spill/vent energy 0.124 kWh; assist 0.0 kWh.
Open-cycle venting is why a buoyancy demonstrator can rotate and generate while still showing strongly negative net
plant power under honest compression accounting.

5. Pneumatic chain: receiver, regulation, FAD vs demand
Live sensors: tank 6.33 bara, header 2.09 bara, outlet 1.01 bara, flow 2.12 m3/min. Injection demand 2.22 m3/min; avg
fill 9 s; docked 3. Compressor FAD 3.2 m3/min matches design airDemand 3.2 m3/min at the design point -
volume-balanced, not pressure-economic.
HMI design warning: receiver 8 bar while injection needs 1.90 bara implies 15.84 kW compression where ~4.89 kW
would do; surplus throttled at the regulator.
  Throttle fraction ~= (15.84 - 4.89) / 15.84 ~= 69% of design compressor shaft power

6. Energy balance

6.1 Design-basis balance
Compressor power (designBasis): 15.843 kW
Mechanical shaft (design): 3.114 kW
Electrical out (design): 2.586 kW
Net plant power (design): -13.257 kW
Air-limited tower speed: 2.755 rpm
Generator nameplate (not delivered): 8 kW
Design-basis net is in the -13 kW class (-13.26 kW). Electrical output is air-limited near 2.6 kW, not nameplate-limited.

6.2 Live snapshot balance
Compressor (live): 13.57 kW
Generated (live): 1.83 kW
Net plant (live): -11.74 kW
Energy in / out (kWh ledger): 4.404 / 0.766
Throttle loss (kWh ledger): 2.032
Compression loss (kWh ledger): 1.321
Live net (-11.7 kW class) matches the design diagnosis: compression input dominates generation. Cumulative throttle
energy already exceeds electrical output in the run ledger.

6.3 Why net is negative (physics, not a sensor fault)
 - Open-cycle venting rejects most compression enthalpy.
 - Receiver cut-out 8 bara >> dock need 1.9 bara => irreversible throttle.
 - Air rate x fill dwell caps tower near 2.75 rpm => ~2.6 kW electrical ceiling.
 - Buoyancy recovers displacement work over the rise - not full compressor work.

7. Receiver cut-out and optimum pressure matching
Engineering optimum for stock hardware: receiver cut-out ~2.3 bara (dock 1.9 + ~0.4 bar margin), regulator ~2.3 bara,
load request <= 2.6 kW. This removes ~11.0 kW of avoidable throttle waste and moves net from ~-13 kW toward a few
kilowatts negative - an honest stock-plant optimum, not yet net-positive.
Smart pulse/learn can trim average compressor power via lower duty, but with receiver still at 8 bara it is second-order
versus cut-out reduction.

8. Demo versus engineering fidelity; demoAssist
Grounding snapshot fidelityMode=engineering. Live demoAssist=0.0 N*m; assist energy=0.0 kWh. designBasis still
publishes what-if demoAssistTorqueNm=17841.5 N*m; that figure must not be labeled buoyancy and must not enter
engineering acceptance torque. Demo modes that inject assist can spin the chain while contaminating the scientific
story; engineering runs require assist = 0.

9. Discussion: what the demo proves vs net >= 0

9.1 What KPP-01 honestly demonstrates
 - Hydrostatic dock fill near 1.9 bara is physically consistent.
 - Buoyancy can produce multi-kN*m torque and drive a geared generator at ~2.8 rpm tower speed.
 - Air-limited electrical output ~2.6 kW is achievable without demo assist.
 - Open-cycle net electrical power is negative under full compression accounting.

9.2 What would be required for net >= 0
 - Pressure-match receiver/regulator to dock (necessary, not sufficient).
 - Recover vented air (closed cycle) or expand the vent stream through a work-producing device.
 - Minimize structural mass and parasitic drag; right-size FAD to true demand.
 - Never use demoAssist as a substitute for buoyancy in acceptance tests.
A larger generator nameplate does not fix an air- and pressure-limited source. Claiming 8 kW delivered from this stock
configuration would be scientifically false.

10. Conclusions
(1) Design-basis net plant power is -13.26 kW with 15.84 kW compression and 2.59 kW electrical - air-limited, not
nameplate-limited. (2) Live engineering snapshot (2026-09-18T19:08:26.372Z) shows the same qualitative balance (net
-11.74 kW; demoAssist 0). (3) Receiver cut-out at 8 bara against dock need 1.9 bara dominates avoidable loss via
regulator throttle (~69% of design compressor shaft power). (4) Open-cycle venting sets a structural efficiency ceiling;
net >= 0 needs recovery architecture. (5) Demo assist must remain excluded from buoyancy accounting in engineering
fidelity.

11. Nomenclature
bara: bar absolute
P_dock: Absolute injection pressure at dock depth
FAD: Free air delivery of compressor (m3/min)
tau, omega: Torque (N*m), angular velocity (rad/s)
designBasis: HMI design-point block (not live sensors)
demoAssist: Non-buoyancy assist torque (must be 0 in eng.)
receiverSP: Receiver cut-out pressure setpoint
regulatorSP: Downstream regulated header setpoint
netPlantPowerKW: generatedKW - compressorKW (plant gate)

12. Data citation
Live values: GET http://127.0.0.1:4173/api/grok/snapshot capturedAt=2026-09-18T19:08:26.372Z, plant.status=running,
fidelityMode=engineering. Design figures: snapshot.designBasis (compressorPowerKW, designElectricalKW,
netPlantPowerKW, injectionPressureBara, airDemandM3Min, gear ratio, demoAssistTorqueNm). Author role: read-only
physics; no START/STOP/setpoint writes for this manuscript.

- End of paper -
