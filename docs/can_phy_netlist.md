> **Opus review (2026-11-02):** termination switch changed from the SMD DIP switch (C6331180, rated 25 mA = exactly the 25 mA worst-case leg current, no derating margin) to **JP1, a 2×2 2.54 mm header + 2 shunts (3 A rating)**. All former SW1 references below now read JP1. Termination sits on the **bus side of L2** (§1 of phase5 doc governs; §7 rule 2 corrected).

# CAN PHY preliminary netlist: transceiver, termination, bus protection, connectors (for KiCad capture)

Status: PRELIMINARY, derived from `phase5_can_phy.md` (Opus design, values unchanged), `ENGINEERING_SPEC.md`, `mcp2515_block_netlist.md`, `power_stage_netlist.md` and the datasheets read for V-10 ... V-14 (section 8). Written 2026-11-02.
Library check: local KiCad install `/Applications/KiCad/KiCad.app/Contents/SharedSupport/{symbols,footprints}`.
Tags: **[DS]** datasheet read (table/page given), **[LIST]** JLC/LCSC listing only, **[CALC]** calculation, **(interp.)** my interpretation, **NOT FOUND** not available, **FLAG** differs from, or is not stated in, the Opus doc.
RefDes already used elsewhere (not reused): U1-U4, Y1, D1/D2, L1, FB1, F1, J1/J5/J2, LED1, SJ1, C1-C18, R1-R10, TP_VIN/5V/3V3/VIO/GND/CLK/INT/TXCAN/RXCAN. U2 and its decoupling C12/C13/C14 are defined in `power_stage_netlist.md` and only cross-referenced here. New numbers in this file: D3, L2, R11-R17, C19, JP1, J3, J4, TP_CANH, TP_CANL, TP_STB.

## 1. Net names

| Net | Meaning |
|---|---|
| `TXCAN` | U1 pin 1 to U2 pin 1 (defined in `mcp2515_block_netlist.md`, not repeated in the tables below except U2) |
| `RXCAN` | U2 pin 4 to U1 pin 2 (same) |
| `STB` | U2 pin 8, R15 pin 1, TP_STB |
| `CANH_T`, `CANL_T` | transceiver side of the optional CMC: U2 pins 7 / 6, R16 / R17 pin 1, L2 pins 1 / 2 |
| `CANH`, `CANL` | bus side: R16 / R17 pin 2, L2 pins 4 / 3, D3 pins 1 / 2, JP1 bus pins, J3 / J4 pins 1 / 2, TP_CANH / TP_CANL |
| `TERM_H` | JP1 pole a output to the two H-leg resistors R11, R12 (FLAG: net name added, between JP1a and the H leg as requested) |
| `TERM_L` | JP1 pole b output to the two L-leg resistors R13, R14 |
| `SPLIT` | junction of the four 120 ohm resistors and C19 |
| `+5V` | buck output (U2 VCC) |
| `VIO` | host logic rail (U2 pin 5) |
| `GND` | single ground |

## 2. ASCII schematic

```
 +5V ─┬─ C13 10u ─ GND                        VIO ─┬─ C14 100n ─ GND
      ├─ C12 100n ─ GND                            │
      │         U2 TCAN1042HGVDR (SOIC-8)          │
      │      ┌─────────────────────────────┐       │
 TXCAN ──────┤1 TXD                  STB 8├────────┴─ STB ─┬─ R15 10k ─ GND   (VIL max 0.3·VIO; Istb ≤ 20 µA → 0.2 V)
 GND ────────┤2 GND                 CANH 7├─ CANH_T         └─ TP_STB
 +5V ────────┤3 VCC                 CANL 6├─ CANL_T
 RXCAN ──────┤4 RXD                  VIO 5├─ VIO
      │      └─────────────────────────────┘
                                     
 CANH_T ─┬─[ R16 0R (default fit) ]──────────┬── CANH ─┬── TP_CANH
         └─[ L2 ACT45B-510-2P pin1→pin4 DNP ]┘          │
 CANL_T ─┬─[ R17 0R (default fit) ]──────────┬── CANL ─┼── TP_CANL
         └─[ L2 pin2→pin3 (DNP) ]────────────┘          │
                                                         │
   D3 PESD2CANFD27V-TR (SOT-23): pin1 ── CANH, pin2 ── CANL, pin3 ── GND (2 vias at pad)
   J3 / J4 KF128-5.08-3P (parallel): 1 CANH, 2 CANL, 3 GND

 Termination (switchable, split), JP1 = 2x2 header, shunts 1-2 and 3-4:

 CANH ── JP1 pin1 ─[shunt]─ pin2 ── TERM_H ─┬─ R11 120R ─┐
                                        └─ R12 120R ─┤
                                                     ├─ SPLIT ── C19 4.7n/50V ── GND
                                        ┌─ R13 120R ─┤
 CANL ── JP1 pin3 ─[shunt]─ pin4 ── TERM_L ─┴─ R14 120R ─┘
```
Both poles open: both legs are disconnected from the bus, C19 and the resistors hang on isolated nets (no asymmetric load, as Opus s3.1 requires).

