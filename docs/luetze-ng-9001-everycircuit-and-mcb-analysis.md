# Lütze NG-9001 DC 24-110V (Art.-Nr. 819001): EveryCircuit schematic + 2-pole vs 1-pole MCB check

## Scope and limitations
- This note provides a **functional equivalent schematic** for simulation in EveryCircuit.
- The **exact internal factory schematic is not public** in the available product text.
- Datasheet inputs used here (from provided text):
  - Input range: DC 16.8 V to 137.5 V (nominal 24 V to 110 V systems)
  - Output: 2 channels, each 5.3 V / 2.1 A up to +50 °C
  - Derated output above +50 °C: 1.5 A per channel
  - Galvanic isolation input/output: yes

## EveryCircuit-ready schematic (functional equivalent)
Use this as a practical simulation model for behavior, sizing, and protection checks.

```text
V1 (DC source, 24...110 V) -> QF1 (MCB) -> D1 (reverse-polarity diode) -> TVS1 (surge clamp to 0V)
-> C_IN (electrolytic input capacitor)
-> SW1 (ideal PWM switch, 50...150 kHz) -> T1 primary (isolated HF transformer)
-> D2 + D3 (secondary Schottky rectifiers)
-> L_OUT + C_OUT (output filter)
-> CH_A load (5.3 V / 2.1 A)
-> CH_B load (5.3 V / 2.1 A)

0V_in and 0V_out remain galvanically isolated.
```

### Suggested simulation starting values
- V1: 24 V, then 110 V sweep (and optional 16.8...137.5 V sweep)
- QF1: ideal breaker element for first-pass simulation
- D1: low-drop power diode model
- TVS1: stand-off around input system level, clamp above normal max input
- SW1: duty cycle closed-loop to hold 5.3 V output
- T1: isolated transformer model (coupled inductors acceptable)
- D2/D3: Schottky models
- C_IN / C_OUT sized to keep ripple and transient dip reasonable

## Core calculations

### 1) Output power (full load, <= +50 °C)
- Per channel: 
  - `P_ch = V_out * I_ch = 5.3 V * 2.1 A = 11.13 W`
- Two channels:
  - `P_out_total = 2 * 11.13 W = 22.26 W`

### 2) Output power (derated, > +50 °C)
- Per channel derated:
  - `P_ch_derated = 5.3 V * 1.5 A = 7.95 W`
- Two channels:
  - `P_out_derated_total = 2 * 7.95 W = 15.90 W`

### 3) Estimated input current at full load
For converter efficiency `eta`:
- `P_in = P_out / eta`
- `I_in = P_in / V_in`

At full-load output 22.26 W:

| Assumed efficiency | Pin (W) | Iin @ 110 V | Iin @ 24 V |
|---|---:|---:|---:|
| 80% | 27.83 | 0.253 A | 1.160 A |
| 85% | 26.19 | 0.238 A | 1.091 A |
| 90% | 24.73 | 0.225 A | 1.030 A |

### 4) Estimated input current at derated load (> +50 °C)
At derated output 15.90 W:

| Assumed efficiency | Pin (W) | Iin @ 110 V | Iin @ 24 V |
|---|---:|---:|---:|
| 80% | 19.88 | 0.181 A | 0.828 A |
| 85% | 18.71 | 0.170 A | 0.780 A |
| 90% | 17.67 | 0.161 A | 0.736 A |

## 2-pole MCB -> 1-pole MCB change impact

### Electrical behavior difference
- **2-pole MCB:** disconnects both DC conductors from the converter.
- **1-pole MCB:** disconnects only one conductor; the other stays connected.

For an isolated DC/DC in rolling stock systems, replacing 2-pole with 1-pole can cause:
1. **Incomplete isolation for maintenance**
   - One conductor may remain live relative to chassis/PE due to upstream bonding, EMC paths, or surge components.
2. **Fault-current path persistence**
   - With one line still connected, some internal protection networks may stay energized during a fault or service condition.
3. **Protection concept mismatch**
   - Original coordination/selectivity and isolation assumptions may rely on both poles opening.
4. **Standards/compliance risk**
   - Railway installation rules or project safety case may require two-pole disconnection for this branch.

### Calculation-based check (steady-state only)
Steady-state input current is low (roughly 0.16...1.16 A across listed conditions), so the main risk is **not** normal current magnitude.
The main risk is **safety and fault isolation behavior** when one conductor remains connected.

### Recommendation
- Keep **2-pole MCB** unless system-level engineering confirms 1-pole remains compliant and safe.
- Before any change, verify at minimum:
  1. DC system grounding/floating arrangement
  2. Upstream protection and selectivity study
  3. Required disconnection method in project/rail standards
  4. Maintenance isolation procedure and lockout expectations

## What this gives the requester
- A direct EveryCircuit-implementable functional equivalent.
- Checked power/current calculations at full-load and derated conditions.
- Clear engineering implications of changing 2-pole protection to 1-pole.
