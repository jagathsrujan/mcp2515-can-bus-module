# Discrete / passive / connector datasheet extract (research only, no selection)

Author: Sonnet research pass, 2026-11-02. Project spec: `docs/ENGINEERING_SPEC.md` (9 V adapter, operate 7-16 V per DR-01, CAN bus miswire to +/-27 V per DR-05, JLCPCB Economic PCBA, Basic parts preferred, 0603 passives).

## How to read this file

- **Source** column: official datasheet document, table/section and PDF page. Page numbers are *PDF page index* of the file I read, not the printed page number, unless stated "printed".
- Where the manufacturer site blocked my fetcher (HTTP 403 / Akamai), I read the **same document hosted by LCSC or a distributor mirror**. That is flagged "MIRROR" with the exact URL and revision date. Opus should confirm no newer revision exists before final selection.
- "LCSC # / Basic?" comes from the JLCPCB parts API (`selectSmtComponentList`) queried 2026-11-02. `Basic` = `componentLibraryType: base`. `Ext` = Extended. Stock is a snapshot, not a guarantee.
- **NOT FOUND** = I could not read it from a datasheet. Nothing in this file is from memory.
- *(CALC)* = my arithmetic from the quoted numbers. *(interp.)* = interpretation / assumption.
- "JLC listing" = attribute text from the JLCPCB/LCSC catalogue (secondary source, not a datasheet).

Scratch copies of every datasheet read are in the session scratchpad `.../scratchpad/ds/` (not part of the project).

---

## 0. Cross-cutting findings that affect several categories

1. **Gate voltage during an input surge exceeds every candidate P-FET's Vgs rating if the TVS is downstream of the FET (D-006) and the gate is tied to GND through a resistor.** The TVS clamps the FET source side to Vc (24.4 V for SMAJ15A, 29.2 V for SMAJ18A/SMBJ18A, Section 2), so Vgs = -Vc, which is beyond +/-20 V as well as +/-12 V. A gate zener (or other gate clamp) is needed for *all* P-FET candidates, not just the +/-12 V AO3401A. *(CALC from Sections 1 and 2.)*
2. **BZT52C12 (12 V zener) cannot protect the AO3401A.** Zener max Vz = 12.7 V at 5 mA vs AO3401A Vgs abs max +/-12 V. A 10 V (or 9.1 V) zener would be needed with AO3401A; see Section 1.
3. **All 24 V-class CAN TVS parts (VRWM 24 V, VBR min 25.5-26.7 V) are in or near breakdown at a sustained 27 V DC miswire (DR-05).** SOT-23 TVS arrays have only pulse (8/20 us) ratings in their datasheets, no continuous-power rating. Either DR-05 is relaxed to <= 24 V DC, or a 27 V part is needed (PESD2CANFD27V-TR, LCSC C552488: JLC listing only, datasheet NOT READ).
4. **120 Ohm termination under a 24 V miswire:** 24 V across 2 x 60.4 Ohm = 0.199 A; each 0603 resistor dissipates ~2.4 W vs 0.1 W rating (24x over) *(CALC)*. JP1 must be open in any miswire scenario, or the termination resistors will burn out. Also a TVS in snapback/clamp would not reduce this since DC.
5. **MLCC DC-bias loss is severe (Section 8): 22 uF/25 V X5R 0805 retains only ~12 % (-87.6 %) at 16 V; 10 uF/25 V X5R 0805 -81.6 % at 12 V.** Bulk capacitance on a 12-16 V input must be sized from the biased value, or X7R/50 V parts used. This feeds the Phase 3 regulator-input-cap choice.
6. 60.4 Ohm and 62 Ohm 0603 1 % are **Extended** only on JLC. 120 Ohm 0603 1 % is Basic. Basic-only alternative for the split termination: 2 x 120 Ohm in parallel per leg *(interp.; 4 parts)*.

---

## 1. Reverse-polarity P-MOSFET (SOT-23), gate clamp zener, Schottky alternative

### 1.1 Candidate spec tables

#### AO3401A (Alpha & Omega) - LCSC C15127

| Parameter | Value | Source |
|---|---|---|
| Vds max | -30 V | AOS AO3401A datasheet, Rev 3.1 Dec 2023, Absolute Maximum Ratings, p.1. https://www.aosmd.com/res/data_sheets/AO3401A.pdf |
| Vgs max | **+/-12 V** | same, p.1 (**DISQUALIFIER for direct gate-to-GND at 16 V: Vgs = -16 V; gate clamp mandatory**) |
| Id continuous | -4.0 A (TA 25 C), -3.2 A (70 C); IDM -27 A | same, p.1 |
| Rds(on) | -10 V, -4.0 A: typ 41, max 50 mOhm; -4.5 V, -3.5 A: typ 47, max 60 mOhm; -2.5 V, -2.5 A: typ 60, max 85 mOhm | same, Electrical Characteristics, p.2 |
| Vgs(th) | min -0.5, typ -0.9, max -1.3 V (Vds=Vgs, Id=-250 uA) | same, p.2 |
| Pd | 1.4 W (25 C), 0.9 W (70 C) | same, p.1 |
| RthetaJA | t<=10 s: typ 70 / max 90 C/W; steady state: typ 100 / max 125 C/W; RthetaJL 63/80 C/W | same, Thermal Characteristics, p.1 (read from rendered page) |
| Pinout | SOT-23: pin 1 G, pin 2 S, pin 3 D | same, p.1 Top View drawing |
| LCSC / Basic? | **C15127, Basic**, stock 1.71 M, AOS brand. (C347476/C306862 etc. are other-vendor "AO3401A" Extended.) | JLCPCB API 2026-11-02 |

#### AO3407A - LCSC C347478 (UMW clone; no AOS-branded AO3407A found)