## 3. Part table

| RefDes | Value | MPN | LCSC (class, stock 2026-11-02) | KiCad symbol | KiCad footprint |
|---|---|---|---|---|---|
| U2 | CAN transceiver, 5 V VCC + VIO, SOIC-8 | TCAN1042HGVDR (TI) | C124014 (Ext, 2253) | **CREATE** `TCAN1042HGV`: copy `Interface_CAN_LIN:MCP2562-E-SN` (pins 1 TXD, 2 VSS, 3 VDD, 4 RXD, 5 Vio, 6 CANL, 7 CANH, 8 STBY = same numbers and functions as TCAN1042HGV Table 6-1); rename 2 -> GND, 3 -> VCC, 8 -> STB. Symbol default footprint is already SOIC-8. The library has TCAN1043xDxQ1 only | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` |
| C12 / C13 / C14 | 100 nF, 10 uF, 100 nF (U2 VCC, VCC bulk, VIO) | see `power_stage_netlist.md` | C14663 / C15850 / C14663 (Basic) | `Device:C` | 0603 / 0805 / 0603 (already listed there) |
| R15 | 10 kohm 1 % 0603 (STB pull-down) | 0603WAF1002T5E (UNI-ROYAL) | C25804 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R16, R17 | 0 ohm 0603 (CMC bypass, fitted by default) | 0603WAF0000T5E | C21189 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| L2 (**DNP**) | CMC 51 uH, 4.5x3.2 mm, 200 mA, 2 lines | ACT45B-510-2P-TL003 (TDK) | C76584 (Ext, only **10** in stock; alternates C51949466 Tokmas 10 940, C20624726 cjiang 69 953, C2761988 KOHER 485; not placed, DNP) | `Device:L_Coupled_1423` (pins 1-4 and 2-3 are the two windings, matches the TDK circuit diagram) | **CREATE** `L_CommonModeChoke_TDK_ACT45B` (4 pads, section 5 / V-14; the stock `L_CommonModeChoke_Coilcraft_1812CAN` has different pad positions) |
| R11, R12 | 120 ohm 1 % 0603 (H leg, in parallel = 60 ohm) | 0603WAF1200T5E | C22787 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| R13, R14 | 120 ohm 1 % 0603 (L leg, in parallel = 60 ohm) | 0603WAF1200T5E | C22787 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| C19 | 4.7 nF 50 V X7R 0603 (C_SPLIT) | 0603B472K500NT (FH) | C53987 (Basic, 840 933) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| JP1 | 2x2 header 2.54 mm + 2 shunts (Opus: primary, 3 A rating) | SH-PZ254V-2x2P (SHOU HAN), shunt 2.54 Short Circuit Cap (BOOMELE) | C55629823 (Ext, 1760); shunt C100114 (hand fitted, not JLC assembled) | `Connector_Generic:Conn_02x02_Odd_Even` (pin 1 CANH, 2 TERM_H, 3 CANL, 4 TERM_L; shunt 1-2 and 3-4) | `Connector_PinHeader_2.54mm:PinHeader_2x02_P2.54mm_Vertical` |
| D3 | 2-line CAN ESD/TVS, VRWM 27 V, SOT-23 | PESD2CANFD27V-TR (Nexperia; datasheet type PESD2CANFD27V-T) | C552488 (Ext, 1595) | `Power_Protection:NUP2105L` (pin 1 K, 2 K, 3 A; same SOT-23 pinout as PESD2CANFD27V: pins 1, 2 = lines, 3 = common). Set Value / MPN / LCSC to the Nexperia part. FLAG: pin names differ from the Nexperia names (K1, K2, CC), numbers match | `Package_TO_SOT_SMD:SOT-23` (symbol default; Nexperia solder land p6 of datasheet not compared) |
| J3, J4 | 3-pos screw terminal 5.08 mm, 1 CANH 2 CANL 3 GND | KF128-5.08-3P-AA (Cixi Kefa) | C474953 (Ext, 6719) | `Connector:Screw_Terminal_01x03` | `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-3-5.08_1x03_P5.08mm_Horizontal` (same approach as J5; KF128 drill / pad sizes NOT checked against the KF128 sheet) |
| TP_CANH, TP_CANL, TP_STB | test pads | n/a | n/a | `Connector:TestPoint` | `TestPoint:TestPoint_Pad_D1.5mm` |

New Extended-class types added by this block: C552488, C55629823, C474953 (U2 C124014 is already counted; L2 is DNP). Everything else is Basic.

## 4. Pin-by-pin connections

### U2 TCAN1042HGVDR, SOIC-8 (SLLSES7D Table 6-1 p5)
| Pin | Name | Net | Connection / note |
|---|---|---|---|
| 1 | TXD | `TXCAN` | from U1 pin 1, < 10 mm; TP_TXCAN is on the same net (U1 netlist) |
| 2 | GND | `GND` | 2 vias at the pin |
| 3 | VCC | `+5V` | C12 100 nF at pin, C13 10 uF [power netlist]; TI s11 p29 recommends ~4.7 uF bulk + 0.1 uF at each supply pin, consistent |
| 4 | RXD | `RXCAN` | to U1 pin 2 |
| 5 | VIO | `VIO` | C14 100 nF at pin |
| 6 | CANL | `CANL_T` | to R16/R17 area (R17 pin 1, L2 pin 2) |
| 7 | CANH | `CANH_T` | to R16 pin 1, L2 pin 1 |
| 8 | STB | `STB` | R15 10 kohm to GND, TP_STB |

### R15 (STB pull-down), R16 / R17 (bypass), C19
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| R15 | `STB` | `GND` |
| R16 | `CANH_T` | `CANH` |
| R17 | `CANL_T` | `CANL` |
| C19 | `SPLIT` | `GND` (own via, short) |

### L2 ACT45B-510-2P-TL003 (DNP; TDK ACT45B datasheet 20180521 p5: pins 1-4 one winding, 2-3 other winding, "No polarity")
| Pin | Net | Position (datasheet top view) |
|---|---|---|
| 1 | `CANH_T` | top left |
| 2 | `CANL_T` | bottom left |
| 3 | `CANL` | bottom right |
| 4 | `CANH` | top right |

With L2 fitted, R16 and R17 are removed (they are in parallel with the windings otherwise and would short the choke).

### Termination: JP1 (2x2 header, two shunts) and R11-R14
JP1 symbol `Connector_Generic:Conn_02x02_Odd_Even`; shunts across 1-2 and 3-4 enable termination.
| JP1 pin | Net | Note |
|---|---|---|
| 1 | `CANH` | pole a, bus side |
| 2 | `TERM_H` | pole a, termination side |
| 3 | `CANL` | pole b, bus side |
| 4 | `TERM_L` | pole b, termination side |

| Ref | Pin 1 | Pin 2 |
|---|---|---|
| R11 | `TERM_H` | `SPLIT` |
| R12 | `TERM_H` | `SPLIT` |
| R13 | `TERM_L` | `SPLIT` |
| R14 | `TERM_L` | `SPLIT` |

Silk: "TERM 120R: ON = both switches ON (1 and 2)". Fallback header: pin 1 `CANH`, 2 `TERM_H`, 3 `CANL`, 4 `TERM_L`, shunts on 1-2 and 3-4.

### D3 PESD2CANFD27V-TR, SOT-23 (Nexperia datasheet 11 Aug 2020 Table 2 p2: 1 K1, 2 K2, 3 CC)
| Pin | Net |
|---|---|
| 1 | `CANH` |
| 2 | `CANL` |
| 3 | `GND` (2 vias directly at the pad) |

### J3, J4 KF128-5.08-3P (parallel, daisy chain)
| Pin | Net |
|---|---|
| 1 | `CANH` |
| 2 | `CANL` |
| 3 | `GND` |

### Test points
| Ref | Net | Silk |
|---|---|---|
| TP_CANH | `CANH` | CANH |
| TP_CANL | `CANL` | CANL |
| TP_STB | `STB` | STB |

## 5. Capture notes
- Create symbol `TCAN1042HGV` (copy of MCP2562-E-SN, rename pins 2/3/8) and footprint `L_CommonModeChoke_TDK_ACT45B` in the project library. Everything else exists in the local libraries.
- Custom L2 footprint (V-14, TDK ACT45B datasheet 20180521 p5 "Recommended land pattern", dimensions in mm): 4 pads 1.35 x 0.90 (derived from overall 5.9 x 3.4 and inner gaps 3.2 x 1.6); centres x = +/-2.275, y = +/-1.25 (KiCad y down: pad 1 (-2.275, -1.25), pad 2 (-2.275, +1.25), pad 3 (+2.275, +1.25), pad 4 (+2.275, -1.25)). Body 4.5 +/-0.2 x 3.2 +/-0.2 x 2.8 +/-0.2 (height); terminal width 0.65, length 0.7. Courtesy outline ~5.9 x 3.4 plus 0.25.
- Optional exact JP1 footprint (V-12, SHOU HAN drawing p1, 2P): pads 1.1 x 2.2 (long axis across the body), centres x = +/-4.4, y = +/-1.27; outer extent 11.0, inner gap 6.6; body 5.08 (A, 2P) x 6.15 x 3.2 high, leads extend 0.795 beyond a 9.4 body width.
- ERC: CANH/CANL/CANH_T/CANL_T only meet passives and the bidirectional U2 pins, so no driver warnings expected. No PWR_FLAG needed here (+5V and VIO already flagged in the other netlists). `SPLIT` is a floating-passive net (resistors + C19 only); fine, no flag.
- DNP L2 in the schematic (`sch_set_dnp`) and keep R16/R17 fitted in the default BOM variant.
- Layout rules are in Opus s7 and not repeated. TP_CANH/TP_CANL pads sit on the pair as short stubs (< 2 mm).

## 6. Flags and errors found in the Opus doc (not silently fixed)
1. **Termination position is inconsistent.** s1 block diagram puts the termination and D3 on the `CANH`/`CANL` bus side of L2/0 ohm. s7 rule 2 says "J -> D3 -> L2/0 ohm -> termination -> U2", i.e. termination on the transceiver side. This netlist follows the s1 diagram (matches the net list given for this task). Electrically identical with 0 ohm fitted; it matters only when L2 is fitted. Decide before Phase 7.
2. **Stale JP1 references** in `ENGINEERING_SPEC.md` (s4 interface table line "JP1 termination", D-004 "enabled by jumper JP1", checklist line "JP1 open") conflict with the new 2-pole JP1.
3. **D3 table row mislabels the clamp**: "Clamp 38/44 V [LIST]" mixes two numbers; per the datasheet 38 V is VBR max and the clamp is VCL typ 33 / max 44 V at IPP = 1 A (not at the 2.5 A rating). Also "Ppk" is not given in the datasheet (only IPPM = 2.5 A).
4. Minor: the s1 diagram label `STB_N?` is a leftover; the pin is plain `STB` (active-high standby, tied low).
5. s3.2 fault limit and s3.1 corner frequency re-computed: 2*sqrt(0.1 W * 120 ohm) = 6.9 V and 1/(2*pi*30 ohm*4.7 nF) = 1.13 MHz [CALC], both correct.

## 7. Verification results V-10 ... V-14 (2026-11-02)

**V-10 PESD2CANFD27V-T (Nexperia).** Source: Nexperia product data sheet "PESD2CANFD27V-T ESD protection for In-vehicle networks", 11 August 2020, 13 pages, via LCSC (`https://datasheet.lcsc.com/datasheet/pdf/a3a080d7c0f544f97b8a44bd2d6d6184.pdf?productCode=C552488`, listing `https://www.lcsc.com/product-detail/C552488.html`). nexperia.com returns "Access Denied" to scripted requests, so the Nexperia-hosted file was not read; the 11 March 2019 objective version (`https://static.chipdip.ru/lib/099/DOC032099825.pdf`) differs (VCL typ 35 V, IPPM 1 A, Cd 6 pF at 0 V, Rdyn 0.3 ohm) and is NOT used. The -TR in the LCSC part number is the reel ordering code of the same die (datasheet lists "-T" only).
| Parameter | Value | Table / page |
|---|---|---|
| VRWM | 27 V max (Tamb 25 C) | Table 1 p1, Table 6 p4 |
| VBR | min 28 V, typ -, max 38 V (IR = 10 mA) | Table 6 p4 |
| VCL | typ 33 V, max 44 V at IPPM = **1 A**, 8/20 us; not specified at 2.5 A | Table 1 p1, Table 6 p4 (TLP curves Fig 5-6 p5, typical) |
| IPPM (rated peak pulse current) | 2.5 A, 8/20 us (IEC 61000-4-5), pin 1 or 2 to pin 3 | Table 1 p1, Table 5 p3 |
| Ppk | NOT FOUND (not stated; only IPPM) | |
| Cd per line | typ 5.2 pF, max 6 pF (1 MHz, VR = +/-2.5 V) | Table 6 p4 |
| Line-to-line match dCd/Cd | typ 0.5 % (no max) | Table 6 p4 |
| IEC 61000-4-2 | **20 kV contact** (ten pulses); air discharge NOT FOUND; ISO 10605 20 kV (150 pF) / 17 kV (330 pF) | Table 5 p3 |
| Other | IRM typ 1 nA, max 50 nA (27 V); Rdyn typ 0.8 ohm (TLP 10 A); Tj max 175 C | Table 6 p4, Table 5 p3 |
| Pinout / package | SOT23 (TO-236AB): 1 = K1, 2 = K2, 3 = CC (common); bidirectional characteristic (Fig 3 p4, negative TLP Fig 6 p5) | Table 2 p2 |
| AEC-Q101 | qualified | p1 features, s10 p4 |
Ordering/stock (JLC API 2026-11-02): C552488 Ext, 1595 in stock.
**Decision rule s4.2: PASS.** VRWM 27 V >= 27 V (no margin; a 27-28 V charger miswire sits at/above VRWM, VBR min is 28 V); clamp: 44 V max at 1 A, estimated about 33 V + 2.5 A x 0.8 ohm = 35 V typ at 2.5 A [CALC] (Fig 5 typical curve) < 70 V (U2 limit) with margin; IEC 20 kV contact >= 15 kV; SOT-23 with 1, 2 = lines and 3 = common. Keep NUP2105L as the documented fallback only.

