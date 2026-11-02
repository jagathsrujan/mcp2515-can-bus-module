# Power-stage research extract (Phase 3) - research only, no selection

Author: Sonnet research pass, 2026-11-02. Project: `docs/ENGINEERING_SPEC.md`, `docs/phase2_component_selection.md`.
Input chain studied: 9 V adapter (7-16 V operating, DR-01) -> SS34 -> SMAJ18A (Vc 29.2 V) -> 5 V regulator (100 mA design, 150 mA max) -> transceiver; optional 3.3 V LDO from 5 V.

## How to read this file

- Every number carries a source: document, table/section/figure, **PDF page index** of the file I read (not the printed page unless stated).
- **[DS]** = read from a manufacturer datasheet. **[LIST]** = JLCPCB/LCSC catalogue attribute only (datasheet not read for that value). **[SIM]** = Samsung SEM product-page simulation library. **(CALC)** = my arithmetic from quoted numbers. **(interp.)** = my interpretation. **(from curve)** = read from a graph, accuracy about +/-1 % (efficiency) or +/-0.02 V (Vf).
- **NOT FOUND** = not in the document I could read. Nothing here is from memory.
- JLC data: JLCPCB parts API `selectSmtComponentList`, queried 2026-11-02. `base` = Basic, `expand` = Extended. Stock is a snapshot.
- Scratch copies of datasheets and rendered pages are in the session scratchpad `.../scratchpad/ds/` and `.../scratchpad/img/` (not part of the project).

---

## 1. AP63205WU-7 (Diodes Inc.) deep extract

**Source document:** DS41326 Rev. 3-2, November 2024, "AP63200/AP63201/AP63203/AP63205, 3.8 V to 32 V input, 2 A low-Iq synchronous buck with enhanced EMI reduction". URL: https://www.diodes.com/assets/Datasheets/AP63200-AP63201-AP63203-AP63205.pdf (18 pages; same revision also at https://www.mouser.com/datasheet/2/115/AP63200_AP63201_AP63203_AP63205-1525010.pdf). Page numbers below are PDF pages = printed "n of 18". JLC: C2071056, **Extended**, stock 27 344.

### 1.1 Pinout (TSOT-26, top view) - Pin Descriptions table, p2 and Pin Assignments p1

| Pin | Name | Function (p2) |
|---|---|---|
| 1 | FB | Feedback sensing. For AP63205 tied directly to VOUT (Fig. 1 / Fig. 21, p2, p9); fixed 5 V |
| 2 | EN | Enable. "Attach to VIN or leave open for automatic startup"; precision threshold 1.18 V |
| 3 | VIN | Power input, 3.8-32 V |
| 4 | GND | Power ground |
| 5 | SW | Switch node |
| 6 | BST | High-side gate-drive boost; 100 nF to SW |

Pin 1 FB, 2 EN, 3 VIN on one side; 4 GND, 5 SW, 6 BST on the other (Pin Assignments figure, p1; recommended-layout figure p15 confirms). Package: TSOT26, 2.9 x 1.6 mm body, pitch 0.95 mm, suggested pad C 0.95 / X 0.70 / Y 1.00 / Y1 3.20 mm (Package Outline Dimensions, p17). MSL 1.

### 1.2 Electrical numbers (all VIN = 12 V, 25 C unless noted; Electrical Characteristics table p5; Abs Max / Recommended / Thermal tables p4)

| Parameter | Value | Where |
|---|---|---|
| VIN recommended range | 3.8-32 V; TA -40...+85 C | Recommended Operating Conditions p4 |
| VIN abs max | -0.3...+35.0 V DC; **-0.3...+40.0 V for 400 ms** | Abs Max p4 |
| VEN abs max | -0.3...+35.0 V | Abs Max p4 |
| VSW | -1.0...VIN+0.3 V | Abs Max p4 |
| VBST | VSW-0.3...VSW+6.0 V | Abs Max p4 |
| UVLO rising | 3.30 / **3.50** / 3.70 V (min/typ/max); hysteresis 440 mV typ | p5 |
| UVLO falling | text says "falls below 3.1 V ... disabled" (Undervoltage Lockout section, p11); 3.50-0.44 = 3.06 V (CALC) | p11, p5 |
| EN logic high (rising) | 1.15 / 1.18 / 1.23 V | p5 |
| EN logic low (falling) | 1.05 / 1.10 / 1.15 V | p5 |
| EN input current | 5.5 uA at VEN = 1.5 V; 1.5 uA at 1 V | p5 |
| **EN tied to VIN?** | **Yes.** Pin table (p2): "Attach to VIN or leave open". Enable section (p11): "EN pin is a high voltage pin and can be directly connected to VIN". EN abs max 35 V equals VIN DC abs max, so with the 29.2 V TVS clamp it is within rating. 1.5 uA internal pull-up and 4 uA hysteresis source exist (p10, Fig. 22) | p2, p4, p10-11 |
| Output accuracy AP63205 | VFB (= VOUT) 4.95 / **5.00** / 5.05 V in CCM, i.e. +/-1 % | p5 |
| fsw (AP63205) | **1100 kHz** typ; **frequency spread spectrum +/-6 %** (about 1.03-1.17 MHz (CALC)) | p5; p1 |
| Minimum on-time | 80 ns typ | p5 |
| Maximum duty / dropout | **NOT FOUND** (no max-duty spec, no dropout curve). Text p14 (Bootstrap Capacitor): when VIN approaches VOUT it "enters low dropout mode by holding Q1 on for multiple clock cycles ... Q2 is forced to refresh ... effective duty cycle approximately 100 %". Dropout estimate (CALC): 0.15 A x (125 mOhm HS + inductor DCR 0.1 Ohm) = 34 mV, so VIN > about 5.1 V keeps 5 V (interp.); 7 V minimum input is well above | p14 |
| Min-on-time check at VIN max | 5 V/32 V = 15.6 % duty; 0.156/1.1 MHz = 142 ns > 80 ns (CALC); at the 29.2 V clamp 156 ns | CALC |
| Shutdown current | 1 / 3 uA (typ/max), VEN = 0 | p5 |
| **Quiescent current** | **22 uA typ** (VEN open, VFB = 1.0 V, non-switching) for AP63200/203/205; AP63201 is 258 uA. No max given | p5 |
| RDS(on) | HS 125 mOhm typ, LS 68 mOhm typ | p5 |
| Peak current limit (HS) | 2.5 / 2.8 / 3.1 A | p5 |
| Valley current limit (LS) | 2.5 / 3.2 / 3.9 A | p5 |
| Soft-start | **4 ms** typ (no min/max) | p5; p9-10 |
| Hiccup | text p10: "wait time 512 cycles, restart 8192 cycles"; text p11 (Overcurrent): hits limit 2 ms then shuts down, restarts after 16 ms. The two statements do not agree at 1.1 MHz (0.47 ms / 7.4 ms (CALC) vs 2 ms / 16 ms) - datasheet inconsistency | p10, p11 |
| Thermal shutdown | table p5: +160 C typ, hysteresis +25 C. Text p11 (Thermal Shutdown): +150 C, recovery +130 C nominal - inconsistent. Abs-max TJ 160 C (p4) but text p12 says TJ "should not exceed +125 C" | p4, p5, p11, p12 |
| **theta-JA / theta-JC** | **89 C/W** / 39 C/W, TSOT26, FR-4 single-layer 2 oz, minimum recommended pad | Thermal Resistance p4 |
| Derating curve | Fig. 24 p12, 12 V in (not read in detail; shows output current vs ambient) | p12 |

