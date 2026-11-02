# Phase 3 — Power Supply Engineering

Status: **Opus design PROPOSED, awaiting user approval** (2026-11-02). Sonnet deliverables reviewed: `power_budget.xlsx`, `power_stage_netlist.md`, `research/f1_ptc_and_filter_parts_extract.md`.
Evidence: `research/power_stage_extract.md` (Sonnet, URLs + pages), earlier extracts. Tags: **[DS]**, **[CALC]**, **[ASSUME]**, **[TEST]**, **[LIST]** (JLC listing only), **(curve)** read from a datasheet graph.

---

## 1. Requirements recap
| Item | Value |
|---|---|
| Input | 9 V adapter, operating 7–16 V, survive −16 V reverse, survive surge/hot-plug (DR-01) |
| Rails | `+5V` (transceiver VDD, LED, optional 3.3 V LDO); `+3V3` optional (VIO only via SJ1); `VIO` from host (not generated here) |
| Load (design) | 100 mA continuous, 150 mA max on +5V |

## 2. Current budget **[DS] / [CALC]**
| Load | Rail | Typ | Max | Source |
|---|---|---|---|---|
| TCAN1042HGV VCC, dominant into 60 Ω | +5V | 40 mA | 70 mA | SLLSES7D §7.7 p9 |
| TCAN1042HGV VCC, recessive | +5V | 1.5 mA | 2.5 mA | same |
| Power LED (KT-0603R, 1.5 kΩ) | +5V | 2 mA | 2.1 mA | (5 − 1.8)/1.5 k |
| XC6206 → VIO (only if SJ1 closed, no host) | +5V | 6 mA | 12 mA | MCP2515 IDD 10 mA max (DS20001801K Table 13-1) + TCAN IIO 0.3 mA + pull-ups ~1 mA + XC6206 Iq 1 µA |
| **+5V worst case** | | | **≈ 85 mA** | sum of max |
| Design value | | | **100 mA cont. / 150 mA** | ≥ 15 % margin + bus-fault short-circuit current **[ASSUME, to verify against TCAN1042 IOS in Phase 5]** |
| Host VIO load (normal use, SJ1 open) | VIO | ~6 mA | 10.8 mA | MCP2515 10 mA + TCAN IIO 0.3 mA + pull-ups |

## 3. LDO vs buck decision
### 3.1 LDO dissipation (5 V out, 100 mA, after 0.34 V Schottky) **[CALC]**
| VIN | P_LDO | ΔT @ 64 °C/W (SOT-223, JEDEC 2s2p) | ΔT @ 90 °C/W (realistic 2-layer) | TJ at 85 °C ambient |
|---|---|---|---|---|
| 9 V | 0.37 W | 23 K | 33 K | 118 °C |
| 12 V | 0.67 W | 43 K | 60 K | 145 °C |
| 16 V | 1.07 W | 68 K | 96 K | **153–181 °C — exceeds 150 °C** |

Only 40 V-class LDOs survive the 29.2 V TVS clamp; of those stocked at JLC (TPS7B6950-Q1 SOT-223, NCV8664C SOT-223, TPS7B8150 DGN) all fail the 16 V / 85 °C corner thermally. TPS70950 (30 V recommended max, 0.8 V from the clamp) and LM2931A (100 mA, 29.5 V OV-shutdown) were rejected **[DS, power_stage_extract §3]**.

### 3.2 Buck (AP63205WU-7)
| Check | Value | Source |
|---|---|---|
| VIN range / abs max | 3.8–32 V / 35 V DC, 40 V for 400 ms | DS41326 p4 **[DS]** |
| vs TVS clamp 29.2 V | 5.8 V margin to abs max | **[CALC]** |
| Output | fixed 5.0 V ±1 % | p5 **[DS]** |
| fsw | 1.1 MHz, ±6 % spread spectrum | p5 **[DS]** |
| Min on-time at worst VIN | D = 5/28.86 = 0.173 → t_on 158 ns > 80 ns t_on(min) | **[CALC]**, p5 |
| Low-VIN | at 7 V in: 6.66 V after D1, D = 0.75; device runs ~100 % duty in dropout, dropout ≈ 0.15 A × (0.125 + 0.1) Ω ≈ 34 mV | p14 **[DS]** + **[CALC]** |
| Efficiency | 91.8 % @ 12 V / 100 mA; 90.1 % @ 10 mA (curve, Fig. 2 p2); 9 V curve NOT in datasheet → assume ≥ 90 % **[ASSUME]** | **(curve)** |
| Loss at 12 V, 100 mA | 0.5 W × (1/0.918 − 1) = **45 mW**; ΔT = 45 mW × 89 °C/W = **4 K** | **[CALC]** |
| Iq | 22 µA | p5 |
| Soft-start | 4 ms | p5 |

