# Phase 6 — Schematic Electrical Review (Opus)

Status: **Schematic APPROVED by Opus, awaiting user approval** (2026-11-03)
Project: `kicad/MCP2515_CAN/` (KiCad 10.0.8) — root + 3 hierarchical sheets (Power, Controller, CAN PHY), 73 components, 43 nets.
Outputs: `schematic/MCP2515_CAN_schematic.pdf`, sheet PNGs, `MCP2515_CAN.net`, `MCP2515_CAN_BOM.csv`, `erc_report.txt`, `netlist_crosscheck.md` (Sonnet), `BOM_diff_vs_master.md` (Sonnet).

## 1. Independent checks performed by Opus
| Check | Method | Result |
|---|---|---|
| ERC | re-ran `kicad-cli sch erc` myself | **0 violations**, no exclusions |
| Annotation | `kicad-cli sch export netlist` warned "annotation errors" | **Fixed:** ref `J1B` (non-numeric suffix) renamed **J5** in schematic and all docs; warning gone, ERC still 0 |
| Netlist | own s-expression parser over a fresh netlist export (not Sonnet's script) | every net below matches the netlist docs |
| Fields | parser checked MPN / Manufacturer / LCSC on every BOM part | none missing |
| Symbol pin maps | read KiCad library files for polarity-sensitive / regulator symbols | see §3 |
| Visual | opened all sheet renders | readable; cosmetic overlaps noted in §5 |

## 2. Net-by-net verification (fresh export)
| Net | Members | vs design |
|---|---|---|
| VIN_RAW | J1.1, J5.1, F1.1 | ✓ |
| VIN_F | F1.2, D1.2 (anode) | ✓ |
| VIN_PROT | D1.1 (cathode), D2.1 (cathode), C1.1 (+), FB1.1, TP1 | ✓ |
| VIN_BUCK | FB1.2, C2, C3, C4, U3.3 IN, U3.2 EN | ✓ EN tied to VIN |
| SW / BST | U3.5, L1.1, C7.2 / U3.6, C7.1 | ✓ |
| +5V | L1.2, C5, C6, U3.1 FB, R1, U2.3 VCC, C12, C13, U4.3 VIN, C8, TP2 | ✓ |
| +3V3 | U4.2 VOUT, C9, SJ1.1, TP3 | ✓ |
| VIO | J2.1, U1.18, U2.5, R2, R10, C10, C11, C14, C17, SJ1.2, TP4 | ✓ (exactly the planned set) |
| GND | 34 pins incl. U1.9, U2.2, U3.4, U4.1, Y1.2/4, D2.2, D3.3, D4.1 (K), J1.2, J2.2, J3.3, J4.3, R15.2 | ✓ |
| SPI | SCK J2.3–R5–U1.13; MOSI J2.4–R6–U1.14; MISO U1.15–R8–J2.5; CS J2.6–R7–U1.16 (+R10 pull-up on U1 side); INT U1.12–R9–J2.7 (+TP7) | ✓ |
| Reset | RST_N: J2.8, R2 (to VIO), C18, R3 → RST_U: U1.17 | ✓ |
| Oscillator | OSC1: U1.8, Y1.1, C15; OSC2: U1.7, R4; OSC2_X: R4, Y1.3, C16 | ✓ (matches Fig 8-1) |
| TXCAN / RXCAN | U1.1–U2.1 (TP8); U1.2–U2.4 (TP9) | ✓ |
| STB | U2.8, R15 (to GND), TP12 | ✓ Normal mode |
| CANH_T / CANL_T | U2.7, R16, L2.1 / U2.6, R17, L2.2 | ✓ |
| CANH / CANL | J3.1, J4.1, D3.1, JP1.1, R16, L2.4, TP10 / J3.2, J4.2, D3.2, JP1.3, R17, L2.3, TP11 | ✓ |
| TERM_H / TERM_L / SPLIT | JP1.2–R11,R12 / JP1.4–R13,R14 / R11–R14 pin 2 + C19 | ✓ both legs switched |
| CLKOUT | U1.3, TP6 only | ✓ intentional |
| No-connects | U1 pins 4, 5, 6, 10, 11; J1 pin 3 | ✓ NC-flagged |

## 3. Symbol / pin-map verification against datasheets
| Part | KiCad symbol | Datasheet | Result |
|---|---|---|---|
| U1 | MCP2515-xSO | DS20001801K Table 1-1 | 18/18 ✓ (also visible on sheet render) |
| U2 | project TCAN1042HGV | SLLSES7D Table 6-1 | 8/8 ✓ |
| U3 | Regulator_Switching:AP63205WU: 1 FB, 2 EN, 3 IN, 4 GND, 5 SW, 6 BST | DS41326 p1 | ✓ |
| U4 | Regulator_Linear:XC6206PxxxMR: 1 GND, 2 VO, 3 VI | Torex pin table | ✓ |
| D1 | Device:D_Schottky 1 = K, 2 = A | cathode to VIN_PROT | ✓ forward-biased in normal polarity |
| D2 | Diode:SMAJ18A (extends SM6T6V8A; unidirectional despite pin names "A1/A2"), pin 1 = cathode end, footprint D_SMA pad 1 = cathode | cathode to VIN_PROT | ✓ |
| D4 | Device:LED 1 = K → GND, 2 = A → LED_A | | ✓ |
| C1 | C_Polarized pin 1 = + → VIN_PROT | | ✓ |
| D3 | NUP2105L-style symbol, pins 1/2 lines, 3 common | Nexperia PESD2CANFD27V | ✓ pin numbers (names differ, cosmetic) |

## 4. Brief-mandated review items
| Item | Finding |
|---|---|
| Pin assignments | All IC pins verified (§3) |
| Supply compatibility | U1/U2-VIO on VIO (3.0–5.5 V); U2 VCC on +5V; U3 IN ≤ 29.2 V clamp vs 35 V abs; U4 IN 5 V vs 6 V max ✓ |
| Current | +5V worst 114 mA vs 150 mA design; VIO ≈ 11 mA from host ✓ |
| Pull-ups/downs | RST_N 10 k↑ + 10 nF; CS 10 k↑ (U1 side); STB 10 k↓; INT push-pull (none needed); TXnRTS internal 100 k ✓ |
| Decoupling | U1: C17; U2: C12, C13 (VCC), C14 (VIO); U3: C2–C4 in, C5–C6 out, C7 BST; U4: C8, C9; VIO entry: C10, C11 ✓ |
| Oscillator | 16 MHz, 10 pF C0G ×2, R4 0 Ω option, RF internal ✓ |
| Protection | F1 → D1 → D2 + C1 → FB1; D3 at bus; split termination switched both legs ✓ |
| Connectors | J1 (centre +), J5 DNP alt, J2 1 VIO 2 GND 3 SCK 4 MOSI 5 MISO 6 CS 7 INT 8 RST, J3/J4 1 CANH 2 CANL 3 GND ✓ |
| Conflicts | Only SJ1 (+3V3 vs host VIO) — by design, documented on schematic note ✓ |

**No electrical issues found. Schematic approved for PCB (Opus).**

## 5. Open items carried to Phase 7 (non-electrical)
| ID | Item |
|---|---|
| P-01 | `BarrelJack_CUI_PJ-002AH` footprint is **provisional** (pad positions from KiCad generic BarrelJack_Horizontal; slot sizes from research doc). Must be checked against the CUI drawing p2 before layout. |
| P-02 | KF128-5.08 drill/pad vs Phoenix MKDS 5.08 footprints (J3, J4, J5) not verified. |
| P-03 | Custom L2 footprint uses derived pad size (1.35 × 0.9 mm) — DNP, low risk. |
| P-04 | Cosmetic: a few overlapping texts on the sheets (Y1 value/GND label, D3 value over JP1 labels, J5 value). Label-stub style is readable but sparse. Tidy before release; no netlist impact. |
| P-05 | Two extra PWR_FLAGs (VIN_BUCK, +5V) added by Sonnet because the regulator/bead symbol pin types don't mark those rails as driven — justified. |
