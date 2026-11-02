# MCP2515 controller / oscillator / host-interface preliminary netlist (for KiCad capture)

Status: PRELIMINARY, derived from `phase4_mcp2515_oscillator.md` (Opus design, values unchanged), `ENGINEERING_SPEC.md`, `power_stage_netlist.md` and `research/ic_datasheet_extract.md`. Written 2026-11-02.
Library check: local KiCad install `/Applications/KiCad/KiCad.app/Contents/SharedSupport/{symbols,footprints}`.
Tags: **[DS]** datasheet, **[LIST]** JLC/LCSC listing only, **(interp.)** my interpretation, **NOT FOUND** not available, **FLAG** differs from or is not stated in the Opus doc.
RefDes: U1, Y1, J2, SJ1, C1-C14, R1, FB1, D1/D2, L1, U2-U4, LED1, TP_VIN/5V/3V3/VIO/GND are already used by `power_stage_netlist.md` / the spec. New numbers here: C15-C18, R2-R10 and TP_CLK/INT/TXCAN/RXCAN. (U1, Y1 are the spec's names; J2 is shared with the power netlist, where only pins 1-2 were listed, this file lists all 8.)

## 1. Net names

| Net | Meaning |
|---|---|
| `VIO` | host-supplied logic rail (J2 pin 1), U1 VDD, U2 VIO |
| `GND` | single ground |
| `SCK`, `MOSI`, `CS_N` | host-side SPI nets at J2 pins 3, 4, 6 |
| `MISO`, `INT_N` | host-side nets at J2 pins 5, 7 (driven by U1 through R8 / R9) |
| `RST_N` | J2 pin 8, R2 pull-up, C18 (power-on-reset RC) |
| `SCK_U`, `MOSI_U` | U1-side nets after R5 / R6 |
| `CS_U` | U1-side net after R7, R10 pull-up (FLAG: pull-up placed on the U1 side of R7; the Opus doc does not say which side) |
| `MISO_U`, `INT_U` | U1 pins 15 / 12, before R8 / R9 |
| `RST_U` | U1 pin 17, after R3 (FLAG: net name added, not in the requested list) |
| `OSC1` | U1 pin 8, Y1 pad 1, C15 |
| `OSC2` | U1 pin 7, R4 pin 1 |
| `OSC2_X` | R4 pin 2, Y1 pad 3, C16 (crystal side of R_S, as Fig 8-1: C2 is on the crystal side of RS) |
| `TXCAN` | U1 pin 1 to U2 pin 1 (TXD) |
| `RXCAN` | U2 pin 4 (RXD) to U1 pin 2 |
| `CLKOUT` | U1 pin 3 (CLKOUT/SOF) to TP_CLK only |

## 2. ASCII schematic

```
                                                                         VIO
                                                                          │
              ┌────────────────────── C17 100n ───────── U1 pin18 VDD ◄───┤
   J2                                                                     │
 1 VIO ───────┴───────────────────────────────────────────────────────────┘
 2 GND ─────── GND
                         U1 MCP2515-I/SO (SOIC-18W)
                    ┌───────────────────────────────┐
                    │1 TXCAN ─────────────────────► TXCAN ──► U2 pin1 TXD ; TP_TXCAN
                    │2 RXCAN ◄───────────────────── RXCAN ◄── U2 pin4 RXD ; TP_RXCAN
                    │3 CLKOUT/SOF ── CLKOUT ── TP_CLK
                    │4,5,6 TXnRTS ── NC (internal 100k pull-ups)
  Y1 16 MHz  OSC2_X │7 OSC2 ◄──┐
   pad3 ─┬── R4 0R ─┘          │   OSC2 net
         └── C16 10p ── GND    │
   pad1 ─┬──────────────────── │8 OSC1        OSC1 net
         └── C15 10p ── GND    │
   pads 2,4 ── GND             │9 VSS ── GND
                    │10,11 RXnBF ── NC (BFPCTRL reset = high-Z)
 J2-7 INT_N ◄─ R9 33R ◄─ INT_U ┤12 INT
 J2-3 SCK   ──  R5 33R ─ SCK_U ┤13 SCK
 J2-4 MOSI  ──  R6 33R ─ MOSI_U┤14 SI
 J2-5 MISO  ◄─  R8 33R ◄ MISO_U┤15 SO
 J2-6 CS_N  ──  R7 33R ─ CS_U ─┤16 CS ── R10 10k ── VIO
 J2-8 RST_N ── R3 1k ─ RST_U ──┤17 RESET
      │                        │18 VDD ── VIO, C17 100n ── GND
      ├── R2 10k ── VIO        └────────────────────────────────┘
      └── C18 10n ── GND
 TP_INT on INT_N (J2 side of R9)
```

## 3. Part table

| RefDes | Value | MPN | LCSC (class) | KiCad symbol | KiCad footprint |
|---|---|---|---|---|---|
| U1 | CAN controller, SPI, SOIC-18W | MCP2515-I/SO (Microchip) | C12368 (Ext) | `Interface_CAN_LIN:MCP2515-xSO` (18 pins, numbers match DS Table 1-1, see section 5) | `Package_SO:SOIC-18W_7.5x11.6mm_P1.27mm` (symbol default; LCSC package "SOIC-18-300mil") |
| Y1 | 16 MHz crystal, 3.2x2.5, CL 9 pF [LIST] | X322516MLB4SI (YXC) | C13738 (Basic) | `Device:Crystal_GND24` (pins 1, 3 crystal; 2, 4 case/GND) | `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm` (hand-solder variant `..._HandSoldering` for prototypes; pad 1 = bottom-left in KiCad, 1 and 3 diagonal: matches YXC "crystal connection" drawing, pads 2 and 4 GND) |
| C15 | 10 pF C0G 50 V +/-5 % 0603 (OSC1) | CL10C100JB8NNNC (Samsung) | C1634 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| C16 | 10 pF C0G 50 V +/-5 % 0603 (OSC2_X) | CL10C100JB8NNNC | C1634 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| R4 | 0 ohm 0603 (R_S, fitted 0 ohm) | 0603WAF0000T5E (UNI-ROYAL) | C21189 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| C17 | 100 nF 50 V X7R 0603 (U1 VDD) | CC0603KRX7R9BB104 (Yageo) | C14663 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| C18 | 10 nF 50 V X7R 0603 (C_RST) | 0603B103K500NT (FH) | C57112 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| R2 | 10 kohm 1 % 0603 (R_PU, RST_N to VIO) | 0603WAF1002T5E (UNI-ROYAL) | C25804 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R3 | 1 kohm 1 % 0603 (R_SER, RST_N to U1) | 0603WAF1001T5E | C21190 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R5 | 33 ohm 1 % 0603 (SCK) | 0603WAF330JT5E | C23140 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R6 | 33 ohm 1 % 0603 (MOSI) | 0603WAF330JT5E | C23140 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R7 | 33 ohm 1 % 0603 (CS_N) | 0603WAF330JT5E | C23140 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R8 | 33 ohm 1 % 0603 (MISO, at U1) | 0603WAF330JT5E | C23140 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R9 | 33 ohm 1 % 0603 (INT_N, at U1) | 0603WAF330JT5E | C23140 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R10 | 10 kohm 1 % 0603 (CS pull-up to VIO) | 0603WAF1002T5E | C25804 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| (RF) | 1 Mohm 0603 | 0603WAF1004T5E | C22935 (Basic) | `Device:R` | **NOT USED**: RF is internal (V-02, section 7). Listed only as the fallback part. |
| J2 | 1x8 2.54 mm header | SH-PZ254V-1x8P | C55629815 (Ext) | `Connector_Generic:Conn_01x08` | `Connector_PinHeader_2.54mm:PinHeader_1x08_P2.54mm_Vertical` |
| TP_CLK, TP_INT, TP_TXCAN, TP_RXCAN | test pads | n/a | n/a | `Connector:TestPoint` | `TestPoint:TestPoint_Pad_D1.5mm` |

Class of C32949 (commonly quoted 10 pF C0G) is 0402, not 0603; C1634 is the 0603 Basic part. Stock snapshot (jlcsearch, 2026-11-02): C1634 1.2 M, C57112 5.6 M, C23140 4.3 M, C21189 20.9 M, C25804 40 M, C12368 only 928 (Ext), C13738 361 k.

## 4. Pin-by-pin connections

### U1 MCP2515-I/SO, SOIC-18W (DS20001801K Table 1-1 p4)
| Pin | Name | Net | Connection / note |
|---|---|---|---|
| 1 | TXCAN | `TXCAN` | to U2 pin 1 (TXD), short; TP_TXCAN |
| 2 | RXCAN | `RXCAN` | from U2 pin 4 (RXD); TP_RXCAN |
| 3 | CLKOUT/SOF | `CLKOUT` | TP_CLK only (output, active /8 = 2 MHz after reset) |
| 4 | TX0RTS | NC | no-connect flag; internal 100 kohm pull-up to VDD |
| 5 | TX1RTS | NC | no-connect flag; same |
| 6 | TX2RTS | NC | no-connect flag; same |
| 7 | OSC2 | `OSC2` | to R4 pin 1 |
| 8 | OSC1 | `OSC1` | Y1 pad 1, C15 |
| 9 | VSS | `GND` | |
| 10 | RX1BF | NC | no-connect flag; high-Z at reset |
| 11 | RX0BF | NC | no-connect flag; high-Z at reset |
| 12 | INT | `INT_U` | to R9 pin 1 |
| 13 | SCK | `SCK_U` | from R5 pin 2 |
| 14 | SI | `MOSI_U` | from R6 pin 2 |
| 15 | SO | `MISO_U` | to R8 pin 1 |
| 16 | CS | `CS_U` | from R7 pin 2, R10 pin 2 |
| 17 | RESET | `RST_U` | from R3 pin 2 |
| 18 | VDD | `VIO` | C17 within a few mm |

### Y1 YXC X322516MLB4SI (YSX321SL series sheet, p1 "Top View Crystal Connection")
| Pad | Function | Net |
|---|---|---|
| 1 | crystal terminal | `OSC1` |
| 2 | case / GND | `GND` |
| 3 | crystal terminal | `OSC2_X` |
| 4 | case / GND | `GND` |

### R4, C15, C16 (oscillator)
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| R4 (0 ohm) | `OSC2` | `OSC2_X` |
| C15 | `OSC1` | `GND` |
| C16 | `OSC2_X` | `GND` |

### C17, C18
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| C17 | `VIO` | `GND` |
| C18 | `RST_N` | `GND` |

### Reset network
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| R2 | `VIO` | `RST_N` |
| R3 | `RST_N` | `RST_U` |

### SPI / INT series and pull resistors
| Ref | Pin 1 | Pin 2 | Note |
|---|---|---|---|
| R5 | `SCK` | `SCK_U` | place at J2 |
| R6 | `MOSI` | `MOSI_U` | place at J2 |
| R7 | `CS_N` | `CS_U` | place at J2 |
| R8 | `MISO_U` | `MISO` | place at U1 (source termination) |
| R9 | `INT_U` | `INT_N` | place at U1 |
| R10 | `VIO` | `CS_U` | pull-up (FLAG: U1 side of R7) |

### J2 host header (order per spec s4)
| Pin | Net | Direction |
|---|---|---|
| 1 | `VIO` | in (power) |
| 2 | `GND` | |
| 3 | `SCK` | host to U1 |
| 4 | `MOSI` | host to U1 |
| 5 | `MISO` | U1 to host |
| 6 | `CS_N` | host to U1 |
| 7 | `INT_N` | U1 to host (push-pull, not wire-OR-able) |
| 8 | `RST_N` | host to U1 |

### Test points
| Ref | Net | Silk |
|---|---|---|
| TP_CLK | `CLKOUT` | CLK |
| TP_INT | `INT_N` | INT |
| TP_TXCAN | `TXCAN` | TXCAN |
| TP_RXCAN | `RXCAN` | RXCAN |

### U2 pins used by this block (TCAN1042HGV, SLLSES7D Table 6-1 p5: pin 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STB)
| Pin | Name | Net |
|---|---|---|
| 1 | TXD | `TXCAN` |
| 4 | RXD | `RXCAN` |
| 8 | STB | must be tied to GND (internal pull-up = Standby; see `ic_datasheet_extract.md`); not part of this block |

## 5. KiCad symbol check: `Interface_CAN_LIN:MCP2515-xSO`
Pins read from `Interface_CAN_LIN.kicad_sym`: 1 TXCAN, 2 RXCAN, 3 CLKOUT/SOF, 4 ~TX0RTS, 5 ~TX1RTS, 6 ~TX2RTS, 7 OSC2, 8 OSC1, 9 VSS, 10 ~RX1BF, 11 ~RX0BF, 12 ~INT, 13 SCK, 14 SI, 15 SO, 16 ~CS, 17 ~RESET, 18 VDD. All 18 match DS20001801K Table 1-1 PDIP/SOIC pin column. Pin types: TXCAN/CLKOUT/SO/INT/RXnBF/OSC2 outputs, others inputs, VSS/VDD power_in. Symbol default footprint `SOIC-18W_7.5x11.6mm_P1.27mm` matches the LCSC "SOIC-18-300mil" part. `Device:Crystal_GND24` pins 1/3 crystal, 2/4 GND, consistent with the YXC drawing.

## 6. Notes for capture
- `VIO` has no driving pin besides J2-1 (passive): add PWR_FLAG as in the power netlist.
- Unused outputs (pins 3 handled by TP, 10, 11) and TXnRTS inputs get no-connect flags; ERC may warn on the output pins, justify with a note (reset value of BFPCTRL disables the pins).
- Fit R4 as 0 ohm by default; it is the footprint for the optional drive-level resistor (Opus doc s1.4).
- Layout rules (crystal guard ring, 5 mm limit, 10 mm from U3/L1) are in the Opus doc s1.6; not repeated.

## 7. Verification results V-01 ... V-06 (2026-11-02)
Sources: MCP2515 DS20001801K (printed pages), TCAN1042H/HG/HGV/HV SLLSES7D, LCSC C13738 sheet. URLs are in `research/ic_datasheet_extract.md` s0 (MCP2515 `https://ww1.microchip.com/downloads/aemDocuments/documents/APID/ProductDocuments/DataSheets/MCP2515-Family-Data-Sheet-DS20001801K.pdf`; TCAN `https://www.ti.com/lit/ds/symlink/tcan1042h.pdf`; crystal `https://www.lcsc.com/datasheet/lcsc_datasheet_2403291504_YXC-Crystal-Oscillators-X322516MLB4SI_C13738.pdf`, which serves the YXC sheet "Crystal Unit YSX321SL", 1 page).

**V-01 crystal.** The only sheet for C13738 is the series sheet YSX321SL (3.20x2.50x0.70 mm, 8-64 MHz), p1: CL "10 pF, 20 pF, or specify"; drive level DL "10-200 uW Max (100 uW typical)"; ESR 16-31 MHz **50 ohm max**; shunt capacitance C0 **3 pF max**; f_tol +/-10 ppm or +/-20 ppm; temp +/-20 ppm; aging +/-3 ppm/yr max. **CL = 9 pF appears only in the LCSC/JLC listing attribute [LIST]; the datasheet does not state 9 pF** (part-specific CL NOT FOUND in a datasheet). C1 = C2 = 10 pF stays consistent (2*(9-4) = 10 pF with 4 pF stray; with 10 pF CL it would be 12 pF). Confirm CL with YXC/LCSC if frequency accuracy matters; the bench CLKOUT measurement (Opus s1.3) settles it.

**V-02 RF.** Fig 8-1 (p55, "Crystal/Ceramic Resonator Operation") draws RF inside the MCP2515 block (dashed outline, between OSC1 and OSC2, in parallel with the Sleep inverter); RS is drawn outside the block. Note 2: "The Feedback Resistor (RF) is typically in the range of 2 to 10 M". The datasheet text never says "internal" in words; the drawing shows it inside the device. **Conclusion: internal (drawing-based), no external 1 Mohm.** Note 1 wording: "A Series Resistor (RS) may be required for AT strip cut crystals."

**V-03 TCAN1042H(G)V loop delay.** Table 7.8 Switching Characteristics, p12, TA -55 to 125 C, STB = 0 V, RL = 60 ohm, CL = 100 pF, CL(RXD) = 15 pF, Fig 8-4; applies to all variants including HGV (VCC 4.5-5.5 V; VIO not separately specified). tPROP(LOOP1) (recessive to dominant): min NOT GIVEN, typ 100 ns, **max 160 ns**. tPROP(LOOP2) (dominant to recessive): min NOT GIVEN, typ 110 ns, **max 175 ns**. MCP2515 internal TXCAN-to-RXCAN delay: **NOT FOUND** (DS only says TDELAY "typically 1-2 TQ", p43, with the rule PropSeg+PS1 >= TDELAY, no ns value).
Max bus length L = (t_PROP/2 - t_loop)/5 ns/m, controller delay not included:
| Rate | t_PROP (Opus table) | L at t_loop max 175 ns | L at t_loop typ 110 ns |
|---|---|---|---|
| 1 Mbit/s | 375 ns | **2.5 m** | 15.5 m |
| 500 kbit/s | 625 ns | **27.5 m** | 40.5 m |
| 250 kbit/s | 1.25 us | **90 m** | 103 m |
| 125 kbit/s | 2.5 us | **215 m** | 228 m |
This uses PropSeg only (strict rule given in the task). Adding the MCP2515 delay would cut these further and cannot be quantified. The 1 Mbit/s case is tight; option (not applied): PRSEG = 3 (4 TQ), PHSEG1 = 0 (1 TQ), PS2 = 2: t_PROP = 500 ns gives 15 m at 175 ns, sample point still 75 %, CNF2 = 0x83; tolerance min(PS1,PS2)/(2(13NBT-PS2)) = 1/204 = 0.49 % still OK.

**V-04 reset values.** BFPCTRL (0Ch, Register 4-3, p29): all implemented bits reset 0, so B1BFE = B0BFE = 0, "pin goes to a high-impedance state": **RXnBF high-Z at reset confirmed**. TXRTSCTRL (0Dh, Register 3-2, p19): B2RTSM/B1RTSM/B0RTSM = 0 (digital input), B2RTS..B0RTS (bits 5:3) = x (pin state), bits 7:6 unimplemented; i.e. 0b00xxx000. CANCTRL (xFh, Register 10-1, p60): reset 1000 0111 = 0x87 (REQOP = 100 Configuration, ABAT 0, OSM 0, CLKEN = 1, CLKPRE = 11): **CLKOUT enabled, divide-by-8 (2 MHz at 16 MHz) confirmed**; s8.2 p55 repeats "active upon system Reset and default to ... divide-by-8".

**V-05 bit timing.** DS s5.2 / s5.5 p40, p43, Registers 5-1 (CNF1, 2Ah, p44), 5-2 (CNF2, 29h, p44), 5-3 (CNF3, 28h, p45).
Rules: PropSeg 1-8 TQ; PS1 1-8 TQ; PS2 2-8 TQ (PS2min = IPT = 2 TQ); SJW 1-4 TQ; BRP[5:0] 0-63; PropSeg + PS1 >= PS2; PropSeg + PS1 >= TDELAY (typ 1-2 TQ); **PS2 >= SJW** (the Opus doc writes "PS2 > SJW", stricter than the datasheet, harmless); SyncSeg fixed 1 TQ; total NBT is the sum, DS gives no explicit NBT range; typical sample point 60-70 % [DS p43] (Opus uses 75/87.5 % per CiA, tagged [ASSUME], fine). TQ = 2(BRP+1)/FOSC per Register 5-1 (Eq 5-2 prints "2*BRP*TOSC", a datasheet inconsistency; the register map is right). Field encoding: PRSEG = Prop-1 (bits 2:0), PHSEG1 = PS1-1 (bits 5:3), PHSEG2 = PS2-1 (CNF3 bits 2:0), SJW bits = SJW-1 (bits 7:6), SAM bit 6, BTLMODE bit 7.
| Rate | BRP | NBT | Prop/PS1/PS2/SJW | Rules | CNF1 | CNF2 | CNF3 |
|---|---|---|---|---|---|---|---|
| 1M | 0 | 8 (1+3+2+2) | 3/2/2/1 | all pass; 5 >= 2; SP 6/8 = 75 % | 00 000000 = 0x00 | 1 0 001 010 = 0x8A | 0x01 (PHSEG2 = 001) |
| 500k | 0 | 16 (1+5+8+2) | 5/8/2/1 | all pass; 13 >= 2; SP 14/16 = 87.5 % | 0x00 | 1 0 111 100 = 0xBC | 0x01 |
| 250k | 1 | 16 | 5/8/2/1 | TQ 250 ns, bit 4 us | 0x01 | 0xBC | 0x01 |
| 125k | 3 | 16 | 5/8/2/1 | TQ 500 ns, bit 8 us | 0x03 | 0xBC | 0x01 |
**All CNF hex values and all four table rows check out; no rule violation found.** TQ and bit times recomputed: 125 ns*8 = 1 us, 125 ns*16 = 2 us, 250 ns*16 = 4 us, 500 ns*16 = 8 us. t_PROP column = Prop x TQ correct. Oscillator tolerance column recomputed: 1M min(0.625 %, 0.98 %) = 0.625 %; others min(0.3125 %, 0.485 %) = 0.31 %: correct. Not verified in the datasheet: the doc claim that SAM = 0 is "required at 1 Mbit/s" (no such statement found in DS s5); harmless since SAM = 0 is chosen anyway.

**V-06 TCAN1042H(G)V short-circuit current.** Table 7.7 Electrical Characteristics (continued), p10, "Short-circuit steady-state output current, dominant, Normal mode" IOS(SS_DOM): CANH shorted (VCANH -5 to 40 V, CANL open, TXD = 0 V): **-100 mA (limit column; magnitude 100 mA)**; CANL shorted (VCANL -5 to 40 V, CANH open): **+100 mA max**. No typ given. Recessive IOS(SS_REC) -5 to +5 mA (-27 V <= VBUS <= 32 V). CANH to GND and CANL to supply are inside the tested ranges. Average formula in s9.3.6.

## 8. Differences / flags against the Opus doc
1. No CNF register or bit-timing error found (section 7, V-05). Only wording: "PS2 > SJW" vs datasheet "PS2 >= SJW".
2. Opus s1.4 says Fig 8-1 note 1 shows RS "for low-drive-level crystals"; the note actually says "may be required for AT strip cut crystals" (the practical use is the same).
3. CL = 9 pF is from the LCSC listing only; the datasheet says 10 pF / 20 pF / specify.
4. Opus s1.5 RF fallback is not needed (internal per Fig 8-1 drawing).
5. Opus s5 lists "OSC2: R_S -> Y1 pin 3, C2": C2 placed on the crystal side of R_S (Y1 pad 3 net `OSC2_X`) as in Fig 8-1.
6. 1 Mbit/s bus-length budget is only 2.5 m worst case (section 7, V-03).
7. U1 pins 3, 10, 11 are outputs left open: ERC "output not connected" needs no-connect flags.