### 1.3 Light-load behaviour and efficiency

- Mode (PFM section p10): AP63200/203/205 run forced-PWM at heavy load; COMP is clamped at a level that corresponds to **450 mA peak inductor current**; as load goes to zero they enter PFM, low-side switch is forced off on zero-cross (DCM), "does not sink current from the output" at light load. AP63201 is the always-PWM variant.
- Claims p1: "up to 80 % efficiency at 1 mA, up to 88 % at 5 mA".
- Efficiency, **Figure 2 p2** (curve labelled VIN = 12 V/VOUT = 5 V; Fig. 2 is on the AP63205 typical-application page; Figs. 4-5 p6 are headed "AP63200", 500 kHz, so I used Fig. 2). **(from curve, Fig. 2, p2)**:

| IOUT | VIN = 12 V, VOUT = 5 V | VIN = 24 V, VOUT = 5 V |
|---|---|---|
| 5 mA | 88.9 % | 79.9 % |
| 10 mA | 90.1 % | 81.9 % |
| 20 mA | 90.6 % | 82.9 % |
| 50 mA | 91.2 % | 83.6 % |
| 100 mA | 91.8 % | 84.4 % |
| 150 mA | 92.3 % | 85.4 % |

- **VIN = 9 V curve: NOT FOUND** (datasheet plots only 12 V and 24 V). Interpolating between the 12 V and 24 V curves gives no useful 9 V number; 9 V should be at or slightly above the 12 V values (interp., unverified).
- Loss at 100 mA, 12 V (CALC from 91.8 %): 0.5 W x (1/0.918 - 1) = 45 mW; temperature rise = 45 mW x 89 C/W = 4 K. Compared with an LDO at 12 V/100 mA (0.70 W).
- Load/line regulation (Figs. 6-7 p6, AP63200 plots): about 5.15-5.17 V at light load, 12 V in (from curve, indicative only).

### 1.4 Datasheet component recommendations and equations

| Item | Datasheet text | Where |
|---|---|---|
| Typical circuit AP63205 | L 4.7 uH, C1 (input) 10 uF, C2 (output) **2 x 22 uF**, C3 (BST) 100 nF; EN to VIN; FB to VOUT | Fig. 1 p2; Table 3 p13 |
| Inductor value | "For most designs" **L = VOUT x (VIN - VOUT) / (VIN x dIL x fSW)** (Eq. 7); choose dIL = **30-50 % of 2 A**; ILPEAK = ILOAD + dIL/2 (Eq. 8) | p13 |
| Inductor selection | "approximately 2.2 uH to 10 uH with a DC current rating of at least 35 % higher than the maximum load current"; DCR "less than 100 mOhm" for highest efficiency; "larger inductance for improved efficiency under light load" | p13 |
| Input capacitor | Cin RMS rating > half of max load current; "electrolytic, or ceramics with low ESR"; tantalum must be surge-rated; "ceramic > 10 uF is sufficient for most applications" | p13 (section 11) |
| Output capacitor | **VOUT_ripple = dIL x ESR (Eq. 9)**; "22 uF to 68 uF ceramic is sufficient"; transient: **COUT > L x (IOUT + dIL/2)^2 / ((dV + VOUT)^2 - VOUT^2) (Eq. 10)** with dV = max overshoot | p14 (section 12) |
| Bootstrap | ceramic 100 nF between BST and SW; boot UV protection: if CBST < 2.3 V, Q2 turns on 220 ns to refresh above 2.85 V | p14 (section 13) |
| Programmable UVLO | R5 = (0.932 x VON - VOFF)/4.1 uA (Eq. 1); R6 = 1.1 x R5/(VOFF - 1.1 V + 5.5 uA x R5) (Eq. 2); optional EN-to-GND capacitor Cd[nF] = 1.27 x ts[ms] (Eq. 3) | p10-11 |

Ripple numbers for this project, fSW = 1.1 MHz nominal, Eq. 7 rearranged dIL = 5 x (VIN-5)/(VIN x L x 1.1 MHz) **(CALC)**:

| VIN | 4.7 uH | 6.8 uH | 10 uH | ILPEAK at 150 mA, 4.7 uH |
|---|---|---|---|---|
| 7 V | 0.276 A | 0.191 A | 0.130 A | 0.29 A |
| 9 V | 0.430 A | 0.297 A | 0.202 A | 0.37 A |
| 12 V | 0.564 A | 0.390 A | 0.265 A | 0.43 A |
| 16 V | 0.665 A | 0.459 A | 0.312 A | 0.48 A |
| 29.2 V (TVS clamp) | 0.800 A | 0.553 A | 0.376 A | 0.55 A |

Notes (CALC / interp.): (a) the datasheet's 30-50 % ripple rule refers to 2 A max load (0.6-1 A); at our 0.1-0.15 A load every option above is in DCM at light load, so the PFM peak clamp (450 mA, p10) is the relevant stress: choose Isat comfortably above 0.5 A, i.e. 1 A or more. (b) The "35 % above max load" rule gives only 0.2 A for 150 mA, so any >= 1 A part satisfies it; but the output short-circuit peak limit is up to 3.1 A, so a 1-2.5 A Isat part will saturate during a short (the hiccup restart, p11, limits average stress). (c) Output ripple Eq. 9 uses ESR only; adding the capacitive term dIL/(8 fSW Ceff) with Ceff = 2 x 22 uF 10 V X5R at 5 V (Section 7: -71 %, 12.8 uF total) gives 0.43 A/(8 x 1.1 MHz x 12.8 uF) = 3.8 mV at 9 V in.

### 1.5 PCB layout guidelines (Layout section p15, "PCB Layout" bullets 1-6; Fig. 25 recommended layout)

