# JLCPCB capabilities research (Phase 1)

Retrieved 2026-11-02. All dimensions in mm unless stated. Values were extracted by an automated page-to-text fetch; re-check against the live pages before fab release.

Source keys:
- **[CAP]** https://jlcpcb.com/capabilities/pcb-capabilities
- **[PCBA]** https://jlcpcb.com/capabilities/pcb-assembly-capabilities
- **[IMP]** https://jlcpcb.com/impedance
- **[EXTRA]** https://jlcpcb.com/help/article/In-what-cases-will-there-be-charged-extra
- **[KICAD]** https://jlcpcb.com/help/article/how-to-generate-the-bom-and-centroid-file-from-kicad
- **[RAIL]** https://jlcpcb.com/help/article/how-to-add-edge-rails-fiducials-for-pcb-assembly-order
- **[TERMS]** https://jlcpcb.com/help/article/61-Terms-and-Conditions-of-JLCPCB-Assembly-Service
- **[PRICE]** https://jlcpcb.com/help/article/pcb-assembly-price (only seen via search-result snippet; page not fetched)
- **[PREF]** https://jlcpcb.com/help/answers/detail/837-What-is-the-practical-difference-between-Basic-and-Preferred (snippet only)

Fetch problems: `jlcpcb.com/help/article/PCB-Assembly-Design-Guidelines` returned only navigation (no data). `jlcpcb.com/parts` and the via tenting/plugging help article returned no relevant content. Anything from those is marked unverified.

Important: [CAP] publishes one table for "1-2 layer" and one for multilayer; it does not publish separate 2-layer and 4-layer sheets for every item. Where the page does not distinguish, the same value is shown in both columns.

## 1. Track width / spacing (1 oz outer copper) [CAP]

| Item | 2-layer | 4-layer | Note |
|---|---|---|---|
| Absolute min track / space | 0.10 / 0.10 (4/4 mil) | 0.09 / 0.09 (3.5/3.5 mil) | 3 mil only in BGA fan-out |
| Surcharge zone | not stated | 3.0-3.5 mil trace/space = +20% of order for 4-8 layers [EXTRA] | Stay above 3.5 mil (0.09) |
| Track width tolerance | +/-20% | +/-20% | |
| SMD pad-to-pad (different nets) | 0.15 | 0.15 | |
| Pad to track | 0.10 | 0.10 (0.09 at BGA) | |
| Same-net spacing | 0.25 | 0.25 | |
| Hatched-grid copper | 0.25 | 0.25 | |
| Min SMD pad | 0.25 x 0.25 | 0.25 x 0.25 | |
| Recommended (ours) | 0.15 / 0.15 | 0.15 / 0.15 | Project choice with margin, not a JLC statement |

## 2. Vias and holes [CAP]

| Item | Value |
|---|---|
| Min via hole / via diameter | 0.15 / 0.25 (0.1 / 0.2 conditional, higher cost) |
| Preferred min via hole | 0.20 |
| Via diameter rule | hole + 0.10 minimum (hole + 0.15 preferred) |
| Cost note | A 0.1 mm hole with 0.3 mm pad on 2/4-layer carries a fee (about $16.29/order per search snippet, unverified); holes below 0.3 mm drill are listed as "small holes" extra-charge item in [EXTRA] (see section 9) |
| Recommended min PTH (component) | 0.50 |
| Min NPTH | 0.50 |
| Plated slot width | 0.50 (2-layer), 0.35 (multilayer); length >= 2x width |
| Non-plated slot width | 1.0 |
| Drill tolerance (PTH) | +0.13 / -0.08 |
| Slot tolerance | plated +0.13 / -0.08; non-plated +/-0.2 |
| Hole position tolerance | +/-0.075 |
| PTH annular ring | 2-layer 1 oz: recommended 0.25, absolute min 0.18. 4-layer 1 oz: recommended 0.20, min 0.15 |
| NPTH pad annular ring | >= 0.45 recommended |
| Via hole to track | 0.20 |
| PTH hole to track | 0.28 (0.35 recommended) |
| NPTH to track | 0.20 |
| Inner-layer via to copper (4-layer) | 0.20 |
| Inner-layer PTH pad to copper (4-layer) | 0.30 |
| Via hole-to-hole | 0.20 |
| Pad hole-to-hole | 0.45 |

Drill-to-copper for pours: not published as one number; use the rows above (via hole to track 0.20, PTH to track 0.28/0.35, inner-layer 0.20/0.30).

