# Phase 5 — CAN Transceiver, Termination and Bus Protection

Status: **Opus design PROPOSED, awaiting user approval** (2026-11-02). Sonnet verification (V-10…V-14) and `can_phy_netlist.md` reviewed; revisions marked **(rev)**.
Transceiver source: TI TCAN1042H/HG/HGV/HV datasheet SLLSES7D, https://www.ti.com/lit/ds/symlink/tcan1042h.pdf (values via `research/ic_datasheet_extract.md` §B4). Tags: **[DS]**, **[CALC]**, **[ASSUME]**, **[TEST]**.

---

## 1. Block diagram
```
 U1 TXCAN ──► U2 pin1 TXD                       ┌── R_T1a‖R_T1b (60 Ω) ──┐
 U1 RXCAN ◄── U2 pin4 RXD                       │                        SPLIT ── C_SPLIT 4.7 nF ── GND
 +5V ─────── U2 pin3 VCC (100 nF + 10 µF)       │  ┌─ R_T2a‖R_T2b (60 Ω) ┘
 VIO ─────── U2 pin5 VIO (100 nF)          JP1a │  │ JP1b
 GND ─────── U2 pin2 GND                        │  │
 TP_STB ──── U2 pin8 STB ── 10 kΩ ── GND        │  │
 U2 pin7 CANH ── CANH_T ─[L2 CMC (DNP) ∥ R 0 Ω]─ CANH ─┴──┼──── D3 pin1 ── J3/J4 pin1
 U2 pin6 CANL ── CANL_T ─[L2 CMC (DNP) ∥ R 0 Ω]─ CANL ────┴──── D3 pin2 ── J3/J4 pin2
                                                         D3 pin3 ── GND ── J3/J4 pin3
```

## 2. Transceiver connections — U2 TCAN1042HGVDR (SOIC-8)
| Pin | Name | Net | Design | Evidence |
|---|---|---|---|---|
| 1 | TXD | TXCAN | direct from U1 pin 1, < 10 mm | VIH 0.7·VIO / VIL 0.3·VIO; internal pull-up → recessive if U1 unpowered **[DS §7.7, §9.3.5]** |
| 2 | GND | GND | 2 vias to plane at pin | |
| 3 | VCC | +5V | 100 nF X7R 0603 at pin + 10 µF 25 V 0805 | 4.5–5.5 V; dominant ICC 70 mA max **[DS §7.4, §7.7]**; buck 5.0 V ± 1 % + ripple well within range |
| 4 | RXD | RXCAN | direct to U1 pin 2 | VOH ≥ 0.8·VIO, VOL ≤ 0.2·VIO **[DS]**; compatibility with MCP2515 checked in Phase 4 §4 |
| 5 | VIO | VIO | 100 nF at pin | 3.0–5.5 V **[DS §7.4]** = DR-02 |
| 6 | CANL | CANL_T | | ±70 V DC, ±30 V common mode **[DS §7.1, §7.7]** |
| 7 | CANH | CANH_T | | same |
| 8 | STB | STB | **10 kΩ to GND** + test pad TP_STB | Internal pull-up → standby if open **[DS §9.3.5]**; pull-down forces Normal mode. Internal pull-up 2–20 µA (IIL, Table 7.7 p9) → STB ≤ 0.20 V vs VIL 0.9 V at VIO 3.0 V **(rev, V-11)** |

Mode table **[DS Table 9-2]**: STB low + VCC & VIO above UV → Normal; VCC < UV or VIO < UV → "Protected" (bus high-Z, RXD high/high-Z). Closes spec §4 items 1–2.

Bit-rate limit: TCAN1042HGV 5 Mbit/s capable; system limited to **1 Mbit/s by the MCP2515** (CAN 2.0B, no FD). TXD dominant time-out 1.2–3.8 ms → minimum bit rate 11 bits / 1.2 ms ≈ 9.2 kbit/s **[DS / CALC]** — all target rates OK.

## 3. Termination (D-019 refined)
### 3.1 Topology — split termination with a 2-pole switch
- Each leg 60 Ω = two 120 Ω 1 % 0603 in parallel (Basic C22787). Differential total 120 Ω ± 1 % **[CALC]**.
- C_SPLIT 4.7 nF 50 V X7R 0603 (Basic C53987) from the split node to GND. Common-mode low-pass corner f = 1/(2π · (60 ∥ 60) · 4.7 nF) = **1.13 MHz** **[CALC]** — attenuates common-mode noise above the 1 Mbit/s fundamental.
- **Switch change: single 1×2 jumper → 2-pole JP1.** A single switch can only break one leg; with it open, the other leg plus C_SPLIT still hang on one bus line (60 Ω + 4.7 nF to GND on CANL only), adding asymmetric capacitance — exactly what split termination is meant to avoid. **(rev)** A 2-pole switch breaks both legs. The SMD DIP switch found (C6331180) is rated 24 V / 25 mA, and the worst-case leg current is VOD(D)max / 120 Ω = 3.0 V / 120 Ω = **25 mA** — zero derating margin, so it is rejected. **Selected: JP1 = 2×2 2.54 mm header (SH-PZ254V-2x2P, C55629823) + two shunts (C100114, hand-fitted), 3 A rating.** Silk: "TERM 120Ω — fit both".

### 3.2 Termination power **[CALC]**
- Normal dominant: V_diff 1.5–3.0 V into 50–65 Ω **[DS SLLSES7D Table 7.7, verified V-13]**; recessive common mode 2.0–3.0 V; per 120 Ω resistor V = V_diff/2 → P = (1.5 V)²/120 Ω = **19 mW** at 3 V (dominant duty ≤ 100 %) vs 100 mW rating — OK.
- Fault survivability: each resistor ≤ 100 mW ⇒ continuous V_diff ≤ 2·√(0.1 × 120) = **6.9 V**. A CANH short to a 9–16 V rail with termination fitted will overheat the 0603s. **Limitation, documented:** termination is for normal operation; the transceiver itself survives ±70 V. Not designed around (1206/2512 parts would only raise this to ~11–22 V).

