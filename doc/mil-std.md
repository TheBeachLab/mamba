# Mamba MIL-STD Validation

## Scope

This document defines the environmental validation approach for the Mamba autonomous drone platform. Testing is self-performed and documented per the tailored methods below. Compliance is claimed **per individual test method**, not to MIL-STD-810H as a whole.

**Applicable standards:**

| Standard | Title | Application |
|---|---|---|
| MIL-STD-810H | Environmental Engineering Considerations and Laboratory Tests | Methods 501.7, 514.8, 516.8 |
| MIL-STD-31000 | Technical Data Packages | Design documentation structure |
| MIL-HDBK-217F | Reliability Prediction of Electronic Equipment | MTBF calculation |

**Platform under test:**

| Subsystem | Component | Critical interfaces |
|---|---|---|
| Flight controller | Pixhawk 1 (hw v2.4.5) | GPS connector, peripheral VDD line |
| Propulsion | RCTIMER 5010 360KV × 4 | Arm mounting, ESC wiring |
| Frame | 2mm CF plate + 13.2mm CF tubes | Arm-to-base joints, vibration damping |
| Compute | Raspberry Pi 5 4GB + Hailo8 | USB/CSI connections, thermal |
| Power | Skywalker Quattro 25A×4 + BEC | Power module connector |
| Telemetry | SiK radio pair | TELEM1 serial port |
| Navigation | u-blox LEA-6H GPS + compass | GPS port (known power issue, see [guidance](guidance.md)) |

> **Known issue:** GPS VDD peripheral line drops to ~1V if GPS is connected during boot. This is a pre-existing hardware limitation of the Pixhawk clone and must be accounted for in all test procedures. GPS must be connected **after** boot or powered via TELEM1 VDD brick line.

---

## 1. MIL-STD-810H, Method 514.8 — Vibration

### 1.1 Purpose

Verify that the Mamba airframe, avionics mounting, and electrical connections survive and operate correctly under vibration levels representative of multirotor flight. Of particular concern:

- Pixhawk 1 mounting (IMU sensitivity to frame vibration)
- Carbon fiber arm-to-base joints (adhesive/mechanical retention)
- GPS connector reliability (known fragile connector)
- RPi 5 + Hailo8 module retention and thermal interface
- Propeller balance and motor shaft integrity

### 1.2 Tailoring rationale

MIL-STD-810H Method 514.8 defines multiple Categories. For a small UAS platform, **Category 24 (Jet Aircraft Stores — Materiel Installed in Jet Aircraft)** is overly severe. The tailored approach uses:

- **Procedure I — General Vibration** as baseline
- Vibration profile derived from **actual flight data** (accelerometer logs from Pixhawk) rather than generic profiles, per 810H guidance on "use measured data when available"
- Test duration: 1 hour per axis (3 axes) — reduced from the 810H default of 4 hours per axis for qualification, acceptable for engineering development testing

### 1.3 Test equipment required

| Equipment | Specification | Alternative |
|---|---|---|
| Accelerometer | ADXL345 or MPU-6050 breakout, ≥200 Hz sample rate | Pixhawk onboard IMU log (VIBE message) |
| Data logger | Raspberry Pi or laptop with serial/I2C logging | QGroundControl .ulog export |
| Vibration source | Electrodynamic shaker (preferred) | Controlled motor run at various RPM on fixed test stand |
| Mounting fixture | Rigid plate replicating operational mounting | The actual airframe, fixed to bench |

### 1.4 Test procedure

**Pre-test:**

1. Photograph all connectors, joints, and mounting points. Record torque values on all fasteners.
2. Record baseline IMU data: power on Pixhawk, log 60s of stationary VIBE data. Record X, Y, Z vibration levels and clipping counts.
3. Perform functional check: GPS lock, telemetry link, RPi boot and Hailo8 inference test.
4. Mark all connectors with alignment marks (paint pen) to detect any movement.

**Test execution (motor-induced vibration):**

Since a shaker table may not be available, the following motor-run profile substitutes:

