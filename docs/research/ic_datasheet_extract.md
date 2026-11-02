# IC datasheet extract — MCP2515 CAN module (Phase 2 research, no selections)

Author: Sonnet research pass, 2026-11-02. Scope: facts only. "NOT FOUND" = could not be retrieved or is not in the document. `[CALC]` = my arithmetic from datasheet numbers. Page numbers are the *printed* datasheet page numbers.
LCSC/JLC data: jlcsearch.tscircuit.com (JLCPCB catalogue mirror) queried by LCSC number on 2026-11-02; stock is a snapshot and moves daily. "Basic/Pref/Ext" = JLCPCB part class (Basic = no setup fee, Pref = Preferred Extended, Ext = Extended).

## 0. Source documents actually read

| Part | Document used | URL | Note |
|---|---|---|---|
| MCP2515 | DS20001801K (Apr 2021, 96 pp) | https://ww1.microchip.com/downloads/aemDocuments/documents/APID/ProductDocuments/DataSheets/MCP2515-Family-Data-Sheet-DS20001801K.pdf | cross-checked against DS20001801J: https://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf (same Table 13-2) |
| MCP2515 errata | DS80179G (Mar 2007, "Rev. B Silicon Errata") | https://ww1.microchip.com/downloads/en/DeviceDoc/80179g.pdf | newest errata I could retrieve; no newer one found |
| MCP2561/2 | DS20005167C (2014, 30 pp) | https://ww1.microchip.com/downloads/en/DeviceDoc/20005167C.pdf | newer revision may exist; not retrievable |
| MCP2561/2FD | DS20005284A (2014, 32 pp) | https://ww1.microchip.com/downloads/en/DeviceDoc/20005284A.pdf | |
| TJA1051 | NXP Rev. 9, 28 Nov 2017 (25 pp) | official https://www.nxp.com/docs/en/data-sheet/TJA1051.pdf is blocked (HTTP error); read from mirror https://cdn.components.ru/media/PDF/332758.pdf (header confirms NXP Rev. 9) | older Rev. 7 mirror also read |
| SN65HVD230 | SLOS346O (Apr 2018) | https://www.ti.com/lit/ds/symlink/sn65hvd230.pdf | |
| TCAN1042H/HG/HGV/HV | SLLSES7D (Oct 2021) | https://www.ti.com/lit/ds/symlink/tcan1042h.pdf | covers H (±70 V) variants only; non-H TCAN1042V datasheet not read |
| TCAN1462(V)-Q1 | SLLSFF2B (Oct 2024) | https://www.ti.com/lit/ds/symlink/tcan1462-q1.pdf | |
| TPS7B6925/33/50-Q1 | SLVSCJ8B (Jan 2015) | https://www.ti.com/lit/ds/symlink/tps7b69-q1.pdf | |
| TLV767 | SLVSE84D (Jul 2021) | https://www.ti.com/lit/ds/symlink/tlv767.pdf | |
| TPS7A25 | SBVS372C (Dec 2022) | https://www.ti.com/lit/ds/symlink/tps7a25.pdf | |
| LM2936 | SNOSC48O (Dec 2014) | https://www.ti.com/lit/ds/symlink/lm2936.pdf | |
| TPS560430 | SLVSE22B (Jun 2018) | https://www.ti.com/lit/ds/symlink/tps560430.pdf | |
| TPS62933 | SLUSEA4D (Aug 2022) | https://www.ti.com/lit/ds/symlink/tps62933.pdf | |
| AP63205 | DS41326 Rev 3-2 (Nov 2024) | https://www.diodes.com/assets/Datasheets/AP63200-AP63201-AP63203-AP63205.pdf | |
| AMS1117 | Advanced Monolithic Systems original (undated copy) | https://files.waveshare.com/upload/b/b8/AMS1117-datasheet.pdf | mirror copy, not AMS site (AMS site fails TLS). A SLKOR clone datasheet (https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/8122/AMS11173.3SOT223.pdf) disagrees on abs-max Vin (30 V) — see §C |
| TLV755P | SBVS320D (Sep 2024) | https://www.ti.com/lit/ds/symlink/tlv755p.pdf | |
| AP2112 | DS39724 Rev 2-2 (Jun 2017) | https://www.diodes.com/assets/Datasheets/AP2112.pdf | |
| L78M05, LD1117S50, XC6206, ME6211, MP1584 | **NOT RETRIEVED** (st.com, advanced-monolithic.com, torexsemi.com, microne, monolithicpower all refused/timed out/blocked) | — | marked NOT FOUND below; only secondary search-snippet facts, labelled as such |

---

# A) MCP2515 (Microchip) — stand-alone CAN controller

All values: DS20001801K unless stated. Table refs are datasheet table numbers.

