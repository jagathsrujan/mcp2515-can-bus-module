# Final Design Validation Report — MCP2515 CAN Bus Module, Rev A

Author: Opus (lead hardware engineer) · Date: 2026-11-03 · Status: **Validated for prototype fabrication, subject to the open items in §9**

Design record: `../ENGINEERING_SPEC.md` (decisions D-001…D-032, change log) · Phase reports `../phase2…phase7_*.md` · Research extracts `../research/*.md` · KiCad project `../../kicad/MCP2515_CAN/`.

---

## 1. Requirement traceability
| ID | Requirement | Implementation | Evidence | Status |
|---|---|---|---|---|
| R-01 | MCP2515 controller + SPI | U1 MCP2515-I/SO, SPI ≤ 10 MHz | DS20001801K Table 13-6; phase4 | ✓ |
| R-02 | Transceiver, up to 1 Mbit/s | U2 TCAN1042HGVDR (5 Mbit/s capable) | SLLSES7D; phase5 | ✓ |
| R-03 | 9 V DC supply | J1 barrel (+J5 DNP terminal), 7–16 V operating | phase3 | ✓ |
| R-04 | Regulation | U3 AP63205 5 V buck; U4 XC6206 3.3 V (optional) | phase3 | ✓ |
| R-05 | 3.3 V and 5 V hosts | Host-referenced VIO domain (MCP2515 VDD + TCAN VIO = host rail, 3.0–5.5 V) — no level shifters | D-001; phase4 §4 logic table | ✓ |
| R-06 | CAN_H/CAN_L connection | J3, J4 (daisy-chain) H/L/GND | phase5 | ✓ |
| R-07 | Configurable termination | Split 2 × 60 Ω + 4.7 nF, both legs switched by JP1 | phase5 §3 | ✓ |
| R-08 | Reverse-polarity + transient | D1 SS34, D2 SMAJ18A, F1 PTC, C1 damping | phase3 §4 | ✓ |
| R-09 | CAN ESD | D3 PESD2CANFD27V (±20 kV contact) | phase5 §4 | ✓ |
| R-10 | Filtering/decoupling | Every IC decoupled per datasheet; π-filter on input | phase6 review §4 | ✓ |
| R-11 | JLC manufacturable | Rules with margin, DRC 0, Economic PCBA single-sided | spec §6; phase7 review | ✓ |
| R-12 | Copper pours | GND both layers, local power pours, 126+ stitching vias, crystal guard | phase7 | ✓ |

## 2. Electrical compatibility (summary of verified links)
| Link | Driver → receiver | Margin |
|---|---|---|
| Host → U1 SPI | host VOH ≈ VIO → VIH 0.7·VIO | same rail ✓ |
| U1 → host (MISO, INT) | VOH ≥ VIO − 0.7 V | ✓ |
| U1 TXCAN → U2 TXD | VOL 0.6 V / VOH VDD−0.7 V vs VIL 0.3·VIO / VIH 0.7·VIO | ✓ (load < 0.2 mA) |
| U2 RXD → U1 RXCAN | VOL ≤ 0.3 mV at 1 µA load vs VIL 0.15·VDD | ✓ (calc, phase4 §4) |
| U2 STB | 10 kΩ ↓ vs 2–20 µA pull-up → ≤ 0.2 V vs VIL 0.9 V | ✓ |
| Bus | VOD 1.5–3.0 V into 50–65 Ω; ±30 V CM; ±70 V fault | ✓ |

## 3. Power supply operation
| Item | Value |
|---|---|
| +5V load | 114 mA worst case (incl. 100 mA bus-short IOS) vs 150 mA design; buck rated 2 A |
| Buck loss / ΔT | ≈ 45 mW / ≈ 4 K |
| Input current | 125 mA worst case (7 V, 150 mA) — F1 holds ≥ 150 mA at 85 °C |
| Abs-max chain at TVS clamp 29.2 V | SS34 40 V · C1 35 V · C2/C3 50 V · AP63205 35 V (40 V/400 ms) — all pass |
| Reverse 16 V | blocked by D1 (VRRM 40 V) |
| Sustained > ~20 V | D2 conducts → F1 trips (protected, not operated) |

## 4. Component ratings (derating summary)
| Part | Stress | Rating | Margin |
|---|---|---|---|
| D1 SS34 | 0.125 A, 16 V reverse | 3 A, 40 V | ≫ 2× |
| D2 SMAJ18A | VWM vs 16 V operating | 18 V | 1.13× (by design: lowest clamp) |
| F1 | 125 mA | 150–170 mA hold @ 85 °C | 1.2× (hot) |
| C1 | 29.2 V clamp | 35 V (44 V surge) | 1.2× |
| C2/C3 | 29.2 V | 50 V | 1.7× |
| C5/C6 | 5 V | 25 V | 5× (DC-bias accounted: 27.2 µF effective) |
| L1 | ≤ 0.55 A peak | Isat 2.35 A | 4× |
| U3 | 29.2 V | 35 V | 1.2× |
| U4 | 5 V | 6 V op / 7 V abs | 1.2× |
| R11–R14 | 19 mW | 100 mW | 5× (normal); fails under bus-to-supply fault with JP1 fitted (documented) |
| D3 | 27 V standoff | — | 24 V nominal systems only |
| JP1 header | 25 mA | 3 A | ≫ |
| Thermal | max single-part dissipation ≈ 45 mW | — | no thermal design needed |

