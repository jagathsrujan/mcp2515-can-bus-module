# Phase 7 — PCB Design Review (Opus)

Status: **PCB APPROVED by Opus after one correction round, awaiting user approval** (2026-11-03)
Board: `kicad/MCP2515_CAN/MCP2515_CAN.kicad_pcb` — 70.1 × 50.1 mm, 2-layer 1.6 mm 1 oz, all SMD on top. Sonnet report: `pcb/layout_report.md`; renders, layer images, STEP in `pcb/`.

## 1. Independent verification (Opus)
| Check | Result |
|---|---|
| DRC `--all-track-errors --schematic-parity` (re-run by Opus after corrections) | **0 violations, 0 unconnected, 0 parity, 0 footprint errors** |
| ERC (schematic after footprint-field update) | 0 |
| Board rules in project file | clearance 0.15, edge 0.5, track 0.15, via 0.3/0.6, annular 0.15, hole-hole 0.5, silk 0.15 / text 1.0 × 0.15 — all ≥ JLC limits with margin (spec §6) |
| Netclasses | Default 0.2/0.2; Power 0.5/0.2; SW 0.6/0.25; CAN 0.25, gap 0.2 — patterns bound to the right nets |
| Board size | 70 × 50 mm (plan allowed ≤ 70 × 50; 60 × 45 could not fit connectors + keep-outs) — still in JLC ≤ 100 × 100 price tier |

## 2. Measured key checks (Opus, pcbnew script)
| Item | Target | Measured | Verdict |
|---|---|---|---|
| Y1 ↔ L1 / U3 | ≥ 10 mm | 28.1 / 25.3 mm | ✓ |
| Y1 ↔ CAN pair | ≥ 5 mm | 11.1 mm | ✓ |
| Oscillator traces | ≤ 5 mm, no vias | Y1.1–U1.8 4.5 mm; Y1.3–R4–U1.7 6.3 mm; no vias; GND guard on top, solid GND below | ✓ (OSC2 side 1.3 mm over, accepted — symmetric load caps at Y1, guard present) |
| Decoupling (pad centre to IC pin) | ≤ ~2 mm edge gap | C17–U1.18 2.31, C12–U2.3 2.38, C14–U2.5 2.35, C4–U3.3 2.18, C8/C9–U4 2.0 mm (pad edges ≤ 1.9 mm) | ✓ |
| Buck hot loop | small | C4–U3 loop ≈ 4.1 mm², SW 0.6 mm, no vias on SW | ✓ |
| SMD courtyard to edge | ≥ 2.5 mm | min 2.7 mm (after D2 moved) | ✓ |
| GND stitching | edge ≈ 5 mm, grid ≈ 8 mm | 126+ GND vias, edge ring visible | ✓ |
| Bottom layer | unbroken GND | one solid pour; only short jumpers (SPI fan-out, STB, RXCAN, INT, RST, CANL × 2 near D3) | ✓ acceptable |
| CAN pair | coupled, matched | J→D3 legs matched 0.00 mm; 4 vias (2 short bottom jumpers forced by alternating H/L/G pin order of identical outward-facing J3/J4) | ✓ acceptable (see §3) |

## 3. Review findings and dispositions
| # | Finding | Disposition |
|---|---|---|
| R-1 | **D3 was 12–18 mm from the connectors with the bus able to branch before it** (first layout) | **Corrected**: D3 moved to the right edge between J3/J4; routing order J3/J4 → D3 → termination → R16/R17 → U2; D3 GND pin on 2 dedicated vias + 2 stitching vias. |
| R-1b | After correction, routed connector→D3 length is 19–26 mm (meanders for zero mismatch) vs the ≤ 5 mm aim — the 5.08 mm pitch with GND between H and L on two connectors makes ≤ 5 mm impossible without moving a connector | **Accepted with rationale**: inductance *in front of* the TVS is in series with the ESD source and does not raise the clamp voltage the transceiver sees; what matters is (a) nothing branches off before D3 ✓, (b) low-inductance TVS GND ✓, (c) more trace between D3 and U2 than between J and D3 is not required. Residual risk: ESD energy radiating from 2–2.5 cm of unclamped trace — mitigated by solid GND beneath. **[TEST: IEC 61000-4-2 ±8 kV contact on J3/J4 if equipment available]** |
| R-2 | D2 courtyard 2.25 mm from edge | **Corrected** (min now 2.7 mm) |
| R-3 | "9V + centre" silk ambiguous between J1 and J5; J5 had no + mark | **Corrected** (J5 "+" at pin 1; text moved to J1) |
| R-4 | Some power tracks 0.3 mm instead of 0.5 mm | Accepted: worst-case current 0.13 A; 0.3 mm 1 oz carries > 0.8 A at 10 °C rise **[CALC, IPC-2152 rule of thumb]** |
| R-5 | No VIN_BUCK pour (only VIN_PROT and +5V pours) | Accepted: FB1 → U3 path is short and 0.5 mm; input current ≤ 0.13 A |
| R-6 | CAN pair coupled at 0.2 mm gap only near U2 | Accepted: at ≤ 1 Mbit/s differential impedance control is not required (rise time ≫ trace delay); both lines run together over solid GND |
| R-7 | KF128 drill was 1.3 mm in the Phoenix footprint vs 1.4 mm required | **Corrected** by Sonnet (project footprints, 1.4 mm drill / 2.6 mm pad), schematic footprint field updated, ERC 0 |
| R-8 | PJ-002AH footprint (P-01) | Verified against CUI drawing p2 by Sonnet; courtyard/silk fixed |
| R-9 | J1 has no 3D model; J3–J5 use the Phoenix model | Cosmetic (renders only) |
| R-10 | MCP `emc_*` checks need a live KiCad GUI session | Not run; manual review above covers the same items |

## 4. Brief-mandated review items
| Item | Verdict |
|---|---|
| Signal integrity | SPI ≤ 10 MHz over short 0.2 mm traces with 33 Ω series R; CAN ≤ 1 Mbit/s matched pair; ✓ |
| CAN differential routing | matched, over GND, D3 first, termination at connector side ✓ (R-1b) |
| Grounding | single solid bottom GND, top GND fill, 126+ stitching vias ✓ |
| Decoupling placement | ✓ (§2) |
| Switching regulator noise | compact hot loop, shielded L1, SW minimal, 25–28 mm from crystal ✓ |
| Oscillator interference | guard pour, no foreign nets beneath ✓ |
| Protection placement | F1/D1/D2/C1 in flow order at J1; D3 at J3/J4 ✓ |
| Thermal | max dissipation ≈ 45 mW (U3), 44 mW (D1) — no thermal concern ✓ |
| Manufacturability | JLC rules with margin; 3 fiducials; M3 holes with keep-out; silk ≥ 1 mm off pads; DFM profile check passed (MCP) ✓ |

## 5. Carried to Phase 8
- Generate Gerbers/drill, JLC BOM + CPL (rotation check in JLC preview), schematic PDF, STEP (exists), bring-up checklist.
- Phase-6 BOM CSV regenerated with the KF128 footprint names (done by Opus).