| Parameter | Value | Source |
|---|---|---|
| Vds max | -30 V | AOS AO3407A datasheet Rev 5.1 Jul 2023, p.1. https://www.aosmd.com/res/data_sheets/AO3407A.pdf |
| Vgs max | +/-20 V (still < 24-29 V surge; gate clamp needed per 0.1) | same, p.1 |
| Id continuous | -4.3 A (25 C), -3.5 A (70 C); IDM -25 A | same, p.1 |
| Rds(on) | -10 V, -4.3 A: typ 34, max 48 mOhm; -4.5 V, -3 A: typ 54?, max 78 mOhm (table text order: 34/48, 52/68 at 125 C, 54/78) | same, p.2 (typ for 4.5 V not unambiguous in extracted text; max values 48 / 78 mOhm are clear from Product Summary p.1) |
| Vgs(th) | min -1.4, typ -1.9, max -2.4 V | same, p.2 |
| Pd | 1.4 W / 0.9 W | same, p.1 |
| RthetaJA | same layout as AO3401A: 70/90 (t<=10 s), 100/125 steady state C/W (read from text, ordering inferred from AO3401A page image) | same, p.1 |
| LCSC / Basic? | **C347478, Ext**, UMW (Youtai) brand, stock 170 k. Electrical numbers above are AOS's, not UMW's: UMW datasheet NOT READ | JLCPCB API |

#### DMP3098L (Diodes Inc) - LCSC C150492 (DMP3098L-7)

| Parameter | Value | Source |
|---|---|---|
| Vds max | -30 V | Diodes DMP3098L, DS31447 Rev 8-2, Oct 2013, Maximum Ratings, p.2. https://www.diodes.com/datasheet/download/DMP3098L.pdf (the `/assets/Datasheets/` path 404s) |
| Vgs max | +/-20 V | same, p.2 |
| Id continuous | -3.8 A (VGS -10 V, TA 25 C), -2.9 A (70 C); IDM -11 A | same, p.2 |
| Rds(on) | -10 V, -3.8 A: typ 56, max 70 mOhm; -4.5 V, -3.0 A: typ 98, max 120 mOhm | same, Electrical Characteristics, p.2 |
| Vgs(th) | min -1.0, typ -1.8, max -2.1 V | same, p.2 |
| Pd / RthetaJA | 1.08 W; RthetaJA 115 C/W (2 oz, 0.5 in^2 Cu, t<=5 s, Note 5) | same, p.2 |
| LCSC / Basic? | **C150492, Ext**, stock 165 k | JLCPCB API |
| Note | Old datasheet revision (2013); no newer one found at the Diodes URL I used. | |

#### Si2301CDS (Vishay) - LCSC C10487 (Si2301CDS-T1-GE3) - **DISQUALIFIED**

| Parameter | Value | Source |
|---|---|---|
| Vds max | **-20 V** | Vishay Si2301CDS, doc 68741 Rev C, 25-Oct-10, Absolute Maximum Ratings, p.1. https://www.vishay.com/doc?68741 |
| Vgs max | **+/-8 V** | same, p.1 |
| Id | -3.1 A (TC 25 C); -2.3 A (TA 25 C, 1x1 in FR4, t=5 s) | same, p.1 |
| Rds(on) | -4.5 V, -2.8 A: typ 90, max 112 mOhm; -2.5 V, -2.0 A: typ 110, max 142 mOhm | same, MOSFET Specifications, p.2 |
| Vgs(th) | -0.4 to -1.0 V | same, p.2 |
| RthJA | 5 s typ 120 / max 145 C/W; steady state max 175 C/W (note d) | same, p.1 |
| LCSC / Basic? | **C10487, Basic**, stock 124 k | JLCPCB API |
| Disqualifier | Vds -20 V < 24 V miswire margin and < TVS clamp; Vgs +/-8 V is unusable at 16 V without a heavy gate divider. | |

#### DMG2305UX (Diodes Inc) - LCSC C150470 (DMG2305UX-7), C144153 (-13) - **DISQUALIFIED**

| Parameter | Value | Source |
|---|---|---|
| Vds max | **-20 V** | Diodes DMG2305UX datasheet, Maximum Ratings, p.2. https://www.diodes.com/datasheet/download/DMG2305UX.pdf |
| Vgs max | **+/-8 V** | same, p.2 |
| Id | -4.2 A (VGS -4.5 V, 25 C steady state), -5.0 A (t<10 s) | same, p.2 |
| Rds(on) | -4.5 V, -4.2 A: typ 40, max 52 mOhm; -2.5 V, -3.4 A: typ 52, max 100 mOhm | same, Electrical Characteristics, p.2 |
| Pd / RthetaJA | 1.4 W; 90 C/W steady state, 64 C/W t<10 s | same, p.2 |
| LCSC / Basic? | C150470 Ext (stock 4.0 k), C144153 Ext (stock 11.8 k) | JLCPCB API |
| Disqualifier | Same as Si2301: 20 V / 8 V. | |

#### DMP4065S (Diodes Inc, -40 V part) - LCSC C182476 (DMP4065S-7)

| Parameter | Value | Source |
|---|---|---|
| Vds max | -40 V | Diodes DMP4065S, DS37646 Rev 3-2, Aug 2021, Maximum Ratings, p.2. https://www.diodes.com/datasheet/download/DMP4065S.pdf |
| Vgs max | +/-20 V | same, p.2 |
| Id continuous | -3.4 A (1 in^2 Cu, TA 25 C), -2.4 A (min pad); IDM -20 A | same, p.2 |
| Rds(on) | -10 V, -4.2 A: typ 64, max 80 mOhm; -4.5 V, -3.3 A: typ 85, max 100 mOhm | same, p.3 |
| Vgs(th) | min -1.0, max -3.0 V | same, p.3 |
| Pd / RthetaJA | 1.4 W / 90 C/W (1 in^2 Cu); 0.72 W / 171 C/W (min pad) | same, p.2 |
| LCSC / Basic? | **C182476, Ext**, stock 9.6 k (lowest stock of the group); C5122018 TECH PUBLIC clone Ext stock 1.3 k | JLCPCB API |

#### Gate-clamp zener candidates

