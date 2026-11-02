# Phase 6 — Schematic Capture Plan (Opus → Sonnet)

Status: plan APPROVED for execution (2026-11-02). Tool: KiCad 10.0.8 via KiCad MCP (connection checked: CLI + IPC 10.0.8 available).

## 1. Sources of truth (do not re-derive)
Net-level content is fully defined, pin by pin, in:
1. `power_stage_netlist.md` — blocks 1–2 (power input & protection, regulation)
2. `mcp2515_block_netlist.md` — blocks 3, 4, 7 (MCP2515, crystal, SPI header J2)
3. `can_phy_netlist.md` — blocks 5–6 (transceiver, protection, termination, connectors). **JP1 = 2×2 header** (Opus override at top of file).
Values / MPN / LCSC: `BOM_master.csv`. Design reasoning: `phase3…phase5_*.md`. If two sources disagree, stop and report — do not choose.

## 2. Project
- Path: `<project>/kicad/MCP2515_CAN/MCP2515_CAN.kicad_pro` (+ `.kicad_sch`, `.kicad_pcb` empty for now).
- Project libraries: `MCP2515_CAN.kicad_sym` (custom symbols), `MCP2515_CAN.pretty` (custom footprints), registered in project `sym-lib-table` / `fp-lib-table` with `${KIPRJMOD}` paths.
- Title block on every sheet: Title "MCP2515 CAN Bus Module", Rev "A", Date 2026-11-02, Comment1 "9 V in · host-referenced VIO 3.0–5.5 V · JLCPCB", Comment2 "Sheet: <block names>".

## 3. Sheet structure (hierarchical, brief §Phase 6 blocks 1–7)
| Sheet | File | Blocks | Contents |
|---|---|---|---|
| Root | MCP2515_CAN.kicad_sch | — | 3 sheet symbols, design notes text (§7), mounting holes H1/H2, fiducials FID1–3, logo/board info text |
| 1 Power | power.kicad_sch | 1 Power input & protection, 2 Voltage regulation | J1, J1B, F1, D1, D2, C1, FB1, C2–C4, U3, C7, L1, C5, C6, LED (D4) + R1, U4, C8, C9, SJ1, TPs |
| 2 Controller | controller.kicad_sch | 3 MCP2515, 4 Crystal, 7 SPI interface | U1, Y1, C15, C16, R4, C17, C18, R2, R3, R5–R10, J2, C10, C11, TPs |
| 3 CAN PHY | can_phy.kicad_sch | 5 Transceiver, 6 Bus protection & connector | U2, C12–C14, R15, R16, R17, L2, R11–R14, C19, JP1, D3, J3, J4, TPs |

Use graphic rectangles + text headings inside each sheet to mark the brief's numbered blocks (e.g. "① Power input & protection").

Inter-sheet connectivity:
- Rails by **power symbols**: `GND`, `+5V`, `+3V3` (KiCad `power:` lib), and a custom power symbol **`VIO`** in the project library (power-type, global).
- Signals between sheets by **hierarchical labels/sheet pins**: `TXCAN`, `RXCAN` (Controller ↔ CAN PHY). No other signals cross sheets.
- Local net labels for every named net in the netlist docs (VIN_RAW, VIN_F, VIN_PROT, VIN_BUCK, SW, BST, LED_A, OSC1, OSC2, OSC2_X, SCK, MOSI, MISO, CS_N, INT_N, RST_N, SCK_U, MOSI_U, MISO_U, CS_U, INT_U, RST_U, CLKOUT, STB, CANH_T, CANL_T, CANH, CANL, TERM_H, TERM_L, SPLIT). No auto-named nets on anything probed.

## 4. Reference designators (KiCad conventions)
Keep the netlist ref-des except:
| Netlist name | KiCad ref | Note |
|---|---|---|
| LED1 | **D4** | LEDs are D in KiCad; value "RED" |
| R_LED | R1 | already R1 |
| TP_VIN, TP_5V, TP_3V3, TP_VIO, TP_GND, TP_CLK, TP_INT, TP_TXCAN, TP_RXCAN, TP_CANH, TP_CANL, TP_STB | **TP1…TP12** in that order | Value field = the label (e.g. "VIN"); footprint `TestPoint:TestPoint_Pad_D1.5mm`; Exclude from BOM + position files |
| — | H1, H2 | `MountingHole:MountingHole_3.2mm_M3` (NPTH, no net), exclude from BOM |
| — | FID1–FID3 | `Fiducial:Fiducial_1mm_Mask2mm`, exclude from BOM |

