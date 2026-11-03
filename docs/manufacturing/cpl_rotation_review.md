# CPL rotation review (Rev A)

**CONFIRM EVERY ROTATION IN JLCPCB'S PLACEMENT PREVIEW BEFORE PAYING.** The corrections below come from a generic database; JLC's own footprint library for a given LCSC part may differ, and no database entry exists for diodes, LED, crystal, inductor, or the through-hole parts. Wrong polarity on D1/D2/D4/C1/U1-U4 produces a dead or damaged board.

## Method

- `kicad-cli pcb export pos --format csv --units mm --side front --use-drill-file-origin` with the board aux origin at (100, 100) = **bottom-left corner of the 70.1 x 50.1 mm outline** (board spans x 100..170, y 50..100). Output Y is positive upward (checked: all 53 parts have 0 < X < 70, 0 < Y < 50), so no Y sign flip is needed. X/Y = footprint origin. All SMD origins coincide with the pad-bbox centre (checked); through-hole origins do not (see notes).
- Rotation = (KiCad rotation + database correction) mod 360, top side only (no bottom-side parts).
- Correction database: Matthew Lai's `cpl_rotations_db.csv` (the file the bouni plugin imports/manages as its rotation corrections): https://github.com/matthewlai/JLCKicadTools/blob/master/jlc_kicad_tools/cpl_rotations_db.csv (raw snapshot saved here as `cpl_rotations_db_snapshot.csv`, 59 rules; file last changed upstream 2024-09-11). Plugin: https://github.com/bouni/kicad-jlcpcb-tools (HEAD 3d27653, 2026-11-02; README "Footprint rotation correction" and `CORRECTIONS.md` define the regex matching used here: pattern searched in the footprint name, longest match wins). bouni's repository itself ships no rotation CSV; it points to Matthew Lai's file.
- JLC CPL columns: Designator, Mid X, Mid Y, Rotation, Layer; mm, no unit suffix, rotation counter-clockwise as in KiCad.

## Per-part table

