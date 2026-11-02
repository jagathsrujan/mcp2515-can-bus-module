# Phase 2 — Component Selection Report

Status: **Opus selections PROPOSED, awaiting user approval** (2026-11-02)
Evidence: `research/ic_datasheet_extract.md`, `research/discrete_datasheet_extract.md` (Sonnet; every value carries URL + table/page). Tags: **[DS]** datasheet, **[CALC]**, **[ASSUME]**, **[TEST]**, **[LIST]** = JLC/LCSC listing only (datasheet not read).

---

## 1. Summary of selections

| Ref | Function | Selected part | Alternate (pin-compatible) | LCSC | JLC class |
|---|---|---|---|---|---|
| U1 | CAN controller | **MCP2515-I/SO** (SOIC-18 W) | MCP2515T-I/SO (reel) | C12368 | Ext |
| U2 | CAN transceiver | **TCAN1042HGVDR** (SOIC-8) | TJA1051T/3/1J (C38695), MCP2562-E/SN (C53609) | C124014 | Ext |
| U3 | 5 V regulator | **AP63205WU-7** buck, fixed 5 V (TSOT-26) — *final LDO-vs-buck call in Phase 3* | LDO path only if a ≥ 30 V, ≤ 1 V-dropout part is found (Phase 3) | C2071056 | Ext |
| U4 | 3.3 V LDO (optional VIO source) | **TLV75533PDBVR** (SOT-23-5) | XC6206P332MR (C5446, Basic) if its datasheet can be verified | C404027 | Ext |
| D1 | Reverse polarity | **SS34** Schottky 40 V/3 A (SMA) — *change from P-FET, see D-006r* | B5819W (C8598, Basic) | C8678 | **Basic** |
| D2 | Input TVS | **SMAJ18A** (Littelfuse, SMA, 400 W) | Diodes SMAJ18A-13-F (C134965) | C148219 | Ext |
| D3 | CAN ESD/TVS | **PESD2CANFD27V-TR** — *conditional, datasheet to be read in Phase 5* | NUP2105LT1G (C14486, official DS read) | C552488 | Ext |
| L2 | CAN common-mode choke | ACT45B-510-2P footprint, **DNP**, 0 Ω bypass fitted | — | C76584 | Ext (not ordered) |
| Y1 | 16 MHz crystal | **YXC X322516MLB4SI** (3225, CL 9 pF, ±10 ppm) | X322516MMB4SI (10 pF) | C13738 | **Basic** |
| R-term | Split termination | **4 × 120 Ω 1 % 0603** (2 ∥ per leg = 60 Ω) | 2 × 60.4 Ω (Ext) | C22787 | **Basic** |
| C-term | Termination centre tap | **4.7 nF 50 V X7R 0603** | — | C53987 | **Basic** |
| LED1 | +5V power LED | **KT-0603R** red | — | C2286 | **Basic** |
| J1 | 9 V barrel jack 5.5×2.1 | **CUI PJ-002AH** (5 A) | DC-005 (C16214, 1 A) | C2961147 | Ext |
| J1b | 9 V screw terminal (alt footprint, DNP) | KF128-5.08-2P | — | C474952 | Ext (not ordered) |
| J3, J4 | CAN terminals (daisy-chain) | **KF128-5.08-3P** ×2 | — | C474953 | Ext |
| J2 | Host header 1×8 2.54 | SH-PZ254V-1x8P | HX PZ2.54-1x8P (C32713274) | C55629815 | Ext |
| JP1 | Termination jumper | 1×2 header + shunt (shunt hand-fitted) | — | C42431798 | Ext |
| — | Passives | 100 nF X7R 50 V 0603 (C14663), 10 µF 25 V X5R 0805 (C15850), 22 µF 25 V X5R 1206 (C12891), 10 k/1 k/1.5 k/4.7 k 1 % 0603 | — | see BOM | **Basic** |

Values of passives (load caps, bulk, pull-ups, buck L/C) are finalised in Phases 3–5; this phase approves part *families* and sources.

---

## 2. Engineering evaluation