| Parameter | Value | Source |
|---|---|---|
| VDD range | 2.7 – 5.5 V (I and E grades) | Table 13-1 "DC Characteristics", p74; front-page features p1 |
| IDD operating | typ 5 mA (front page); max 10 mA @ VDD 5.5 V, FOSC 25 MHz, FCLK 1 MHz, SO open | p1 features; Table 13-1 p74 |
| IDDS (Sleep) | max 5 µA (−40…+85 °C), max 8 µA (−40…+125 °C); typ 1 µA (front page) | Table 13-1 p74; p1 |
| Abs max VDD | 7.0 V | §13.1 Absolute Maximum Ratings, p73 |
| Abs max, all inputs/outputs wrt VSS | −0.6 V to VDD + 1.0 V | §13.1, p73 |
| Register retention voltage | min 2.4 V | Table 13-1 p74 |
| VIH: SCK, CS, SI, TXnRTS | 0.7·VDD … VDD+1 | Table 13-1 p74 |
| VIH: RESET / OSC1 | 0.85·VDD … VDD | Table 13-1 p74 |
| VIH: RXCAN | 2.0 V … VDD+1 | Table 13-1 p74 |
| VIL: SCK, CS, SI | −0.3 … 0.4·VDD | Table 13-1 p74 |
| VIL: RESET | VSS … 0.15·VDD | Table 13-1 p74 |
| VIL: RXCAN, TXnRTS | −0.3 … 0.15·VDD | Table 13-1 p74 |
| VOL | SO, CLKOUT 0.6 V max @ IOL 2.1 mA; INT 0.6 V max @ 1.6 mA; TXCAN 0.6 V @ 6 mA; RXnBF 0.6 V @ 8.5 mA (all at VDD 4.5 V) | Table 13-1 p74 |
| VOH | SO, CLKOUT ≥ VDD−0.5 @ −400 µA; INT ≥ VDD−0.7 @ −1.0 mA; TXCAN, RXnBF ≥ VDD−0.7 @ −3 mA (all at VDD 4.5 V) | Table 13-1 p74 |
| Input leakage | ±1 µA (CS = RESET = VDD); OSC1 ±5 µA | Table 13-1 p74 |
| SPI max clock | 10 MHz, **one value for the whole 2.7–5.5 V range** (no VDD split). Mode 0,0 and 1,1. TCSS/TCSH/TCSD 50 ns; TSU/THD 10 ns; THI/TLO 45 ns; TV (output valid) 45 ns max; TDIS (output disable) 100 ns max | Table 13-6 "SPI Interface AC Characteristics", p77; §12.1 p65 |
| FOSC (clock in) | 1 – 40 MHz at VDD 4.5–5.5 V; **1 – 25 MHz at VDD 2.7–5.5 V** (so 25 MHz is the ceiling at 3.0–4.5 V). TOSC min 25 ns / 40 ns respectively. Duty cycle 0.45–0.55 for external clock | Table 13-2 "Oscillator Timing Characteristics", p75 (identical in DS20001801J) |
| Revision-H note | "voltage range widened to 2.7–5.5 V for E-temp device; IDDS and FOSC differ between I and E" | Appendix A Revision History, Rev H |
| Oscillator type | Crystal or ceramic resonator on OSC1/OSC2, **parallel-cut crystal required** (series cut may give frequency out of spec) | §8.0 p55 |
| Oscillator start-up | Oscillator Start-up Timer holds device in reset for 128 OSC1 cycles after POR, SPI reset, RESET pin, wake-up; no SPI access until expired | §8.1 p55 |
| Crystal load caps (design guidance only, "not optimized") | Table 8-2: 4 MHz 27/27 pF, 8 MHz 22/22 pF, 20 MHz 15/15 pF (HS mode). Table 8-1 (resonators): 8 MHz 27/27 pF, 16 MHz 22/22 pF | §8, p56 |
| Other osc. notes | RS may be needed for low-drive-level crystals / AT-strip cut (Fig 8-1 note 1, Table 8-2 note 3); RF 2–10 MΩ typ (Fig 8-1 note 2); verify over VDD & temperature; higher C increases start-up time | p55–56 |
| Crystal ESR / drive level | **NOT FOUND** (datasheet gives no ESR limit or drive-level spec) | — |
| RESET requirement | tRL (RESET low time) min 2 µs; device must be held in reset ≥2 µs after VDD reaches operating voltage; RC on RESET suggested (Fig 9-1: R, C, diode D, R1 = 1–10 kΩ series). Reset after power-up "highly recommended" (hardware or SPI RESET 0xC0) | Table 13-4 p75; §9.0 p57; §12.2 p65 |
| RESET internal pull-up | **None documented.** Pin table says only "Active-low device Reset input"; only TXnRTS pins are documented with 100 kΩ internal pull-ups to VDD. External pull-up required | Table 1-1 p4 |
| INT output type | Output with specified VOH (≥VDD−0.7 V @ −1 mA) and VOL, i.e. **push-pull (not open-drain)**; datasheet does not call it open-drain. Active low, stays low until all interrupt flags are cleared | Table 13-1 p74; §7 interrupt text (INT driven low…remains low) |
| RXnBF outputs | Push-pull style (VOH/VOL specified); can be disabled to high-impedance via BFPCTRL (§4.x "RXnBF pin functions") | Table 13-1 p74; Register 4-x BFPCTRL |
| SO (MISO) tri-state when CS high | Shown as "High-Impedance" in all SPI read timing diagrams (Figs 12-x) and TDIS (output disable time) = 100 ns max. No sentence says "tri-states" but timing shows it | Table 13-6 p77; Figs 12-2…12-9 |
| CLKOUT default | Active after reset, **divide-by-8** (slowest); CLKEN=1 default; CLKEN=0 → high-Z; idle low in Sleep | §8.2 p55; Register 10-1 CANCTRL bits 2:0, p60 |
| Temperature grades | I: −40…+85 °C; E: −40…+125 °C; AEC-Q100 qualified ("VAO" suffix) | Front page p1; Product Identification System (last page) |
| Operating vs standby: TXnRTS pin | 100 kΩ internal pull-up to VDD | Table 1-1 p4; §3.5 p15 |

**SOIC-18 (300 mil) pinout** (Table 1-1, p4): 1 TXCAN, 2 RXCAN, 3 CLKOUT/SOF, 4 TX0RTS, 5 TX1RTS, 6 TX2RTS, 7 OSC2, 8 OSC1, 9 VSS, 10 RX1BF, 11 RX0BF, 12 INT, 13 SCK, 14 SI, 15 SO, 16 CS, 17 RESET, 18 VDD.
**TSSOP-20:** 1 TXCAN, 2 RXCAN, 3 CLKOUT, 4 TX0RTS, 5 TX1RTS, 6 NC, 7 TX2RTS, 8 OSC2, 9 OSC1, 10 VSS, 11 RX1BF, 12 RX0BF, 13 INT, 14 SCK, 15 NC, 16 SI, 17 SO, 18 CS, 19 RESET, 20 VDD.
**QFN-20 (4×4):** 1 CLKOUT, 2 TX0RTS, 3 TX1RTS, 4 NC, 5 TX2RTS, 6 OSC2, 7 OSC1, 8 VSS, 9 RX1BF, 10 RX0BF, 11 INT, 12 SCK, 13 NC, 14 SI, 15 SO, 16 CS, 17 RESET, 18 VDD, 19 TXCAN, 20 RXCAN, 21 EP (to VSS).

