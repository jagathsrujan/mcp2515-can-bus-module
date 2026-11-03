# MCP2515 CAN Bus Module — End-to-End Design

Hardware design of an MCP2515-based CAN bus module (external 9 V DC input, 3.3 V / 5 V host compatible, JLCPCB fab + assembly).

**Status:** Phase 8 — final validation complete. Rev A is ready for a prototype order once the order-time checks are done. Phases 1–7 approved.

![3D render](docs/pcb/render_iso.png)

## Where to look
| What | Files |
|---|---|
| Spec (source of truth) | [`docs/ENGINEERING_SPEC.md`](docs/ENGINEERING_SPEC.md) |
| Schematic | [PDF](docs/schematic/MCP2515_CAN_schematic.pdf), sheet PNGs and ERC report in [`docs/schematic/`](docs/schematic/) |
| PCB | renders, layer images, DRC report and [`layout_report.md`](docs/pcb/layout_report.md) in [`docs/pcb/`](docs/pcb/), 3D [`MCP2515_CAN.step`](docs/pcb/MCP2515_CAN.step) |
| KiCad project | [`kicad/MCP2515_CAN/`](kicad/MCP2515_CAN/) |
| Manufacturing (JLCPCB) | Gerbers zip, BOM, CPL and rotation review in [`docs/manufacturing/`](docs/manufacturing/) |
| Final docs | validation, BOM, connector pinouts, design notes, testing and commissioning in [`docs/final/`](docs/final/) |
| Phase write-ups | `docs/phase2…phase7_*.md`, block netlists, research in [`docs/research/`](docs/research/) |