### 2.1 CAN controller — MCP2515-I/SO
| Check | Value | Effect on design |
|---|---|---|
| VDD | 2.7–5.5 V; abs max 7.0 V **[DS]** DS20001801K Table 13-1 p74, §13.1 p73 | Confirms D-001: runs directly on host VIO (3.0–5.5 V). |
| Pin abs max | −0.6 V … VDD + 1.0 V **[DS]** §13.1 | Host must not drive SPI before VIO is up — guaranteed because VIO *is* the host rail. |
| SPI | 10 MHz across full 2.7–5.5 V **[DS]** Table 13-6 p77 | DR-03 met at 3.3 V and 5 V. |
| Input thresholds | VIH 0.7·VDD (SCK/SI/CS), VIL 0.4·VDD **[DS]** Table 13-1 | Same-domain host → no translation. |
| FOSC | 1–25 MHz at 2.7–5.5 V; 40 MHz only at ≥ 4.5 V **[DS]** Table 13-2 p75 | 16 MHz crystal is valid at every VIO → constraint §4-3 of the spec closed. |
| IDD | 10 mA max @ 5.5 V, 25 MHz **[DS]** | VIO host load ≤ 10.5 mA total with transceiver IIO. |
| INT | **push-pull** (VOH specified) **[DS]** Table 13-1 | Corrects spec wording ("open-drain-style"); INT cannot be wire-ORed between modules; no pull-up needed. |
| RESET | no internal pull-up; tRL ≥ 2 µs **[DS]** Table 1-1 p4, Table 13-4 | External pull-up + RC required (Phase 4). |
| Errata DS80179G #5 | holding CS low after a TXREQ write may repeat a frame **[DS]** | Firmware note: raise CS within 47 µs (all common libraries do). |
| Stock | 928 @ JLC **[LIST]** | Adequate for prototypes; reel variant C153782 as backup. |

Package choice: SOIC-18 (hand-reworkable, 1.27 mm pitch) over TSSOP-20/QFN-20 which had no JLC stock.

### 2.2 CAN transceiver — TCAN1042HGVDR selected

| Criterion | MCP2562 | TJA1051T/3 | **TCAN1042HGV** | SN65HVD230 |
|---|---|---|---|---|
| VDD 5 V rail (D-002) | ✓ 4.5–5.5 | ✓ 4.5–5.5 | ✓ 4.5–5.5 | ✗ 3.0–3.6 only |
| VIO range vs DR-02 (3.0–5.5) | ✓ 1.8–5.5 | ✓ 2.8–5.5 (VIO UV up to 2.7 V) | ✓ **3.0–5.5** | ✗ no VIO |
| Bus DC fault | ±58 V | ±58 V | **±70 V** | ✗ −4…+16 V |
| Common mode | ±12 V | ±30 V | ±30 V | −2…+7 V |
| IEC 61000-4-2 bus | ±8 kV | ±8 kV | ±8 kV powered / ±15 kV unpowered | not stated |
| TXD dominant time-out | 1.25 ms typ, no min/max | 0.3–5 ms | **1.2–3.8 ms** | none |
| VIO lost, VDD present | high-Z at power-on; *after power-on not stated* | off, zero bus load | **"Protected": bus high-Z, RXD high-Z** (Table 9-2) | n/a |
| VDD lost, VIO present | high-Z | off | high-Z | loads bus 100–350 µA |
| Recessive IDD | 10 mA max | 10 mA max | **2.5 mA max** | 17 mA |
| θJA SOIC-8 | 149.5 °C/W | 155 | **105.8** | 76.8 |
| Mode pin default | STBY ↑ = standby (must tie low) | S ↓ = normal | STB ↑ = standby (must tie low) | Rs |
| JLC stock / price | 525 / $1.37 | 104 951 / $0.60 | 2 253 / $1.06 | 104 398 / $0.69 |

**Decision:** TCAN1042HGVDR. It is the only candidate whose datasheet explicitly specifies safe behaviour for *both* Phase-1 open constraints (spec §4 items 1–2: VIO missing with 9 V present, and 9 V missing with VIO present), with a min/max-specified dominant time-out and the widest fault rating. SN65HVD230 is rejected on four independent grounds (3.3 V-only VCC, no VIO, 16 V bus abs max, loads an unpowered bus).

**Second source:** all three VIO parts share the pinout 1 TXD, 2 GND, 3 VDD, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STB/S **[DS]**. With pin 8 pulled to GND (normal mode on all three: TCAN STB low = normal, TJA S low = normal, MCP STBY low = normal), TJA1051T/3/1J (105 k stock) is a drop-in alternate with no footprint change. Pin 8 will be a 10 kΩ pull-down + test pad so firmware/host can optionally drive standby (Phase 5).