## 5. Oscillator
16 MHz YXC 3225 (±10 ppm, ±20 ppm T), 10 pF C0G ×2, RF internal, R4 0 Ω option. FOSC ≤ 25 MHz at 2.7–5.5 V ✓. Bit timing exact at 125k/250k/500k/1M; tolerance allowance 3 100–4 900 ppm vs ≈ 50 ppm budget ✓. Drive level ≈ 99 µW at 3.3 V; ≈ 227 µW (pessimistic) at 5 V vs 200 µW max → **bench check**.

## 6. CAN termination & bus
120 Ω ± 1 % differential when both JP1 shunts fitted; fully disconnected when removed. Common-mode filter corner 1.13 MHz. Max bus length (worst-case loop delay 175 ns, controller delay excluded): 1M ≈ 15 m (use ≤ 10 m), 500k 52 m, 250k 140 m, 125k 315 m.

## 7. Protection circuitry — placement
F1 → D1 → D2/C1 → FB1 in flow order at J1; D2 GND on vias at pad. D3 at J3/J4 with 2 + 2 GND vias, first element on the bus (routed length to D3 19–26 mm — accepted, phase7 R-1b).

## 8. Signal routing & manufacturability
DRC 0 / parity 0 / ERC 0. Crystal 28 mm from L1; buck hot loop ≈ 4 mm²; decoupling ≤ 2 mm; solid bottom GND; CAN pair matched. JLC: track/space 0.15/0.15 min (0.2 default), via 0.3/0.6, edge 0.5, silk ≥ 1 mm, 3 fiducials, all SMD top (Economic PCBA eligible), board 70 × 50 mm (≤ 100 × 100 tier). Manufacturing package and CPL rotation table: `../manufacturing/` (see §10 review).

## 9. Risk register — every unresolved item
| ID | Risk | Likelihood | Impact | Mitigation / action | Owner |
|---|---|---|---|---|---|
| K-01 | Crystal overdrive at VIO = 5 V (calc. 227 µW vs 200 µW max) | Medium | Crystal aging / frequency shift | Measure on bench; fit R4 ≈ 1 kΩ if needed; re-check start-up at 3.0 V | Bench |
| K-02 | Crystal CL ambiguity (9 pF listing vs 10/20 pF sheet) | Medium | ≤ ±30 ppm offset | Measure CLKOUT; trim C15/C16 | Bench |
| K-03 | Buck PFM ripple at light load not characterised (no 9 V curve) | Low | Transceiver noise | Measure +5V ripple at idle and 1 Mbit/s traffic | Bench |
| K-04 | 1 Mbit/s bus length limited (MCP2515 internal delay unspecified) | Medium | Bit errors on long buses | Use ≤ 10 m at 1 Mbit/s; 500k for longer | User |
| K-05 | D3 standoff 27 V — 24 V vehicle at 28 V charge voltage would conduct | Low (hobby use) | D3 failure | Don't connect to 24 V systems' charging bus | User |
| K-06 | Termination resistors fail if a bus line is shorted to 9–16 V with JP1 fitted | Low | Resistor burn | Documented; transceiver survives ±70 V | User |
| K-07 | Sustained input > 18 V trips F1; TVS may fail short (safe) | Low | Board off | Use 7–16 V supplies only | User |
| K-08 | Connector→D3 routed length 19–26 mm | Low | Reduced ESD robustness | Optional IEC 61000-4-2 ±8 kV test | Bench |
| K-09 | SJ1 closed with host VIO connected → rail contention | Medium (user error) | Host/LDO stress | Silk + doc warning; SJ1 default open | User |
| K-10 | INT is push-pull — cannot be shared between modules | Medium (user error) | Contention | One INT per module (doc) | User |
| K-11 | Centre-negative 9 V adapters (e.g. pedal supplies) | Medium | Module stays off (no damage) | Silk "+ centre" | User |
| K-12 | U1 moved to C153782 (4 875 stock); several Extended parts with modest stock | Medium | Order delay | Check stock at order time; reel alternates listed | Order |
| K-13 | JLC CPL rotations for some packages uncertain | Medium | Mis-placed parts | **Confirm every rotation in JLC's placement preview before paying** | User |
| K-14 | JLC Extended-part fee value unverified (~$3 each, ~13 unique) | — | Cost | Check at order time | User |
| K-15 | KiCad footprints from datasheet drawings (PJ-002AH, KF128, ACT45B) — no physical fit test yet | Low | Re-work | Check first boards; L2 is DNP | Bench |
| K-16 | MCP2515 errata #5 (frame repeat if CS held low after TXREQ) | Low | Duplicate frames | Use standard libraries / raise CS promptly | Firmware |
| K-17 | Thermal/EMC not tested (no emissions pre-scan) | Unknown | Compliance | Pre-scan if the module goes into a product; CMC footprint available | Bench |

