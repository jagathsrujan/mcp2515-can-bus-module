# Phase 4 — MCP2515 Controller and Oscillator

Status: **Opus design PROPOSED, awaiting user approval** (2026-11-02). Sonnet verification of V-01…V-06 and netlist `mcp2515_block_netlist.md` reviewed; revisions marked **(rev)**.
Primary source: MCP2515 datasheet DS20001801K, https://ww1.microchip.com/downloads/aemDocuments/documents/APID/ProductDocuments/DataSheets/MCP2515-Family-Data-Sheet-DS20001801K.pdf (values via `research/ic_datasheet_extract.md` §A). Tags: **[DS]**, **[CALC]**, **[ASSUME]**, **[TEST]**.

---

## 1. Oscillator

### 1.1 Frequency — 16 MHz (confirms D-018)
- FOSC 1–25 MHz at VDD 2.7–5.5 V; 40 MHz only at ≥ 4.5 V **[DS Table 13-2 p75]** → 16 MHz is legal at VIO 3.0–5.5 V.
- Parallel-cut crystal required **[DS §8.0 p55]** — YXC X322516MLB4SI is a parallel (load-capacitance-specified) crystal.
- Oscillator start-up timer: 128 OSC1 cycles (8 µs at 16 MHz) before SPI access **[DS §8.1]**.

### 1.2 CAN bit timing at 16 MHz **[CALC]**
TQ = 2·(BRP+1)/FOSC. Bit = Sync(1) + PropSeg + PS1 + PS2. MCP2515 rules **[DS §5.2, §5.5 p40/p43, verified]**: PropSeg 1–8, PS1 1–8, PS2 2–8, SJW 1–4, BRP 0–63; PropSeg + PS1 ≥ PS2; PS2 ≥ SJW. Sample points follow CiA 301 recommendations (75 % at 1 Mbit/s, 87.5 % below) **[ASSUME — CiA recommendation, not a datasheet value]**.

| Bit rate | BRP | TQ | NBT | Prop | PS1 | PS2 | SJW | Sample pt | t_PROP | Osc. tolerance allowed | CNF1 | CNF2 | CNF3 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 Mbit/s | 0 | 125 ns | 8 | 4 | 1 | 2 | 1 | 75 % | 500 ns | 0.49 % | 0x00 | 0x83 | 0x01 |
| 500 kbit/s | 0 | 125 ns | 16 | 7 | 6 | 2 | 1 | 87.5 % | 875 ns | 0.31 % | 0x00 | 0xAE | 0x01 |
| 250 kbit/s | 1 | 250 ns | 16 | 7 | 6 | 2 | 1 | 87.5 % | 1.75 µs | 0.31 % | 0x01 | 0xAE | 0x01 |
| 125 kbit/s | 3 | 500 ns | 16 | 7 | 6 | 2 | 1 | 87.5 % | 3.5 µs | 0.31 % | 0x03 | 0xAE | 0x01 |

**(rev)** First draft used Prop 3/PS1 2 at 1 Mbit/s and Prop 5/PS1 8 below. Sonnet verified those hex values as correct, but with the TCAN1042 loop delay (t_PROP(LOOP2) 175 ns max, SLLSES7D Table 7.8 p12) they allowed only 2.5 m at 1 Mbit/s and 27.5 m at 500 k. Moving TQs from PS1 into PropSeg keeps the same sample points and tolerances while extending the bus:

| Rate | Max bus length, worst-case t_loop 175 ns | typ t_loop 110 ns | CiA DS-102 guideline |
|---|---|---|---|
| 1 Mbit/s | (500/2 − 175)/5 = **15 m** | 28 m | 40 m |
| 500 kbit/s | (875/2 − 175)/5 = **52 m** | 65 m | 100 m |
| 250 kbit/s | **140 m** | 153 m | 250 m |
| 125 kbit/s | **315 m** | 328 m | 500 m |

t_bus = 5 ns/m **[ASSUME]**; MCP2515 internal input/output delay is not specified in ns (§5.5 only says "typically 1–2 TQ") and is **not** included — treat the 1 Mbit/s figure as ≤ 10 m in practice **[ASSUME]**. CNF2 hex re-derived: 0x80 | (PS1−1)<<3 | (Prop−1) → 0x80|0x00|0x03 = 0x83; 0x80|0x28|0x06 = 0xAE **[CALC]**. (Datasheet Eq 5-2 misprints TQ = 2·BRP·TOSC; Register 5-1 gives 2·(BRP+1)·TOSC, used here.)