**V-11 TCAN1042 STB.** SLLSES7D, Table 7.7 Electrical Characteristics p9 (STB terminal): VIL max 0.3 x VIO (V variants), VIH min 0.7 x VIO; IIL (STB = 0 V, VCC = VIO = 5.5 V) min -20 uA, typ 0, max -2 uA, i.e. the internal pull-up to VIO sources **2 to 20 uA** at 0 V; IIH +/-2 uA. s9.3.7.2 p21: "the internal pull ups on TXD and STB are pulled up to VIO"; s9.3.5 p20: floating STB gives Standby. No resistance value given (equivalent >= 5.5 V / 20 uA = 275 kohm). Worst case with R15 = 10 kohm (+1 %): V_STB = 20 uA x 10.1 kohm = **0.20 V** [CALC]. VIL limit 0.3 x 3.0 V = 0.9 V (VIO min 3.0 V; 0.99 V at 3.3 V) so margin is 4.5x; the largest allowed pull-down is about 45 kohm. Also at VIO = 5.5 V: 20 uA x 10.1 kohm = 0.2 V vs 1.65 V. R15 = 10 kohm confirmed.

**V-12 2-position SMD DIP switch.** Primary: **SHOU HAN 2.54-2P TPGT, LCSC C6331180, Extended, 32 846 in stock, USD 0.45 at 1** (JLC API 2026-11-02); datasheet `https://datasheet.lcsc.com/datasheet/pdf/95468f927ae1465eb5ba36a620e6b421.pdf?productCode=C6331180` (SHOU HAN spec "DIPSWITCH DATASHEET", p3 sections 2 and 3, p1 drawing). Rating: switch ON DC 24 V, 25 mA; contact resistance max 100 mohm (AC 1 kHz, 20 mV; or DC 5 V 10 mA); insulation min 100 Mohm at 500 V; capacitance max 5 pF between adjacent terminals; temperature -30 ... +85 C; life 2000 cycles [LIST]; gull-wing SPST x2, slide/rocker, 2.54 mm pitch, 5.08 mm long (2P). Suitability: termination leg carries at most 3.0 V / 120 ohm = 25 mA [CALC] (dominant, VOD(D) max), i.e. at the 25 mA rating (a switching rating; here it is only carried, never switched under load). 2 x 100 mohm max adds 0.17 % to 120 ohm: negligible. Alternates: DORABO EPM-2P-BK C53085309 (Ext, 74 in stock, 100 mA / 50 V, piano key, 2.54 mm; datasheet not read), hanxia HX 2.54-2P TPPT C7502785 (Ext, 1952, 24 V 25 mA recessed slide), SHOU HAN 2.54-2P TPPT C6709488 (Ext, 941). Fallback: 2x2 header C55629823 (SHOU HAN SH-PZ254V-2x2P, Ext, 1760) + shunt BOOMELE C100114 (hand fitted).
KiCad: symbol `Switch:SW_DIP_x02`. Footprints (local lib `Button_Switch_SMD`, no 2.54 mm exact match for this vendor): `SW_DIP_SPSTx02_Slide_9.78x7.26mm_W8.61mm_P2.54mm` (pads 2.44 x 1.12 at x = +/-4.305, closest) and `SW_DIP_SPSTx02_Slide_6.7x6.64mm_W8.61mm_P2.54mm_LowProfile` (same pads); the 1.27 mm KingTek/Copal/Omron types do not fit. Vendor land pattern (drawing p1): pads 1.1 x 2.2, row centres 8.8 mm apart (11.0 outer, 6.6 inner), so the stock pads are 0.1 mm too far in, 0.43 mm longer inward: acceptable for assembly, or make a custom footprint (section 5).