**Decision D-021: AP63205WU-7 buck converter (confirms D-014).** At worst case it dissipates ~25× less than the best LDO, leaves 5.8 V margin to its abs max at the TVS clamp, needs only 5 external parts, and its spread-spectrum 1.1 MHz switching is far above the CAN (≤ 1 Mbit/s) and SPI bands' sensitive points. The noise cost is handled by layout (§7) and output filtering; the transceiver's PSRR at 1.1 MHz is not specified **[TEST: measure +5V ripple and CAN eye]**.

## 4. Input protection chain (J1 → … → buck VIN)
`J1 → F1 (PTC) → D1 (SS34) → node VIN_PROT: D2 (SMAJ18A) + C1 (100 µF 35 V Al, damping) → FB1 (bead) → node VIN_BUCK: C2, C3 (2 × 10 µF 50 V X5R 1206) + C4 (100 nF) → U3 VIN`

### 4.1 Reverse polarity — D1 SS34 (D-006r)
- VRRM 40 V ≥ 16 V reverse, and ≥ 29.2 V clamp seen when forward **[DS]** MDD SS34 p1.
- Vf ≈ 0.34 V at 100 mA, 0.36 V at 200 mA **(curve, Fig. 1 p2)**; worst-case input current 125 mA (7 V in, 150 mA out, 90 %) → P = 0.35 × 0.125 = **44 mW**, ΔT ≈ 3 K (70 °C/W) **[CALC]**.
- Reverse leakage 0.5 mA max at 40 V, 25 °C — harmless.

### 4.2 Over-current — F1 PTC (new, D-022)
Added because a sustained over-voltage (e.g. 24 V adapter) drives the TVS into continuous conduction; without a fuse the TVS overheats. Requirements: I_hold ≥ 125 mA **at 85 °C** (PTC hold current roughly halves at 85 °C, flagged by Sonnet — so ≥ 0.35 A rated at 25 °C), I_trip ≤ 0.75 A, V_max ≥ 30 V, 1206, R ≤ 2 Ω. **Selected: Bel Fuse 0ZCJ0035AF2E** (C6378052, Ext): 0.35 A hold / 0.75 A trip, 30 V, R 0.3–1.2 Ω, 0.1 s trip at 8 A **[DS]**; hold ≈ 0.15–0.17 A at 85 °C ≥ 125 mA. Drop ≤ 0.15 V at 125 mA **[CALC]**. Alternate BSMD1206-035-30V (C910829). Behaviour: sustained OV > ~20 V → TVS conducts → PTC trips. If the TVS fails, it fails short, which is the safe direction (the PTC then holds the board unpowered). **Documented limitation: sustained inputs above 18 V are protected, not operated.**

### 4.3 Transient — D2 SMAJ18A
VWM 18 V, VBR 20.0–22.1 V, Vc 29.2 V @ 13.7 A, 400 W 10/1000 µs **[DS]** Littelfuse SMAJ p2. Downstream ratings vs 29.2 V: SS34 40 V, C1 35 V, C2/C3 50 V, AP63205 35 V (40 V/400 ms) — all pass **[CALC]**.

### 4.4 Hot-plug and input filter
- An adapter cable (~2 µH **[ASSUME]**) into ~5 µF of effective ceramic forms an under-damped LC with Z0 = √(L/C) ≈ 0.63 Ω; undamped hot-plug overshoot can approach 2 × VIN **[CALC]**.
- **C1 = 100 µF 35 V Al electrolytic, |Z| ≤ 0.36 Ω @ 100 kHz** (FZ100UF35V90RV0064, C2992596; 44 V surge, 280 mA ripple **[DS, KNSCHA p1]**) damps it: ESR ≈ Z0 and C1 ≫ 4 × C_ceramic (Middlebrook damping rule) **[CALC]**. Also gives low-frequency hold-up.
- **FB1 = GZ2012D601TF** (Basic, 0805, 600 Ω @ 100 MHz, 500 mA, 0.3 Ω DCR **[LIST]**): with C1 on the connector side and C2–C4 on the buck side it forms a π-filter that keeps switching harmonics off the adapter cable. Drop 38 mV at 125 mA **[CALC]**. Sonnet read the impedance curve: ≈ 5 Ω at 1 MHz, 52 Ω at 10 MHz, peak ≈ 670 Ω near 130 MHz **[DS, Sunlord catalogue p10]** — so the π-filter attenuates harmonics above ~20 MHz, not the 1.1 MHz fundamental (which the 100 µF + ceramics handle). A 0 Ω 0805 can replace it if the bead causes instability **[TEST]**.