- All four DR-04 rates are exact (0 % rate error).
- Tolerance formula (ISO 11898-1): df ≤ min( SJW / (20·NBT), min(PS1,PS2) / (2·(13·NBT − PS2)) ). The tightest allowance (0.31 % = 3 125 ppm) is ~60× the crystal budget: ±10 ppm initial + ±20 ppm temperature + ±3 ppm/yr aging + load-cap pulling ≈ ±50 ppm **[DS crystal series sheet / CALC]**.
- Max bus length from t_PROP ≥ 2·(t_bus + t_loop): computed in §1.6 once TCAN1042 loop delay is confirmed.
- CNF2 bit 7 BTLMODE = 1 (PS2 set by CNF3); bit 6 SAM = 0 (single sample — triple sampling adds delay and is not recommended at high rates **[ASSUME, not a datasheet requirement]**). CNF3 SOF = 0, WAKFIL = 0.

### 1.3 Load capacitors
CL = C1·C2/(C1+C2) + C_stray → C1 = C2 = 2·(CL − C_stray).
- CL = 9 pF from the JLC/LCSC listing only; the only YXC document (YSX321SL series sheet p1) says "10 pF, 20 pF, or specify" **(rev, V-01)**. 10 pF caps sit inside the 8–12 pF window for CL 9 pF and give CL ≈ 9–10 pF with 4–5 pF stray, so the choice holds either way; bench trim decides.
- C_stray = OSC pin capacitance + trace ≈ 3–5 pF **[ASSUME]** (MCP2515 OSC pin capacitance not specified **[DS NOT FOUND]**).
- → C1 = C2 = 8–12 pF; **selected 10 pF C0G/NP0 50 V 0603** (±5 % or ±0.25 pF). C0G mandatory (no voltage/temperature drift).
- Microchip's own Table 8-2 suggests 15 pF at 20 MHz for a ~20 pF-CL-class crystal — consistent in scale **[DS p56]**.
- Final value set on the bench: measure CLKOUT (enabled, /1) with a frequency counter; target within ±20 ppm of 16.000000 MHz **[TEST]**.

### 1.4 Drive level and series resistor
Estimate (pessimistic: rail-amplitude sine across crystal), I = 2π·f·(C0 + CL)·V_rms, P = I²·ESR with ESR 50 Ω max, C0 3 pF **[CALC]**:
- VIO = 3.3 V: 1.4 mA → ~99 µW (crystal rating 10–200 µW, 100 µW typ) — OK.
- VIO = 5.0 V: 2.1 mA → ~227 µW — **possibly above the 200 µW crystal maximum**.

Mitigation: **R_S footprint (0603) between OSC2 and the crystal, fitted with 0 Ω** (MCP2515 Fig 8-1 shows R_S external; note 1 says it "may be required for AT strip cut crystals" **[DS p55, wording corrected (rev)]** — here it is a drive-level option). C2 sits on the crystal side of R_S (net OSC2_X) per Fig 8-1. If bench measurement at VIO = 5 V shows overdrive (frequency shift vs drive or current-probe measurement), fit R_S ≈ X_C2 = 1/(2π·16 MHz·10 pF) ≈ 1 kΩ, then re-verify start-up at VIO = 3.0 V, −40 °C if possible **[TEST]**. Not fitted by default because MCP2515 oscillator transconductance is not specified, so the start-up margin with R_S cannot be calculated.

### 1.5 Feedback resistor
**RF is internal** — Fig 8-1 (p55) draws RF inside the MCP2515 boundary, R_S outside **(rev, V-02)**. No external feedback resistor.

### 1.6 Layout rules (crystal)
Y1, C1, C2, R_S within 5 mm of pins 7/8; ground guard ring (top GND pour) around them stitched to bottom GND; no other nets under Y1 on either layer; crystal pads 2 & 4 (case) to GND; keep ≥ 10 mm from U3/L1 (spec §6, Phase 3 §7).

---

## 2. Reset
- RESET has **no internal pull-up**; t_RL ≥ 2 µs; reset after power-up "highly recommended" **[DS Table 1-1 p4, Table 13-4, §9.0]**.
- VIH(RESET) 0.85·VDD, VIL 0.15·VDD **[DS Table 13-1]**.