1. The part is specified for 2 A, "heat dissipation is a major concern"; **2 oz copper on both top and bottom** recommended. (We run 0.1-0.15 A, so this is largely moot; interp.)
2. Provide sufficient vias on the **GND side of the input and output capacitors** to conduct heat to the bottom layer.
3. Make the bottom layer under the device a **GND layer**, as large as possible, for heat dissipation.
4. Place the **VIN capacitors as close to the device as possible.**
5. Place the **feedback components as close to FB as possible** (for AP63205 FB connects straight to VOUT; Fig. 25 shows a red feedback trace running from the VOUT pour to pin 1, away from the SW pour).
6. Follow Fig. 25: VIN pour on the left carrying C1, central GND pour with vias between C1 and C2, SW pour (green) on the right side of the IC to the inductor, VOUT pour on the right with C2.

No guidance given on SW-node area, BST routing or loop area beyond these bullets (NOT FOUND).

### 1.6 AP63200 / AP63201 / AP63203 / AP63205 family (Ordering Information p16, Table 1-3 p13)

| Part | JLC (Ext, stock) | Mode | fsw | VOUT | Iq | Recommended L (Tables 1-3, p13) |
|---|---|---|---|---|---|---|
| AP63200WU-7 | C2071868, 5 947 | PWM/PFM | 500 kHz | adjustable (0.8 V ref) | 22 uA | 2.2-10 uH by VOUT (5 V: 10 uH, R1 157k/R2 30k) |
| AP63201WU-7 | C2071044, 2 624 | **PWM only** | 500 kHz | adjustable | **258 uA** | same table |
| AP63203WU-7 | C780769, 1 398 | PWM/PFM | 1.1 MHz | fixed 3.3 V | 22 uA | 3.9 uH |
| **AP63205WU-7** | C2071056, 27 344 | PWM/PFM | 1.1 MHz | **fixed 5 V** | 22 uA | **4.7 uH** |

All four: 3.8-32 V in, 2 A, 4 ms soft-start, +/-6 % FSS, TSOT26, same pinout. An automotive AP63205Q variant has a separate datasheet (p1). For the 5 V rail only AP63205 (fixed) or AP63200 (adjustable, 500 kHz, needs a divider and 10 uH) are relevant; AP63200 at 500 kHz would need a larger L for the same ripple.

---

## 2. Inductor candidates (4.7 uH primary; 6.8 and 10 uH alternatives)

**No Basic power inductor was found.** I paged ~2 100 JLC results for "4.7uH", "6.8uH", "10uH", "Power Inductors" etc.; the only Basic (`base`) inductors are multilayer chip parts (C1035 SDFL1608S100KTF 10 uH 0603, 3 mA class; C1046 SDFL2012S100KTF 10 uH 0805, 15 mA) - unusable at this current. All power inductors below are **Extended**.