### 4.5 Buck input capacitors
- Datasheet: ceramic ≥ 10 µF, RMS rating > ½ I_load (p13).
- **C2, C3 = 2 × 10 µF 50 V X5R 1206 (CL31A106KBHNNNE, C13585, Basic)**: effective 2 × 4.4 = 8.8 µF at 9 V, 2 × 2.3 = 4.6 µF at 16 V (Samsung DC-bias data) **[SIM]**. 50 V rating > 29.2 V clamp. RMS current ≈ I_o·√(D(1−D)) = 0.15 × 0.5 = 75 mA — trivial **[CALC]**.
- **C4 = 100 nF 50 V X7R 0603** directly at pins VIN–GND for the high-frequency hot loop.
- Note: at 16 V effective ceramic (4.6 µF) is below 10 µF nominal; acceptable because C1 provides bulk and the load is 7.5 % of the 2 A design point **[ASSUME, TEST input ripple]**.

## 5. Buck power stage values
| Ref | Part | Value | Justification |
|---|---|---|---|
| L1 | Sunlord SWPA4020S4R7MT (C48427, Ext) | 4.7 µH, Isat 2.35 A, Irms 1.34 A, DCR 98 mΩ max, shielded 4×4×2 mm | Datasheet value 4.7 µH (Table 3 p13), DCR < 100 mΩ advice (p13). Ripple ΔI_L = V_o(V_in−V_o)/(V_in·L·f) **[DS Eq. 7]**: 0.24 A @ 7 V, 0.41 A @ 9 V, 0.66 A @ 16 V, 0.80 A @ 29.2 V **[CALC]**. At ≤ 150 mA the stage is in DCM/PFM; peak current is set by the 450 mA PFM clamp or ≤ 0.55 A ripple peak — 4× below Isat. During an output short the 3.1 A limit exceeds Isat briefly; hiccup mode limits stress **[DS p11]**. Shielded = less H-field into crystal/CAN area. |
| C5, C6 | Samsung CL31A226KAHNNNE (C12891, Basic) | 2 × 22 µF 25 V X5R 1206 | Datasheet 2 × 22 µF (p2). Effective 2 × 13.6 = 27.2 µF at 5 V **[SIM]**. Ripple capacitive term ΔI/(8·f·C) = 1.1 mV @ 7 V … 2.8 mV @ 16 V **[CALC]**; ESR term < 2 mV. PFM-mode ripple at light load is larger and must be measured **[TEST]**. Transient Eq. 10 needs ≥ 1.4 µF for 100 mV overshoot at the 29.2 V / 150 mA corner **[CALC, corrected after Sonnet cross-check]** — 19× margin. |
| C7 | 100 nF 50 V X7R 0603 (C14663, Basic) | BST–SW | Datasheet (p14). |
| EN | tied to VIN_BUCK | — | EN abs max 35 V allows direct tie (p4); UVLO 3.5 V rising / 3.06 V falling. Transceiver VCC UV (protected mode below ~4 V) covers brown-out. |
| FB | to +5V at C5, Kelvin trace | — | Fixed-output version; FB routed from the output cap, away from SW (layout bullet 5). |

## 6. Derived rails
### 6.1 +5V distribution
- U2 VCC: 100 nF X7R 0603 at pin + 10 µF 25 V X5R 0805 (C15850, Basic; 4.97 µF effective at 5 V **[SIM]**) — supplies the 70 mA dominant-edge current step locally.
- LED1 + R 1.5 kΩ: 2 mA.
- Test point TP_5V.

### 6.2 Optional +3V3 — U4 changed to XC6206P332MR (D-015r)
| Check | Value |
|---|---|
| VIN | 1.8–6 V operating, 7 V abs max **[DS, Torex XC6206 via LCSC mirror]** vs +5V — OK (+5V cannot exceed 5.05 V) |
| Iout | ≥ 200 mA vs 12 mA load |
| Dropout | 250 mV typ / 680 mV max @ 100 mA → irrelevant with 1.7 V headroom |
| P_diss | 1.7 V × 12 mA = 20 mW **[CALC]** |
| Caps | CIN 1 µF, CL 1 µF ceramic per typical application; fitted: CIN = 1 µF 50 V 0603 (C15849, Basic, 0.66 µF eff. at 5 V) + the +5V rail caps; COUT = 10 µF 25 V 0805 (C15850, Basic, ≈ 7 µF eff. at 3.3 V) **[SIM]** |
| Why change | Phase-2 alternate now verified; Basic part (no Extended fee) vs TLV75533 Extended. Same SOT-23 class. **Pinout differs (1 VSS, 2 VOUT, 3 VIN)** — schematic must use the XC6206 pin map. |