### 2.3 5 V regulator — candidates screened (final choice Phase 3)

Hard limits a regulator must meet: survive the input TVS clamp **29.2 V** (SMAJ18A Vc @ Ipp **[DS]**), operate down to 7 V − D1 drop ≈ 6.6 V **[CALC]**, deliver 100–150 mA.

| Candidate | Result | Reason |
|---|---|---|
| AMS1117-5.0 | ✗ | Abs max Vin 15 V (AMS DS p2) < 16 V operating; needs tantalum Cout; 10 mA min load |
| TLV76750 | ✗ | Abs max 18 V < 29.2 V TVS clamp |
| TPS7A25 | ✗ | Abs max 20 V; 5 V fixed version not orderable |
| LM2936-5.0 | ✗ | 50 mA max |
| TPS7B6950-Q1 | ✗ | Electrically ideal (40 V, 150 mA, ceramic-stable) but **JLC stock 0** |
| L78M05 | ✗ (provisional) | ~2 V dropout vs 1.6 V headroom at 7 V in; ST datasheet not retrieved |
| **AP63205WU-7** | ✓ shortlisted | 3.8–32 V operating, 35 V abs (40 V/400 ms) **[DS]** Diodes DS41326; fixed 5 V; 1.1 MHz with spread spectrum; 5 external parts; dissipation ≈ 0.1 W instead of 1.1 W LDO worst case **[CALC, Phase 3]** |
| TPS560430X | ✓ backup | 4–36 V, but adjustable (2 extra resistors) |

Phase 3 will (a) do the full LDO-vs-buck calculation including a search for a stocked ≥ 30 V LDO, (b) size L/C with DC-bias derating, (c) confirm 2-layer is adequate for the buck hot loop (D-009).

### 2.4 Reverse polarity — design change D-006r (P-FET → Schottky)
Sonnet's research showed the P-FET route has a real failure mode: during a surge the TVS (downstream, per D-006) clamps at up to 29.2 V, so a P-FET gate sees Vgs ≈ −29 V unless clamped, exceeding even ±20 V parts; the only Basic P-FET (AO3401A, Vgs ±12 V) also has Vds −30 V, only 0.8 V above the clamp.

| Option | Parts | Loss at 9 V **[CALC]** | Margin |
|---|---|---|---|
| P-FET + 10 V zener + resistor | 3 (2 Ext) | ≈ 0.5 mW | Vds 30 V vs 29.2 V clamp — 0.8 V |
| **SS34 Schottky** | 1 (Basic) | ≈ 0.3 V × 65 mA ≈ 20 mW (input current with buck: 5 V × 0.1 A / 0.85 / 9 V ≈ 65 mA) | VRRM 40 V vs 16 V reverse / 29.2 V clamp |

**Decision:** SS34. At this current the P-FET's efficiency advantage (≈ 20 mW) is irrelevant; the Schottky removes the gate-overstress failure mode, is one Basic part, and has 10.8 V margin to the TVS clamp. Vf at 65 mA is read from the datasheet curve only (≈ 0.3 V) → **[TEST]**.

### 2.5 Input TVS — SMAJ18A
VWM 18 V ≥ 16 V max operating; VBR 20–22.1 V; Vc 29.2 V @ 13.7 A; 400 W (10/1000 µs) **[DS]** Littelfuse SMAJ p2. SMAJ15A rejected (VWM 15 V < 16 V). Placed after D1 so a reversed supply never forward-biases it. SMBJ18A adds nothing at this power level and costs board area.

### 2.6 CAN bus ESD — PESD2CANFD27V-TR (conditional)
DR-05 asks the bus pins to survive a 24 V-system miswire. The transceiver tolerates ±70 V, so the TVS is the weak link: every 24 V-class CAN TVS (VBR min 25.4–26.7 V) would conduct continuously at 27–28 V and burn. PESD2CANFD27V is the only 27 V standoff option found (6 pF, from JLC listing). **Its datasheet must be read in Phase 5 before final approval**; fallback is NUP2105LT1G (24 V, 30 pF, ±30 kV IEC, official datasheet read) with DR-05 relaxed to ≤ 24 V DC. Capacitance (6–30 pF) is negligible at ≤ 1 Mbit/s **[ASSUME, check in Phase 5]**.

