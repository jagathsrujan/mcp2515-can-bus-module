# Design notes - MCP2515 CAN module Rev A

Frozen design: `kicad/MCP2515_CAN/` (schematic ERC 0; PCB DRC 0, parity 0). Source of truth: [ENGINEERING_SPEC.md](../ENGINEERING_SPEC.md). Tags: [DS] datasheet, [CALC] calculated, [TEST] needs bench confirmation.

## 1. Block summary
- **Power** (Phase 3, [phase3_power_supply.md](../phase3_power_supply.md)): J1 (or DNP J5) -> F1 0.35 A PTC -> D1 SS34 reverse blocker -> D2 SMAJ18A TVS + C1 100 uF damping electrolytic -> FB1 bead -> AP63205WU-7 fixed-5 V buck (L1 4.7 uH, 2 x 22 uF out, 2 x 10 uF + 100 nF in) -> +5V. XC6206 3.3 V LDO (U4) feeds only SJ1 (default open).
  - Buck, not LDO: every LDO fails thermally at 16 V / 85 C; buck loss is about 45 mW [CALC].
  - Schottky, not P-FET, for polarity: the P-FET would see Vgs/Vds overstress at the 29.2 V TVS clamp (D-006r).
  - PTC added so a sustained over-voltage (e.g. 24 V adapter) makes the TVS trip F1 instead of cooking D2.
  - Optional/untested: 0 ohm instead of FB1 if the bead causes instability.
- **Controller/oscillator** (Phase 4, [phase4_mcp2515_oscillator.md](../phase4_mcp2515_oscillator.md)): MCP2515 powered from host **VIO** (3.0-5.5 V, no level shifters, D-001); 16 MHz parallel crystal Y1 with 2 x 10 pF C0G; R4 0 ohm footprint in series with OSC2 (drive-level option, fit ~1 k only if overdriven at 5 V); RESET = 10 k pull-up + 10 nF + 1 k series, also on J2-8; 33 ohm series on SCK/MOSI/MISO/CS/INT; 10 k CS pull-up; INT is push-pull; unused TXnRTS/RXnBF not connected; CLKOUT only to TP6.
- **CAN PHY** (Phase 5, [phase5_can_phy.md](../phase5_can_phy.md)): TCAN1042HGVDR (VCC +5V, VIO from host, STB 10 k to GND = Normal); bus order J3/J4 -> D3 PESD2CANFD27V (27 V) -> termination (4 x 120 ohm as 2 x 60 ohm legs + 4.7 nF to GND, both legs switched by JP1 + 2 shunts) -> optional L2 CM choke (DNP, 0 ohm R16/R17 bypass) -> U2.
- **Schematic/PCB** ([phase6_schematic_review.md](../phase6_schematic_review.md), [phase7_pcb_review.md](../phase7_pcb_review.md)): 3 hierarchical sheets; 70 x 50 mm, 2-layer 1.6 mm 1 oz, all SMD on top, solid bottom GND, 126+ stitching vias, GND guard under/around Y1, buck hot loop about 4 mm^2, Y1 28 mm from L1/U3, 3 fiducials, 2 x M3.
- **Manufacturing rules** (spec section 6, [research/jlcpcb_capabilities.md](../research/jlcpcb_capabilities.md)): 0.15 / 0.15 mm track/space, 0.3 / 0.6 mm vias, 0.5 mm copper-to-edge; all above JLC minimums.

## 2. Firmware notes
1. **Reset first**: send SPI RESET (`0xC0`) after power-up (the on-board RC resets at power-on, but firmware should not rely on it), then wait >= 128 OSC cycles (8 us at 16 MHz; use >= 10 ms in practice). After reset the chip is in Configuration mode: `CANSTAT (0x0E) = 0x80`, `CANCTRL (0x0F) = 0x87`.
2. **Oscillator / bit timing at FOSC = 16 MHz** (Phase 4 D-025; all rates exact, tolerance allowance >= 0.31 %). Tell your library the crystal is **16 MHz** (`MCP_16MHZ`).

   | Bit rate | CNF1 | CNF2 | CNF3 | Sample point |
   |---|---|---|---|---|
   | 1 Mbit/s | 0x00 | 0x83 | 0x01 | 75 % |
   | 500 kbit/s | 0x00 | 0xAE | 0x01 | 87.5 % |
   | 250 kbit/s | 0x01 | 0xAE | 0x01 | 87.5 % |
   | 125 kbit/s | 0x03 | 0xAE | 0x01 | 87.5 % |

   Write CNF only in Configuration mode. Library preset tables (mcp_can, autowp) use different but valid segment splits; for mixed networks the sample points should be in the same range (about 75-87.5 %) - if in doubt write the raw CNF values above.