**SJ1 rule (silkscreen):** close SJ1 *only* when no host is driving J2-VIO; otherwise the host rail and the LDO output fight. A series Schottky was considered and rejected: its 0.3 V drop would push VIO to 3.0 V, the bottom of DR-02.

### 6.3 VIO (from host)
Local bulk 10 µF 0805 (C15850) + 100 nF at J2 to absorb host-cable inductance; per-IC 100 nF at U1 VDD and U2 VIO.

## 7. Layout and copper-pour rules for the power stage (feeds Phase 7)
From AP63205 PCB layout section p15 (bullets 1–6, Fig. 25) plus project rules (spec §6):
1. Hot loop C4/C2 → VIN pin → GND pin closed within ~3 mm on top layer; C4 the closest part.
2. SW node: minimum copper area that carries the current (short, ≥ 0.5 mm wide stub to L1); no pour flood on SW; no other nets under SW or L1 on the bottom except solid GND.
3. VIN_BUCK, +5V and GND as top-layer pours around U3 per Fig. 25; GND pour between C2 and C5 with ≥ 4 stitching vias (0.3/0.6) to the bottom plane.
4. Bottom layer under U3/L1/C2–C6: unbroken GND.
5. FB trace from C5 positive pad, routed away from SW/L1.
6. Keep U3/L1 ≥ 10 mm from Y1 (crystal) and ≥ 5 mm from the CAN pair **[ASSUME]**; L1 shielded and orientation of its start-of-winding (dot) toward U3 SW.
7. VIN_PROT pour from J1 → F1 → D1 → D2 → C1 wide (≥ 1 mm); D2 cathode/anode pads with ≥ 2 GND vias at the pad.
8. Thermal: no extra thermal design needed (largest dissipation 45 mW); pours give margin.

## 8. 2-layer adequacy (D-009 check)
Hot loop fits in < 3 × 3 mm with TSOT-26 + 1206 caps on top, solid ground beneath on the bottom (1.6 mm away — larger loop than 4-layer, but the switched current is ≤ 0.55 A peak, not 2 A). **Decision: 2-layer confirmed** **[ASSUME → TEST: near-field probe / CAN eye with buck running]**.

## 9. Power-up sequence and fault behaviour
| Scenario | Result |
|---|---|
| 9 V applied, no host | +5V up in ~4 ms soft-start; U2 sees VCC but no VIO → "Protected" mode, bus high-Z (TCAN1042 Table 9-2) **[DS]**. MCP2515 unpowered. |
| Host (VIO) applied, no 9 V | MCP2515 runs on VIO; U2 VCC absent → bus high-Z **[DS]**; host can still talk SPI (loopback mode works). |
| Reverse 9 V | D1 blocks; no current; TVS never forward-biased. |
| 24 V adapter | TVS conducts above ~20 V → F1 trips; board unpowered. |
| Output short on +5V | AP63205 current limit 2.5–3.1 A, hiccup **[DS p11]**; F1 hold 0.25 A at the input side limits input power. |
| CAN pin shorted to +16 V | handled by U2 ±70 V and D3 (Phase 5). |

## 10. Unresolved / to test
| ID | Item |
|---|---|
| U-07 | AP63205 efficiency at 9 V not plotted — measure **[TEST]** |
| U-08 | PFM light-load output ripple at 2–10 mA (CAN idle) — measure; add 0 Ω→bead option on transceiver VCC if > 50 mV p-p **[TEST]** |
| U-09 | ~~PTC selection~~ closed: 0ZCJ0035AF2E |
| U-10 | Transceiver bus short-circuit current IOS to confirm 150 mA design value (Phase 5) |
| U-11 | ~~Bead/electrolytic datasheets~~ closed (values confirmed) |
| U-12 | J1 polarity: barrel jacks are assumed centre-positive. Many 9 V adapters (e.g. guitar pedal) are centre-negative — D1 makes that harmless (module simply stays off); silk will state "⊕ centre" |
| U-13 | New symbol/footprint needed: TCAN1042 symbol, PJ-002AH footprint (Phase 6/7) |