Circuit (net `RST_N`):
| Part | Value | Why |
|---|---|---|
| R_PU | 10 kΩ to VIO | Holds RESET high when host leaves J2-RST floating |
| C_RST | 10 nF X7R 0603 to GND | Power-on reset: τ = 100 µs; reaches 0.85·VIO in 1.9·τ ≈ 190 µs ≫ 2 µs and ≫ OST 8 µs **[CALC]** |
| R_SER | 1 kΩ, RST_N → U1 pin 17 | Datasheet Fig 9-1 R1 (1–10 kΩ): limits current into the pin's clamp diode if VIO collapses while C_RST is charged **[DS]** |
| J2-RST | wired to RST_N | Host may drive reset (push-pull or open-drain). A host driver (≤ 50 Ω) discharges 10 nF in < 1 µs, so a host pulse ≥ 10 µs low satisfies t_RL ≥ 2 µs with margin **[CALC]**. After release, RST_N recovers through R_PU in ≈ 190 µs before U1 leaves reset. |

The Fig 9-1 discharge diode is omitted: with 10 nF / 10 kΩ, C_RST discharges to VIO through R_PU in ~100 µs, and R_SER limits pin current to (V_C − VIO − 0.6)/1 kΩ < 5 mA briefly **[CALC]**. Firmware should still issue SPI RESET (0xC0) at start-up.

## 3. Supply and decoupling
- U1 pin 18 VDD = VIO; pin 9 VSS = GND.
- 100 nF X7R 50 V 0603 at pin 18–9, via to GND plane at the cap pad.
- VIO bulk (10 µF 0805 + 100 nF) already at J2 (Phase 3 §6.3).
- IDD ≤ 10 mA **[DS]** — covered by the host VIO budget.

## 4. Host SPI / interrupt interface (J2)
| J2 pin | Net | Direction | U1 pin | Series R | Pull | Notes |
|---|---|---|---|---|---|---|
| 1 | VIO | in (power) | 18 | — | — | 3.0–5.5 V from host |
| 2 | GND | — | 9 | — | — | |
| 3 | SCK | host → U1 | 13 | 33 Ω at J2 | — | ≤ 10 MHz; modes 0,0 / 1,1 **[DS Table 13-6]** |
| 4 | MOSI | host → U1 (SI) | 14 | 33 Ω at J2 | — | |
| 5 | MISO | U1 (SO) → host | 15 | 33 Ω at U1 (source) | — | Tri-state when CS high (t_DIS 100 ns) → bus-shareable |
| 6 | CS_N | host → U1 | 16 | 33 Ω at J2 | **10 kΩ to VIO** | Keeps U1 deselected while host boots (ESP32/STM32 pins float at reset) |
| 7 | INT_N | U1 → host | 12 | 33 Ω at U1 | — | **Push-pull** active-low **[DS]** — no pull-up; **cannot be wire-ORed** with other modules |
| 8 | RST_N | host → U1 | 17 | via 1 kΩ R_SER | 10 kΩ to VIO | §2 |

Series resistors: source-series damping for 10–20 cm jumper-wire harnesses (source impedance ~20–40 Ω + 33 Ω ≈ wire Z0 of ~70–100 Ω **[ASSUME]**) and ESD current limiting at the board edge. Timing cost: 33 Ω × ~15 pF ≈ 0.5 ns, negligible vs 50 ns SPI windows **[CALC]**.

Logic compatibility (same domain VIO) **[DS / CALC]**:
| Link | Driver | Receiver | Result |
|---|---|---|---|
| Host → SCK/SI/CS | host VOH ≈ VIO | VIH 0.7·VIO, VIL 0.4·VIO | OK at any VIO |
| SO/INT → host | VOH ≥ VIO − 0.5/0.7 V | host CMOS 0.7·VIO class | OK |
| TXCAN → TCAN TXD | VOL 0.6 V @ 6 mA, VOH VDD − 0.7 @ 3 mA | VIL 0.3·VIO (≥ 0.9 V), VIH 0.7·VIO | OK; actual load < 0.2 mA so VOL/VOH far better |
| TCAN RXD → RXCAN | VOL 0.2·VIO @ 2 mA (0.66 V at 3.3 V); VOH 0.8·VIO | VIL 0.15·VDD (0.5 V at 3.3 V); VIH 2.0 V | **Spec-sheet VOL > VIL at 2 mA**, but RXCAN draws ≤ 1 µA: driver R ≤ 0.66 V/2 mA = 330 Ω → VOL ≤ 0.3 mV **[CALC]**. OK. |