Isat definition (Sunlord catalogue notes *3/*4, p16): Isat = DC current at which L drops about 30 % from its zero-current value; Irms = DC current causing a 40 K temperature rise from 20 C ambient; "Rated current: Isat or Irms, whichever is smaller". JLC lists the Max-column values. Construction (Sunlord p1): "magnetic-resin shielded", closed magnetic circuit, wire-wound.

| MPN | LCSC | Class | L | Isat (max / typ) | Irms (max / typ) | DCR max / typ | Size (L x W x H max) | Shielded | Stock | Source |
|---|---|---|---|---|---|---|---|---|---|---|
| **Sunlord SWPA4020S4R7MT** | **C48427** | Ext | 4.7 uH +/-20 % | 2.35 / 2.50 A | 1.34 / 2.00 A | 98 / 75 mOhm | 4.0 x 4.0 x 2.0 mm | yes (magnetic-resin) | **30 140** | Catalogue p7 table "SWPA4020S Series"; dims p1-2 |
| Sunlord SWPA3015S4R7MT | C19398 | Ext | 4.7 uH | 1.10 / 1.40 A | 1.09 / 1.30 A | 163 / 125 mOhm | 3.0 x 3.0 x 1.5 mm | yes | 25 332 | Catalogue p5 table "SWPA3015S Series" |
| Sunlord SWPA4020S6R8MT | C83425 | Ext | 6.8 uH | 2.20 / 2.40 A | 1.04 / 1.60 A | 163 / 125 mOhm | 4.0 x 4.0 x 2.0 | yes | 4 838 | p7 |
| Sunlord SWPA4020S100MT | C82398 | Ext | 10 uH | 1.60 / 1.70 A | 0.90 / 1.20 A | 215 / 165 mOhm | 4.0 x 4.0 x 2.0 | yes | 16 318 | p7 |
| Sunlord MWSA0402S-4R7MT | C408337 | Ext | 4.7 uH | 3.0 A [LIST] | 2.8 A [LIST] | 105 mOhm [LIST] | 4.4 x 4.2 mm (height not read) | molded (series name; datasheet table not read) | **51 749** | JLC attributes only |
| Cenker CKCS4020-4.7uH/M | C354581 | Ext | 4.7 uH | 2.35 A [LIST] | 1.34 A [LIST] | 75 mOhm [LIST] | 4 x 4 x 2 | "Magnetic Shielded" [LIST] | 1 789 | JLC attributes |
| Cenker CKCSA4020-4.7uH/M | C354583 | Ext | 4.7 uH | 4.0 A [LIST] | 2.85 A [LIST] | 110 mOhm [LIST] | 4 x 4 | shielded [LIST] | 15 566 | JLC attributes |
| Cenker CKCS3015-4.7uH/M | C354565 | Ext | 4.7 uH | 1.1 A [LIST] | 1.09 A [LIST] | 125 mOhm [LIST] | 3 x 3 x 1.5 | shielded [LIST] | 57 743 | JLC attributes |
| Taiyo Yuden NRS4018T4R7MDGJ | C92961 | Ext | 4.7 uH | 2.0 A [LIST] | 1.7 A [LIST] | 84 mOhm [LIST] | 4 x 4 x 1.8 | shielded (NRS series; [LIST] says ferrite shielded) | 16 676 | JLC attributes |
| TDK SPM4020T-4R7M-LR | C694586 | Ext | 4.7 uH | 3.5 A [LIST] | 2.5 A [LIST] | 147 mOhm [LIST] | 4.4 x 4.1 | molded/shielded (series) | 776 | JLC attributes |

Not usable: Cenker CKCS3012 (C354557, 1.0 A, 2 171 stock; 3 x 3 x 1.2), TDK SPM4020T-4R7M (C6680489, stock 0), Taiyo Yuden NR4018T4R7M (C92936, stock 0).

(CALC) checks against the AP63205 numbers: SWPA4020S4R7MT Isat 2.35 A is below the 2.5-3.1 A peak limit, but far above the 0.55 A worst ripple peak at 150 mA (1.4 / 29.2 V) and above the 0.45 A PFM peak clamp; DCR 98 mOhm max just meets the datasheet's "< 100 mOhm" advice (p13). SWPA3015S4R7MT (Isat 1.10 A, DCR 163 mOhm) also clears the 0.55 A need with 2x margin but misses the "< 100 mOhm" advice (conduction loss at 0.1 A is 1.6 mW, negligible). 6.8 uH and 10 uH rows give lower ripple (Section 1.4) at the cost of higher DCR (163-215 mOhm) and lower Isat.

---

## 3. 5 V LDO alternatives (>= 30 V abs max, >= 150 mA, dropout <= 1 V at 100 mA, ceramic-stable, JLC stock > 1 000)

**No Basic LDO of this class exists** - every part below is Extended. Dissipation (CALC): P = (VIN - 5 V) x 0.1 A: 7 V 0.20 W; 9 V **0.40 W**; 12 V 0.70 W; 16 V **1.10 W**; at 150 mA/16 V 1.65 W. Temperature rise = P x theta-JA.

### 3.1 Summary table

| Part (MPN) | LCSC, class, stock | VIN abs / rec | Iout | Dropout at 100 mA (typ / max) | Iq | Cout requirement | theta-JA | Verdict |
|---|---|---|---|---|---|---|---|---|
| **TPS7B6950-Q1** (TI), SOT-223 DCY | C108469 (QDCYRQ1) Ext, 4 396 | 45 V (200 ms) / **40 V** | 150 mA | **450 / 800 mV** | 15 uA typ, 25 uA max | **2.2-100 uF effective, ESR 1 mOhm-2 Ohm** (ceramic OK) | **64.2** (SOT-223, JEDEC 2s2p), 210.4 (SOT-23-5 DBV) | **Viable in SOT-223**; DBV fails thermally |
| TPS7B6950 SOT-23-5 (DBV) | C174327 (DBVR) Ext, 6 181; C132202 (QDBVRQ1) 2 005 | same | 150 mA | 450 / 800 mV | 15/25 uA | same | 210.4 | 0.4 W -> 84 K rise (9 V); 1.1 W -> 231 K (16 V) **fails** |
| **TPS7B8150** (TI), HVSSOP-8 DGN / WSON DRV | C2864577 (DGN) Ext, **495 (<1 000)**; C2876221 (Q1 DRV) 2 964; C2873388 (DRV) 0 | 45 V (200 ms) / **40 V** | 150 mA | **180 / 350 mV** (DGN); 200 / 390 mV (DRV) | **1.9 uA typ**, 3.5 uA max (0 mA) | **1-200 uF, ESR 1 mOhm-5 Ohm** | **63.9** (DGN), 72.8 (DRV) | Viable electrically; JLC stock of the DGN part < 1 000; DRV is a 2 x 2 mm WSON (hard to hand-rework) |
| TPS7B8250-Q1 (300 mA version) | C424924 (QDGNRQ1) 3 551; C2862522 (QDRVRQ1) 1 311; C2860622 (TO-252) 51 | 45 V / 40 V | 300 mA | 700 mV max at 200 mA (100 mA spec not read) | 2.7 uA typ (light load), 5 uA max | 1-200 uF, ESR 1 mOhm-5 Ohm | 63.9 (DGN), 72.8 (DRV), **31.1 (TO-252 KVU)** | Viable; automotive-grade only (Q1) at JLC |
| **TPS70950** (TI TPS709, 5 V) | C96028 (DBVR) Ext, **6 203**; C485909 (QDBVRQ1) 11 334; C527527 (DRV) 1 985 | **32 V** / **30 V** | 150 mA | **~460 / ~850 mV (CALC)**: datasheet gives 245/500 mV at 50 mA and 690/1 200 mV at 150 mA only | 1.4 uA typ, 2.25 uA max (no load); 350 uA at 150 mA | **1.5 uF effective min, 47 uF max, ESR 0-0.2 Ohm** (ceramic) | **212.1** (DBV), **73.1** (DRV) | Marginal: 30 V rec max with 29.2 V clamp (0.8 V margin), 1.2 V max dropout at 150 mA > 1 V; DBV fails thermally; **EN must not be tied to VIN (abs max 7 V)** |
| **NCV8664C 5.0 V** (onsemi), SOT-223 / DPAK | C556405 (SOT-223 ST) Ext, 3 856; C604073 (DPAK) 28; C904690 (DPAK) 8 | 45 V DC (rated -42 V reverse) / 4.5-**45 V** | 150 mA | **230 / 500 mV** | 21 uA typ, 29 uA max (100 uA load) | **10 uF** (Fig. 2/3 p4; ESR stability region Fig. 4 p5, not read); ceramic permitted | 109 (SOT-223, 100 mm2 1 oz), 87.4 (DPAK) | Viable; 1.1 W at 16 V -> 120 K rise (SOT-223) means TJ 145 C at 25 C ambient - marginal at 16 V |
| NCV4275A 5.0 V (onsemi), DPAK-5 | C709694 (ADT50) Ext, 3 666; C513032 (CDT) 681 | 45 V / 6-40 V (VOUT spec) | **450 mA** | not given at 100 mA; 250/500 mV at 300 mA | **140 uA typ**, 200 uA max at 1 mA | 22 uF, ESR < 4.5 Ohm | not read (DPAK package) | Viable, oversized; reset output and 140 uA Iq; a 5-pin DPAK |
| LM2931AD-5.0 (onsemi) SOIC-8 | C134292 (R2G) Ext, 2 584; C135506 (G) 99 | **40 V DC**, 60 V (100 ms) / regulation guaranteed 6-26 V; output shuts down above **26-40 V (29.5 typ)** (Vth(OV)) | **100 mA** | 160 / **600 mV** | 0.4 mA typ at 10 mA (1.0 max); 5.8 mA typ at 100 mA | **>= 10 uF (22 uF optimum), stable region ESR 0-0.4 Ohm** (Fig. 23 p10); ceramic OK | **160** (SOIC-8, min pad; falls to ~40 with copper, Fig. 24 p10), 92 (DPAK) | **Fails**: only 100 mA, 29.5 V typ overvoltage shutdown right at the 29.2 V clamp |
| HT7550-1 (Holtek) SOT-89 | C16106 Ext, **189 980** (also UMW C347189 377 072) | **33 V abs**, 30 V in | **100 mA min** (150 mA typ) at VIN = 7 V | **NOT FOUND** at 100 mA (only 25/55 mV at 1 mA) | 2.5 uA typ, 4.0 max | **NOT FOUND** in the text I read (figures only) | **200** (SOT-89), 500 (SOT-23-5); PD 0.5 W / 0.2 W | **Fails**: 16 V x 0.1 A = 1.1 W > 0.5 W PD limit; no thermal shutdown listed; dropout at load unspecified |
| LP2950-5.0 / LP2951 (TI) TO-92 / SOIC-8 | C129349 LP2950-50LPRE3 (TO-92) 2 404; C414606 LP2951DR 1 401 | **35 V (legacy) / 42 V (new chip)** abs; 30 V rec | **100 mA** | 380/450 mV (legacy, 25 C); 340/420 mV (new chip); 600/570 mV over temp | 50 uA typ (new chip) | 1-100 uF, ESR 0-2 Ohm (new chip); legacy 2.2 uF, ESR 30 mOhm-5 Ohm | 140/132.6 (TO-92), 97/123 (SOIC-8) | **Fails**: 100 mA only; same thermal issue |
| L78L05 (ST), SOIC-8 / SOT-89 | C7543 (SOIC-8) 294 970; C42738 190 304; C81154 208 574 | **30 V** (for 5 V option) | 100 mA | **1.7 V typ at 40 mA** (no max given at 100 mA) | 6 mA class | 0.1 uF | not read | **Fails**: dropout 1.7 V > 1 V, 100 mA, mA-level quiescent |
| LM2936 (TI) 5.0 V | C3746550 (M-5.0) stock 0 | 40 V | **50 mA** | 400 mV at 50 mA [LIST] | 9 uA | - | - | **Fails**: 50 mA, stock 0 |
| Others in JLC search | LR8342A-T50 (C29781218, SOT-23-3L, 40 V, 250 mA, 1.2 V at 250 mA [LIST], stock 1 242) not read in a datasheet; TPS7A4901 (C16430, adjustable, 35 V abs/33 V out, 150 mA, 260 mV at 100 mA [LIST], stock 6 764) needs divider, unread; NCV8675 (C463375, D2PAK-5, 45 V, 350 mA, stock 41) too little stock | | | | | | | not evaluated |

### 3.2 Source detail for the rows I read

- **TPS7B69xx-Q1:** TI SLVSCJ8B (Nov 2014, rev Jan 2015), https://www.ti.com/lit/ds/symlink/tps7b69-q1.pdf. Abs Max p4 (IN -0.3...45 V, "withstands 45 V for 200 ms"); Recommended Operating p4 (VI 4-40 V; CO 2.2-100 uF effective; ESR 0.001-2 Ohm); Thermal Information p4 (DCY 64.2, DBV 210.4 C/W); Electrical Characteristics p5 (5 V version: VI 5.5-40 V, IQ 15 uA typ/25 uA max, VDROP 220/400 mV at 50 mA, **450/800 mV at 100 mA**, IOCL 150-500 mA, UVLO up 3.65 V/down 3.0 V, Tsd 175 C); output capacitor text p13 section 9.2.2.2. The JLC datasheet listed for C174327 (TPS7B6950DBVR, TA -40...105 C) is the same family; I did not read a separate non-Q1 revision.
- **TPS7B81 / TPS7B82-Q1:** TI SBVS402 (Apr 2020) https://www.ti.com/lit/ds/symlink/tps7b81.pdf and TI SLVSDQ0K (Sep 2025) https://www.ti.com/lit/ds/symlink/tps7b82-q1.pdf. Abs Max p4 (VIN -0.3...45 V, 45 V for 200 ms; VEN up to VIN); Recommended p4 (VIN 3-40 V, COUT 1-200 uF, ESR 1 mOhm-5 Ohm); Thermal p4/p5; Electrical p5 (TPS7B81 5 V: dropout at 100 mA 180/350 mV DGN, 200/390 mV DRV; at 150 mA 270/540 and 325/585 mV; IQ 1.9 uA typ (3.5 max DRV/5 max DGN for TPS7B82) at 0 mA; UVLO 2.7 V; current limit 180-510-690 mA; Tsd 175 C; TPS7B82-Q1 max dropout 700 mV at 200 mA for 5 V). Note: JLC parts for TPS7B8150 are DGN (non-Q1, C2864577) and Q1 variants; the Q1 listing is automotive-qualified, otherwise equivalent (interp.).
- **TPS709:** TI SBVS186H (Jul 2021) https://www.ti.com/lit/ds/symlink/tps709.pdf. Abs Max p4 (VIN -0.3...32 V; VEN, VOUT up to 7 V); Recommended p4 (VIN 2.7-30 V); Thermal p4 (DBV 212.1, DRV 73.1); Electrical p5 (TPS70950 dropout: 50 mA 245/500 mV, 150 mA 690/1 200 mV; IGND 1.4/2.25 uA at 0 mA, 350 uA at 150 mA; current limit 200/320/500 mA; Tsd 158 C); EN text p13 section 7.4 ("Do not connect EN to VIN ... clamped by a 6.5-V Zener ... do not exceed 7 V"); capacitor text p14 section 8.1.1 (effective COUT 1.5 uF min for VOUT >= 1.5 V, 47 uF max, ESR 0-0.2 Ohm; input cap 0.1-2.2 uF, "necessary if line transients greater than 10 V in magnitude are anticipated").
- **NCV8664C:** onsemi NCV8664C/D Rev. 3 Nov 2018, LCSC mirror https://www.lcsc.com/datasheet/lcsc_datasheet_2005282035_onsemi-NCV8664CST50T3G_C556405.pdf. Features p1; Operating range and Max Ratings p2 (VIN -42...+45 V); Thermal p3; Electrical p3 (5 V: dropout 230 typ/500 max at 100 mA, 270/600 at 150 mA; Iq 21/29 uA at 100 uA; ground current 0.5 mA typ at 50 mA, 3.1 mA at 150 mA; current limit 150-500 mA; TSD 150-200 C); Cout 10 uF for 5 V version in Fig. 2/3 p4.
- **LM2931A:** onsemi LM2931/D Rev. 29 Oct 2021, LCSC mirror https://www.lcsc.com/datasheet/lcsc_datasheet_2210210230_onsemi-LM2931AD-5-0R2G_C134292.pdf. Max Ratings p3 (VI 40 V cont., 60 V peak <= 100 ms; theta-JA SOP-8 160); Electrical p4 (dropout 100 mA 0.16 typ/0.6 max; bias 5.8 mA at 100 mA; Vth(OV) 26/29.5/40 V); Application p8 and Fig. 22/23 p10 (Cout >= 10 uF, 22 uF optimum, |ZO| <= 0.4 Ohm; stable region extends to ESR 0 above 10 uF).
- **HT7550-1:** Holtek HT75xx-1 Rev. 2.40 Aug 2018, LCSC mirror https://www.lcsc.com/datasheet/lcsc_datasheet_2404031722_Holtek-Semicon-HT7550-1_C16106.pdf (official link in JLC data: https://www.holtek.com/documents/10179/116711/HT75xx-1v260.pdf, not fetched). Abs Max "-0.3 V to 33 V", theta-JA / PD table p2; HT7550-1 electrical table p7 (IOUT 100 mA min/150 mA typ at VIN = 7 V; VDIF 25 typ/55 max mV at 1 mA; ISS 2.5/4.0 uA).
- **LP2950/LP2951:** TI SLVS582L (Jun 2026) https://www.ti.com/lit/ds/symlink/lp2950.pdf. p1 features; Abs Max p4; Recommended p5 (VIN 2-30 V, IL 0-100 mA, COUT 1-100 uF, ESR 0-2 Ohm new chip, 30 mOhm-5 Ohm legacy); Thermal p5; Electrical p6 (5 V accuracy, dropout at 100 mA).
- **NCV4275A:** onsemi NCV4275A/D Rev. 4 Oct 2014 (text from the JLC datasheet attached to C709694, no public URL given by JLC). Max Ratings p2 (VI -42...45 V); Electrical p4; output cap spec in Pin Function table p2 ("22 uF, ESR < 4.5 Ohm"). theta-JA not read.
- **L78L05:** ST DocID2145 Rev. 22, LCSC mirror https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_STMicroelectronics-L78L05ACD13TR_C7543.pdf. Abs Max (VI 30 V for VO 3.3-9 V, IO 100 mA) p3-4; "Vd Dropout voltage 1.7 V" at IO = 40 mA, Table 3 electrical characteristics (L78L05AC/AB) p6.

### 3.3 Thermal check for the viable parts (CALC)

| Part / package | 9 V in, 0.1 A (0.40 W) | 16 V in, 0.1 A (1.10 W) | 16 V, 0.15 A (1.65 W) |
|---|---|---|---|
| TPS7B6950 SOT-223 (64.2 C/W) | +26 K | +71 K (TJ 96 C at 25 C amb; **156 C at 85 C amb > 150 C**) | +106 K |
| TPS7B8150 DGN (63.9) | +26 K | +70 K | +105 K |
| TPS7B8150 DRV (72.8) | +29 K | +80 K | +120 K |
| TPS7B8250 KVU TO-252 (31.1) | +12 K | +34 K | +51 K |
| NCV8664C SOT-223 (109) / DPAK (87.4) | +44 K / +35 K | +120 K / +96 K | +180 K / +144 K |
| TPS7B6950 DBV (210.4) / TPS70950 DBV (212) | **+84 K** | **+231 K (fails)** | fails |
| LM2931A SOIC-8 (160 min pad) | +64 K | +176 K (fails) | fails |
| AP63205 buck, for comparison (89 C/W, 12 V, 100 mA: 45 mW) | +4 K | about +5 K | - |

JEDEC theta-JA values are for 2s2p boards with 2 oz copper (TI notes); real 2-layer boards with small pads run higher (interp.).

---

## 4. XC6206P332MR (Torex) - LCSC C5446

Source: Torex ETR0305_004b "XC6206 Series", 17 pages, read from the LCSC mirror https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_Torex-Semicon-XC6206P332MR-G_C5446.pdf (torexsemi.com returned "Access Denied" for www.torexsemi.com/file/xc6206/XC6206.pdf; the JLC data field for the official link is empty). Part on JLC: C5446 "XC6206P332MR-G", SOT-23-3L, **Basic**, stock **520 752**, 0.148 USD at 1-49 pcs.

| Parameter | Value | Where |
|---|---|---|
| VIN abs max | **7.0 V** | Absolute Maximum Ratings p3 |
| VIN operating | **1.8-6.0 V** ("Maximum operating voltage 6.0 V") | Electrical Characteristics p4; features p1 |
| Output | 3.3 V, 2 % grade: 3.234-3.366 V at IOUT = 30 mA, VIN = VOUT + 1 V | Electrical chart E-1 p5 |
| Max output current | >= **200 mA** (3.3 V row) | E-2 p5 |
| Dropout at 30 mA (Vdif1) | 75 mV typ / 350 mV max | E-4 p5 |
| Dropout at 100 mA (Vdif2) | 250 mV typ / 680 mV max | E-5 p5 |
| **Dropout at 50 mA** | **NOT in table.** Linear interpolation (CALC, interp.): about 125 mV typ / 445 mV max | CALC |
| Supply current IDD | **1.0 uA typ / 3.0 uA max** (VCE = VIN) | p4 |
| Load regulation | 60 mV max (1-100 mA) | E-3 p5 |
| Short-circuit current | 100 mA typ (foldback limiter) | E-6 p5; description p1 |
| Cin / Cout | Typical application circuit: **CIN 1.0 uF ceramic, CL 1.0 uF ceramic**; "compatible with low ESR ceramic capacitors" | p1 |
| Package power | SOT-23 **250 mW**; SOT-89 500 mW | p3 |
| Pinout SOT-23 | **pin 1 = VSS (GND), pin 2 = VOUT, pin 3 = VIN** (differs from many LDOs) | Pin Number table p2 |
| Temperature | -40...+85 C | p3 |

(CALC) from a 5.0 V rail at 50 mA: headroom 1.7 V, P = 1.7 V x 0.05 A = 85 mW < 250 mW. Meets the 5.5 V transient need? 5 V +/- 3 % = 5.15 V max, under the 6.0 V operating limit and 7.0 V abs max. Note: TLV75533PDBVR (Phase 2 choice) was not re-read here.

---

## 5. Schottky forward voltage (reverse-protection diode) - read from curves

Both datasheets give **no Vf table value below 1 A**; only the forward-characteristic graph, whose lowest plotted current is **0.1 A**. The 50 mA values below are therefore extrapolated off the plot.

| Part | Vf at 50 mA | Vf at 100 mA | Vf at 200 mA | Source |
|---|---|---|---|---|
| **SS34** (MDD, C8678, Basic, stock 3 244 970; SMA, 40 V, 3 A) | **NOT plotted** (axis starts at 0.1 A); log-linear extrapolation of the solid curve (0.10 A -> 0.34 V, 1.0 A -> 0.41 V, slope about 0.07 V/decade) gives about 0.31 V (extrapolation, +/-0.03 V) | **about 0.34 V** (from curve, Fig. 3, solid line "SS32-SS35" which includes SS34) | **about 0.36 V** (from curve, Fig. 3) | MDD SS32-SS3200 Rev 2024A5, "Fig. 3 Typical Forward Characteristic", p2, https://www.lcsc.com/datasheet/lcsc_datasheet_2407101107_MDD-Microdiode-Semiconductor-SS34_C8678.pdf. Curve test temperature not stated on the figure (25 C assumed (interp.)). The page note says "The curve above is for reference only." Table value: 0.55 V at 3.0 A (p1 per `discrete_datasheet_extract.md`). 1 A from the same curve: about 0.41 V |
| **B5819W** (Changjiang, C8598, Basic, SOD-123, stock 960 792; 1 A, 40 V) | **NOT plotted** (axis starts at 0.1 A); extrapolation about 0.27-0.29 V | **about 0.30 V** (from curve, "Forward Characteristics", Ta = 25 C curve) | **about 0.35 V** (from curve) | CJ B5817W-B5819W, D Mar 2015, "Typical Characteristics" p2, https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_Jiangsu-Changjing-Electronics-Technology-Co---Ltd--B5819W-SL_C8598.pdf. Table: 0.6 V max at 1 A (p1). Curve shows Ta = 25 C and 100 C (100 C curve about 0.07 V lower at 0.1 A) |

(CALC) Dissipation at 100 mA: SS34 0.34 V x 0.1 A = 34 mW; at the 126 mA worst buck input current (7 V, 150 mA load, 85 % efficiency, interp.) about 0.35 V x 0.126 = 44 mW. The diode drop at 7 V min input leaves about 6.65 V, comfortably above the AP63205 UVLO (3.5 V) and the 5.1 V dropout floor (Section 1.2).

---

## 6. Input filter parts: ferrite beads and bulk capacitor

### 6.1 Ferrite beads, 600 Ohm @ 100 MHz (JLC attributes only [LIST]; datasheets not read; impedance-vs-DC-bias NOT FOUND)

Required rating: input current into the buck at the worst case = 5 V x 0.15 A / (0.85 x 7 V) = **0.126 A** (CALC, 85 % efficiency interp.); at 9 V, 100 mA about 0.065 A.

| MPN | LCSC | Class | Size | Rated current | DCR | Stock | Note |
|---|---|---|---|---|---|---|---|
| **Sunlord GZ2012D601TF** | **C1017** | **Basic** | 0805 | **500 mA** | 300 mOhm | 491 477 | meets >= 500 mA, Basic |
| Sunlord GZ1608D601TF | C1002 | **Basic** | 0603 | **200 mA** | 450 mOhm | 581 182 | Basic; only 200 mA but 1.6 x the 126 mA worst case |
| Murata BLM18AG601SN1D | C19330 | Ext | 0603 | 500 mA | 380 mOhm | 514 818 | |
| Murata BLM18EG601SN1D | C20345 | Ext | 0603 | 500 mA | 350 mOhm | 42 997 | |
| TDK MPZ1608S601ATA00 | C107335 | Ext | 0603 | 1 A | 150 mOhm | 681 581 | |
| Murata BLM21PG601SN1D | C41556732 | Ext | 0805 | 1.4 A | 140 mOhm | 2 830 | |
| Murata BLM21AG601SN1D | C85837 | Ext | 0805 | 700 mA | 210 mOhm | 61 949 | |
| TDK MPZ2012S601ATD25 | C81034 | Ext | 0805 | 2 A | 100 mOhm | 197 828 | also C21519 (AT000) 106 723 |
| FH CBW201209U601T | C21286 | Ext | 0805 | 1.5 A | 220 mOhm | 162 470 | |
| Sunlord GZ1608D601CTF | C74330 | Ext | 0603 | 500 mA | 400 mOhm | 18 868 | |
| Taitech FCM1608KF-601T03 | C141723 | Ext | 0603 | 350 mA | 500 mOhm | 533 435 | |

(CALC) Voltage drop across C1017 at 126 mA: 38 mV; power 4.8 mW.

### 6.2 Bulk electrolytic / hybrid-polymer options, 47-100 uF at 35 V (SMD can). All **Extended**; no Basic aluminum electrolytic found in about 1 200 JLC results

**Voltage rating (CALC):** the TVS clamps at 29.2 V (SMAJ18A, 13.7 A) which exceeds 25 V; use **35 V rated** parts. 25 V parts in the listing (e.g. C178540, C311671) are below the clamp.

| MPN | LCSC | Cap / V | Size | ESR @100 kHz | Ripple current | Life | Stock | Type |
|---|---|---|---|---|---|---|---|---|
| FZ100UF35V90RV0064 | C2992596 | 100 uF 35 V | 6.3 x 7.7 mm | **360 mOhm** | 280 mA @100 kHz | 3 000 h/105 C | 15 898 | aluminum |
| Panasonic EEEFPV101XAP | C178585 | 100 uF 35 V | 6.3 x 7.7 | **160 mOhm** | 600 mA @100 kHz | 2 000 h/105 C | 2 548 | aluminum |
| RVE100UF35V67RV0072 | C2836437 | 100 uF 35 V | 6.3 x 7.7 | 600 mOhm | 230 mA @120 Hz | 2 000 h/105 C | 670 241 | aluminum |
| RVE35V100M6X8 | C48971002 | 100 uF 35 V | 6.3 x 7.7 | 600 mOhm | 230 mA @120 Hz | 2 000 h/105 C | 8 862 | aluminum |
| 118EC240 | C5355875 | 100 uF 35 V | 6.3 x 7.7 | **35 mOhm** | 2 A @100 kHz | - | 2 947 | hybrid polymer |
| Panasonic EEEFP1V470AP | C401754 | 47 uF 35 V | 6.3 x 5.8 | 260 mOhm | 300 mA @100 kHz | 2 000 h/105 C | 3 579 | aluminum |
| 189RV0005 | C7471867 | 47 uF 35 V | 5 x 5.7 | 360 mOhm | 240 mA @100 kHz | 2 000 h/105 C | 8 635 | aluminum |
| RAHVK470M1GDC8002EA | C19271513 | 47 uF 35 V | 6.3 x 5.8 | 60 mOhm | - | 125 C class | 13 000 | hybrid polymer |
| GYC1V470MCQ1GS | C1848745 | 47 uF 35 V | 6.3 x 5.8 | 60 mOhm | 900 mA @100 kHz | 2 000 h/135 C | 7 247 | hybrid polymer |

Source for all: JLC attributes [LIST], 2026-11-02; datasheets not read. Ripple-current ratings of 230-600 mA exceed the 126 mA worst-case input current by a wide margin (CALC) - the input ripple is carried by the ceramic Cin, so the bulk cap mainly damps the cable resonance.

Cable-damping rule of thumb (interp., not from a datasheet): an adapter cable of about 2 uH with an effective ceramic Cin of about 3 uF (Section 7 at 12 V) has characteristic impedance sqrt(L/C) = 0.8 Ohm; a damping electrolytic with ESR near that value (0.16-0.6 Ohm) and capacitance >= about 4 x Cin (12 uF) damps it. 100 uF / 0.36 Ohm satisfies this. Hot-plug overshoot of an undamped LC can approach 2 x VIN (32 V at 16 V in), which is within the AP63205 abs max of 35 V DC / 40 V 400 ms (p4) but above the 29.2 V TVS clamp (CALC).

---

## 7. MLCC DC-bias (Samsung SEM simulation library, **[SIM]**)

Source: Samsung SEM product-page data embedded in https://product.samsungsem.com/mlcc/<PN>.do (graph "DCBias", delta-C %, 25 C). Test conditions per page: 22 uF parts 120 Hz / 0.5 Vrms; 10, 4.7, 1 uF parts 1 kHz / 1.0 Vrms. Fetched 2026-11-02. Values are the nearest sampled point (x within 0.1 V of the stated voltage); effective capacitance = nominal x (1 + delta) (CALC). This is manufacturer simulation data, not a controlled datasheet table.

| Part (Samsung) | LCSC, class, stock | Spec | at 5 V | at 9 V | at 12 V | at 16 V |
|---|---|---|---|---|---|---|
| **CL21A226MPQNNN** 22 uF 10 V X5R 0805 +/-20 % | C29277 (CL21A226MPQNNNE) Ext, 586 599 | 0805 | **-71.0 % (6.4 uF)**; at 10 V -88.3 % | -86.3 % (3.0 uF) | beyond rating | beyond rating |
| **CL21A226MOQNNN** 22 uF 16 V X5R 0805 | C98190 (CL21A226MOQNNNE) Ext, 834 221 | 0805 | **-62.6 % (8.2 uF)** | -82.3 % | -87.2 % | -91.5 % (1.9 uF) |
| CL21A226MAQNNN 22 uF 25 V X5R 0805 (previous extract, `discrete_datasheet_extract.md` section 8.3) | C45783 **Basic** | 0805 | -51.3 % (10.7 uF) | -74.9 % | -81.4 % | -87.6 % |
| CL31A226KAHNNN 22 uF 25 V X5R 1206 (previous extract; re-fetched, identical) | C12891 **Basic** | 1206 | **-38.0 % (13.6 uF)** | -65.4 % (7.6 uF) | -73.9 % (5.7 uF) | -82.3 % (3.9 uF) |
| **CL31A106KBHNNN** 10 uF 50 V X5R 1206 +/-10 % | C13585 (CL31A106KBHNNNE) **Basic**, 3 250 868 | 3.2 x 1.6 x 1.6 mm | -27.2 % (7.3 uF) | -56.0 % (4.4 uF) | **-66.0 % (3.4 uF)** | -76.6 % (2.3 uF) |
| **CL31B106KBHNNN** 10 uF 50 V X7R 1206 | C89632 (CL31B106KBHNNNE) Ext, 779 839 | 1206 | -15.0 % (8.5 uF) | -41.6 % (5.8 uF) | **-53.4 % (4.7 uF)** | -66.1 % (3.4 uF) |
| **CL31A475KBHNNN** 4.7 uF 50 V X5R 1206 | C12881 (CL31A475KBHNNNE) Ext, 9 305 | 1206 | -6.1 % (4.4 uF) | -16.4 % (3.9 uF) | **-24.1 % (3.6 uF)** | -34.4 % (3.1 uF) |
| CL31B475KBHNNN 4.7 uF 50 V X7R 1206 (page returns data identical to the X5R part - treat with suspicion) | C51205 (CL31B475KBHNNNE) Ext, 212 141 | 1206 | -6.1 % | -16.4 % | -24.1 % | -34.4 % |
| **CL21A475KBQNNN** 4.7 uF 50 V X5R 0805 | C98192 (CL21A475KBQNNNE) Ext, 313 242 | 0805 | -31.8 % (3.2 uF) | -59.5 % (1.9 uF) | -69.1 % (1.45 uF) | -78.8 % (1.0 uF) |
| **CL10A105KB8NNN** 1 uF 50 V X5R 0603 | C15849 (CL10A105KB8NNNC) **Basic**, 12 123 305 | 0603 | -34.4 % (0.66 uF) | -61.8 % (0.38 uF) | -70.9 % (0.29 uF) | -80.0 % (0.20 uF) |

Reading these (CALC): at 12-16 V the 25 V 22 uF 1206 part (5.7/3.9 uF) retains more than the 50 V 10 uF 1206 X5R (3.4/2.3 uF), but has only 25 V rating against a 29.2 V TVS clamp; the 4.7 uF 50 V 1206 X5R (3.6/3.1 uF) keeps more than the 10 uF 50 V X5R part at 12-16 V and is Extended. CL21B475KBQNNN / CL10B105KB8NNN pages had no DC-bias data (NOT FOUND). 100 nF X7R 50 V (C14663) DC-bias: NOT FOUND (carried over).

Output capacitor impact (CALC): the datasheet's 2 x 22 uF output (p2, Table 3 p13) is 2 x 6.4 = 12.8 uF at 5 V with 10 V parts, 2 x 8.2 = 16.4 uF with 16 V parts, 21.4 uF with the 25 V 0805 Basic parts, 27.2 uF with 25 V 1206 Basic parts; all exceed the 1.5-2.2 uF an LDO such as XC6206 needs and are well within what the AP63205 can tolerate (datasheet range "22 uF to 68 uF" nominal, p14).

---

## 8. Consolidated NOT FOUND / caveats

1. AP63205 efficiency at VIN = 9 V (no curve; only 12 V and 24 V, Fig. 2 p2).
2. AP63205 maximum duty cycle / dropout voltage curve; Fig. 24 derating curve not read in detail.
3. AP63205 datasheet inconsistencies: thermal shutdown 160 C/25 C hysteresis (table, p5) vs 150 C/130 C (text, p11); hiccup 512/8192 cycles (p10) vs 2 ms/16 ms (p11); UVLO falling 3.1 V (text) vs 3.06 V (table-derived).
4. No Basic-class power inductor or LDO exists on JLC for this duty; every viable inductor and LDO is Extended.
5. SS34 and B5819W Vf at 50 mA (curves start at 0.1 A); Vf values are curve-read (+/-0.02-0.03 V).
6. HT7550-1 dropout at 100 mA and Cout requirement; NCV4275A theta-JA; TPS7B8250 dropout at 100 mA; TPS7B6950 non-Q1 datasheet; NCV8664C ESR stability region value (Fig. 4 p5).
7. Ferrite bead and electrolytic data are JLC listing attributes only; no impedance-vs-DC-bias curves read.
8. Samsung simulation library values are not controlled specification numbers; X7R 4.7 uF page appears to duplicate the X5R data.
9. torexsemi.com is blocked; XC6206 read from the LCSC mirror (ETR0305_004b); confirm no newer Torex revision.