| Parameter | BZT52C12 (Changjiang) LCSC C43491 | MMSZ5242BT1G (onsemi) LCSC C82050 | BZT52C10 (Changjiang) LCSC C2103 |
|---|---|---|---|
| Vz nominal / range | 12 V; 11.4-12.7 V @ 5 mA | 12 V; 11.40-12.60 V @ 20 mA | 10 V; 9.4-10.6 V @ 5 mA |
| Power | 500 mW (ceramic PCB); RthetaJA 250 C/W | 500 mW on FR-5 at TL 75 C; RthetaJA 340 C/W | same family as C43491 |
| Package | SOD-123 | SOD-123 | SOD-123 |
| Source | Jiangsu Changjiang BZT52Cxx datasheet (LCSC copy, MIRROR) https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_Jiangsu-Changjing-Electronics-Technology-Co---Ltd--BZT52C12_C43491.pdf, Electrical table p.1; Note: Table row `BZT52C12 WH 12 11.4 12.7 5 25 150 ...` | onsemi MMSZ52xxB datasheet (LCSC copy, MIRROR) https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_onsemi-MMSZ5242BT1G_C82050.pdf, Electrical Characteristics table | same PDF as C43491, row `BZT52C10 WF 10 9.4 10.6 5 20 150 ...` |
| Basic? | **Ext** (stock 211 k) | **Ext** (stock 36.6 k) | **Ext** (stock 67 k) |
| Flag | Vz max 12.7 V **> AO3401A Vgs 12 V abs max**; OK for +/-20 V parts | same flag | OK for AO3401A (10.6 V max < 12 V) |

No Basic 12 V / 10 V zener found on JLC (searched BZT52C12/C10/C9V1 and MMSZ5242B: all Extended).

#### Schottky alternative to the P-FET

| Parameter | SS34 (MDD) LCSC C8678 | B5819W SL (Changjiang) LCSC C8598 |
|---|---|---|
| Package | SMA (DO-214AC) | SOD-123 |
| Vrrm / Io | 40 V / 3.0 A | 40 V / 1 A (IFRM 1.5 A) |
| Vf at 1 A / 3 A | SS34: 0.55 V at 3.0 A (1 A value NOT in table) | B5819W: 0.6 V max at 1 A, 0.9 V max at 3 A |
| **Vf at 100 mA** | **NOT FOUND** (only the typical forward curve, no table value) | **NOT FOUND** (curve only) |
| Ir | 0.5 mA at 25 C, 5 mA at 125 C (rated Vr) | 1 mA at 40 V (Max) |
| Rth | RthetaJA 70 C/W (5x5 mm pad) | RthetaJA 200 C/W |
| Source | MDD SS32-SS3200, Rev 2024A5, Max Ratings and Electrical Characteristics table, p.1; curves p.2. https://www.lcsc.com/datasheet/lcsc_datasheet_2407101107_MDD-Microdiode-Semiconductor-SS34_C8678.pdf (MIRROR) | CJ B5817W-5819W, D Mar 2015, Electrical Characteristics, p.1. https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_Jiangsu-Changjing-Electronics-Technology-Co---Ltd--B5819W-SL_C8598.pdf (MIRROR) |
| Basic? | **Basic**, stock 3.2 M | **Basic**, stock 961 k |
| Note | 3 A part; 1 W dissipation at 1 A not an issue at the 100 mA budget | 100 mA budget fits; Io 1 A |

### 1.2 Comparison (P-MOSFETs)

| Part | LCSC | Basic? | Vds | Vgs max | Rds(on) max -4.5 V / -10 V | Id (25 C) | RthetaJA (steady) | Status |
|---|---|---|---|---|---|---|---|---|
| AO3401A | C15127 | **Basic** | -30 V | **+/-12 V** | 60 / 50 mOhm | -4.0 A | 125 max (100 typ) | OK only with <=10 V gate clamp |
| AO3407A | C347478 | Ext | -30 V | +/-20 V | 78 / 48 mOhm | -4.3 A | 125 max (100 typ) | gate clamp still needed (0.1); datasheet is AOS, part is UMW |
| DMP3098L-7 | C150492 | Ext | -30 V | +/-20 V | 120 / 70 mOhm | -3.8 A | 115 | gate clamp needed; highest Rds(on) |
| Si2301CDS | C10487 | Basic | **-20 V** | **+/-8 V** | 112 / n.a. | -3.1 A (TC) | 145 max (5 s) / 175 max SS | **DISQUALIFIED** |
| DMG2305UX-7 | C150470 | Ext | **-20 V** | **+/-8 V** | 52 / n.a. | -4.2 A | 90 | **DISQUALIFIED** |
| DMP4065S-7 | C182476 | Ext | -40 V | +/-20 V | 100 / 80 mOhm | -3.4 A (1 in^2) | 90 (1 in^2), 171 (min pad) | OK; only -40 V choice, low stock |

Conduction loss at the 100 mA budget is negligible for all (100 mA x 0.1 Ohm = 10 mV, *CALC*). Rds(on) does not discriminate; Vgs and Vds do.

---

## 2. Input TVS (SMA/SMB)

Operating limit from spec DR-01: 16 V; reverse-miswire 16 V; downstream regulator abs-max is a Phase 3 input.