## 5. CAN controller connections and unused pins
| Pin | Name | Connection | Justification |
|---|---|---|---|
| 1 | TXCAN | → U2 pin 1 TXD, direct, short (< 10 mm) | |
| 2 | RXCAN | ← U2 pin 4 RXD, direct | |
| 3 | CLKOUT/SOF | NC + test pad TP_CLK | Default after reset is **enabled, FOSC/8 = 2 MHz** **[DS §8.2]** → firmware should set CLKEN = 0 (high-Z) except during frequency calibration; no long trace (EMI) |
| 4, 5, 6 | TX0RTS, TX1RTS, TX2RTS | NC | Internal 100 kΩ pull-ups to VDD **[DS Table 1-1]** define them; default function is digital input |
| 7 | OSC2 | R_S (0 Ω) → net OSC2_X → Y1 pad 3 + C2 10 pF | §1 |
| 8 | OSC1 | Y1 pin 1, C1 10 pF | §1 |
| 9 | VSS | GND | |
| 10, 11 | RX1BF, RX0BF | NC | Outputs; BFPCTRL reset value disables them (high-Z) **[DS Reg 4-3]** |
| 12 | INT | INT_N via 33 Ω to J2-7 | §4 |
| 13–16 | SCK, SI, SO, CS | J2 via 33 Ω | §4 |
| 17 | RESET | RST_N via 1 kΩ | §2 |
| 18 | VDD | VIO | §3 |

## 6. Firmware notes (for the testing procedure)
- Issue SPI RESET (0xC0) after power-up; wait ≥ 128 OSC cycles.
- Set CNF1–3 per §1.2; disable CLKOUT (CANCTRL.CLKEN = 0) unless calibrating.
- Errata #5 (DS80179G): raise CS within 47 µs after a TXREQ write / RTS.
- INT is push-pull: one INT line per module.

## 7. Verification results (Sonnet, reviewed by Opus)
| ID | Result | Status |
|---|---|---|
| V-01 | Crystal: CL "10/20 pF or specify" (sheet) vs 9 pF (listing); drive 10–200 µW; ESR 50 Ω; C0 3 pF | closed — 10 pF caps, bench trim |
| V-02 | RF internal (Fig 8-1) | closed |
| V-03 | t_PROP(LOOP2) 110 typ / 175 max ns | closed — bit timing revised |
| V-04 | BFPCTRL = 0x00 (RXnBF high-Z), TXRTSCTRL inputs, CANCTRL = 0x87 (CLKOUT /8 on) | closed |
| V-05 | All CNF values and rules verified bit by bit | closed |
| V-06 | TCAN1042 IOS(SS_DOM) 100 mA max → +5V worst case ≈ 100 + 2 + 12 = 114 mA < 150 mA design | closes Phase 3 U-10 |

## 8. Pin-by-pin review (Opus) — `mcp2515_block_netlist.md` vs DS20001801K Table 1-1 (SOIC)
All 18 U1 pins match: 1 TXCAN, 2 RXCAN, 3 CLKOUT, 4–6 TXnRTS (NC), 7 OSC2, 8 OSC1, 9 VSS, 10 RX1BF (NC), 11 RX0BF (NC), 12 INT, 13 SCK, 14 SI, 15 SO, 16 CS, 17 RESET, 18 VDD. KiCad `Interface_CAN_LIN:MCP2515-xSO` symbol pin numbers also match (Sonnet check). Accepted Sonnet additions: net RST_U (U1 side of R_SER), CS_N pull-up placed on the U1 side of its 33 Ω so it holds CS high even with J2 unplugged. Y1 pads 1/3 crystal, 2/4 GND. No issues open.

## 9. Parts added this phase (all JLC Basic)
10 pF C0G 50 V 0603 C1634 ×2; 10 nF X7R 50 V 0603 C57112 ×1; 33 Ω 1 % 0603 C23140 ×5; 1 kΩ C21190 ×1; 0 Ω 0603 C21189 ×1 (R_S); 10 kΩ C25804 ×2; 100 nF C14663 ×1.
