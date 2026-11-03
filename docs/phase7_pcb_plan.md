# Phase 7 — PCB Layout Plan (Opus → Sonnet)

Status: plan APPROVED for execution (2026-11-03). Board: `kicad/MCP2515_CAN/MCP2515_CAN.kicad_pcb` (currently empty). Schematic is frozen — no electrical changes; report anything that seems to need one.

Inputs: `ENGINEERING_SPEC.md` §6 (JLC rules + copper-pour plan), `phase3_power_supply.md` §7, `phase4_mcp2515_oscillator.md` §1.6, `phase5_can_phy.md` §7, `phase6_schematic_review.md` §5 (P-01…P-03).

---

## 0. Before placement — footprint verification (P-01, P-02)
1. **J1 PJ-002AH**: read the CUI drawing (LCSC datasheet C2961147, p2; or cui.com / sameskydevices.com PJ-002AH). Fix `MCP2515_CAN:BarrelJack_CUI_PJ-002AH` pad/slot positions and sizes (slots plated, finished slot ≥ pin + 0.2 mm). Pin 1 centre (+), 2 sleeve, 3 switch. If the drawing truly cannot be read, use KiCad `Connector_BarrelJack:BarrelJack_CUI_PJ-063AH_Horizontal`-class geometry only if dimensions are confirmed equal; otherwise stop and report.
2. **KF128-5.08 (J3, J4, J5)**: read the Kefa datasheet (LCSC C474952/C474953): pin diameter → finished hole ≥ pin + 0.2 mm, pad ≥ hole + 0.5 mm (0.25 mm annular). Adjust a project copy of the Phoenix 5.08 footprint if needed; record dimensions.
3. Every footprint pin 1 / polarity vs datasheet: U1–U4, D1–D4, C1, Y1, L2 — list in report.