**Orderable variants** (Product Identification System, last page): MCP2515-I/SO, MCP2515-E/SO, MCP2515T-I/SO (tape & reel), MCP2515T-E/SO; -I/ST, -E/ST (TSSOP-20); -I/ML, -E/ML (QFN-20); -I/P (PDIP). "VAO" suffix = AEC-Q100.

**Errata** (DS80179G, https://ww1.microchip.com/downloads/en/DeviceDoc/80179g.pdf, pp1–2). Silicon rev B4 (in production since week 38/2005) fixes items 1–3. Remaining relevant items:
- #5 SPI module (all revisions incl. B4): holding CS low long after an SPI command that sets TXREQ (SPI RTS in mode 1,1, or Byte-Write of TXREQ in either mode) can make the frame repeat. Workaround: raise CS within <47 µs of the command, use SPI RTS in mode 0,0, or use TXnRTS pins. (→ MCU driver must release CS promptly.)
- #4 CAN module: buffer corruption if a higher-priority buffer becomes pending exactly during the trigger pulse of a lower-priority one; workaround: RTS all buffers together / send one at a time.
- #1 oscillator (rev ≤B0 only): oscillator may keep running in Sleep if OSC1 DC level too low — fixed in B2.
- No clarification/correction list entries ("None"). Whether a newer errata revision exists: NOT FOUND.

**LCSC / JLCPCB** (snapshot 2026-11-02):

| MPN | Package | LCSC | JLC class | Stock | Price @1 |
|---|---|---|---|---|---|
| MCP2515-I/SO | SOIC-18-300mil | C12368 | Ext | 928 | $1.71 |
| MCP2515T-I/SO | SOIC-18-300mil | C153782 | Ext | 0 (reel) | $1.54 |
| MCP2515-I/ST, -I/ML, -E/SO | — | NOT FOUND | — | — | — |

---

# B) CAN transceivers

## B1. MCP2562 / MCP2562FD (Microchip) — DS20005167C / DS20005284A

(FD datasheet electrical tables identical to MCP2562 for every row checked: abs max, IDD, IIO, thermal, permanent-dominant time.)

| Parameter | Value | Source |
|---|---|---|
| VDD | 4.5 – 5.5 V (abs max 7.0 V) | §2.2 DC Characteristics p9; Abs Max p8 |
| VIO | 1.8 – 5.5 V (abs max 7.0 V) | §2.2 p10; p8 |
| IDD | recessive typ 5 / max 10 mA; dominant typ 45 / max 70 mA (RL = 60 Ω) | §2.2 p9 |
| IIO (VIO current) | recessive typ 4 / max 30 µA; dominant typ 85 / max 500 µA | §2.2 p10 |
| Standby: IDDS / IIO(S) | 5 µA typ, 15 µA max (incl. IIO); VIO standby 0.3 / 1 µA | §2.2 p9–10 |
| Max bit rate | MCP2562: 1 Mbit/s (fBIT 14.4 kHz–1000 kHz). MCP2562FD: optimized for CAN FD 2, 5, 8 Mbit/s | §2.3 AC Characteristics Table, p13; FD p1 |
| Bus pin DC abs max | CANH, CANL (and SPLIT) −58 V to +58 V; transient (ISO-7637) −150 V…+100 V | Abs Max p8 |
| Bus common-mode range | operation ensured −12 V … +12 V by characterization (tested −2…+7 V) | §2.2 Note 3, p10 |
| ESD on CANH/CANL | IEC 61000-4-2 **±8 kV for MCP2562** (±14 kV applies to MCP2561; front-page bullet is ambiguous); HBM ±8 kV; other pins HBM ±4 kV; MM ±300 V; CDM ±750 V | Abs Max p8; p1 |
| TXD dominant time-out | "permanent dominant detection" tPDT **1.25 ms typ**; min/max NOT GIVEN. Min bit rate 14.4 kHz. Same detect on bus in standby (RXD forced recessive) | §1.5 p4; AC Table item 11 p13 |
| TXD/STBY thresholds | VIH min 0.7·VIO; VIL max 0.3·VIO | §2.2 p11 |
| RXD output | VOH ≥ VIO − 0.4 V @ −1 mA; VOL ≤ 0.4 V @ 4 mA | §2.2 p11 |
| VDD absent, VIO present | CANH/CANL stay **high-impedance** until both VDD and VIO exceed UV thresholds; VDD UV: POR high 3.8–4.3 V, low 3.4–4.0 V | §1.6 p4; §2.2 p9 |
| VIO absent / below UV, VDD present | Same §1.6 statement: bus high-Z until *both* rails above UV (VIO UV typ 1.2 V). Behavior if VIO drops *after* power-on is not stated explicitly (text covers only VDD brown-out going high-Z). Bus wake-up needs both rails valid | §1.6 p4; §1.1.2 p3 |
| Unpowered leakage | CANH/CANL ±5 µA max @ VDD = VIO = 0 V, 5 V on bus; "CAN bus pins are disconnected when device is unpowered" | §2.2 ILI p10; p1 |
| Back-powering VIO→VDD | NOT FOUND (not discussed) | — |
| STBY pin | pin 8; **high = Standby**, low = Normal. Internal MOS pull-up to VIO (typ 660 kΩ @5 V, 1.1 MΩ @3.3 V, 4.4 MΩ @1.8 V) ⇒ **defaults to Standby if left open; must be tied low** | §1.1, §1.7.9 p5; Table 1-1 |
| TXD pull-up | internal 33 kΩ nominal to VIO (MCP2562) → recessive if floating | §1.7.1 p5 |
| Pinout SOIC-8 / DFN-8 | 1 TXD, 2 VSS, 3 VDD, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STBY, 9 EP (DFN, to VSS) | Table 1-2 p4 |
| Propagation | TXD→bus dominant ≤70 ns; TXD→RXD ≤125 ns (↓)/235 ns (↑) (MCP2562). FD: max prop delay 120 ns | §2.3 p13; FD p1 |
| Thermal | θJA SOIC-8 149.5 °C/W; DFN 3×3 56.7; PDIP 89.3. Thermal shutdown typ 175 °C | §2.5 p16; §2.2 p11 |
| Temperature | E: −40…+125 °C; H: −40…+150 °C | p1, PIS |
| Orderable | MCP2562-E/SN, MCP2562T-E/SN, -H/SN, -E/MF (DFN), -E/P; MCP2562FD-E/SN, MCP2562FDT-E/SN, -E/MF… | Product Identification System p27 (FD p~29) |

LCSC: MCP2562-E/SN **C53609** (SOIC-8, Ext, stock 525, $1.37); MCP2562T-E/SN **C191417** (Ext, 8687, $0.98); MCP2562FD-E/SN **C124016** (Ext, 572, $1.72); MCP2562FDT-E/SN **C511335** (Ext, 4160, $1.54).

## B2. TJA1051T/3 (NXP) — Rev. 9

| Parameter | Value | Source (Rev. 9, page "n of 25") |
|---|---|---|
| VCC | 4.5 – 5.5 V (abs max −0.3…+7 V any non-bus pin) | Table 7 p8; Table 5 Limiting values p7 |
| VIO | **2.8 – 5.5 V** (only /3 and TK/3 variants); VIO input tied to VCC internally on versions without the pin | Table 7 p8; Table 2 note |
| ICC | recessive typ 5 / max 10 mA; dominant typ 50 / max 70 mA; silent mode 0.1/1/2.5 mA; bus-short dominant up to 110 mA | Table 7 p8 |
| IIO | recessive typ 80 / max 250 µA; dominant typ 350 / max 500 µA | Table 7 p8 |
| Max bit rate | Guaranteed to 5 Mbit/s in CAN FD fast phase; minimum 20 kbit/s (set by TXD time-out) | p1; §7.2.1 p6 |
| Bus DC abs max | CANH, CANL −58 … +58 V; V(CANH−CANL) −27 … +27 V; transients pulses 1 −100 V, 2a +75 V, 3a −150 V, 3b +100 V | Table 5 p7 |
| Common-mode range | receiver thresholds guaranteed −30 V ≤ VCANH, VCANL ≤ +30 V | Table 7 p9 (Vth(RX)dif etc.) |
| ESD | IEC 61000-4-2 ±8 kV on CANH/CANL; HBM ±8 kV bus, ±4 kV other pins; MM ±300 V; CDM ±750 V corner / ±500 V other | Table 5 p7 |
| TXD dominant time-out | tto(dom)TXD min 0.3 / typ 1 / max 5 ms (≥0.8 ms for SAE J2284; 0.3 ms ISO 11898-2:2016 legacy) | Table 8 p10 and note [4] p11 |
| TXD/S thresholds | VIH min 0.7·VIO (max VIO+0.3; if VCC>VIO, max VCC+0.3); VIL max 0.3·VIO | Table 7 p8–10 note [5] |
| RXD | VOH: IOH −1 mA min at VRXD = VIO−0.4 V; IOL 5 mA typ at 0.4 V | Table 7 p8 |
| VCC absent, VIO present | VCC under-voltage (3.5–4.5 V) → transceiver switches off and **disengages from bus (zero load)** | §7.2.3 p6 |
| VIO absent / low, VCC present | VIO UV (1.3 / 2.0 / 2.7 V min/typ/max) → switches off, disengages from bus (zero load) until both recover | §7.2.3 p6; Table 7 p8 |
| Unpowered | "ideal passive behavior"; leakage −5…+5 µA @ VCC = VIO = 0 V (or shorted via 47 kΩ), bus 5 V | p1; Table 7 p9 |
| Back-powering | NOT FOUND (not discussed) | — |
| Silent pin | name **S**, pin 8, **high = Silent mode** (transmitter off), low = Normal; internal **pull-down** to GND ⇒ defaults to **Normal** if floating. TXD has internal pull-up to VIO | §7.1.2, §7.2.2 p5–6 |
| Pinout SO8 (/3) | 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 S. HVSON8 (TK/3) same pin order + exposed pad | Table 3 p4 |
| Thermal | Rth(vj-a) SO8 155 K/W; HVSON8 55 K/W | Table 6 p7 |
| Temperature | Tvj −40…+150 °C, AEC-Q100 | Table 5 p7; p1 |
| Variants | TJA1051T (no VIO), TJA1051T/3 (SO8, VIO), TJA1051T/E (EN pin), TJA1051TK/3 (HVSON8, VIO). The suffix "/1J" (reel/packing code, e.g. TJA1051T/3/1J) is NOT explained in this datasheet | Table 2 p2–3 |

LCSC: TJA1051T/3/1J (NXP, SOIC-8-150mil) **C38695** — Ext, stock **104 951**, $0.60. Tokmas TJA1051T/3 (third-party clone, not NXP) **C5342108** — Ext, 14 628, $0.57. TJA1051T/3/CM,118 (NXP) **C2671195** — Ext, stock 11.

## B3. SN65HVD230 (TI) — SLOS346O

| Parameter | Value | Source |
|---|---|---|
| VCC | **3.0 – 3.6 V only** (abs max 6 V) | §8.3 Recommended Operating Conditions p6; §8.1 p5 |
| VIO pin | **none**; D (TXD) and R (RXD) referenced to VCC (3.3 V); digital pins abs max −0.5…VCC+0.5 V | §8.1 p5; pin table p4 |
| ICC | no-load dominant and recessive typ 10 mA / max 17 mA; standby (Rs = VCC) typ 370 / max 600 µA | §8.5 Driver electrical characteristics p7 |
| Max bit rate | 1 Mbit/s | p1 |
| Bus pin DC abs max | −4 … +16 V; transient ±25 V through 100 Ω. **Does not meet ±27 V (DR-05)** | §8.1 p5 |
| Common-mode (recommended) | −2 … +7 V (single pin −2.5 … +7.5 V) | §8.3 p6 |
| ESD | HBM ±16 kV on CANH/CANL/GND; ±4 kV all pins; CDM ±1 kV. IEC 61000-4-2: NOT FOUND | §8.2 p6 |
| TXD dominant time-out | NOT FOUND (no such feature in text) | — |
| Thresholds | VIH(D,R) 2 V, VIL 0.8 V (absolute, not VIO-relative) | §8.3 p6 |
| VCC absent behaviour | bus input current at VCC = 0 V: +100…+350 µA (bus at +7 V), −100…−20 µA (bus at −2 V) ⇒ **loads the bus when unpowered** | §8.6 Receiver "II bus input current" p7–8 |
| Mode pin | **Rs (pin 8)**: strong pull-down = high speed; 10–100 kΩ to GND = slope control; pull-up (>0.75·VCC) = standby/listen-only. Internal default: NOT STATED | Pin table p4; §8.3 p6 |
| Pinout SOIC-8 (D) | 1 D (TXD), 2 GND, 3 VCC, 4 R (RXD), 5 Vref (VCC/2), 6 CANL, 7 CANH, 8 Rs | p4 |
| Thermal | RθJA 76.8 °C/W (D) | §8.4 p6 |
| Temperature | −40 … +85 °C | §8.3 p6 |

LCSC: SN65HVD230DR **C12084** — **Preferred (Pref Ext)**, stock **104 398**, $0.69.
Disqualifiers vs D-001/D-002/D-003 (as stated in spec): 3.3 V VCC only (cannot sit on 5 V rail, abs max 6 V), no VIO pin, bus abs max 16 V < 27 V requirement, no TXD time-out, loads bus when unpowered.

## B4. TI VIO-pin candidates

Two TI families have a VIO pin; both pin-compatible with the 8-pin SOIC footprint (pin 5 = VIO).

### TCAN1042HGV / TCAN1042HV (TI) — SLLSES7D (best commercial-grade TI VIO candidate found; **H = ±70 V bus**, G = 5 Mbit/s)

| Parameter | Value | Source |
|---|---|---|
| VCC | 4.5 – 5.5 V (abs max −0.3…7 V) | §7.4 p8; §7.1 p6 |
| VIO | **3.0 – 5.5 V** recommended (abs max −0.3…7 V; logic inputs ≤ VIO+0.3 V) | §7.4 p8; §7.1 p6 |
| ICC | dominant (RL 60 Ω) typ 40 / max 70 mA; (RL 50 Ω) typ 45 / max 80 mA; recessive typ 1.5 / max 2.5 mA; standby 0.5/5 µA (V variants, VCC not needed) | §7.7 Electrical Characteristics p9 (min/typ column assignment of 1st number is my reading of extraction) |
| IIO | normal mode typ 90 / max 300 µA (RXD floating); standby 12/17 µA | §7.7 p9 |
| Max bit rate | 2 Mbit/s (all), 5 Mbit/s for "G" devices | p1 §3 |
| Bus DC abs max | −70 … +70 V (H variants); −58…+58 V for non-H | §7.1 p6; p1 |
| Common mode | −30 … +30 V (normal mode, STB = 0) | §7.7 receiver table p12 |
| ESD | HBM ±16 kV bus; ±6 kV all pins; CDM ±1.5 kV; IEC 61000-4-2 contact ±15 kV unpowered / ±8 kV powered (SOIC); IEC EFT ±4 kV | §7.2 p6; §7.3 p7 |
| TXD dominant time-out | min 1.2 ms, max 3.8 ms (typ not extracted); min data rate = 11 bits / tDTO | §7.7 switching table; §9.3.1 |
| Logic thresholds (V variants) | TXD, STB VIH 0.7·VIO, VIL 0.3·VIO; RXD VOH ≥ 0.8·VIO @ −2 mA, VOL ≤ 0.2·VIO @ 2 mA | §7.7 p9–10 |
| VCC absent, VIO present | STB low → "Protected": bus **high-impedance**, RXD high (recessive). STB high → Standby: bus output recessive, RXD = wake request | Table 9-2 p20 |
| VIO absent/UV, VCC present | "Protected": bus high-impedance, RXD high-impedance. VIO UV detect 1.3–2.75 V, hysteresis 80 mV | Table 9-2 p20; §7.7 p9 |
| Unpowered | ideal passive; bus leakage ≤ 4.8 µA @ VCC=VIO=0, bus 5 V; logic pins ±1 µA | §9.3.4 p20; §7.7 |
| Floating pins | TXD pulled up (recessive); **STB pulled up ⇒ defaults to Standby if floating; tie STB to GND for normal operation** | §9.3.5 p20 |
| Pinout SOIC-8 (V variants) | 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STB (active-high standby) | Table 6-1 p5 |
| Thermal | RθJA D (SOIC) 105.8 °C/W; DRB (VSON) 48.3; TTSD 170 °C | §7.5 p8 |
| Temperature | TA −55…+125 °C (electrical tables), TJ −55…+150 °C | §7.7 p9; §7.1 p6 |
| Orderable | TCAN1042HGVDR, TCAN1042HGVDRG4, TCAN1042HVDR, TCAN1042HVDRG4 (SOIC-8 reels); VSON options TCAN1042HGVD**RB**… NOT extracted | orderable addendum p~38–41 |

LCSC: TCAN1042HGVDR **C124014** (Ext, 2253, $1.06); TCAN1042HVDR **C127869** (Ext, 990, $0.89); TCAN1042HGVDRQ1 C132550 (Ext, 109); TCAN1042HVDRQ1 C485807 (Ext, 2976); TCAN1042VDRQ1 C485806 (Ext, 8328); TCAN1042HDR (no VIO) C130131.

### TCAN1462V-Q1 (TI, automotive Q1 only) — SLLSFF2B (CAN FD + SIC, 8 Mbit/s)

| Parameter | Value | Source |
|---|---|---|
| VCC / VIO | 4.5–5.5 V (abs max 6 V) / **1.7 – 5.5 V** (abs max 6 V) | §6.4 p6; §6.1 p5 |
| ICC | dominant (60 Ω) typ 45 / max 70 mA; recessive typ 4.5 / max 8 mA; standby (V devices) 0.2 typ/2 µA max @ ≤125 °C | §6.6 p6 |
| IIO | dominant typ 125 / max 300 µA; recessive typ 25 / max 48 µA | §6.6 p7 |
| Max bit rate | classic CAN and CAN FD up to 8 Mbit/s | p1 |
| Bus abs max | ±58 V; VDIFF ±45 V | §6.1 p5 |
| Common mode | −12 … +12 V | §6.8 p8 |
| ESD | HBM bus ±10 kV (class 3B), others ±4 kV; CDM ±750 V; IEC 62228-3/ISO 10605 ±8 kV contact; SAE J2962-2 powered air ±15 kV | §6.2–6.3 p5 |
| TXD time-out | tTXD_DTO min 1.2 / max 4.0 ms | §6.9 switching table p9 |
| Thresholds | TXD/STB VIH 0.7·VIO, VIL 0.3·VIO (V devices) | §6.8 p8–9 |
| UV levels | VCC rise 4.2–4.4 / fall 3.5–4.0 V; VIO rise 1.6–1.65 / fall 1.4–1.5 V | §6.6 p7 |
| UV behaviour (V) | VCC<UV & VIO ok: STB=VIO → standby / STB=GND → "Protected"; VCC ok & VIO<UV or both<UV → Protected, bus high-impedance, RXD high-Z. (Bus-column wording for the VCC<UV rows is ambiguous in my text extraction.) | Table 8-2 §8.3.6 |
| Unpowered | ideal passive; bus leakage ±5 µA @ VCC=VIO=GND | §8.3.7; §6.8 |
| Floating pins | TXD pull-up; **STB pull-up ⇒ default low-power standby**; STB may be tied to GND | §8.3.8 Table 8-3 |
| Pinout SOIC-8/SOT-23-8 | 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO (NC on non-V), 6 CANL, 7 CANH, 8 STB | Table 5-1 p4 |
| Thermal | RθJA SOIC 120, SOT-23 (DDF) 115.3, VSON (DRB) 52.8 °C/W | §6.5 p6 |
| Temperature | TJ −40…+150 °C, AEC-Q100 grade 1 | §6.4 p6; p1 |
| Orderable names for the V variant | NOT FOUND in extracted text (device comparison lists TCAN1462V-Q1 only) | — |

LCSC: TCAN1462V — NOT FOUND. Only TCAN1462DRQ1 (non-V) **C22445414** (Ext, 837, $1.00).

## B5. Transceiver comparison (all rows from tables above)

| Item | MCP2562-E/SN | MCP2562FD-E/SN | TJA1051T/3 | SN65HVD230 | TCAN1042HGV/HV | TCAN1462V-Q1 |
|---|---|---|---|---|---|---|
| VDD / VCC | 4.5–5.5 | 4.5–5.5 | 4.5–5.5 | **3.0–3.6** | 4.5–5.5 | 4.5–5.5 |
| VIO | 1.8–5.5 | 1.8–5.5 | **2.8**–5.5 | none | 3.0–5.5 | 1.7–5.5 |
| Dominant IDD (typ/max mA) | 45/70 | 45/70 | 50/70 | 10/17 (no load) | 40/70 | 45/70 |
| Recessive IDD (typ/max mA) | 5/10 | 5/10 | 5/10 | 10/17 | 1.5/2.5 | 4.5/8 |
| VIO current dominant (max) | 500 µA | 500 µA | 500 µA | — | 300 µA | 300 µA |
| Max bitrate | 1 M | 8 M (FD) | 5 M (FD) | 1 M | 2 M / 5 M (G) | 8 M (FD, SIC) |
| Bus abs max | ±58 V | ±58 V | ±58 V (±27 diff) | −4…+16 V | ±70 V (H) | ±58 V |
| Common mode | ±12 | ±12 | ±30 | −2…+7 | ±30 | ±12 |
| ESD bus (IEC / HBM) | ±8 kV / ±8 kV | ±8 kV / ±8 kV | ±8 kV / ±8 kV | n/a / ±16 kV | ±15 (unpwr) ±8 (pwr) / ±16 kV | ±8 kV / ±10 kV |
| TXD DTO | 1.25 ms typ only | same | 0.3–1–5 ms | none | 1.2–3.8 ms | 1.2–4.0 ms |
| Standby/silent pin default | **STBY pulled up = Standby** | same | **S pulled down = Normal** | Rs: not stated | **STB pulled up = Standby** | **STB pulled up = Standby** |
| VCC absent, VIO present | bus high-Z | bus high-Z | bus off (zero load) | n/a | high-Z (STB low) | see table |
| VIO absent, VCC present | bus high-Z (at power-on) | same | bus off | n/a | high-Z | high-Z |
| Unpowered bus leakage | ±5 µA | ±5 µA | ±5 µA | 100–350 µA | ≤4.8 µA | ±5 µA |
| θJA SOIC-8 | 149.5 | 149.5 | 155 | 76.8 | 105.8 | 120 |
| Temp | −40…125 (E) | −40…125 | −40…150 Tvj | −40…85 | −55…125 | −40…150 |
| LCSC | C53609 Ext | C124016 Ext | C38695 Ext | C12084 Pref | C124014 / C127869 Ext | not found |

Flags (facts, not selections):
- MCP2562 / TCAN1042 / TCAN1462V: STBY/STB defaults to **standby** when open — a pull-down/tie to GND is needed for the module to transmit. TJA1051 S defaults to Normal.
- TJA1051T/3: VIO minimum 2.8 V (OK for 3.0 V host, but only 0.2 V below the spec floor).
- TCAN1042HGV: VIO recommended range 3.0–5.5 V matches DR-02 exactly.
- SN65HVD230: disqualified on VCC (3.3 V only), no VIO, bus abs max.

---

# C) 5 V regulators (7–16 V in, 100 mA continuous / 150 mA design)

Dissipation context `[CALC]`: (16−5) V × 0.10 A = 1.10 W; × 0.15 A = 1.65 W; at 9 V, 0.1 A: 0.40 W. Temperature rise = P × θJA only where θJA is a board-realistic figure (JEDEC values are for comparison).

## C1. LDOs

| Parameter | AMS1117-5.0 (SOT-223) | L78M05 (DPAK) | TLV76750 (TLV767 fixed 5 V) | TPS7B6950-Q1 (SOT-223) | TPS7A25 | LM2936-5.0 |
|---|---|---|---|---|---|---|
| Source | AMS orig. copy (Waveshare mirror) p2–4 | ST datasheet NOT RETRIEVED | TI SLVSE84D | TI SLVSCJ8B | TI SBVS372C | TI SNOSC48O |
| Vin abs max | **15 V** (Absolute Max Ratings p2) — *16 V input exceeds it*. SLKOR clone says 30 V (clone datasheet p2; vendor-dependent) | 35 V per distributor listings (secondary) | 18 V (§6.1 p5) | 45 V (200 ms); 40 V recommended (§7.1, 7.3 p4) | 20 V (§6.1 p4) | 40 V operating; 60 V HV version |
| Vin recommended max | 12 V in the electrical-spec conditions (p2–3) | NOT FOUND | **16 V** (§6.3 p5) | 40 V (min 5.5 V for 5 V output at 1 mA; ±3 % spec from 6.5 V) | 18 V | 40 V |
| Max Iout | 800 mA (AMS) / 1 A | 500 mA | 1 A | **150 mA** | 300 mA | **50 mA — disqualified (<100 mA)** (p1) |
| Dropout @100 mA | not specified; max 1.3 V at full 800 mA ("decreasing at lower load currents") p3 | secondary: ≈2 V (test current not verified) | ≈0.09 V `[CALC]` from 0.9 V typ/1.5 V max @1 A (RDS(on) scaling §8.3.2); no 100 mA row | typ 450 / max 800 mV @100 mA (§7.5 p5) | typ ~64–120 mV (50/150 mA rows; 5 V version) | typ ~0.2 V @50 mA |
| Iq / min load | quiescent 5 typ/10 mA max; **min load 10 mA** (Table p3) | ~6 mA (secondary) | 50 µA typ/80 max (60/95 fixed) §6.5 p6 | 15 µA typ/25 max (§7.5 p5) | 2 µA typ/4.5 µA max | ≤15 µA |
| Output cap | 22 µF **solid tantalum** (ESR ≤0.5 Ω in clone; AMS: "22 µF solid tantalum ensures stability") p4 — ceramic-only is a stability risk | NOT FOUND | 1–220 µF, ESR 2–500 mΩ, effective ≥0.5 µF (§6.3 p5) | 2.2–100 µF, ESR 1 mΩ–2 Ω, stable with ceramic (p1, §7.3 p4) | 1–2.2–100 µF (derate to 50 %) §6.3 p5 | 10 µF * (p1) |
| Input cap | 10 µF tantalum typ. (clone p5) | NOT FOUND | ≥1 µF | cover schematic shows 10 µF & 4.7 µF (which is CIN not stated) | ≥1 µF | 100 nF * |
| Fixed/adj | fixed 5.0 V ±2 % (4.90–5.10) | fixed 5 V ±2 % (AB/AC) / ±4 % (C) secondary | fixed 5.0 V (TLV76750) ±1 % | fixed ±3 % | **fixed 5.0 V ordering code not in orderable list** (only TPS7A2501 adj. listed) | fixed |
| θJA | SOT-223 90 °C/W (p2, "46…>90 depending on copper") | NOT FOUND | DGN 60.1, DRV 77.7 (§6.4 p6) | DCY(SOT-223) 64.2; DBV 210.4 (§7.4 p4) | DRV 73.3 (§6.4 p5) | — |
| ΔT at 1.1 W `[CALC]` | 99 K (θJA 90) | — | 66 K (DGN) / 85 K (DRV) | 70 K | 81 K | — |
| Package | SOT-223/TO-252 | DPAK/TO-220 | HVSSOP-8 (DGN), WSON-6 2×2 (DRV) | SOT-223-4 (DCY), SOT-23-5 | WSON-6 2×2 | SOIC/SOT-223/TO-252 |
| Ext. parts | 2 caps (tantalum) | 2 caps | 2 caps | 2 caps | 2 caps + PG pullup | 2 caps |
| Orderable | AMS1117-5.0 | L78M05ABDT-TR, L78M05ACDT-TR, L78M05CDT-TR | TLV76750DGNR, TLV76750DRVR, TLV76750DRVRG4 | TPS7B6950QDCYRQ1 (also TPS7B6950-Q1 SOT-23-5 DBV) | TPS7A2501DRVR (adj only) | LM2936 family |
| LCSC | **C6187 Basic**, stock 128 344, $0.23 | L78M05ABDT-TR **C58069 Basic** 168 191 $0.20; L78M05CDT-TR C55509 Ext 119 988 $0.15; ACDT C173987 Ext 3 913 | **NOT FOUND** | C108469 Ext, **stock 0** (DBV C174327 stock 0) | NOT FOUND | NOT FOUND |

Notes: TPS7A25 (WSON 2×2) and AMS1117 abs-max 15 V are flagged. LM2936: 50 mA — disqualified for 100 mA. LD1117S50 (ST) and MP1584: official datasheets NOT RETRIEVED, no LCSC number found; the TI LM1117 datasheet (https://www.ti.com/lit/ds/symlink/lm1117.pdf, 800 mA 15 V-class) is on file but was not extracted.

## C2. Buck converters (5 V out)

| Parameter | TPS560430 (X = 1.1 MHz PFM) | TPS62933 | AP63205 (fixed 5 V) | MP1584 |
|---|---|---|---|---|
| Source | TI SLVSE22B | TI SLUSEA4D | Diodes DS41326 R3-2 | NOT RETRIEVED |
| Vin range (rec.) | 4 – 36 V (§7.3 p4) | 3.8 – 30 V (§8.3 p5) | 3.8 – 32 V (Recommended Operating Cond.) | NOT FOUND |
| Vin abs max | 38 V | 32 V | 35 V DC; 40 V for 400 ms | NOT FOUND |
| Iout | 0.6 A | 3 A (2 A TPS62932) | 2 A | NOT FOUND |
| Iq (non-switching) | 80 typ / 120 µA max (§7.5 p5) | 12 µA typ | 22 µA typ | NOT FOUND |
| Output | adjustable (1.000 V ref) — **no fixed 5 V option** (fixed 3.3 V X3F only) | adjustable (0.8 V ref) | **fixed 5.0 V** (AP63205WU-7) | NOT FOUND |
| fsw | 1.1 MHz (X) / 2.1 MHz (Y) | 200 kHz–2.2 MHz via RT; 500 kHz with RT floating (5 V ref design) | 1.1 MHz (+ ±6 % spread spectrum) | NOT FOUND |
| Mode | PFM or FPWM versions | PFM (TPS62933), FCCM (F), OOA (O) | PWM/PFM | NOT FOUND |
| Package / θJA | SOT-23-6 (DBV) 173 °C/W (not for design use) | SOT583 (DRL) 112.2 JEDEC / 60.2 on EVM | TSOT26 89 °C/W | NOT FOUND |
| Recommended L / C for 5 V | L 18 µH, COUT 22 µF/10 V, RFBT 88.7 kΩ, RFBB 22.1 kΩ (Table 1 p15); CIN: NOT extracted (10 µF-class) | 6.8 µH, COUT 2×22 µF, CIN 2×10 µF + 100 nF, boot 100 nF, SS 33 nF, R 511k/88.7k etc. (Fig 10-1, Table 10-1 p28; 5.5–30 V in, 3 A ref design) | 4.7 µH, C1 (in) 10 µF, C2 (out) 2×22 µF, C3 (boot) 100 nF (Table 3) | NOT FOUND |
| Ext. parts (approx.) | L, CIN, COUT, CBOOT, RFBT, RFBB = 6 | L, 2 CIN + 100 nF ×3, 2 COUT, CSS, 5–7 resistors ≈ 15 | L, CIN, 2 COUT, CBOOT = 5 | NOT FOUND |
| Capacitor ESR constraint | none stated beyond output ripple; ceramic expected | peak-current-mode, internally compensated; ceramics used | "22–68 µF ceramic", ESR sets ripple (Eq. 9) | NOT FOUND |
| Orderable | TPS560430XDBVR | TPS62933DRLR | AP63205WU-7 (Q-grade AP63205QWU-7 exists) | MP1584EN-LF-Z (unverified) |
| LCSC | C524782 Ext, 33 424, $0.60 | C3200405 Ext, 23 952, $0.53 | C2071056 Ext, 16 759, $0.44 | NOT FOUND (C14259 resolves to MP2359DJ, not MP1584) |

Dissipation of switchers not tabulated (efficiency curves not extracted).

---

# D) 3.3 V LDO from 5 V, ≤50 mA

| Parameter | AMS1117-3.3 | XC6206P332MR | TLV75533P (TLV75533PDBVR) | AP2112K-3.3 | ME6211C33M5G |
|---|---|---|---|---|---|
| Source | AMS orig. copy (Waveshare) | NOT RETRIEVED (torex 403) | TI SBVS320D | Diodes DS39724 R2-2 | NOT RETRIEVED |
| Vin abs max | 15 V | NOT FOUND (secondary snippet: output 150–250 mA, dropout 0.25–0.68 V — unverified) | 6.0 V (§5.1 p4) | 6.5 V (Abs Max p3) | NOT FOUND |
| Vin rec. | 4.75–12 V in spec table | NOT FOUND | 1.45 – 5.5 V (§5.3 p4) | 2.5 – 6.0 V | NOT FOUND |
| Max Iout | 800 mA | NOT FOUND | 500 mA | 600 mA | NOT FOUND |
| Dropout | max 1.3 V @800 mA (none at 50 mA) | NOT FOUND | max 238 mV @500 mA (p1); 50 mA row NOT in table (scales with current) | typ 5 / max 8 mV @10 mA; 125/200 mV @300 mA (3.3 V table p8) | NOT FOUND |
| Iq | 5 typ/10 mA max, **min load 10 mA** | NOT FOUND | 25 µA typ (14–31 µA at 25 °C, 40 µA max hot) §5.5 p5 | 55 µA typ / 80 µA max | NOT FOUND |
| Cout | 22 µF tantalum (AMS) | NOT FOUND | 1 – 200 µF, CIN ≥ 1 µF (§5.3) | 1 µF ceramic stable (X7R/X5R recommended, p1 & note 4 p3) | NOT FOUND |
| Package / θJA | SOT-223 90 °C/W | SOT-23-3L | SOT-23-5 (DBV) 231 JEDEC / 100.8 EVM; DYD 92.5/60.3; DQN X2SON | SOT-25 184, SO-8 114 °C/W | SOT-23-5 |
| Power at 5→3.3 V, 50 mA `[CALC]` | 85 mW | 85 mW | 85 mW | 85 mW | 85 mW |
| LCSC | C6186 **Basic**, 1 782 182, $0.20 | C5446 (XC6206P332MR-G) **Basic**, 521 014, $0.12 | C404027 Ext, 103 076, $0.15 | C51118 (AP2112K-3.3TRG1) Ext, 7 369, $0.16 | C82942 (ME6211C33M5G-N) Ext, 859, $0.05 |

Flags: AMS1117-3.3 has a 10 mA minimum load (3.3 V rail would need a ≥10 mA bleed) and needs a tantalum-class output cap; this applies equally to the 5 V AMS1117.

---

# E) Summary of items NOT FOUND

- Official datasheets not retrievable: ST L78M05 / LD1117S50, Torex XC6206, Microne ME6211, MPS MP1584, AMS (original site), NXP TJA1051 (mirror used).
- MCP2515 crystal ESR / drive level; newer errata than DS80179G.
- MCP2562: tPDT min/max, VIO-dropping-after-power-on behaviour, back-power data.
- TJA1051 back-power data; meaning of "/1J" suffix.
- SN65HVD230 IEC ESD, Rs default state, TXD time-out.
- TCAN1462V-Q1 orderable names and any LCSC listing; TCAN1042H min/typ column split on a few ICC rows.
- LCSC numbers: TLV76750, LD1117S50, MP1584, TPS7A2550, TCAN1462V, L7805-class TO-263.
- Datasheet rows not verified in this pass: efficiency curves for buck candidates, input capacitor for TPS560430/TPS7B6950.