## 10. Manufacturing package review (Opus)
Package: `../manufacturing/` (Sonnet). Design files unchanged during export (PCB md5 identical).

| Check | Method | Result |
|---|---|---|
| Gerber set | file list | F/B Cu, Mask, Paste, Silk, Edge.Cuts, job file, Protel extensions — complete for 2-layer ✓ |
| Export settings | JLC KiCad guide (https://jlcpcb.com/help/article/how-to-generate-gerber-and-drill-files-in-kicad-9) | X2 on per JLC's current guide; mask subtracted from silk ✓ |
| Outline | parsed Edge.Cuts gerber | 70 × 50 mm (X 100–170, Y −100…−50 in absolute board coords) ✓ |
| Drill alignment | parsed PTH/NPTH Excellon vs Edge.Cuts | same absolute origin; NPTH (103.5, −53.5) and (166.5, −97.0) inside outline at the H1/H2 positions ✓ |
| Drill sizes | drill report | PTH 0.30 × 150 (vias), 1.0 × 15 incl. 3 slots (J1), 1.4 × 8 (KF128 J3/J4/J5); NPTH 3.2 × 2 — smallest 0.3 mm = no surcharge ✓ |
| JLC BOM | inspected | Comment / Designator / Footprint / LCSC; DNP (J5, L2), TPs, holes, fiducials, SJ1, shunts excluded ✓ |
| CPL | inspected + rule DB | mm, bottom-left aux origin, Y up; corrections from the community rotation DB applied to U1, U2, U3, U4, D3, C1 (SOIC 270, TSOT 180, SOT-23 −90, CP_Elec 180) — consistent with that DB ✓. **Diodes D1/D2, LED D4, crystal Y1 and THT parts have no DB rule → must be checked in JLC's 3D placement preview.** A reversed D2 would short the input; a reversed D1 would leave the board unpowered. |
| Stock | JLC API (2026-11-03) | all assembled parts ≥ 50 in stock; **U1 C12368 flagged "no longer manufactured" by JLC (896 left)** — replacement listing C153782 MCP2515T-I/SO (same device, tape & reel, 4 875 stock). L2 stock 0 (DNP, no effect). |
| Cost | Sonnet estimate | ≈ $8.50 components/board (≈ $42.5 for 5), excl. PCB, assembly, ~12 Extended setup fees |

**Verdict:** manufacturing package is complete and consistent. **U1 switched to C153782 (MCP2515T-I/SO) on 2026-11-03 at the user's request** — schematic, PCB fields, JLC BOM, BOM_final and BOM_master updated; ERC 0, DRC/parity 0 re-verified. Remaining order-time action: verify every polarity/rotation in JLC's placement preview (risk K-13).

## 11. Deliverables index (brief §6)
| # | Deliverable | File(s) |
|---|---|---|
| 1 | Architecture & block diagram | `../ENGINEERING_SPEC.md` §2 |
| 2 | Component comparison & selection | `../phase2_component_selection.md` |
| 3 | Datasheet reference document | `../research/ic_datasheet_extract.md`, `discrete_datasheet_extract.md`, `power_stage_extract.md`, `f1_ptc_and_filter_parts_extract.md`, `jlcpcb_capabilities.md` |
| 4 | Power supply calculations | `../phase3_power_supply.md`, `../power_budget.xlsx` |
| 5 | Electrical calculations | `../phase4_mcp2515_oscillator.md`, `../phase5_can_phy.md` |
| 6 | Full schematic | `../schematic/MCP2515_CAN_schematic.pdf` |
| 7 | KiCad project | `../../kicad/MCP2515_CAN/` |
| 8 | PCB layout | `../../kicad/MCP2515_CAN/MCP2515_CAN.kicad_pcb`, `../pcb/` renders, STEP |
| 9 | Final BOM with MPNs | `BOM_final.csv/.xlsx`, `../manufacturing/MCP2515_CAN_RevA_BOM_JLC.csv` |
| 10 | Electrical verification report | `../phase6_schematic_review.md` + this document |
| 11 | PCB design review | `../phase7_pcb_review.md` |
| 12 | Testing & commissioning | `testing_commissioning.md`, `connector_pinouts.md`, `design_notes.md` |
| — | Manufacturing files | `../manufacturing/` (Gerber zip, BOM, CPL, rotation review, stock check) |