## 3. Board edge [CAP]

| Item | Value |
|---|---|
| Copper to routed edge / slot | >= 0.20 |
| Copper to V-cut edge | >= 0.40 |
| Routed outline tolerance | +/-0.2 regular, +/-0.1 high-precision (min 50 x 50) |
| V-cut tolerance | +/-0.4 |
| V-cut panel size | min 70 x 70, max 475 x 475 |
| V-cut board spacing | min 2 (3 recommended) |
| Component body to board edge (assembly) | >= 2.5 [TERMS] |
| Hole-to-hole | see section 2 (0.20 via, 0.45 pad) |
| Recommended (ours) | copper >= 0.5 from routed edge; components >= 3 from edge |

## 4. Solder mask [CAP]

| Item | Value |
|---|---|
| Mask expansion | 1:1 (no automatic expansion by JLC; set in KiCad) |
| Mask opening to trace clearance | >= 0.09 |
| Min mask dam/bridge, 1 oz | 0.10 (green, red, yellow, blue, purple); 0.13 (black, white) |
| Min mask dam, 2 oz | 0.20 |
| Ink thickness | >= 10 um |
| Colors | green, purple, red, yellow, blue, white, black |
| Plugged vias | max via diameter 0.5; >= 0.35 clearance to other openings |
| Via filling (epoxy/copper paste) | 0.15-0.55; default for 6+ layers |

Unverified (not on fetched pages): which color is the no-surcharge default. Green is the conventional default; [EXTRA] item 8 mentions red mask as an example of a "special process" fee, so assume green is the cheap option. Default via treatment (tented vs open) and its cost: could not be verified from the fetched pages.

## 5. Silkscreen [CAP]

| Item | Value |
|---|---|
| Min line width | 0.15 |
| Min text height | 1.0 (40 mil) |
| Width:height ratio | 1:6 |
| Silk to pad | 0.15 |

## 6. Copper pours / planes

| Item | Value | Source |
|---|---|---|
| Hatched-grid copper line | 0.25 | [CAP] |
| Pour-to-track / pour-to-pad clearance | use track-to-track 0.10 minimum; project uses 0.2-0.3 | [CAP] |
| Pour to PTH / via hole | >= 0.28 / 0.20 (see sec. 2) | [CAP] |
| Inner plane clearance to via / PTH | 0.20 / 0.30 | [CAP] |
| Pour to board edge | >= 0.20 routed, >= 0.40 V-cut | [CAP] |
| Thermal relief spoke / copper balance / island removal | NOT published on the fetched pages. Unverified (from memory / general practice): solid pours are fine, remove dead islands in KiCad (Remove islands: Always), use thermal reliefs on THT pads, balance copper top/bottom to limit warp | n/a |

## 7. Size, panel, fiducials

| Item | Value | Source |
|---|---|---|
| Min board size (bare) | 3 x 3 (thickness >= 0.6) | [CAP] |
| Max 2-layer / 4-layer | 670 x 600 / 663 x 593 | [CAP] |
| Economic PCBA single-board size | 10 x 10 to 470 x 500 | [PCBA] |
| Standard PCBA single-board size | 70 x 70 to 460 x 500 | [PCBA] |
| Economic PCBA panel | 10 x 10 to 250 x 250 | [PCBA] |
| Edge rails | Economic: "not necessary"; Standard: "necessary". Help article says rails/fiducials are required for standard PCB assembly; min rail width 5; rails delivered attached by default | [PCBA], [RAIL] |
| Fiducials | Economic: not necessary; Standard: necessary. If used: 1.0 dia un-drilled copper pad, mask opening 2.0 dia, 3-4 per board along edges, >= 3.35 from board edge (keep-out included) | [PCBA], [RAIL] |
| Panel tooling | tooling edge min 3 (5 for SMT); tooling holes 2.0 dia; fiducials 1.0 dia centered 3.85 from panel edge | [CAP] |
| Missing rails/fiducials | JLC adds or modifies them by default if absent or incompatible; may use mouse bites instead of V-cut for small/thin boards | [TERMS] |

Not verified: whether Economic PCBA on a single (non-panel) board needs no fiducials at all. The comparison table says "Not necessary" for Economic; the help text is generic. Treat own fiducials as optional but cheap insurance.

## 8. Assembly [PCBA], [TERMS], [KICAD], [PRICE]/[PREF] where noted