Note: with the termination jumper fitted, a 24 V miswire puts ≈ 2.4 W into each 0603 leg **[CALC]** — no practical termination survives that; documented as a limitation, not designed around.

### 2.7 Crystal — 16 MHz YXC X322516MLB4SI (Basic)
- 16 MHz < 25 MHz FOSC limit at VDD 2.7–5.5 V **[DS]** → valid for 3.3 V and 5 V hosts.
- Exact bit timing at 16 MHz (TQ = 2·(BRP+1)/FOSC, BRP = CNF1 register value) **[CALC]**: 1 Mbit/s = 8 TQ × 125 ns (BRP 0), 500 k = 16 TQ × 125 ns (BRP 0), 250 k = 16 TQ × 250 ns (BRP 1), 125 k = 16 TQ × 500 ns (BRP 3). All of DR-04 achievable with integer settings; full sample-point design in Phase 4.
- 20 MHz would give 10 TQ at 1 Mbit/s but no Basic 3225 part exists; 16 MHz is sufficient.
- CL = 9 pF (JLC attribute) vs "10 pF / 20 pF or specify" in the series sheet → load caps ≈ 10 pF C0G (2 × (9 − 4 pF stray)) **[CALC]**, frequency to be measured on the first board **[TEST]**.
- MCP2515 datasheet gives no ESR/drive-level limit (NOT FOUND); crystal ESR 50 Ω max, drive 100 µW typ — gain margin to be confirmed on the bench **[TEST]**.

### 2.8 Termination — 4 × 120 Ω Basic + 4.7 nF X7R
Two 120 Ω in parallel per leg = 60 Ω; total 120 Ω ± 1 % differential, satisfies "120 Ω ± 2 %" key check **[CALC]**. Avoids two Extended fees for 60.4 Ω parts. Power at nominal 2 V differential: 33 mW total, ≈ 8 mW per resistor vs 100 mW **[CALC]**. 4.7 nF C0G 0603 is not stocked; X7R ±10 % is acceptable because the centre-tap cap is a common-mode filter whose exact value is non-critical.

### 2.9 Connectors
All THT connectors are Extended at JLC. PJ-002AH chosen over DC-005 for its 5 A rating and a real CUI datasheet (DC-005 has no ratings document). KF128-5.08 series chosen because KF301/DG301 are actually 5.00 mm pitch (would not match 5.08 mm footprints). JP1 shunt is a loose accessory and is fitted by hand.

---

## 3. JLC cost implications (for user decision)
Extended parts loaded by JLC ≈ **11 unique** (U1, U2, U3, U4, D2, D3, L1 buck inductor, J1, J3/J4, J2, JP1). At ≈ $3 each **[unverified fee, search snippet]** ≈ $33 per order in setup fees, independent of quantity. Options:
- Hand-solder the 4 THT connector types → saves ≈ $12.
- Drop U4 (optional 3.3 V LDO) → saves ≈ $3.

## 4. Review findings (Opus on Sonnet's work)
- Research quality good: all IC values traced to official datasheets; MIRROR sources flagged honestly.
- Accepted findings that changed the design: SN65HVD230 disqualified; P-FET gate-overstress → D-006r; termination via 4 × 120 Ω Basic; KF301 pitch mismatch; AMS1117/TLV767 abs-max failures; STB/STBY default-to-standby trap; INT is push-pull.
- Gaps carried forward: ST L78M05 and Torex XC6206 datasheets not retrieved; PESD2CANFD27V datasheet not read; crystal drive level / MCP2515 oscillator gain; Schottky Vf at 65 mA (curve only); JLC Extended fee value.

## 5. Unresolved issues
| ID | Issue | Resolve in |
|---|---|---|
| U-01 | LDO vs buck final; buck inductor/caps; 2-layer adequacy for buck | Phase 3 |
| U-02 | PESD2CANFD27V datasheet verification, else NUP2105L + DR-05 relaxed | Phase 5 |
| U-03 | Crystal load caps & oscillator margin | Phase 4 + bench |
| U-04 | Bulk input capacitance sized with DC-bias derating (22 µF 1206 keeps only ~7.6 µF at 9 V) | Phase 3 |
| U-05 | XC6206 (Basic) could replace TLV75533 if datasheet verified | Phase 3 |
| U-06 | THT connectors: JLC-assembled or hand-soldered | User |