| Step | Throttle | Duration | RPM (approx.) | Rationale |
|---|---|---|---|---|
| 1 | 25% | 15 min | ~3000 | Ground idle / pre-flight |
| 2 | 50% | 20 min | ~5500 | Hover |
| 3 | 75% | 15 min | ~7500 | Cruise / maneuver |
| 4 | 100% | 5 min | ~9000 | Max thrust |
| 5 | Sweep 25→100→25% | 5 min | Variable | Resonance sweep |

Repeat with each axis oriented vertically (rotate airframe on fixture) if using a shaker. If using motor-run, a single orientation (normal flight attitude) is acceptable.

**During test:**

- Log Pixhawk VIBE data continuously (.ulog)
- Monitor for audible rattling, visible connector loosening
- At each throttle step, verify telemetry link active and GPS lock maintained

**Post-test:**

1. Repeat all pre-test inspections. Compare photographs.
2. Check all alignment marks for movement.
3. Record post-test VIBE baseline (same 60s stationary test).
4. Perform full functional check.
5. Inspect propeller balance and arm joints for cracking or delamination.

### 1.5 Pass/fail criteria

| Parameter | Pass | Fail |
|---|---|---|
| Pixhawk VIBE levels | X, Y, Z < 30 m/s² | Any axis ≥ 60 m/s² |
| Clipping events | 0 during stationary post-test | Any clipping in post-test |
| Connector displacement | No movement on alignment marks | Visible displacement |
| Structural integrity | No cracks, delamination, or loosened fasteners | Any structural damage |
| Functional check | All subsystems operational | Any subsystem failure |

### 1.6 Deliverables

- [ ] Test plan (this document)
- [ ] Pre-test photographs and baseline data
- [ ] Raw .ulog files for all test steps
- [ ] Post-test photographs and comparison
- [ ] Test report with pass/fail determination

---

## 2. MIL-STD-810H, Method 501.7 — High Temperature

### 2.1 Purpose

Verify that Mamba electronics and structural components operate correctly at elevated temperatures representative of hot climate field deployment. Critical concerns:

- Raspberry Pi 5 thermal throttling (known to throttle at 85°C die temperature)
- Hailo8 accelerator thermal limits
- LiPo battery performance degradation and safety
- Carbon fiber epoxy matrix softening (typically Tg > 120°C, not expected to be limiting)
- Pixhawk IMU accuracy drift with temperature

### 2.2 Tailoring rationale

Method 501.7 defines storage and operational temperature categories. For a tactical UAS operating in European/Mediterranean climates:

- **Operational hot temperature:** 45°C (Category A2 — Basic Hot)
- **Storage hot temperature:** 60°C (Category A2)
- **Test duration:** 4 hours at operational temperature (reduced from 810H qualification default of 7×24h cycles, appropriate for development testing)
- **LiPo batteries excluded from storage test** — tested per manufacturer limits only, not subjected to 60°C due to safety risk

### 2.3 Test equipment required

| Equipment | Specification | Alternative |
|---|---|---|
| Climate chamber | Programmable to 60°C ±2°C | Insulated box + ceramic heater + PID controller |
| Temperature sensors | K-type thermocouples or DS18B20 digital | RPi internal temp sensor + IR thermometer |
| Data logger | Multi-channel thermocouple logger | RPi logging via 1-Wire bus |
| Timer | — | Software timer |

### 2.4 Test procedure

**Pre-test:**

1. Record ambient temperature and humidity.
2. Attach temperature sensors to: Pixhawk case, RPi 5 SoC (via `vcgencmd measure_temp`), Hailo8 module, ESC heat sink, motor windings (IR spot check), battery surface.
3. Perform baseline functional check at ambient temperature.

**Test execution — Operational (45°C):**

| Step | Action | Duration |
|---|---|---|
| 1 | Place fully assembled drone (without propellers) in chamber | — |
| 2 | Ramp to 45°C at ≤1°C/min | ~20 min |
| 3 | Soak at 45°C | 60 min (thermal stabilization) |
| 4 | Power on all electronics, begin functional test | — |
| 5 | Run Pixhawk self-test, verify IMU calibration | 10 min |
| 6 | Run Hailo8 inference benchmark (fixed model, record FPS) | 10 min |
| 7 | Activate telemetry link, verify stable connection | 10 min |
| 8 | Maintain powered operation at 45°C | 120 min |
| 9 | Ramp down to ambient at ≤1°C/min | ~20 min |