| Ref | Footprint | KiCad rot | DB rule matched | Correction | Final rot | Type | Confidence / note |
|---|---|---|---|---|---|---|---|
| C1 | CP_Elec_6.3x7.7 | 0 | ^CP_Elec_6.3x7.7 | +180 | **180** | SMD | Polarised: confirm negative band/chamfer vs silk in JLC preview (DB says 180 for CP_Elec_6.3x7.7) |
| C2 | C_1206_3216Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C3 | C_1206_3216Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C4 | C_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C5 | C_1206_3216Metric | 180 | - | +0 | **180** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C6 | C_1206_3216Metric | 180 | - | +0 | **180** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C7 | C_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C8 | C_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C9 | C_0805_2012Metric | 180 | - | +0 | **180** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C10 | C_0805_2012Metric | -90 | - | +0 | **270** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C11 | C_0603_1608Metric | -90 | - | +0 | **270** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C12 | C_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C13 | C_0805_2012Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C14 | C_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C15 | C_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C16 | C_0603_1608Metric | -90 | - | +0 | **270** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C17 | C_0603_1608Metric | 180 | - | +0 | **180** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C18 | C_0603_1608Metric | 180 | - | +0 | **180** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| C19 | C_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| D1 | D_SMA | 180 | - | +0 | **180** | SMD | Polarised SMA, no DB rule: confirm cathode band at pad 1 (K) side |
| D2 | D_SMA | 180 | - | +0 | **180** | SMD | Polarised SMA, no DB rule: confirm cathode band at pad 1 (K) side; wrong = TVS shorts input in normal polarity |
| D3 | SOT-23 | 0 | ^SOT-23 | -90 | **270** | SMD | SOT-23 DB rule -90 is generic; confirm pin 1 (CANH) location |
| D4 | LED_0603_1608Metric | 0 | - | +0 | **0** | SMD | LED, no DB rule: confirm cathode marking matches pad 1 (K) - JLC LED 0603 orientation varies by vendor |
| F1 | Fuse_1206_3216Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| FB1 | L_0805_2012Metric | 0 | - | +0 | **0** | SMD | Non-polar, low risk |
| J1 | BarrelJack_CUI_PJ-002AH | 0 | - | +0 | **0** | THT | THT: origin = footprint origin, pad-bbox centre is (-2.88,-1.97) mm away; verify position/rotation in preview |
| J2 | PinHeader_1x08_P2.54mm_Vertical | -90 | - | +0 | **270** | THT | THT header: origin is pin 1 (pad centre is 8.89 mm away along -X); verify body centre/pin 1 in preview |
| J3 | KF128-5.08-3P_Horizontal | 90 | - | +0 | **90** | THT | THT terminal: origin = pin 1, pad centre +5.08 mm in Y; wire-entry must face the +X board edge; verify |
| J4 | KF128-5.08-3P_Horizontal | 90 | - | +0 | **90** | THT | THT terminal: origin = pin 1, pad centre +5.08 mm in Y; wire-entry must face the +X board edge; verify |
| JP1 | PinHeader_2x02_P2.54mm_Vertical | 0 | - | +0 | **0** | THT | THT header: origin is pin 1 (pad centre +1.27,-1.27); verify in preview |
| L1 | L_Sunlord_SWPA4020S | 0 | - | +0 | **0** | SMD | Shielded inductor is non-polar electrically; check winding-start dot only cosmetically |
| R1 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R2 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R3 | R_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R4 | R_0603_1608Metric | -90 | - | +0 | **270** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R5 | R_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R6 | R_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R7 | R_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R8 | R_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R9 | R_0603_1608Metric | 90 | - | +0 | **90** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R10 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R11 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R12 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R13 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R14 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R15 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R16 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| R17 | R_0603_1608Metric | 0 | - | +0 | **0** | SMD | Non-polar 2-terminal, symmetrical footprint: low risk |
| U1 | SOIC-18W_7.5x11.6mm_P1.27mm | 180 | ^SOIC- | +270 | **90** | SMD | SOIC DB rule 270 generic for SOIC-; confirm pin-1 dot at pad 1 (KiCad rot 180 + 270 = 90) |
| U2 | SOIC-8_3.9x4.9mm_P1.27mm | 0 | ^SOIC- | +270 | **270** | SMD | SOIC-8 DB rule 270; confirm pin-1 dot |
| U3 | TSOT-23-6 | 0 | ^TSOT-23 | +180 | **180** | SMD | TSOT-23-6 DB rule 180; confirm pin 1 (FB) location |
| U4 | SOT-23-3 | 0 | ^SOT-23 | -90 | **270** | SMD | SOT-23-3 DB rule -90 generic; confirm pin 1 = VSS(GND) location; XC6206 pinout is 1 GND 2 VOUT 3 VIN |
| Y1 | Crystal_SMD_3225-4Pin_3.2x2.5mm | -90 | - | +0 | **270** | SMD | 4-pad crystal, no DB rule: 180 deg error only swaps the two crystal pads (harmless); 90/270 error would swap crystal<->GND pads - confirm |

## Summary

- 53 assembled parts. Database corrections applied to 6 parts: C1 (+180), D3 (-90), U4 (-90), U1 (+270), U2 (+270), U3 (+180). All other 47 parts matched no rule (correction 0).
- Uncertain / must be eyeballed in the JLC preview: **C1** (electrolytic polarity), **D1, D2** (SMA cathode, no rule), **D4** (LED cathode, no rule), **D3, U4** (SOT-23 generic -90), **U1, U2** (SOIC 270), **U3** (TSOT-23-6 180), **Y1** (4-pad crystal, no rule; only a 90/270 error is harmful), **J1, J2, J3, J4, JP1** (through-hole, no rule, origin offsets).
- Through-hole origin vs pad-bbox centre (mm): J1 (-2.88, -1.97); J2 (-8.89, 0.00) rotated -90 so pin 1 is at the origin and the header extends toward -X from it; J3/J4 (0, +5.08); JP1 (+1.27, -1.27). If JLC's preview shows these parts displaced, shift by these amounts (JLC's library origin is usually the body centre while KiCad's is pin 1).
- If the preview shows a part rotated wrongly, fix it in JLC's editor (rotate/flip) - do not change the PCB.
- DNP/unfitted parts excluded from CPL: J5, L2, SJ1. Test points, holes, fiducials are `exclude_from_pos_files` in the PCB.
- THT parts (J1-J4, JP1): economy PCBA may not place THT; confirm in the quote whether JLC solders them or whether they must be hand-fitted (then drop them from BOM/CPL).
