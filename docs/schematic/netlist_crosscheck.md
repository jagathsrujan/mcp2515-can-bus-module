# Netlist cross-check (script generated)

Expected = pin tables parsed from `power_stage_netlist.md`, `mcp2515_block_netlist.md`, `can_phy_netlist.md` (section 4) + prose-only caps C8-C14; actual = `kicad-cli sch export netlist` of the saved schematic. Ref mapping: LED1->D4, R_LED->R1, TP_*->TP1..TP12. 'NC' = no-connect (netlist `unconnected-*`). Hierarchical/local prefixes stripped.

**Pins checked: 167, mismatches: 0**

## Parser notes (rows not machine-parsed / doc conflicts)

- unparsed net: mcp2515_block_netlist.md / U2 pins used by this block (TCAN1042HGV, SLLSES7D Table 6-1 p5: pin 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STB) pin 8: 'must be tied to GND (internal pull-up = Standby; see `ic_datasheet_extract.md`); not part of this block'

## Schematic pins with no entry in the netlist docs (informational)

none

## Plan section 9 key checks

- PASS **U1 pin count 18 (want 18)**
- PASS **U2 pin count 8 (want 8)**
- PASS **U3 pin count 6 (want 6)**
- PASS **U4 pin count 3 (want 3)**
- PASS **Y1 pin count 4 (want 4)**
- PASS **VIO members**
- PASS **+5V members**
- PASS **TXCAN**
- PASS **RXCAN**
- PASS **STB**
- PASS **CANH**
- PASS **CANL**
- PASS **Nets with a single connection: none** (intentional no-connect pins excluded; CLKOUT has 2: U1-3, TP6)
