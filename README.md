# StockPulse
StockPulse - Real-time stock market app for Android

## Engineering note: Lütze NG-9001 (DC 24-110V to dual 5.3V outputs)

This repository issue requested both:
1. A simplified EveryCircuit-style schematic approximation.
2. An engineering check of changing upstream protection from a 2-pole MCB to a 1-pole MCB.

### 1) Simplified EveryCircuit-style schematic (functional equivalent)

The exact internal manufacturer schematic is not publicly provided.  
Based on datasheet text (`galv. Trennung E/A: ja`), the converter is treated as an isolated switched DC/DC stage with a high-frequency transformer.

```text
Input +24...110Vdc
      |
   [MCB]
      |
 [Reverse-polarity protection]
      |
 [TVS / suppressor diode clamp]
      |
 [EMI/input capacitor]
      |
 [PWM switch + controller]
      |
      | primary
   [HF Transformer]  <-- galvanic isolation barrier
      | secondary
 [Rectifier + LC filter] ---> +5.3V_CH1 (max 2.1A @ T<=50°C)
 [Rectifier + LC filter] ---> +5.3V_CH2 (max 2.1A @ T<=50°C)
      |
     0V_out (isolated from input side by transformer)
```

Use this as an EveryCircuit approximation with a switched transistor stage, transformer coupling, secondary rectification, output capacitors, and channel loads.

### 2) 2-pole MCB -> 1-pole MCB change analysis

#### Datasheet values used
- Input operating range: 16.8 to 137.5 Vdc (nominal family 24 to 110 Vdc).
- Output: 5.3 V ±0.25 V.
- Output current: 2.1 A per channel (up to +50°C), two channels.
- I/O galvanic isolation: yes.

#### Output and input power/current estimates
- Per channel output power:  
  `P_ch = 5.3 V * 2.1 A = 11.13 W`
- Total output power:  
  `P_out = 2 * 11.13 W = 22.26 W`

Assuming typical converter efficiency `η = 0.85`:
- `P_in = P_out / η = 22.26 / 0.85 = 26.19 W`

Input current at key voltages:
- `I_in@110V = 26.19 / 110 = 0.238 A`
- `I_in@24V = 26.19 / 24 = 1.09 A`

So normal steady-state current is low/moderate; the dominant risk in 2-pole vs 1-pole change is fault isolation behavior, not nominal load current.

#### Engineering implications of replacing 2-pole with 1-pole

With a 2-pole DC MCB, both conductors are disconnected.  
With a 1-pole DC MCB, one conductor remains connected and can stay live relative to chassis/ground through external bonds, EMC paths, leakage, or fault paths.

Likely consequences:
1. **Incomplete isolation during maintenance**
   - One line remains energized, increasing shock/arc risk during service.
2. **Different ground-fault behavior**
   - Fault current may return via unintended paths; trip/selectivity assumptions can change.
3. **Protection concept drift**
   - Original design intent for double-pole disconnection is no longer preserved.
4. **Compliance risk**
   - Railway/industrial safety expectations often require clear all-pole isolation in DC circuits.

#### Decision guidance

Do **not** downgrade to 1-pole MCB unless a full system protection study confirms:
- supply grounding topology (floating vs bonded),
- all fault-loop impedances and prospective fault currents,
- breaker DC interrupt rating and coordination/selectivity with upstream protection,
- maintenance isolation requirements and applicable railway/industrial standards.

In short: for this isolated wide-input railway converter, changing from 2-pole to 1-pole is usually a safety/reliability regression unless explicitly validated by system-level calculations and compliance review.