**V-13 TCAN1042 driver and application notes.** SLLSES7D Table 7.7 p10: VOD(DOM) (CANH - CANL, TXD = 0, STB = 0) for **50 ohm <= RL <= 65 ohm: min 1.5 V, max 3.0 V** (so 60 ohm: 1.5 ... 3.0 V); 45 ohm <= RL < 50 ohm: min 1.4 V, max 3.0 V; RL = 2240 ohm: 1.5 ... 5.0 V. VO(REC) (CANH and CANL, recessive, no load) min 2.0 V, typ 0.5 x VCC, max 3.0 V; VOD(REC) (RL = 60 ohm) -120 ... +12 mV; VO(DOM) CANH 2.75 ... 4.5 V, CANL 0.5 ... 2.25 V (50-65 ohm). VSYM is specified with Rterm = 60 ohm and Csplit = 4.7 nF (Table 7.7 p10), supporting C19 = 4.7 nF. Application text: s10.2.2.1 p27 and Fig 10-2: split termination (2 x RTERM/2 + CSPLIT) "improves the electromagnetic emissions behavior ... by eliminating fluctuations in the bus common-mode voltages", no CSPLIT value recommended; s11 p29: ~4.7 uF bulk near VCC and 0.1 uF at each supply pin; s12.1 p30: place TVS and bus filter capacitors as close to the connector as possible, "a series common mode choke (CMC) can be placed on the CANH and CANL lines between the transceiver and connector" (optional, not shown in the example), use a bidirectional TVS or varistor rated for the application, at least two vias for protection-device and bypass grounds, design protection "in the direction of the signal path", split termination with centre tap to GND via C3, check termination resistor power for fault cases, pin 5 bypass on V variants, STB pull-down to GND if only normal mode is used. No TI statement on CMC ringing; Opus's "TI and NXP recommend evaluating per system" stays [ASSUME]. ESD on the device: IEC 61000-4-2 +/-15 kV unpowered, +/-8 kV powered (s7.3 p7; earlier extract).

**V-14 ACT45B-510-2P-TL003 land pattern.** TDK "ACT45B type" datasheet `cmf_automotive_signal_act45b_en`, 21 May 2018, p5 (copy `https://static.chipdip.ru/lib/988/DOC011988231.pdf`; product.tdk.com returns Access Denied to scripted requests): recommended land pattern overall 5.9 x 3.4 mm, inner gap 3.2 mm (x) and 1.6 mm (y); pad size is not printed, derived (interp.) 1.35 x 0.9. Circuit diagram: windings 1-4 and 2-3, "No polarity". Part ratings (LCSC C76584 listing [LIST]): 51 uH @ 100 kHz, 2.8 kohm @ 10 MHz, DCR 1 ohm max, 200 mA, 50 V.