## 5. Symbols / footprints
- Standard library symbols where they exist and **pin numbers match the datasheet** (Sonnet verified: `Interface_CAN_LIN:MCP2515-xSO`, `Regulator_Switching:AP63205WU`, `Regulator_Linear:XC6206PxxxMR`, `Device:Crystal_GND24`).
- Create in project lib:
  - `TCAN1042HGV` — copy of `MCP2562-E-SN`, pins 1 TXD (in), 2 GND (power in), 3 VCC (power in), 4 RXD (out), 5 VIO (power in), 6 CANL (bidir), 7 CANH (bidir), 8 STB (in). Footprint SOIC-8_3.9x4.9mm_P1.27mm.
  - `VIO` power symbol.
- Create in project footprint lib:
  - `BarrelJack_CUI_PJ-002AH` from the CUI PJ-002AH drawing p2 (slots 1.0 × 3.5 / 1.0 × 3.0) — pin 1 centre (VIN_RAW), 2 sleeve (GND), 3 switch (NC, no-connect on symbol).
  - `L_CommonModeChoke_TDK_ACT45B` from `can_phy_netlist.md` V-14 dimensions.
- J3/J4: `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-3-5.08_1x03_P5.08mm_Horizontal` — check KF128 drill (datasheet) ≤ footprint drill − 0.1 mm clearance; if not, copy and adjust in project lib. J1B likewise (2-pin).
- Every symbol gets fields: **Value, Footprint, Datasheet (URL), MPN, Manufacturer, LCSC** — from `BOM_master.csv`. DNP parts (J1B, L2) get the DNP attribute and stay in BOM marked DNP. Shunts for JP1 and SJ1 are not schematic parts (note text only).

## 6. Electrical rules
- No-connect flags on: U1 pins 4, 5, 6, 10, 11; J1 pin 3 (switch).
- PWR_FLAG on: `VIN_RAW` (J1), `GND`, `VIO` (at J2 pin 1). Add PWR_FLAG on any other rail only if ERC complains *and* the rail is genuinely driven (e.g. +5V from U3 if its output pin is not power-output type) — list each in the report.
- Junction dots at every T; no 4-way junctions; wires on 1.27 mm (50 mil) grid.
- Run ERC; target **0 errors, 0 warnings**; any waiver must be an ERC exclusion with a comment and listed in the report.

## 7. Notes to place as schematic text (root sheet)
1. "SJ1 (VIO ← +3V3): close ONLY when no host drives J2-VIO."
2. "JP1: fit BOTH shunts (1-2, 3-4) to enable 120 Ω split termination; remove both to disable."
3. "Input: 7–16 V DC, centre-positive. Reverse polarity blocked by D1; sustained >18 V trips F1."
4. "MCP2515 16 MHz bit timing (CNF1/CNF2/CNF3): 1M 00/83/01 · 500k 00/AE/01 · 250k 01/AE/01 · 125k 03/AE/01. Disable CLKOUT (CANCTRL.CLKEN=0)."
5. "L2 DNP: R16/R17 0 Ω fitted. If L2 fitted, remove R16/R17."
6. "R4 (OSC2 series) 0 Ω; fit ~1 kΩ only if crystal overdrive measured at VIO 5 V."

## 8. Outputs required from Sonnet
1. Saved project; ERC report file (`kicad-cli sch erc`) with result.
2. Schematic PDF export (`docs/schematic/MCP2515_CAN_schematic.pdf`) and per-sheet PNG renders.
3. Netlist export (`kicad-cli sch export netlist`) and a **script-generated cross-check** (`docs/schematic/netlist_crosscheck.md`): for every pin in the three netlist docs, expected net vs actual net — list mismatches (expected: none).
4. BOM export from the schematic (CSV with Value, Reference, Footprint, MPN, Manufacturer, LCSC, DNP) → compare against `BOM_master.csv`, list differences.
5. Short report: anything deviating from this plan and why.

## 9. Key checks (Opus will verify)
- U1 18 pins, U2 8 pins, U3 6 pins, U4 3 pins, Y1 4 pads on the nets stated in the netlist docs.
- `VIO` touches only: J2-1, U1-18, U2-5, R2, R10, C10, C11, C14, C17, SJ1, TP4.
- `+5V` touches: L1, C5, C6, U3-FB, R1, U2-3, C12, C13, U4-VIN, C8, TP2.
- TXCAN = U1-1 ↔ U2-1; RXCAN = U1-2 ↔ U2-4; STB = U2-8, R15, TP12.
- CANH/CANL bus nets: J3-1/2, J4-1/2, D3-1/2, JP1-1/3, R16/R17-2, L2 bus pins, TP10/TP11.
- No net has only one connection (except intentional test-pad-only nets: CLKOUT).
- ERC clean.