**Test execution — Storage (60°C):**

| Step | Action | Duration |
|---|---|---|
| 1 | Remove LiPo batteries. Place drone in chamber | — |
| 2 | Ramp to 60°C at ≤1°C/min | ~35 min |
| 3 | Soak at 60°C, unpowered | 4 hours |
| 4 | Ramp down to ambient at ≤1°C/min | ~35 min |
| 5 | Allow 1 hour ambient stabilization, then full functional check | — |

**During test:**

- Log all temperatures at 10-second intervals
- Monitor RPi throttle flag: `vcgencmd get_throttled` (bit 19 = over-temperature)
- Record Hailo8 inference performance every 15 minutes

**Post-test:**

1. Visual inspection for deformation, discoloration, delamination.
2. Full functional check at ambient temperature.
3. Compare Hailo8 inference FPS to baseline.
4. Check all connectors and solder joints.

### 2.5 Pass/fail criteria

| Parameter | Pass | Fail |
|---|---|---|
| RPi 5 SoC temperature | < 85°C at 45°C ambient | ≥ 85°C (throttling) |
| Hailo8 inference rate | ≥ 90% of ambient baseline FPS | < 80% of baseline |
| Pixhawk IMU | Passes self-test, no thermal cal errors | Self-test failure |
| Telemetry link | Stable connection throughout | Link loss > 5 seconds |
| Structural | No visible deformation or delamination | Any structural change |
| Post-storage functional | All systems nominal at ambient | Any subsystem failure |

### 2.6 Deliverables

- [ ] Test plan (this document)
- [ ] Temperature log (all sensors, full duration)
- [ ] RPi throttle flag log
- [ ] Hailo8 benchmark results (ambient vs. hot)
- [ ] Pre/post photographs
- [ ] Test report with pass/fail determination

---

## 3. MIL-STD-810H, Method 516.8 — Shock

### 3.1 Purpose

Verify that Mamba survives handling drops and hard landings without loss of function. This is particularly relevant for:

- Pixhawk IMU (accelerometer and gyroscope damage from impact)
- GPS connector (known fragile, see [guidance notes](guidance.md))
- Carbon fiber tube arm joints (brittle failure mode under impact)
- RPi 5 and Hailo8 board-level connections (BGA solder joints)
- Propeller retention on motor shafts

### 3.2 Tailoring rationale

Method 516.8 defines multiple procedures. The applicable ones for a portable UAS:

- **Procedure I — Functional Shock:** simulate hard landing
- **Procedure IV — Transit Drop:** simulate handling/transport drop

Tailored shock levels:

| Procedure | Shock specification | Rationale |
|---|---|---|
| Functional shock (hard landing) | 20g, 11ms half-sine pulse | Representative of 2m/s vertical impact on hard surface |
| Transit drop | 1.0m drop onto 50mm plywood over concrete | Per 810H Table 516.8-II for items 0–9 kg |

### 3.3 Test equipment required

| Equipment | Specification | Alternative |
|---|---|---|
| Drop test rig | Guided drop fixture with quick-release | Measured height drop from string release |
| Impact surface | 50mm plywood over concrete floor | Per 810H spec |
| Accelerometer | ADXL377 (±200g range) or similar high-g sensor | Pixhawk IMU may clip (limited to ±16g on MPU-6000) |
| High-speed camera | ≥240 fps | Smartphone slow-motion |
| Measuring tape | Steel tape, calibrated | — |

### 3.4 Test procedure

**Pre-test:**

1. Full functional check: Pixhawk self-test, GPS lock, telemetry, RPi + Hailo8 boot and inference.
2. Photograph all structural joints, connectors, propeller retention.
3. Record accelerometer baseline noise floor.
4. Remove propellers for safety. Install with landing gear as in operational configuration.

**Test execution — Transit drop (Procedure IV):**

