# Power-stage preliminary netlist (for KiCad capture)

Status: PRELIMINARY, derived from `phase3_power_supply.md` (Opus design, values unchanged) and `research/power_stage_extract.md`. Written 2026-11-02.
Library check: local KiCad install `/Applications/KiCad/KiCad.app/Contents/SharedSupport/{symbols,footprints}` (symbol and footprint names below were found there unless marked **CUSTOM**).
Tags: **[DS]** datasheet, **[LIST]** JLC/LCSC listing only, **(interp.)** my interpretation, **NOT FOUND** not available. RefDes numbers for new parts (C8+, R1, TPs) are proposals; annotate globally later.

## 1. Net names

| Net | Meaning |
|---|---|
| `VIN_RAW` | J1 / J1B positive, before F1 |
| `VIN_F` | after F1 (PTC), D1 anode |
| `VIN_PROT` | after D1 (Schottky): D2 TVS cathode, C1 bulk (+), FB1 pin 1 |
| `VIN_BUCK` | after FB1: C2, C3, C4, U3 VIN, U3 EN |
| `SW` | U3 pin 5, L1 pin 1, C7 pin 2 |
| `BST` | U3 pin 6, C7 pin 1 |
| `+5V` | L1 pin 2, C5, C6, U3 FB (pin 1), loads |
| `LED_A` | R_LED pin 2, LED1 anode |
| `+3V3` | U4 output (optional rail, reaches VIO only via SJ1) |
| `VIO` | host-supplied logic rail at J2 pin 1 |
| `GND` | single ground |

## 2. ASCII schematic of the chain

```
 J1 pin1 (centre +) ─ VIN_RAW ─[F1 PTC]─ VIN_F ─►|─ VIN_PROT ─[FB1 600R@100MHz]─ VIN_BUCK ─┬─ U3 pin3 VIN
 (J1B pin1 DNP, parallel)                      D1 SS34   │                                  ├─ U3 pin2 EN
                                               K=band    ├─ C1 100u/35V (+) ─ GND           ├─ C2 10u/50V ─ GND
                                                         └─ D2 SMAJ18A (K) ─ A ─ GND        ├─ C3 10u/50V ─ GND
                                                                                            └─ C4 100n ─ GND
                    U3 AP63205WU-7                                                          
          ┌──────────────────────────┐
 +5V ─────┤1 FB                 BST 6├─ BST ─[C7 100n]─ SW
 VIN_BUCK ┤2 EN                  SW 5├─ SW ───[L1 4.7u]──┬── +5V ─┬─ C5 22u ─ GND
 VIN_BUCK ┤3 VIN                GND 4├─ GND              │        ├─ C6 22u ─ GND
          └──────────────────────────┘                   │        ├─ TP_5V
                                                         │        ├─ R_LED 1k5 ─ LED_A ─ LED1 ─ GND
 +5V ─┬─ C8 1u ─ GND                                     │        ├─ U2 pin3 VCC (+C12 100n, C13 10u)
      └─ U4 XC6206 pin3 VIN                              │        └─ U4 pin3 VIN
          U4 pin1 VSS ─ GND ; U4 pin2 VOUT ─ +3V3 ─┬─ C9 10u ─ GND
                                                   ├─ TP_3V3
                                                   └─ SJ1 (open) ─ VIO ─┬─ J2 pin1 ─ C10 10u, C11 100n ─ GND
                                                                         ├─ TP_VIO
                                                                         └─ U2 pin5 VIO (+C14 100n)
```

## 3. Part table (symbol / footprint / MPN / LCSC)

