# Manufacturing package - MCP2515 CAN module Rev A (JLCPCB)

Generated 2026-11-03 from `kicad/MCP2515_CAN/MCP2515_CAN.kicad_pcb` (unchanged; md5 before/after export identical) with KiCad 10.0.8 `kicad-cli`. Nothing was uploaded anywhere; nothing committed.

| File | Purpose |
|---|---|
| `MCP2515_CAN_RevA_gerbers.zip` | Upload to JLCPCB (15 files: 9 Gerber layers, 2 drill, 2 drill maps, drill report, `.gbrjob`) |
| `gerbers/` | Unzipped copy of the above |
| `MCP2515_CAN_RevA_BOM_JLC.csv` | JLC BOM: Comment, Designator, Footprint, LCSC Part # (31 lines, 53 parts) |
| `MCP2515_CAN_RevA_CPL_JLC.csv` | JLC CPL: Designator, Mid X, Mid Y, Rotation, Layer (mm, 53 rows, all Top) |
| `cpl_rotation_review.md` | Every part: KiCad rotation, DB correction, final rotation, uncertainty notes |
| `cpl_rotations_db_snapshot.csv` | Snapshot of the rotation correction database used |
| `jlc_stock_check.md` | Stock / class refresh of every LCSC part |

## 1. Gerber export settings and source

Followed JLCPCB's current KiCad 9 guide: https://jlcpcb.com/help/article/how-to-generate-gerber-and-drill-files-in-kicad-9 (fetched 2026-11-03; the guide also exists for KiCad 4-8 - this board is KiCad 10, same dialogs).

| JLC guide item | Used |
|---|---|
| Layers (2-layer) F.Cu, B.Cu, F.Mask, B.Mask, F.Silkscreen, B.Silkscreen, Edge.Cuts (+ F.Paste, B.Paste for SMT) | All nine plotted |
| Protel filename extensions | Yes (.gtl .gbl .gts .gbs .gto .gbo .gtp .gbp .gm1) |
| Extended X2 format | **Kept ON** - the current JLC guide says tick it, so "no X2" was not applied; netlist attributes included |
| Coordinate format 4.6 mm, plot mode filled, scale 1:1, drill marks none | Yes (`--precision 6`) |
| Check zone fills before plotting | Pours are stored filled; export did not modify the board (md5 identical). DRC was 0 per Phase 7 |
| Subtract soldermask from silkscreen | Yes (`--subtract-soldermask`; not mentioned by the guide, requested by project brief) |
| Reference designators / values | Plotted as set per footprint (not excluded) |
| Drill: Excellon, mm, decimal zeros, absolute origin, alternate oval-hole mode, **separate PTH and NPTH**, drill map | Yes (`MCP2515_CAN-PTH.drl`, `-NPTH.drl`, maps as Gerber X2 `*-drl_map.gbr`) |
| Zip all of it | `MCP2515_CAN_RevA_gerbers.zip` |

The guide's "plot board edge on all layers" note was not applied (Edge.Cuts is its own file, which JLC reads).

## 2. Gerber sanity check

| Check | Result |
|---|---|
| Files | `-F_Cu.gtl -B_Cu.gbl -F_Mask.gts -B_Mask.gbs -F_Paste.gtp -B_Paste.gbp -F_Silkscreen.gto -B_Silkscreen.gbo -Edge_Cuts.gm1`, `-PTH.drl -NPTH.drl`, two drill maps, `-drill.rpt`, `-job.gbrjob` |
| Layer count | 2 copper (job file `LayerNumber` 2; Cu L1 Top / L2 Bot), board thickness 1.6 mm |
| Board outline | Edge.Cuts centreline rectangle 70 x 50 mm, 1 mm corner radius, 0.1 mm line; job file reports board size **70.1 x 50.1 mm** (outline X 100..170, Y 50..100 in KiCad coordinates) - matches the Phase 7 spec |
| PTH drills | T1 **0.300 mm x 150** (vias), T2 1.000 mm x 15 (12 round: J2 x8 + JP1 x4; **3 slots** of J1 at (107.7,89.5)-(107.7,91.5), (109.7,95.2)-(111.7,95.2), (113.7,89.25)-(113.7,91.75)), T3 1.400 mm x 8 (J3/J4/J5 terminals) = 173 plated holes |
| NPTH drills | 3.200 mm x 2 (M3 mounting holes H2 at (103.5,53.5), H1 at (166.5,97.0)) |
| Smallest hole | **0.30 mm** = at JLC's no-surcharge limit (>= 0.3 mm) |
| J1 slots | present as G85 slot records (alternate oval mode) in the PTH file |
| Origin | Gerbers and drill both in absolute KiCad coordinates (no aux/drill-origin offset) |

JLC's online Gerber viewer / a third-party viewer should still be used once after upload as JLC recommends; this was not done (no uploads).
Drill/DFM numbers match `docs/pcb/drc_report.txt` (DRC 0 / parity 0).

## 3. Assembly files - assumptions
- BOM includes every assembled part: all schematic parts except **DNP J5, L2**, **SJ1** (solder jumper), loose **shunts** for JP1 (C100114, hand-fitted), test points TP1-TP12, mounting holes H1/H2 and fiducials FID1-FID3 (the latter are not in the schematic BOM).
- Through-hole J1-J4 and JP1 are included. If the chosen JLC service does not solder THT parts, remove them from BOM/CPL and hand-solder.
- Positions: aux origin (100,100) = bottom-left corner of the board; Y positive up. Rotation corrections: see `cpl_rotation_review.md`. **Rotations must be confirmed in JLC's placement preview before ordering.**
- Stock/class and one flag (U1 MCP2515-I/SO "no longer manufactured" by JLC's listing, alternative C153782): `jlc_stock_check.md`.