| Drop | Orientation | Height | Surface |
|---|---|---|---|
| 1 | Flat (bottom down, normal landing attitude) | 1.0 m | 50mm plywood/concrete |
| 2 | Nose down (forward arm leading) | 1.0 m | 50mm plywood/concrete |
| 3 | Side (left arms leading) | 1.0 m | 50mm plywood/concrete |
| 4 | Inverted (top down) | 1.0 m | 50mm plywood/concrete |
| 5 | Corner (single arm leading) | 1.0 m | 50mm plywood/concrete |

For each drop:

1. Suspend drone at specified height and orientation using quick-release mechanism.
2. Start high-g accelerometer and high-speed camera recording.
3. Release. Allow drone to come to rest.
4. Record peak g-level and pulse duration from accelerometer.
5. Visual inspection for damage before proceeding to next drop.
6. **Stop testing if structural failure is detected** — document and report.

**Test execution — Functional shock (Procedure I):**

If a shock machine is available, apply 20g half-sine 11ms pulse in each of 3 orthogonal axes, both directions (6 shocks total). If not, the transit drop test above provides adequate engineering-level shock data.

**Post-test (after all drops):**

1. Full functional check (identical to pre-test).
2. Detailed photographic inspection of all joints, connectors, boards.
3. Check propeller shaft runout (spin by hand, visual wobble check).
4. Verify Pixhawk IMU calibration is still valid (compare pre/post accelerometer offsets).
5. Verify GPS lock acquisition time is within normal range (< 60 seconds cold start).

### 3.5 Pass/fail criteria

| Parameter | Pass | Fail |
|---|---|---|
| Structural integrity | No cracks, delamination, broken joints | Any structural failure |
| Pixhawk function | Passes self-test, IMU cal within ±5% of pre-test | Self-test failure or cal drift > 10% |
| GPS | Cold start lock < 60s post-test | No lock or lock > 120s |
| RPi 5 + Hailo8 | Boots and runs inference normally | Boot failure or inference errors |
| Telemetry | Link established normally | Link failure |
| Propeller shafts | No visible runout | Wobble or shaft bend |
| Connectors | All seated, no displacement on alignment marks | Any connector unseated |

### 3.6 Deliverables

- [ ] Test plan (this document)
- [ ] Pre-test photographs and functional check record
- [ ] High-speed video of each drop
- [ ] Accelerometer data for each drop (peak g, pulse duration)
- [ ] Post-test photographs and functional check record
- [ ] Test report with pass/fail determination

---

## 4. MIL-STD-31000 — Technical Data Package

### 4.1 Purpose

Structure the Mamba design documentation as a Technical Data Package (TDP) conforming to MIL-STD-31000. This is a **documentation standard**, not a test. It demonstrates the ability to deliver engineering data in a format accepted by defense procurement.

### 4.2 Applicability

MIL-STD-31000 defines three TDP types:

| Type | Content | Mamba applicability |
|---|---|---|
| Conceptual | Functional requirements, trade studies | Partial — README + design rationale |
| Developmental | Drawings, specs, test plans | **Primary focus** |
| Production | Full manufacturing package | Out of scope |

### 4.3 Required TDP elements for Mamba (Developmental level)

The following mapping shows how existing and planned Mamba documentation maps to MIL-STD-31000 requirements:

| MIL-STD-31000 Element | Mamba document | Status |
|---|---|---|
| **System specification** | README.md + operational concept | ⬜ Expand |
| **Engineering drawings** | hardware/*.FCStd (FreeCAD) | ⬜ Export to PDF/STEP |
| **Parts list / BOM** | doc/design.md (partial) | ⬜ Formalize as structured BOM |
| **Interface documents** | doc/guidance.md (wiring, pinouts) | ⬜ Expand with connector tables |
| **Test plans** | doc/mil-std.md (this document) | ✅ |
| **Test reports** | — | ⬜ After test execution |
| **Software documentation** | doc/ai.md, doc/vision.md | ⬜ Expand with SW architecture |
| **Associated data** | img/ directory | ✅ |
| **Quality provisions** | — | ⬜ Add inspection criteria |

### 4.4 Implementation plan

**Phase 1 — Drawing package:**

1. Export FreeCAD models (drprt.FCStd, frame.FCStd) to:
   - STEP AP214 format (3D exchange)
   - PDF drawings with title block, revision, dimensions, tolerances
2. Create assembly drawing showing all components in context.
3. Assign drawing numbers: `MAMBA-DWG-001` through `MAMBA-DWG-NNN`.

**Phase 2 — Bill of Materials:**

Create structured BOM in tabular format:

| Item | Part number | Description | Qty | Manufacturer | Specification |
|---|---|---|---|---|---|
| 1 | MAMBA-001 | Base plate, 2mm CF | 1 | Custom | MAMBA-DWG-001 |
| 2 | MAMBA-002 | Arm tube, 13.2mm OD CF | 4 | Generic | ASTM D3039 |
| 3 | RCTIMER-5010 | Motor 360KV pancake | 4 | RCTIMER | Vendor datasheet |
| 4 | RCTIMER-16x55 | Propeller 16×5.5" CF | 4 | RCTIMER | Vendor datasheet |
| 5 | PX4-FMUv2 | Pixhawk 1 (hw 2.4.5) | 1 | 3DR/clone | PX4FMUv2.4.5 schematic |
| 6 | UBEC-SW4x25 | Skywalker Quattro ESC | 1 | Hobbyking | Vendor datasheet |
| 7 | SiK-TEL | SiK telemetry radio | 2 | Generic | SiK firmware spec |
| 8 | UBLOX-LEA6H | GPS + compass module | 1 | u-blox | LEA-6H datasheet |
| 9 | RPI5-4G | Raspberry Pi 5 4GB | 1 | Raspberry Pi Ltd | Product brief |
| 10 | HAILO-8 | Hailo8 AI accelerator | 1 | Hailo | HAILO-8 datasheet |

**Phase 3 — Interface Control Document (ICD):**

Document all electrical interfaces:

- Pixhawk ↔ GPS (UART + I2C, known VDD issue)
- Pixhawk ↔ ESC (PWM channels 1–4)
- Pixhawk ↔ SiK radio (TELEM1 UART, 57600 baud)
- Pixhawk ↔ RPi 5 (MAVLink over serial or UDP)
- RPi 5 ↔ Hailo8 (PCIe M.2)
- Power distribution diagram with voltage rails and current budgets

### 4.5 Deliverables

- [ ] Drawing package (STEP + PDF)
- [ ] Structured BOM (CSV + PDF)
- [ ] Interface Control Document
- [ ] TDP index document listing all data items

---

## 5. MIL-HDBK-217F — Reliability Prediction

### 5.1 Purpose

Calculate predicted Mean Time Between Failures (MTBF) for the Mamba electronics using MIL-HDBK-217F methodology. This is a **calculation exercise**, not a physical test. It demonstrates the ability to perform quantitative reliability engineering.

### 5.2 Method

MIL-HDBK-217F provides two prediction methods:

- **Parts Count** — uses generic failure rates by component category. Simpler, suitable for early design.
- **Parts Stress** — uses detailed operating conditions (voltage derating, temperature, etc.). More accurate.

For Mamba, the **Parts Count** method is appropriate. The Parts Stress method can be applied selectively to critical components (Pixhawk, RPi 5).

### 5.3 Environmental profile

MIL-HDBK-217F defines environments with different π_E (environmental factor) multipliers:

| Environment code | Description | π_E multiplier (typical) |
|---|---|---|
| G_B | Ground, Benign | 1.0 |
| A_UF | Airborne, Uninhabited Fighter | 12.0 |
| A_RW | Airborne, Rotary Wing | 8.0 |

For Mamba, **A_RW (Airborne, Rotary Wing)** is the closest match. This is conservative for a small multirotor but demonstrates worst-case thinking.

### 5.4 Parts count reliability prediction

The following table uses generic base failure rates (λ_b) from MIL-HDBK-217F Tables for each component category, adjusted by the environmental factor.

| Component | Category (217F) | Qty | λ_b (per 10⁶ hrs) | π_E (A_RW) | λ_p = Qty × λ_b × π_E |
|---|---|---|---|---|---|
| Pixhawk (microprocessor-based) | Hybrid/MOS complex IC | 1 | 0.060 | 8.0 | 0.480 |
| Raspberry Pi 5 SoC | VLSI MOS microprocessor | 1 | 0.14 | 8.0 | 1.120 |
| Hailo8 accelerator | VLSI MOS ASIC | 1 | 0.14 | 8.0 | 1.120 |
| BLDC motors (5010) | Motor, AC | 4 | 5.0 | 8.0 | 160.000 |
| ESC (4-in-1, power MOSFET) | Power semiconductor | 4 ch | 0.50 | 8.0 | 16.000 |
| u-blox GPS module | Hybrid IC | 1 | 0.060 | 8.0 | 0.480 |
| SiK radio module | RF/transmitter hybrid | 2 | 0.10 | 8.0 | 1.600 |
| Connectors (est. 20 pins) | Connector, general | 20 | 0.010 | 8.0 | 1.600 |
| Wiring harness (est.) | Wiring, general | 1 | 0.002 | 8.0 | 0.016 |
| LiPo battery | Battery, rechargeable | 1 | 2.0 | 8.0 | 16.000 |
| **TOTAL** | | | | | **λ_SYSTEM ≈ 198.4** |

### 5.5 MTBF calculation

```
MTBF = 1 / λ_SYSTEM = 1 / (198.4 × 10⁻⁶) ≈ 5,040 hours
```

This is the **predicted MTBF** for the full system under airborne rotary-wing environmental conditions.

**Interpretation:**

- At 1 hour average mission time, this predicts approximately 1 failure per 5,040 missions.
- The dominant failure contributors are the **BLDC motors** (81% of total failure rate), followed by **ESC** and **battery** (8% each).
- Electronics-only MTBF (excluding motors, ESC, battery) would be approximately 156,000 hours — indicating that mechanical/electromechanical components drive overall reliability.

### 5.6 Recommendations from reliability analysis

1. **Motor reliability is the critical path.** Consider: bearing quality, motor temperature monitoring, vibration-based predictive maintenance.
2. **ESC derating:** Verify that motor current draw at max throttle does not exceed 80% of ESC rating (25A × 80% = 20A per channel).
3. **Connector reliability:** The known GPS connector issue is consistent with connector failure being a significant contributor. Consider: conformal coating, strain relief, or connector upgrade.
4. **Battery:** Implement cell voltage monitoring and enforce safe discharge limits (no LiPo cell below 3.3V under load).

### 5.7 Deliverables

- [ ] Parts count prediction spreadsheet (this section, expanded to full 217F format)
- [ ] Reliability block diagram
- [ ] Critical items list (top 5 failure contributors)
- [ ] Recommendations for reliability improvement

---

## 6. Test execution tracking

| Test | Standard | Status | Date | Report |
|---|---|---|---|---|
| Vibration | MIL-STD-810H Method 514.8 | ⬜ Planned | — | — |
| High temperature | MIL-STD-810H Method 501.7 | ⬜ Planned | — | — |
| Shock / drop | MIL-STD-810H Method 516.8 | ⬜ Planned | — | — |
| Technical data package | MIL-STD-31000 | 🔨 In progress | — | — |
| Reliability prediction | MIL-HDBK-217F | ✅ Initial calc complete | — | Section 5 above |

---

## References

1. MIL-STD-810H, *Environmental Engineering Considerations and Laboratory Tests*, Department of Defense, January 2019. Available: [everyspec.com](https://everyspec.com/MIL-STD/MIL-STD-0800-0899/MIL-STD-810H_55998/)
2. MIL-STD-31000B, *Technical Data Packages*, Department of Defense, November 2018. Available: [everyspec.com](https://everyspec.com/MIL-STD/MIL-STD-30000-79999/MIL-STD-31000B_50579/)
3. MIL-HDBK-217F Notice 2, *Reliability Prediction of Electronic Equipment*, Department of Defense, February 1995. Available: [everyspec.com](https://everyspec.com/MIL-HDBK/MIL-HDBK-0200-0299/MIL-HDBK-217F_14591/)
4. PX4 Autopilot Documentation, *Vibration Isolation*. Available: [docs.px4.io](https://docs.px4.io/main/en/assembly/vibration_isolation.html)
5. Raspberry Pi 5 Datasheet, *Thermal Management*. Available: [raspberrypi.com](https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html)