| Item | Economic PCBA | Standard PCBA |
|---|---|---|
| Sides | single-sided placement (SMT/THT) | single and double-sided |
| Layers / thickness | 2, 4, 6; 0.8-1.6 mm | 1-32; no limit |
| Quantity | 2-50 pcs | 2-80,000 pcs |
| Stack-up | standard stack-up only | all |
| Min package | 0402 | 01005 |
| Min IC pin pitch | 0.4 | 0.35 |
| Min BGA pitch | 0.5 | 0.3 |
| Reflow | 255 +/-5 C, not adjustable | 240 +/-5 C |
| Gold fingers / castellated / edge plating | not supported | supported |
| Build time | 1-3 days | 4+ days |
| Delivery | single PCB or panel with mouse bites | single or panel (mouse bites or V-cut) |

- **Through-hole:** both services support THT and mixed SMD+THT on one order (THT is wave-soldered). [PCBA]. Economic is one side only; put THT parts so the solder side is the other side (verify in the order preview).
- **Basic vs Extended vs Preferred Extended:** Basic parts: no per-part fee. Preferred Extended: exempt from feeder-loading fee on Economic PCBA [PREF snippet]. Other Extended: per unique part feeder-loading fee, about $3.07 (earlier quoted $3) [PRICE snippet; third-party site cites a lower figure, so the number moves; check live quote]. Economic setup fee about $8.18 and stencil about $1.53 [PRICE snippet, unverified live]. Basic/Extended/Preferred list page (jlcpcb.com/parts) did not fetch usable content.
- **Component spacing:** no numeric value published; IPC-7351B medium/low density is the stated rule. Overlaps or tiny gaps must be avoided [TERMS]. Component body to board edge >= 2.5 [TERMS]. Stacked parts: extra CPL coordinate must differ > 0.2 from base [PCBA].
- **BOM file:** .csv/.xls/.xlsx; minimum columns Comment, Designator, Footprint, JLCPCB/LCSC Part #. Fabrication Toolkit (bennymeg KiCad plugin) outputs Designator, Footprint, Quantity, Value, LCSC Part # [KICAD].
- **CPL file:** Designator, Mid X, Mid Y, Rotation, Layer; millimetres; one file for both sides [KICAD].
- **Rotation caveat:** JLC's 3D library orientation often differs from KiCad footprint zero; keep "Apply automatic component translations" ticked in Fabrication Toolkit [KICAD]; always check the 2D/3D placement preview on the JLC order page and fix polarity/rotation there [TERMS]. Per-footprint offset tables are not published on the fetched pages.

## 9. Pricing thresholds [EXTRA] (also https://jlcpcb.com/help/article/15-in-what-cases-will-there-be-charged-extra via search)

| Trigger | Effect |
|---|---|
| Single PCB up to 102 x 102 mm | special price tier (prototype); larger incurs board-size price rise |
| Area > 650 cm^2 per board | extra charge per 50 cm^2 |
| Board side <= 3 cm in "Single PCB" batch delivery; side < 1.5/1.5-3 cm | small-board / deburring fees ($0.05 / $0.02 per piece) |
| V-cut on single board < 15 mm side | extra fee |
| Drill < 0.3 mm | "small holes" extra fee (exact amount not stated) |
| 4-8 layer trace/space 3.0-3.5 mil | +20% of order |
| ENIG area > 30% | per-% fee |
| Hole density > 150,000/m^2 | extra fee |
| Non-standard options (1.6 mm is standard; red mask, leaded HASL, 1 oz on some builds shown as warning) | checkout "special process" fee |
| Multiple designs in one Gerber | per-design fee |
| Slots 0.8-1.0 mm with long path (>= 80 m/m^2) | routing fee |

Not found: any explicit "via < 0.3 mm" or "track < 0.127 mm" surcharge on 2-layer. Note the page lists "holes smaller than 0.3 mm drill" as extra charge, which conflicts with the 0.15/0.25 capability; stay at drill >= 0.3 to be safe. The 4-layer vs 2-layer price difference itself is not on the fetched pages.

## 10. Impedance stack-ups, 4-layer [IMP]