| RefDes | Value | MPN | LCSC (class) | KiCad symbol | KiCad footprint |
|---|---|---|---|---|---|
| J1 | DC jack 5.5x2.1, 5 A | CUI PJ-002AH | C2961147 (Ext) | `Connector:Barrel_Jack_Switch` (pins 1/2/3 passive) | **CUSTOM** `PJ-002AH` (generic `Connector_BarrelJack:BarrelJack_Horizontal` will not match; build from CUI p2: slots 1.00x3.50 (pin 1), 1.00x3.00 (pins 2 and 3)) |
| J1B | 2-pin 5.08 terminal, **DNP** | KF128-5.08-2P | C474952 (Ext) | `Connector:Screw_Terminal_01x02` | `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal` (KF128 hole sizes NOT checked) |
| F1 | PTC 0.35 A hold / 0.75 A trip, 30 V | Bel Fuse 0ZCJ0035AF2E (primary, see section 6) | C6378052 (Ext) | `Device:Polyfuse` | `Fuse:Fuse_1206_3216Metric` (hand-solder variant `..._Pad1.42x1.75mm_HandSolder` for prototypes) |
| D1 | Schottky 40 V 3 A | SS34 (MDD) | C8678 (Basic) | `Device:D_Schottky` (pin 1 K, pin 2 A) | `Diode_SMD:D_SMA` (pad 1 = cathode, KiCad convention) |
| D2 | TVS 18 V 400 W uni | SMAJ18A (Littelfuse) | C148219 (Ext) | `Diode:SMAJ18A` (pin 1 = cathode bar side, pin 2 = anode; pin names read A1/A2 in the library, graphic is unidirectional) | `Diode_SMD:D_SMA` (pad 1 = cathode) |
| C1 | 100 uF 35 V Al, 6.3x7.7 | FZ100UF35V90RV0064 (KNSCHA) | C2992596 (Ext) | `Device:C_Polarized` | `Capacitor_SMD:CP_Elec_6.3x7.7` (pad 1 = +, KiCad convention; verify against KNSCHA drawing: negative side has the black marking bar) |
| FB1 | Bead 600 ohm @100 MHz, 500 mA, 0805 | GZ2012D601TF (Sunlord) | C1017 (Basic) | `Device:FerriteBead` | `Inductor_SMD:L_0805_2012Metric` |
| C2, C3 | 10 uF 50 V X5R 1206 | CL31A106KBHNNNE (Samsung) | C13585 (Basic) | `Device:C` | `Capacitor_SMD:C_1206_3216Metric` |
| C4 | 100 nF 50 V X7R 0603 | CC0603KRX7R9BB104 (Yageo) | C14663 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| U3 | 5 V 2 A sync buck, TSOT-26 | AP63205WU-7 (Diodes) | C2071056 (Ext) | `Regulator_Switching:AP63205WU` | `Package_TO_SOT_SMD:TSOT-23-6` |
| C7 | 100 nF 50 V X7R 0603 (BST) | CC0603KRX7R9BB104 | C14663 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| L1 | 4.7 uH, Isat 2.35 A, DCR 98 mohm, 4x4x2 | SWPA4020S4R7MT (Sunlord) | C48427 (Ext) | `Device:L` | `Inductor_SMD:L_Sunlord_SWPA4020S` |
| C5, C6 | 22 uF 25 V X5R 1206 | CL31A226KAHNNNE (Samsung) | C12891 (Basic) | `Device:C` | `Capacitor_SMD:C_1206_3216Metric` |
| LED1 | Red LED 0603 | KT-0603R (Hubei KENTO) | C2286 (Basic) | `Device:LED` (pin 1 K, pin 2 A) | `LED_SMD:LED_0603_1608Metric` |
| R_LED (R1) | 1.5 kohm 1 % 0603 | 0603WAF1501T5E (UNI-ROYAL) | C22843 (Basic) | `Device:R` | `Resistor_SMD:R_0603_1608Metric` |
| U4 | 3.3 V LDO 200 mA, SOT-23 | XC6206P332MR (Torex) | C5446 (Basic) | `Regulator_Linear:XC6206PxxxMR` (pin 1 GND, 2 VO, 3 VI: matches the XC6206 pin map) | `Package_TO_SOT_SMD:SOT-23-3` (symbol default; `SOT-23` also exists) |
| C8 | 1 uF 50 V X5R 0603 (U4 CIN) | CL10A105KB8NNNC (Samsung) | C15849 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| C9 | 10 uF 25 V X5R 0805 (U4 COUT) | CL21A106KAYNNNE (Samsung) | C15850 (Basic) | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` |
| SJ1 | solder jumper +3V3 to VIO, default OPEN | n/a (copper) | n/a | `Jumper:SolderJumper_2_Open` (pin 1 A, 2 B) | `Jumper:SolderJumper-2_P1.3mm_Open_RoundedPad1.0x1.5mm` |
| C10 | 10 uF 25 V X5R 0805 (VIO bulk at J2) | CL21A106KAYNNNE | C15850 (Basic) | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` |
| C11 | 100 nF 50 V X7R 0603 (VIO at J2) | CC0603KRX7R9BB104 | C14663 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| C12 | 100 nF 50 V X7R 0603 (U2 VCC) | CC0603KRX7R9BB104 | C14663 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| C13 | 10 uF 25 V X5R 0805 (U2 VCC bulk) | CL21A106KAYNNNE | C15850 (Basic) | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` |
| C14 | 100 nF 50 V X7R 0603 (U2 VIO; per phase3 s6.3) | CC0603KRX7R9BB104 | C14663 (Basic) | `Device:C` | `Capacitor_SMD:C_0603_1608Metric` |
| U2 (power pins only) | CAN transceiver | TCAN1042HGVDR | C124014 (Ext) | **No TCAN1042 symbol in the local library** (`Interface_CAN_LIN` has TCAN1043 only): create custom symbol; footprint `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` | see pin table below |
| J2 (power pins only) | 1x8 2.54 header | SH-PZ254V-1x8P | C55629815 (Ext) | `Connector_Generic:Conn_01x08` | `Connector_PinHeader_2.54mm:PinHeader_1x08_P2.54mm_Vertical` |
| TP_VIN, TP_5V, TP_3V3, TP_VIO, TP_GND (x2 recommended) | test pads | n/a | n/a | `Connector:TestPoint` | `TestPoint:TestPoint_Pad_D1.5mm` (or `TestPoint_Loop_D1.80mm_Drill1.0mm_Beaded` for the GND scope hook) |

## 4. Pin-by-pin connections

### J1 PJ-002AH (CUI datasheet 10/30/2019, p2 "Schematic" and PCB layout)
| Pin | Function per datasheet drawing | Net |
|---|---|---|
| 1 | centre pin (terminal 1, drawn directly on the plug tip symbol) | `VIN_RAW` |
| 2 | sleeve contact (the spring contact in the schematic) (interp.) | `GND` |
| 3 | switch contact, normally closed to the sleeve (interp. from the arrow symbol) | NC (leave unconnected, add no-connect flag) |

Centre-positive polarity is NOT stated in the datasheet; it is the project's presumption (spec). Reverse plugs are blocked by D1 anyway. Plug mates with 2.1 mm centre pin: datasheet says "centre pin 2.0 mm"; verify against the intended adapter.

### J1B KF128-5.08-2P (DNP, parallel footprint position)
| Pin | Net |
|---|---|
| 1 | `VIN_RAW` (silk "+") (interp.) |
| 2 | `GND` (silk "-") |

### F1 PTC
| Pin | Net |
|---|---|
| 1 | `VIN_RAW` |
| 2 | `VIN_F` |

### D1 SS34 (SMA; cathode = band = pad 1)
| Pin | Net |
|---|---|
| 1 K (cathode, band) | `VIN_PROT` |
| 2 A (anode) | `VIN_F` |

### D2 SMAJ18A (SMA unidirectional; cathode = band = pad 1, anode to ground)
| Pin | Net |
|---|---|
| 1 K (cathode, band) | `VIN_PROT` |
| 2 A (anode) | `GND` |

Orientation check: with VIN positive, the TVS must be reverse-biased (cathode on the positive node); the band on the silk and in the 3D model must be on the `VIN_PROT` side.

### C1 (polarised)
| Pin | Net |
|---|---|
| 1 (+) | `VIN_PROT` |
| 2 (-) | `GND` |

### FB1
| Pin | Net |
|---|---|
| 1 | `VIN_PROT` |
| 2 | `VIN_BUCK` |

### C2, C3, C4
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| C2, C3 | `VIN_BUCK` | `GND` |
| C4 | `VIN_BUCK` (nearest to U3 pin 3) | `GND` (nearest to U3 pin 4) |

### U3 AP63205WU-7 TSOT-26 (DS41326 pin descriptions p2)
| Pin | Name | Net |
|---|---|---|
| 1 | FB | `+5V` (Kelvin trace from C5 positive pad) |
| 2 | EN | `VIN_BUCK` (direct tie allowed, DS p2/p11) |
| 3 | VIN | `VIN_BUCK` |
| 4 | GND | `GND` |
| 5 | SW | `SW` |
| 6 | BST | `BST` |

### C7, L1, C5, C6
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| C7 (100 nF) | `BST` | `SW` |
| L1 | `SW` | `+5V` |
| C5, C6 | `+5V` | `GND` |

### LED1 and R_LED
| Ref | Pin 1 | Pin 2 |
|---|---|---|
| R_LED | `+5V` | `LED_A` |
| LED1 | K: `GND` | A: `LED_A` |

### U4 XC6206P332MR SOT-23 (Torex ETR0305_004b, Pin Number table p2)
| Pin | Name | Net |
|---|---|---|
| 1 | VSS | `GND` |
| 2 | VOUT | `+3V3` |
| 3 | VIN | `+5V` |

CIN C8 (1 uF): pin 1 `+5V`, pin 2 `GND`. COUT C9 (10 uF): pin 1 `+3V3`, pin 2 `GND`.

### SJ1
| Pin | Net |
|---|---|
| 1 (A) | `+3V3` |
| 2 (B) | `VIO` |
Silk: "CLOSE ONLY IF NO HOST DRIVES VIO".

### U2 TCAN1042HGV (power/supply pins only; SOIC-8 order TXD, GND, VCC, RXD, VIO, CANL, CANH, S) **[DS-pending: pin order from the family pinout; confirm against SLLSES7D Table 5-1 when the custom symbol is made]**
| Pin | Name | Net |
|---|---|---|
| 2 | GND | `GND` |
| 3 | VCC | `+5V` |
| 5 | VIO | `VIO` |

Decoupling: C12 (100 nF) and C13 (10 uF) between pin 3 and `GND`, placed at the pin; C14 (100 nF) between pin 5 and `GND`.

### J2 host header (power pins; order per spec s4 "VIO, GND, SCK, MOSI, MISO, CS, INT, RST")
| Pin | Net |
|---|---|
| 1 | `VIO` |
| 2 | `GND` |

C10 (10 uF) and C11 (100 nF): pin 1 `VIO`, pin 2 `GND`, placed at J2.

### Test points
| Ref | Net | Note |
|---|---|---|
| TP_VIN | `VIN_PROT` (after F1/D1; `VIN_RAW` is available at J1) (interp.) | silk "VIN" |
| TP_5V | `+5V` | at C5/C6 |
| TP_3V3 | `+3V3` | |
| TP_VIO | `VIO` | |
| TP_GND | `GND` | add a second one next to TP_5V as a scope ground |

## 5. Notes for capture

- `GND` has no PWR_FLAG issue as long as J1 pin 2 is a power-input type; add PWR_FLAG on `VIN_RAW`/`GND` or ERC will report "power input not driven". Same for `VIO` (only driven from the J2 passive pin).
- Layout items (hot loop, SW area, FB routing) are in `phase3_power_supply.md` section 7 and are not repeated here.
- Open items found while preparing this table: (1) no PJ-002AH footprint and no TCAN1042 symbol in the local libraries, both must be created; (2) KF128-5.08 hole/pad sizes unchecked; (3) CP_Elec_6.3x7.7 pad-1 polarity assumed from KiCad convention; (4) L1 winding-start (dot) location is not stated in the Sunlord data read, so the phase3 section 7 item 6 instruction cannot be implemented from the data read.