3. **CLKOUT**: default after reset is enabled at FOSC/8 = 2 MHz on TP6. Disable it in normal use (`CANCTRL.CLKEN = 0`, i.e. clear bit 2) to cut EMI; leave enabled only for crystal frequency checks.
4. **Errata #5 (DS80179G)**: after a TXREQ write or RTS command, bring CS high within 47 us (do not hold CS low for long after the last data byte); keep SPI transactions short and uninterrupted.
5. **INT is push-pull**: connect one INT line per module to one host pin; do not wire-OR several modules. Enable only the interrupt sources you service (`CANINTE`), clear flags with BIT MODIFY.
6. **SPI**: mode 0,0 (or 1,1), <= 10 MHz (Table 13-6); start at 1-4 MHz during bring-up, especially on jumper wires; MISO is tri-state when CS is high so the SPI bus can be shared.
7. Normal-mode transmit needs another node acking (or use loopback). STB is hard-wired low, so the transceiver is always in Normal mode; a bus with no other node produces error frames - expected.
8. Pins TX0RTS..TX2RTS, RX0BF/RX1BF are not connected: leave BFPCTRL and TXRTSCTRL at reset values.

## 3. Known limitations (from the phase documents)
| # | Limitation | Source |
|---|---|---|
| 1 | Operating input 7-16 V. Sustained 18-20 V+ is protected (TVS then PTC trip), not operated; the board stays off until the PTC cools | phase3 4.2 |
| 2 | Centre-positive barrel jack assumed; centre-negative adapters just leave the board off | phase3 U-12 |
| 3 | 5 V-supplied VIO: crystal drive level may exceed the 200 uW rating (about 227 uW estimated); R4 provides a series-R option, to be checked on the bench | phase4 1.4 |
| 4 | OSC2-side crystal trace is 1.3 mm over the 5 mm aim (symmetric load caps and guard present) | phase7 section 2 |
| 5 | Max bus length at 1 Mbit/s: 15 m worst case from TCAN loop delay, treat as <= 10 m in practice | phase4 1.2 |
| 6 | CAN pair runs 19-26 mm from J3/J4 to D3 (accepted); unclamped trace length leaves a residual ESD radiation risk - IEC 61000-4-2 +-8 kV contact test recommended if equipment is available | phase7 R-1b |
| 7 | D3 standoff 27 V gives no margin for a charging 24 V system (28 V); DR-05 met for 24 V nominal only | phase5 4.2 |
| 8 | Termination resistors (0603, 100 mW) tolerate at most about 6.9 V continuous differential: a CAN pin shorted to a 9-16 V rail with termination fitted overheats them. Transceiver itself survives +-70 V | phase5 3.2 |
| 9 | Module is **non-isolated**; all nodes need a common ground (CAN common-mode range +-12 V) | phase5 6 |
| 10 | AP63205 PFM light-load output ripple (2-10 mA, CAN idle) and 9 V efficiency not characterised - to be measured | phase3 U-07, U-08 |
| 11 | MCP2515 is CAN 2.0B only (no CAN FD), so the bus is capped at 1 Mbit/s although TCAN1042 supports 5 Mbit/s | phase5 2 |
| 12 | Unmeasured: buck-noise coupling into the oscillator/CAN lines, 2-layer EMC (no emc_* checks run in phase 7) | phase3 8, phase7 R-10 |
| 13 | SJ1 must stay open while a host drives VIO | phase3 6.2 |
| 14 | THT connector 3D models are placeholders (J1 none, J3-J5 Phoenix); cosmetic only | phase7 R-9 |
| 15 | JLC lists the chosen MCP2515-I/SO (C12368) as "no longer manufactured" (stock 896); alternative MCP2515T-I/SO C153782. Re-check at order time | `../manufacturing/jlc_stock_check.md` |