## 4. Bus ESD / surge protection — D3
### 4.1 Requirement
- Protect U2 bus pins (IEC 61000-4-2 ±8 kV powered rating **[DS §7.3]**) to at least IEC ±15 kV at the connector.
- Must not conduct during normal operation or common-mode excursions (≤ ±12 V CAN, ±30 V receiver range) and should stand off a 24 V-system miswire (DR-05) **[ASSUME target]**.
- Clamp must stay below U2's ±70 V abs max.
- Low capacitance and matched lines (asymmetry converts common-mode to differential noise).

### 4.2 Candidates
| Part | VRWM | VBR min | Clamp | C/line | IEC | Status |
|---|---|---|---|---|---|---|
| **PESD2CANFD27V-TR** (C552488) | 27 V | 28 V (max 38 V) | 33 typ / 44 V max @ 1 A (8/20 µs); Ipp 2.5 A | 5.2 typ / 6 pF max, 0.5 % matching | ±20 kV contact | **Selected (rev, V-10)** — Nexperia DS 11 Aug 2020 via LCSC copy; AEC-Q101 |
| NUP2105LT1G (C14486) | 24 V | 26.2 V | 40 V max @ 5 A | 30 pF max | ±30 kV | Fallback — official DS read; stands off 24 V nominal but not 27–28 V charging voltage |

Decision rule: VRWM ≥ 27 V ✓, clamp < 70 V ✓ (44 V max at 1 A; VCL at the full 2.5 A not published — but 44 V leaves 26 V to U2's 70 V abs max), IEC ≥ ±15 kV ✓ (20 kV contact), pinout SOT-23 1/2 lines, 3 common ✓. **PESD2CANFD27V accepted (D-017 final).** Limitation: VRWM 27 V has no margin against a 24 V vehicle system at 28 V charge voltage — DR-05 is met for 24 V nominal, not for a charging 24 V system. Lower capacitance (6 pF vs 30 pF) and 0.5 % line matching also beat NUP2105L.

### 4.3 Capacitive loading **[CALC]**
6 pF (or 30 pF) per line vs ISO 11898-2 node capacitance budget — at ≤ 1 Mbit/s even 30 pF adds < 2 ns to edges driven through 60 Ω (τ = 30 Ω × 30 pF ≈ 0.9 ns). Line-to-line matching is inherent in a dual-line array.

## 5. Common-mode choke — L2 (DNP, D-005)
- TDK ACT45B-510-2P (51 µH CM) footprint, **not fitted**; two 0 Ω 0603 (Basic C21189) bridge CANH_T→CANH and CANL_T→CANL by default.
- Fit the choke (and remove 0 Ω) only if EMC testing requires it; CMCs can cause ringing with some transceivers — TI and NXP application notes recommend evaluating per system **[ASSUME]**.
- Footprint: custom 4-pad (not in KiCad stock library) — Phase 6/7.

## 6. Connectors and grounding
- J3, J4: KF128-5.08-3P, pinout **1 = CANH, 2 = CANL, 3 = GND** (silk-labelled), wired in parallel for daisy-chaining.
- GND pin tied to the solid ground plane; no chassis/shield connection (module is non-isolated — documented: all nodes must share a ground within the CAN common-mode range ±12 V).

## 7. Layout rules (feeds Phase 7)
1. D3 right at J3/J4 (< 5 mm), GND pin with ≥ 2 vias directly at the pad; ESD current path must not cross U2.
2. **(rev)** Route CANH/CANL as a coupled pair (0.25 mm / 0.2 mm gap) J3/J4 → D3 → termination (JP1, R11–R14, C19) → L2/0 Ω → U2 — choke nearest the transceiver, matching §1, over unbroken bottom GND, equal lengths (±1 mm), no vias if possible.
3. Termination resistors and JP1 close to the connector side; C_SPLIT with its own GND via.
4. U2 within 15 mm of the connector; TXD/RXD to U1 short.
5. Keep the pair ≥ 5 mm from L1/U3 (switching) and from Y1.
6. GND pour around (not between) CANH/CANL; no pour slivers inside the pair.

## 8. Verification results and review
| ID | Result |
|---|---|
| V-10 | PESD2CANFD27V passes — see §4.2 |
| V-11 | STB pull-up 2–20 µA → 10 kΩ OK |
| V-12 | DIP switch rejected (25 mA rating = 25 mA load); 2×2 header + shunts selected |
| V-13 | VOD(D) 1.5–3.0 V into 50–65 Ω; VSYM specified with Csplit 4.7 nF (supports C19); TI §10.2.2.1/§11/§12.1 guidance on split termination, optional CMC and TVS placement consistent with this design |
| V-14 | ACT45B land pattern 5.9 × 3.4 mm span; windings 1–4, 2–3; pins 1/2 transceiver side — custom footprint in Phase 6/7 |

Opus pin review of `can_phy_netlist.md` against SLLSES7D Table 6-1: U2 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STB — match. D3 pins 1/2 to CANH/CANL, 3 to GND — match Nexperia pinout. J3/J4 1 CANH, 2 CANL, 3 GND. No conflicts.

Symbols/footprints to create (Phase 6/7): TCAN1042HGV symbol (from MCP2562 symbol, pins 2/3/8 renamed), L2 ACT45B footprint, check KF128 drill vs Phoenix 5.08 footprint.