Standard thickness 1.6 mm. Page lists layer thicknesses but not trace-width/impedance results (use https://jlcpcb.com/pcb-impedance-calculator).

| Stack-up | Top Cu | Prepreg | Core | Inner Cu (each) | Prepreg | Bottom Cu | Total (summed) |
|---|---|---|---|---|---|---|---|
| JLC04161H-7628 | 0.035 | 7628 x1, 0.2104 | 1.065 (labeled 1.1) | 0.0152 | 7628 x1, 0.2104 | 0.035 | ~1.586 |
| JLC04161H-3313 | 0.035 | 3313 x1, 0.0994 | 1.265 (labeled 1.3) | 0.0152 | 3313 x1, 0.0994 | 0.035 | ~1.564 |

Other 4-layer codes listed: JLC04161H-1080, -7628A/B/C/E/F/G, -3313A, -1080A/B, -2116, -2116A/B/C/D/E (core 0.15-1.3). Dielectric constants: 7628 = 4.4, 3313 = 4.1, 1080 = 3.91, 2116 = 4.16, core 4.6, mask 3.8 [IMP]; [CAP] quotes 2-layer core 4.5. Impedance tolerance +/-10%. Total thicknesses are my sums of listed layers.

## Recommended project design rules (KiCad Board Setup)

| Rule | Value | Basis |
|---|---|---|
| Min track width / clearance | 0.15 / 0.15 | JLC min 0.10 (2L), 0.09 (4L) |
| Default signal track | 0.20 | project choice |
| Power tracks (9 V, 5 V, 3.3 V) | 0.4-0.5 (size by current) | project choice |
| CAN differential pair | per chosen stack-up if 4-layer; 2-layer: plain 0.25 pair, 120 ohm termination, no controlled impedance needed at <= 1 Mbit/s | project choice |
| Via drill / diameter | 0.30 / 0.60 | JLC min 0.15/0.25; avoids small-hole fee |
| Min via drill / diameter (hard floor) | 0.30 / 0.60 | |
| Min through-hole (component) | 0.60 (pin + clearance; JLC min PTH 0.5) | [CAP] |
| Min annular ring | 0.20 (0.25 on 2-layer) | JLC rec 0.25 / 0.20 |
| Hole-to-hole | 0.5 | JLC 0.20 via / 0.45 pad |
| Copper to edge | 0.5 | JLC 0.20 routed, 0.40 V-cut |
| Pour clearance (track/pad) | 0.25 | project choice |
| Pour min width | 0.25 | [CAP] hatch/grid 0.25 |
| Zone connection | thermal reliefs on THT pads, solid on SMD ground vias; remove islands | general practice, unverified for JLC |
| Thermal relief spoke | 0.30 wide, 0.30 gap | project choice |
| Solder mask expansion | 0.05 | JLC 1:1, mask-to-trace >= 0.09 |
| Solder mask min web | 0.12 (green) | JLC 0.10 |
| Silkscreen line / text height / text thickness | >= 0.15 / >= 1.0 mm / 0.15 | [CAP] |
| Silk to pad | 0.20 | JLC 0.15 |
| Courtyard clearance | 0.25 | project choice |
| Board thickness / copper | 1.6 / 1 oz | standard |

## Implementation constraints for this module

- All SMD on the top side; keeps Economic PCBA eligible (single-sided, 2-50 pcs, 2/4-layer, 0.8-1.6 mm, standard stack-up).
- Passives >= 0402 (0603 preferred); ICs pitch >= 0.4 (Economic).
- Select only Basic or Preferred Extended parts; every other Extended part adds about $3 per unique part.
- THT parts (headers, terminal block, DB9 if any) only on the same placement side as SMD; check JLC wave-solder acceptance in the order preview. Avoid parts without an LCSC number.
- Keep all component bodies >= 2.5 mm (use 3 mm) from the board edge; place mounting holes (3.2 mm NPTH, pad keep-out) accordingly.
- Board <= 102 x 102 mm for the cheap tier, ideally well below; no V-cut for boards with a side < 15 mm.
- Drill >= 0.3 mm everywhere; track/space >= 0.15; avoid 3-3.5 mil on any 4-layer option.
- Add 2-3 fiducials (1 mm copper, 2 mm mask opening, >= 3.35 from edge) as insurance even though Economic says "not necessary".
- Pours: GND pours on both layers (2-layer) or inner plane (4-layer) with island removal; stitch with 0.3/0.6 vias; keep CAN pair over unbroken ground.
- Export with Fabrication Toolkit (bennymeg) using "Apply automatic component translations"; add `LCSC` field to every symbol; verify rotations of SOIC/TQFN/SOT-23 and connectors in the JLC placement preview.
- Keep silkscreen off pads (>= 0.15) and text >= 1.0 mm height.
