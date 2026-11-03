# Schematic BOM vs BOM_master.csv

Schematic BOM: `MCP2515_CAN_BOM.csv` (kicad-cli, grouped by Value+MPN). Master groups mapped to schematic refs via the netlist docs / plan sec. 4 (R_TERM=R11-R14, C_DEC=C4,C7,C11,C12,C14,C17, C_10U=C9,C10,C13, R_10K=R2,R10 + R_STB=R15, LED1=D4, ...).

**Differences (8 items)**

- MPN `` only in schematic: ['SJ1']
- MPN `0603B103K500NT` only in schematic: ['C18']
- MPN `TBD-Basic` (master ['C_RST']) not in schematic
- Schematic part SJ1 (SJ1_+3V3_to_VIO) has no MPN: SJ1 is copper (not a BOM part; master has no row)
- Master `C_RST` MPN `TBD-Basic` / manufacturer `TBD` -> schematic uses `0603B103K500NT` / `FH` from mcp2515_block_netlist.md part table (same LCSC C57112); master row unresolved TBD, not a conflict.
- Master quantities marked TBD (C_DEC, C_10U, R_10K) resolved by schematic: C_DEC 6 (100 nF), C_10U 3 (10 uF 0805), R_10K 2 + R_STB 1 = 3 (same MPN, one schematic line R2,R10,R15).
- R_S (R4) and R_CMC (R16,R17) share MPN 0603WAF0000T5E: one schematic line R4,R16,R17 (grouping only).
- Reference names: master LED1 -> schematic D4 (plan sec. 4); master TP/H/FID rows absent (excluded from BOM by design).