| Parameter | SMAJ15A | SMAJ18A | SMBJ18A |
|---|---|---|---|
| Vendor / datasheet | Littelfuse SMAJ series, rev 11/20/15 (MIRROR https://static.chipdip.ru/lib/299/DOC012299232.pdf); Electrical Characteristics table p.2, ratings p.1. Official page: littelfuse.com SMAJ series (direct PDF returned 403) | same document | Littelfuse SMBJ series, rev 07/04/25 (LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_Littelfuse-SMBJ18A_C151256.pdf, MIRROR); Electrical table p.2, ratings p.1 |
| VR (standoff VWM) | 15.0 V | 18.0 V | 18.0 V |
| VBR min / max @ IT | 16.70 / 18.50 V @ 1 mA | 20.00 / 22.10 V @ 1 mA | 20.00 / 22.10 V @ 1 mA |
| Clamp VC at Ipp | 24.4 V @ 16.4 A | 29.2 V @ 13.7 A | 29.2 V @ 20.6 A |
| Ppk (10/1000 us) | 400 W | 400 W | 600 W |
| Max leakage IR at VR | 1 uA | 1 uA | 1 uA |
| RthetaJA typ | 120 C/W | 120 C/W | 100 C/W |
| Package | SMA = DO-214AC | SMA | SMB = DO-214AA |
| LCSC # (Littelfuse) | C148216 (stock 15.8 k) | C148219 (stock 11.1 k) | C151256 (stock 14.9 k) |
| Basic? | Ext | Ext | Ext |
| Cheaper alternates (JLC listing only) | C113958 MDD 8.3 k; C86819 Brightking 59.9 k; C110520 Diodes SMAJ15A-13-F 12.3 k | C173513 MDD 26.2 k; C134965 Diodes SMAJ18A-13-F 8.5 k | C86368 Starsea 2.0 k; C135055 Diodes 3.0 k |
| Flags | **VR 15 V < 16 V max operating (DR-01)**: at 16 V it is above standoff (leakage > 1 uA, close to VBR min 16.7 V). Vc 24.4 V is the lowest of the three. | VR 18 V > 16 V OK. Vc 29.2 V: downstream parts must tolerate 29.2 V; vs AO3401A Vds -30 V margin is 0.8 V | Same VR and Vc as SMAJ18A at 1.5x surge current |

None are Basic. Stock-wise all fine.

---

## 3. CAN bus dual-line ESD/TVS (SOT-23)

All: SOT-23, pins 1 and 2 = the two protected lines, pin 3 = common (GND) unless stated. Bidirectional.

| Parameter | NXP/Nexperia PESD2CAN | Nexperia PESD2CANFD24V-T | onsemi NUP2105L | TI ESD2CAN24-Q1 | Littelfuse SM24CANB-02HTG | Nexperia PESD1CAN |
|---|---|---|---|---|---|---|
| VRWM | 24 V | 24 V | 24 V min | 24 V (-24..24 V) | 24.0 V (IR<=1 uA) | 24 V |
| VBR | 26.2 / 28 / 30.3 V @ 1 mA | 25.5 / 30 / 35.5 V @ 10 mA | 26.2 min / 32 max @ 1 mA | 25.5 min / 35.5 max @ +/-10 mA, then snapback (VHold 30 V typ) | 26.7 V min @ 1 mA | 25.4 / 27.8 / 30.3 V @ 5 mA |
| Capacitance per line to GND (0 V, 1 MHz) | typ 25 / max 30 pF | typ 5.2 / max 6 pF (@ 2.5 V bias) | typ 26 / max 30 pF | typ 3 / max 5 pF (TI table, "ESD2CAN24-Q1" column) | 30 pF (typ vs max not distinguished in table) | typ 11 / max 17 pF |
| Clamp VC | 34 V max @ 1 A; 41 V max @ 5 A (8/20 us) | 33 typ / 42 max @ 1 A (8/20 us) | 33 typ / 40 max @ 5 A; 37 typ / 44 max @ 8 A (8/20 us) | 37 V (typ, @ 5.7 A 8/20 us); TLP 35 V @ 16 A | 34 V @ 1 A, 46 V @ 8 A, 50 V @ 10 A (8/20 us) | 40 V max @ 1 A; 70 V max @ 3 A |
| Ipp / Ppk (8/20 us) | 5 A / 230 W | 2.6 A | 8 A / 350 W | 5.7 A / 210 W | 10 A / 500 W | 3 A (JLC listing; limiting-values table page NOT captured) / 200 W |
| IEC 61000-4-2 | 30 kV contact (Table 6), class 4 (>8 kV contact, >15 kV air, Table 7) | 23 kV contact | +/-30 kV contact and air; ISO 10605 +/-30 kV | +/-30 kV contact and air | +/-30 kV contact and air | 23 kV contact; class 4 (>8 kV contact) |
| Pinout | 1 = K1, 2 = K2, 3 = common cathode | 1 = K1, 2 = K2, 3 = CC | Style 28: pins 1, 2, 3 all "anode" (die schematic: pin 3 common); line/GND roles = by app drawing | 1, 2 = IO; 3 = GND | Pin 1, 2 lines; pin 3 common | 1 = K1, 2 = K2, 3 = common cathode |
| Automotive | AEC-Q101 | AEC-Q101 | SZ- variant AEC-Q101 | AEC-Q101 | AEC-Q101 | not stated in extracted text |
| Source | PESD2CAN Rev 2, 27 Sep 2012, Tables 1, 2, 5, 6, 7, 8 (PDF p.2-5). Official https://www.nexperia.com/documents/data-sheet/PESD2CAN.pdf (403 to my tool); read LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_1811151539_Nexperia-PESD2CAN-215_C75176.pdf MIRROR. Product page says a 2023 revision may exist: NOT READ. NRND status reported by Octopart: NOT VERIFIED | PESD2CANFD24V-T, 20 Dec 2019, Tables 1, 2, 5, 6 (PDF p.1-4). Read MIRROR https://static.chipdip.ru/lib/959/DOC011959296.pdf. Official: nexperia.com/product/PESD2CANFD24V-T | NUP2105L/D Rev 12, June 2026, Maximum Ratings p.1, Electrical Characteristics p.2. https://www.onsemi.com/pdf/datasheet/nup2105l-d.pdf (official, read directly) | ESD2CANxx24-Q1 SLVSFW5D, Mar 2026, Pin Functions p.3, 5.3 p.4, 5.7 Electrical Characteristics p.6. https://www.ti.com/lit/ds/symlink/esd2can24-q1.pdf (official, read directly) | SM24CANB, Littelfuse Revision 12/08/16 (MIRROR https://datasheet.chipsmall.com/SM24CANB-02HTG/311667.pdf), Abs Max / Electrical tables p.3. Newer rev 04/29/22 at https://www.mouser.com/datasheet/2/240/media-3319137.pdf (blocked): values may differ | PESD1CAN Rev 04, 15 Feb 2008, Table 8 (PDF p.4), ESD Table 6/7 (p.3). LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_1811141231_Nexperia-PESD1CAN-215_C15771.pdf MIRROR |
| LCSC # / Basic? | C75176 (Nexperia PESD2CAN,215) **Ext**, stock 37.9 k; clones C284078 DOWO 96 k, C907830 TECH PUBLIC 100 k (their specs differ: 17-30 pF, 250-420 W - JLC listing only) | C552486 (PESD2CANFD24V-TR) **Ext**, stock 7.9 k; also C28933048 PESD2CAN24T-QR (Nexperia, 13.5 pF, VBR 31 V; JLC listing only) | C14486 (NUP2105LT1G) **Ext**, stock 217 k; C233494 SZNUP2105LT1G Ext 215 k | **NOT FOUND on JLC** for TI ESD2CAN24-Q1; only look-alikes: C2942563 Yangjie ESD2CAN24T2Q Ext 27 pF, C19170841 ElecSuper ESD2CAN24-ES Ext | C151237 **Ext**, stock 482 (low); C2937019 TECH PUBLIC Ext 7.6 k (17 pF, 250 W, different part) | C15771 (PESD1CAN,215) **Ext**, stock 124 k |

Disqualifier / flag summary for Section 3:
- **None is Basic.** Every 24 V CAN TVS is Extended on JLC.
- **DR-05 (+/-27 V DC miswire):** all six have VRWM 24 V and VBR min 25.4-26.7 V, so they conduct at 27 V DC. Only PESD2CANFD27V-TR (C552488: JLC listing says 27 V, 38 V, 44 V, 6 pF; datasheet NOT READ) would stand off 27 V.
- **PESD1CAN:** Vc 70 V max at 3 A: weak clamp for a transceiver whose bus-pin abs-max is typically +/-40 V or lower (check in Phase 5 against the chosen transceiver; NOT READ here).
- **Capacitance:** only PESD2CANFD24V-T (6 pF max) and ESD2CAN24-Q1 (5 pF max) are low enough to matter above 1 Mbit/s; for <= 1 Mbit/s the 25-30 pF parts load the bus lightly vs the transceiver's own pin cap (transceiver value NOT READ here).
- **SM24CANB-02HTG:** stock only 482 at snapshot.
- **ESD2CAN24-Q1:** snapback device (VHold 30 V typ): when used on a node that is powered at up to 24 V bus fault, verify VHold > worst bus DC; NOT FOUND on JLC anyway.
- Pin 3 must go to the GND plane with >= 2 vias (spec section 6 copper plan item 6).

---

## 4. Common-mode choke for CAN (optional footprint, DNP/0 Ohm default)

| Parameter | TDK ACT45B-510-2P-TL003 | TDK ACT1210-510-2P-TL00 | Murata DLW43SH510XK2L | Bourns SRF2012-510Y |
|---|---|---|---|---|
| Inductance (common mode) | 51 uH @ 100 kHz, -30/+50 % | 51 uH @ 100 kHz (JLC listing) | 51 uH @ 1 MHz? (JLC listing, unreliable) | **NOT FOUND** (no JLC hit; Bourns datasheet URL returned a web page, not PDF) |
| Impedance | Zmin 1000 Ohm, Ztyp 2800 Ohm @ 10 MHz (common mode, 100 mV) | 2.6 kOhm @ 10 MHz (JLC listing; 1 kOhm min per C48928231 listing) | **NOT FOUND** | NOT FOUND |
| Rated current | 200 mA | 200 mA (JLC listing) | 230 mA (JLC listing) | NOT FOUND |
| DCR max per winding | 1.0 Ohm | 0.7 Ohm (JLC listing) | 1 Ohm (JLC listing) | NOT FOUND |
| Rated voltage | 50 V DC | 80 V (JLC listing) | 50 V (JLC listing) | NOT FOUND |
| Package / footprint | EIA 1812, 4.5 x 3.2 mm, 4-pad; KiCad `Inductor_SMD:L_1812_4532Metric` is only a rough 2-pad fit, 4-pad CMC footprint **must be custom**: pad/layout drawing is on p.3 of the TDK PDF (graphic, dimensions not extracted as text) | 1210, 3.2 x 2.5 mm (JLC listing) | 4.5 x 3.2 mm (JLC listing) | NOT FOUND |
| Temperature / qual | -40..+150 C, AEC-Q200 | -55..+150 C (JLC listing) | -40..+125 C (JLC listing) | NOT FOUND |
| Source | TDK ACT45B, April 2025, Characteristics and ordering codes table p.4 (ordering code ACT45B-510-2P-TL003, internal B82787C0513H002), features p.2, layout p.3. https://www.tdk-electronics.tdk.com/inf/30/ds/act45b.pdf (official, read) | TDK ACT1210 datasheet URL blocked (product.tdk.com 403). Values only from LCSC listing | Murata datasheet NOT RETRIEVED (murata.com returned HTML). LCSC listing only | NOT FOUND |
| LCSC # / Basic? | C76584 TDK, **Ext**, stock **10** (effectively unobtainable); alternates C48928226 MetalLions ACT45B-510-2P-TF Ext 1.2 k (230 mA, 1 Ohm, 2.8 kOhm typ), C49230370 CND-tek Ext 1.1 k | C95572 TDK **Ext**, stock 29.6 k; C48928231 MetalLions ACT1210-510-2P-TF Ext 1.1 k | C91595 **Ext**, stock 8.4 k (DLW43SH510XK2L) | none found |

Notes: rated current 200 mA is above the ~70 mA CAN dominant budget. Single-ended DC resistance 1 Ohm adds 0.07 V drop per leg at 70 mA *(CALC)*. Footprint is not in the KiCad standard library as a 4-pad CMC (checked `Inductor_SMD.pretty`: only 2-pad chip inductors).

---

## 5. Crystals (16 MHz and 20 MHz)

Stray-capacitance assumption 3-5 pF per pin (brief). Load-cap formula CL = (C1 x C2)/(C1 + C2) + Cstray, so for C1 = C2 = C: C = 2 x (CL - Cstray). *(CALC)*

| Parameter | YXC X322516MLB4SI (16 MHz, SMD 3225) LCSC C13738 | YXC X322520MSB4SI (20 MHz, SMD 3225) LCSC C9004 | YXC X49SD16MSD2SC (16 MHz HC-49S THT) LCSC C16212 | YXC X49SD20MSD2SC (20 MHz HC-49S) LCSC C16370 |
|---|---|---|---|---|
| Frequency tolerance @ 25 C | +/-10 ppm (also +/-20 ppm option in series sheet) | +/-10 ppm | +/-20 ppm | +/-20 ppm |
| Stability over temp | +/-20 ppm, -40..+85 C | +/-20 ppm, -40..+85 C | +/-30 ppm, -20..+70 C | +/-30 ppm, -20..+70 C |
| Load capacitance CL | **9 pF** (JLC listing; the YXC series sheet says 10 pF / 20 pF "or specify") | 20 pF | 20 pF | 20 pF |
| ESR max | 50 Ohm (series sheet: 16-31 MHz = 50 Ohm max) | 50 Ohm | 45 Ohm | 45 Ohm |
| Drive level | 10-200 uW max, 100 uW typical (series sheet) | same series: NOT individually checked | NOT FOUND (datasheet not read) | NOT FOUND |
| Shunt capacitance C0 | 3 pF max (series sheet) | same series | NOT FOUND | NOT FOUND |
| Aging | +/-3 ppm/year max | same series | NOT FOUND | NOT FOUND |
| Package | 3.2 x 2.5 x 0.7 mm, 4 pads; crystal on pads 1 and 3, pads 2 and 4 = GND; suggested land: 1.4 x 1.2 mm pads, 0.8 mm gap across, 0.5 mm gap between rows; KiCad `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm` | same | HC-49S (through-hole), KiCad `Crystal:Crystal_HC49-U_Vertical` or `Crystal_SMD_HC49-SD` (check pitch) | same |
| Source | YXC YSX321SL series sheet, p.1 (specs, ESR table, land pattern). LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_2403291504_YXC-Crystal-Oscillators-X322516MLB4SI_C13738.pdf (image PDF read by eye) MIRROR; CL = 9 pF and +/-10 ppm and +/-20 ppm are from the JLC attributes table | JLC attributes (C9004); series sheet LCSC copy .../YXC-Crystal-Oscillators-X322520MSB4SI_C9004.pdf NOT individually read | JLC attributes (C16212) | JLC attributes (C16370) |
| Basic? | **Basic**, stock 352 k | **Ext**, stock 31.5 k | Ext, stock 19.7 k | Ext, stock 9.7 k |

Load capacitor values *(CALC)*:

| Crystal | CL | Cstray 3 pF | Cstray 4 pF | Cstray 5 pF |
|---|---|---|---|---|
| X322516MLB4SI | 9 pF | 12 pF | 10 pF | 8 pF |
| X322520MSB4SI / HC-49S | 20 pF | 34 pF | 32 pF | 30 pF |

Other Extended 16 MHz 3225 options seen (JLC listing only, datasheet NOT READ): C12668 X322516MOB4SI 12 pF, C112972 X322516MMB4SI 10 pF, C70562 X322516MRB4SI 18 pF, NDK NX3225SA-16MHz C880097 (8 pF, 80 Ohm). No Basic 20 MHz 3225 crystal found. Flag: 20 MHz needs an Extended part; also spec Section 4 item 3 (MCP2515 oscillator limit at VDD = 3.3 V) is a Phase 4 check not done here.

---

## 6. Termination resistors and capacitor

| Item | Candidate | LCSC # | Basic? | Value / rating | Source |
|---|---|---|---|---|---|
| 60.4 Ohm 0603 1 % | YAGEO RC0603FR-0760R4L | C245987 | **Ext**, stock 194 k | 100 mW, 75 V, +/-100 ppm | JLC listing (Yageo datasheet NOT READ) |
| 60.4 Ohm 0603 1 % | UNI-ROYAL 0603WAF604JT5E | C23090 | Ext, stock 262 | 100 mW | JLC listing |
| 62 Ohm 0603 1 % | UNI-ROYAL 0603WAF620JT5E | C23222 | **Ext**, stock 142 k | 100 mW, 75 V | JLC listing |
| 62 Ohm 0603 1 % | YAGEO RC0603FR-0762RL | C245989 | Ext, stock 16 k | 100 mW | JLC listing |
| 120 Ohm 0603 1 % | UNI-ROYAL 0603WAF1200T5E | C22787 | **Basic**, stock 2.03 M | 100 mW, 75 V, +/-100 ppm, -55..+155 C | UNI-ROYAL 0603WAF series datasheet (LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_2206010130_UNI-ROYAL-Uniroyal-Elec-0603WAF1200T5E_C22787.pdf) + JLC listing; table page not individually extracted |
| 4.7 nF 50 V X7R 0603 | FH 0603B472K500NT | C53987 | **Basic**, stock 840 k | 4.7 nF, 50 V, X7R, +/-10 % | JLC attributes; datasheet LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_FH--Guangdong-Fenghua-Advanced-Tech-0603B472K500NT_C53987.pdf |
| 4.7 nF 50 V X7R 0603 | YAGEO CC0603KRX7R9BB472 | C106218 | Ext | | JLC listing |
| 4.7 nF 50 V C0G 0603 | none found in a JLC search | - | - | **NOT FOUND** (C0G at 4.7 nF/50 V/0603 not located; only TDK 130/200 pF C0G showed) | |

Power check *(CALC)*: nominal CAN dominant differential 2 V: 120 Ohm sees 33 mW total, each 60 Ohm leg 17 mW; 100 mW rating is fine. For the 24 V miswire case see Section 0 item 4. Note 4.7 nF centre-tap cap sees only common-mode: 50 V rating OK vs 24-27 V.

---

## 7. Connectors (all through-hole; all Extended on JLC)

| Item | Candidate | LCSC # | Basic? | Ratings | Pin layout / footprint note | Source |
|---|---|---|---|---|---|---|
| DC barrel jack 5.5 x 2.1 mm | CUI PJ-002AH | C2961147 | **Ext**, stock 1.28 k | 24 V DC, **5.0 A**, 2.0 mm centre pin, contact resistance 30-50 mOhm | 3 terminals (centre pin, sleeve, switch); PCB layout p.2 shows slots 1.0 x 3.5 and 1.0 x 3.0, hole spacing; KiCad `Connector_BarrelJack:BarrelJack_Horizontal` is the generic fit, **verify pad geometry against p.2** (not extracted as text). Polarity per spec (centre positive presumed) NOT CONFIRMED | CUI PJ-002AH datasheet dated 10/30/2019, p.1-2 (LCSC copy https://www.lcsc.com/datasheet/lcsc_datasheet_2410010101_CUI-PJ-002AH_C2961147.pdf) |
| DC barrel jack (cheap) | BOOMELE DC-005 2.0 | C16214 | Ext, stock 87.6 k | 12 V, **1 A** (JLC listing) | datasheet is a drawing only (no ratings text); same footprint family | JLC listing |
| Screw terminal 5.00 mm 2-pin / 3-pin | Cixi Kefa KF301-5.0-2P / -3P | C474881 / C474882 | Ext, stock 53 k / 24.7 k | 250 V, 17 A (JLC listing), wire 14-22 AWG | **Pitch is 5.00 mm, not the 5.08 mm in the spec.** Do not mix with a 5.08 footprint. | JLC listing |
| Screw terminal 5.08 mm 2-pin / 3-pin | Cixi Kefa KF128-5.08-2P-AA / -3P-AA | C474952 / C474953 | **Ext**, stock 56 k / 6.7 k | 250 V, 24 A (JLC listing), 12-22 AWG, 5.08 mm | KiCad `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal` / `...MKDS-1,5-3-5.08_1x03_P5.08mm_Horizontal` exist in the local library; KF128 hole/pad dimensions NOT CHECKED against its datasheet | JLC listing |
| Screw terminal DG301-5.0 (DEGSON) | DG301-5.0-02P-12-00A(H) / DG301-5.0-03P-12-00A(H) | C5371917 / C5128849 | Ext, stock 13.7 k / 220 | 250 V, 17.5 A (JLC listing), **5.0 mm pitch** | `TerminalBlock_Degson` library only has DG250 in this install; DG301 not present | JLC listing |
| 2.54 mm 1x8 male header | SHOU HAN SH-PZ254V-1x8P | C55629815 | Ext, stock 1.6 k | 3 A, 1 kV (JLC listing) | `Connector_PinHeader_2.54mm:PinHeader_1x08_P2.54mm_Vertical` (exists); alt hanxia HX PZ2.54-1x8P ZZ C32713274 Ext 19.5 k | JLC listing |
| 2.54 mm 1x2 male header | JXTCONN PH2.54-1X2P-H25 | C42431798 | Ext, stock 34.7 k | 3 A | `Connector_PinHeader_2.54mm:PinHeader_1x02_P2.54mm_Vertical` | JLC listing |
| Jumper shunt 2.54 mm | BOOMELE "2.54 Short Circuit Cap Closed" | C100114 | Ext, stock 363 k | 3 A (JLC listing) | not a footprint part (accessory; JLC will not place it; hand-fit) | JLC listing |

JLC THT assembly of these parts, and which of them JLC can place on the Economic tier, is a Phase 5 question for `research/jlcpcb_capabilities.md`; not resolved here.

---

## 8. LED and general passives

### 8.1 Indicator LEDs 0603

| Parameter | Red: Hubei KENTO KT-0603R (LCSC C2286) | Green: KT-0603G (C12624) |
|---|---|---|
| Basic? | **Basic**, stock 6.97 M | **Ext**, stock (see JLC) |
| Vf | 1.8-2.4 V @ 20 mA (bins VE 1.8-1.9 up to VJ 2.3-2.4); **Vf at 2 mA NOT FOUND** (not specified) | 3.1 V (JLC listing, 525 nm "emerald" InGaN, test current 5 mA); datasheet NOT READ |
| Iv | 145-300 mcd @ 20 mA | 430 mcd (JLC listing) |
| Abs max | IF 25 mA, IFP 60 mA, VR 5 V, Pd 40 mW | NOT READ |
| Source | KENTO KT-0603R approval sheet, Rev A.0, 2018-12-06, section 4-6, p.3-4. https://www.lcsc.com/datasheet/lcsc_datasheet_1810231112_Hubei-KENTO-Elec-KT-0603R_C2286.pdf | JLC listing |
| Footprint | `LED_SMD:LED_0603_1608Metric` | same |

No Basic green 0603 LED located in my searches (all JLC "Green" hits were placeholders or Extended). With +5 V and 1.5 kOhm: red Iled ~ (5 - 2)/1.5k = 2 mA *(CALC, spec 2.5)*; a 3.1 V green would be ~1.3 mA *(CALC)*. At 2 mA the red LED is below its 20 mA test point, so brightness is a bench question.

### 8.2 General passives (all JLC Basic, 2026-11-02)

| Part | LCSC # | Basic? | Stock | Datasheet / source |
|---|---|---|---|---|
| 100 nF 50 V X7R 0603, YAGEO CC0603KRX7R9BB104 | C14663 | **Basic** | 40.1 M | JLC attributes; datasheet LCSC copy .../lcsc_datasheet_2211101700_YAGEO-CC0603KRX7R9BB104_C14663.pdf |
| 10 uF 25 V X5R 0805, Samsung CL21A106KAYNNNE | C15850 | **Basic** | 8.08 M | JLC attributes |
| 22 uF 25 V X5R 0805, Samsung CL21A226MAQNNNE (+/-20 %) | C45783 | **Basic** | 3.62 M | JLC attributes |
| 22 uF 25 V X5R 1206, Samsung CL31A226KAHNNNE (+/-10 %) | C12891 | **Basic** | 1.60 M | JLC attributes |
| 10 uF 25 V X5R 1206, Samsung CL31A106KAHNNNE | C9807 | Ext | - | JLC listing |
| 10 kOhm 0603 1 %, UNI-ROYAL 0603WAF1002T5E | C25804 | **Basic** | 47.2 M | JLC attributes; 100 mW, 75 V, +/-100 ppm |
| 1 kOhm 0603 1 %, 0603WAF1001T5E | C21190 | **Basic** | 30.1 M | same |
| 1.5 kOhm 0603 1 %, 0603WAF1501T5E | C22843 | **Basic** | 3.53 M | same |
| 4.7 kOhm 0603 1 %, 0603WAF4701T5E | C23162 | **Basic** | 14.8 M | UNI-ROYAL 0603WAF datasheet (LCSC copy .../2206010116_UNI-ROYAL-Uniroyal-Elec-0603WAF4701T5E_C23162.pdf) |

0603 resistor power rating: 100 mW (JLC attribute `Power(Watts)=100mW` for all UNI-ROYAL 0603WAF rows; Section 6 above).

### 8.3 MLCC DC-bias derating (Samsung Electro-Mechanics, published simulation data)

Source: Samsung SEM product page data embedded as `datasheetData` graphType "DCBias" (DeltaC %, 25 C), pages https://product.samsungsem.com/mlcc/CL21A226MAQNNN.do (22 uF 0805; test 120 Hz, 0.5 Vrms), https://product.samsungsem.com/mlcc/CL21A106KAYNNN.do (10 uF 0805; 1 kHz, 1.0 Vrms), https://product.samsungsem.com/mlcc/CL31A226KAHNNN.do (22 uF 1206; 120 Hz, 0.5 Vrms). Fetched 2026-11-02. This is the manufacturer's simulation library, not a controlled datasheet table; it matches the "SEM Simulation" tool. Values read to the nearest sampled point.

| DC bias | CL21A226MAQNNN 22 uF 0805 | CL21A106KAYNNN 10 uF 0805 | CL31A226KAHNNN 22 uF 1206 |
|---|---|---|---|
| 5 V | -51.3 % (10.7 uF) | -50.3 % (4.97 uF) | -38.0 % (13.6 uF) |
| 9 V | -74.9 % (5.5 uF) | -75.2 % (2.5 uF) | -65.4 % (7.6 uF) |
| 12 V | -81.4 % (4.1 uF) | -81.6 % (1.8 uF) | -73.9 % (5.7 uF) |
| 16 V | **-87.6 % (2.7 uF)** | -87.5 % (1.25 uF) | **-82.3 % (3.9 uF)** |
| 20 V | -90.5 % | -90.5 % | -86.8 % |
| 25 V | -92.7 % | -92.7 % | -90.0 % |

Absolute uF in brackets are *(CALC)* = nominal x (1 + deltaC). The generic Samsung "CL" datasheet on LCSC (the 140 kB multi-series PDF) contains no DC-bias curve at all. Takeaway: at 16 V all three X5R 25 V parts retain under 20 % of nominal; 9 V adapter operation keeps ~25-35 %. 100 nF X7R 50 V DC-bias: **NOT FOUND** (Yageo curve not read).

---

## 9. Consolidated flags (disqualifiers and gaps)

Disqualifiers
- Si2301CDS (C10487, Basic) and DMG2305UX (C150470/C144153): Vds -20 V and Vgs +/-8 V. Not usable at 16 V in, or 24 V miswire.
- AO3401A (C15127, the only Basic P-FET): Vgs +/-12 V, so a gate clamp is mandatory; BZT52C12 and MMSZ5242B are too high (up to 12.7 V / 12.6 V); BZT52C10 (C2103) fits.
- SMAJ15A: VRWM 15 V < 16 V upper operating limit.
- All 24 V-class CAN TVS (VRWM 24 V): do not stand off a sustained +/-27 V (DR-05). 60.4 Ohm 0603 termination burns at 24 V (Section 0).
- PESD1CAN: 70 V max clamp at 3 A, weak.
- TDK ACT45B-510-2P-TL003 (C76584): stock 10, effectively not orderable; JLC alternates are third-party clones.

Not Basic (each is a JLC Extended fee line): every TVS, every CAN ESD part, every zener, 60.4/62 Ohm resistors, 20 MHz crystal, green LED, all connectors, CMCs, all P-FETs except AO3401A and Si2301CDS.

NOT FOUND list
- Schottky Vf at 100 mA (SS34, B5819W): only graphs, no table value.
- Bourns SRF2012-510Y: no JLC hit, no datasheet; Murata DLW43SH510XK2L and TDK ACT1210 datasheets not retrieved (blocked); CMC pad drawing dimensions (ACT45B p.3 is a graphic).
- TI ESD2CAN24-Q1 on LCSC/JLC.
- PESD2CAN 2023 datasheet revision (product page mentions it), SM24CANB 2022 revision, PESD1CAN Ipp/limiting-values rows (taken from JLC listing).
- Drive level for HC-49S crystals and for the 20 MHz 3225 (series sheet not individually read); CL = 9 pF for the Basic 16 MHz crystal is not in the YXC series sheet (only in JLC attributes).
- C0G 4.7 nF 50 V 0603 (not located).
- 100 nF X7R DC-bias curve; Vf of 2 mA for the LEDs; Basic green 0603 LED.
- KF128/KF301/DG301 terminal block pad dimensions and CUI PJ-002AH polarity/pad numbering.
- Datasheets for Yageo/UNI-ROYAL 60.4 / 62 Ohm resistors (JLC listing only).
- PESD2CANFD27V datasheet (for a 27 V option).
