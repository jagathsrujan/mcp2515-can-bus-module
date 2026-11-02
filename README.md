# MCP2515 CAN Bus Module — End-to-End Design

Work-in-progress hardware design of an MCP2515-based CAN bus module (external 9 V DC input, 3.3 V / 5 V host compatible, JLCPCB fab + assembly).

**Status:** Phase 5 — CAN PHY (awaiting approval). Phases 1–4 approved; Phase 6 schematic plan drafted.

## Contents
- [`docs/ENGINEERING_SPEC.md`](docs/ENGINEERING_SPEC.md) — engineering spec (source of truth): requirements, architecture, decisions
- [`docs/phase2_component_selection.md`](docs/phase2_component_selection.md) — component selection
- [`docs/phase3_power_supply.md`](docs/phase3_power_supply.md) — power supply design
- [`docs/power_stage_netlist.md`](docs/power_stage_netlist.md) — power stage netlist
- [`docs/phase4_mcp2515_oscillator.md`](docs/phase4_mcp2515_oscillator.md), [`docs/mcp2515_block_netlist.md`](docs/mcp2515_block_netlist.md) — MCP2515 + oscillator
- [`docs/phase5_can_phy.md`](docs/phase5_can_phy.md), [`docs/can_phy_netlist.md`](docs/can_phy_netlist.md) — CAN transceiver / physical layer
- [`docs/phase6_schematic_plan.md`](docs/phase6_schematic_plan.md) — schematic plan
- [`docs/BOM_master.csv`](docs/BOM_master.csv), [`docs/power_budget.xlsx`](docs/power_budget.xlsx)
- [`docs/research/`](docs/research/) — datasheet extracts and JLCPCB capabilities

Schematic and PCB (KiCad) not started yet.