## 1. Board
| Item | Value |
|---|---|
| Outline | rectangle **60 × 45 mm** (may grow to ≤ 70 × 50 if placement rules can't be met — report why), corner radius 1 mm, Edge.Cuts 0.1 mm line |
| Origin | bottom-left corner = (0,0) as aux/grid origin (for CPL) |
| Stack-up | 2 layers, 1.6 mm FR-4, 1 oz, green mask, white silk, HASL lead-free |
| Mounting | H1, H2 M3 (3.2 mm NPTH, `MountingHole_3.2mm_M3`), 3.5 mm from edges at two diagonal corners; 3 mm copper/part keep-out around |
| Fiducials | FID1–FID3 (1 mm Cu / 2 mm mask), ≥ 3.5 mm from edge, 3 non-collinear corners (top side) |

## 2. Design rules (Board Setup + netclasses) — from spec §6
| Rule | Value (mm) |
|---|---|
| Min track / clearance (constraints) | 0.15 / 0.15 |
| Min via drill / diameter | 0.3 / 0.6; min annular 0.15 |
| Min through-hole | 0.3; hole-to-hole 0.5 |
| Copper to edge | 0.5 |
| Silk: min width / text height / clearance to pad | 0.15 / 1.0 / 0.15 |
| Mask expansion / min web | 0.05 / 0.1 |
| Courtyard | courtyard overlap = error |

Netclasses:
| Class | Nets | Track | Clearance | Via |
|---|---|---|---|---|
| Default | all signals | 0.20 | 0.20 | 0.3/0.6 |
| Power | VIN_RAW, VIN_F, VIN_PROT, VIN_BUCK, +5V, +3V3, VIO, GND | 0.50 (neck to 0.3 only at fine-pitch pins) | 0.20 | 0.3/0.6 |
| SW | SW | 0.60 | 0.25 | — (no vias) |
| CAN | CANH, CANL, CANH_T, CANL_T | 0.25, diff-pair gap 0.20 | 0.20 | avoid |

Current check (IPC-2152, 1 oz external, 10 °C rise): 0.5 mm carries ≈ 1.2 A ≫ 0.13 A worst-case input; 0.25 mm ≈ 0.7 A ≫ 0.1 A CAN **[CALC — rule of thumb, margin > 5×]**.

## 3. Placement (all SMD on TOP; THT on top)
Zones (top view, x → right, y ↑):
| Zone | Location | Parts |
|---|---|---|
| A — Power entry | left edge, lower half | J1 (opening to the left edge, body flush/overhang ≤ 1 mm as the footprint intends), J5 (DNP) beside it, F1 → D1 → D2/C1 → FB1 in signal-flow order |
| B — Buck | bottom centre-left | U3, C4 (closest to VIN/GND pins), C2, C3, C7, L1, C5, C6 — per phase3 §7 hot-loop rules; SW copper minimal |
| C — Logic | centre / upper-left | U1, Y1 + C15/C16/R4 within 5 mm of U1 pins 7/8, C17 at pin 18, R2/R3/C18, R5–R10 between U1 and J2 |
| D — Host | top edge, left/centre | J2 (pin 1 VIO marked), C10/C11 at J2-1, U4 + C8/C9 + SJ1 near J2 |
| E — CAN | right side | J3 and J4 on the right edge (wire entry facing out), D3 within 5 mm of both, then JP1 + R11–R14 + C19, R16/R17/L2, U2 with C12/C13/C14 at its pins |
| F — Indicators | near J1/U3 | D4 + R1 (visible), TPs on the edges of their zones |

Hard placement constraints:
1. Y1 ≥ 10 mm from L1/U3 SW node; ≥ 5 mm from the CAN pair.
2. Nothing but GND under Y1/C15/C16 on either layer.
3. U2 within 15 mm of J3/J4; TXCAN/RXCAN short (U1 ↔ U2 on the same side, no crossing of the power zone).
4. Decoupling caps ≤ 2 mm from their pins, GND via at each cap's GND pad.
5. All parts' courtyards ≥ 0.5 mm apart (JLC courtesy spacing); consistent orientation of passives (0° or 90°).
6. SMD bodies ≥ 2.5 mm from board edge except edge connectors.
7. Test pads reachable (not under connectors).

## 4. Routing
1. **No autorouter on** CAN pair, buck loop (VIN_BUCK/SW/+5V/FB), oscillator. Freerouting may be used for the remaining low-speed signals only if available; otherwise route by script.
2. CAN pair J3/J4 → D3 → JP1/termination → R16/R17 (L2) → U2: coupled, 0.25/0.20, length mismatch ≤ 1 mm, top layer, no vias, over solid bottom GND.
3. Buck: C4 → U3 VIN/GND loop closed on top; U3 GND pad + C2–C6 GND with ≥ 4 vias to bottom; FB from C5/C6 + pad (Kelvin), away from SW/L1.
4. Oscillator: OSC1/OSC2/OSC2_X ≤ 5 mm each, symmetric, no vias.
5. SPI: R5–R9 near J2 (SCK, MOSI, CS) and near U1 (MISO, INT) per schematic intent; 0.2 mm traces; avoid running parallel to SW.
6. Bottom layer: keep as unbroken GND — signal jumpers on bottom only where unavoidable, each ≤ 10 mm, never under the CAN pair, Y1, or the buck.
7. 45° corners only; no acute angles; teardrops on THT pads optional.

## 5. Copper pours (user requirement R-12, spec §6)
| Pour | Layer | Net | Settings |
|---|---|---|---|
| Z1 | B.Cu | GND | whole board, priority 0, clearance 0.3, min width 0.25, thermal relief on THT pads (spoke 0.4, gap 0.3), solid on vias & SMD, remove islands always |
| Z2 | F.Cu | GND | fill all free top area, same settings, priority 0 |
| Z3 | F.Cu | VIN_PROT / VIN_BUCK | local pours J1 → FB1 → U3 IN (priority 1), wide surge path |
| Z4 | F.Cu | +5V | local pour from L1/C5/C6 to U2 VCC / U4 (priority 1) |
| Keep-outs | F.Cu+B.Cu | — | no copper pour under SW except GND on bottom; pour edge 0.5 mm from Edge.Cuts; 3 mm around mounting holes |

Stitching vias (GND, 0.3/0.6): along the board edge every ≈ 5 mm, ≈ 8 mm grid elsewhere, ring around Y1 guard, next to every GND pad of decoupling caps, both sides of the CAN pair at ≈ 5 mm pitch (not between the traces), ≥ 2 at D2 and D3 GND pads. Then check for top-layer pour islands/slivers.

## 6. Silkscreen (≥ 1.0 mm text, 0.15 mm stroke, off pads)
- Board: "MCP2515 CAN MODULE  Rev A  2026-11" + "JLCJLCJLCJLC" order-number placeholder near an edge.
- J1: "9V ⊕ centre"; J2 pin names "VIO GND SCK MOSI MISO CS INT RST" + pin-1 mark; J3/J4: "H L G"; JP1: "TERM 120Ω fit both"; SJ1: "3V3→VIO (no host)"; TPs labelled; polarity marks D1, D2, D4, C1; LED "5V".

## 7. Checks Sonnet must run and report
1. `kicad-cli pcb drc` (with all-track-errors, schematic parity on): **0 errors, 0 unconnected**; warnings listed and justified.
2. Courtyard and silk-to-pad violation tools (MCP `get_courtyard_violations`, `get_silk_to_pad_violations`).
3. If available: MCP `dfm_load_manufacturer_profile` (JLCPCB) + `dfm_run_manufacturer_check`, `emc_check_differential_pair_symmetry`, `emc_check_ground_plane_voids`, `emc_check_decoupling_placement`, `emc_check_via_stitching` — report results.
4. Renders: top, bottom, iso 3D (`export_3d_render` or `kicad-cli pcb render`) + per-layer SVG/PNG of F.Cu and B.Cu → `docs/pcb/`.
5. STEP export → `docs/pcb/MCP2515_CAN.step`.
6. Report: board size, list of placements (ref, x, y, rot), any deviation, unresolved items.

## 8. Key checks (Opus will verify)
- DRC/parity clean; all nets routed.
- Measured: Y1–L1 distance ≥ 10 mm; CAN pair length mismatch ≤ 1 mm; decoupling cap distances ≤ 2 mm.
- Bottom GND plane continuity under CAN pair, oscillator, buck.
- Hot loop C4–U3 area small (< 10 mm² enclosed).
- JLC rules met (§2); silk legible and off pads; fiducials and holes placed.
