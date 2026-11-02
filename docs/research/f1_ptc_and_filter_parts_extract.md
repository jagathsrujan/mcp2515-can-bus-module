# F1 PTC candidates and FB1 / C1 datasheet confirmation (Phase 3 follow-up)

Sonnet research, 2026-11-02. JLC data from the JLCPCB parts API (`selectSmtComponentList`), stock is a snapshot. Datasheet pages = PDF page index. No Basic-class PTC exists in the roughly 580 JLC PTC listings paged (all 1206/1812/2920 hits are Extended).

## 1. F1 requirement filter
I_hold >= 0.25 A, I_trip <= 0.75 A, V_max >= 30 V, 1206 or 1812, R <= 2 ohm (taken as datasheet R1max, the conservative post-solder maximum). Time to trip is the datasheet "max time to trip" at the stated fault current. 59 listings passed the first three criteria; best by stock/maker below.

| MPN | LCSC | Class | Stock | Ihold / Itrip / Vmax | Rmin / R1max | Max time to trip | Imax | Price (1 pc) | Datasheet (page) |
|---|---|---|---|---|---|---|---|---|---|
| **Bel Fuse 0ZCJ0035AF2E** | C6378052 | Ext | 9 782 | 0.35 A / 0.75 A / 30 V | 0.30 / 1.2 ohm | 0.1 s @ 8 A | 40 A | $0.28 | https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2312110255_Bel-Fuse-0ZCJ0035AF2E_C6378052.pdf (electrical p1, derating p3: 48 % of Ihold at 85 C = 0.17 A) |
| BHFUSE BSMD1206-035-30V | C910829 | Ext | 32 223 | 0.35 / 0.75 / 30 V | 0.25 / 1.5 ohm | 0.1 s @ 8 A | 40 A | $0.051 | https://www.lcsc.com/datasheet/lcsc_datasheet_2305121424_BHFUSE-BSMD1206-035-30V_C910829.pdf (electrical p2, derating p5: 0.15 A at 85 C) |
| Littelfuse 1206L035/30WR (WR = tape-and-reel ordering suffix, same electrical as 1206L035/30) | C2153989 | Ext | 4 662 | 0.35 / 0.75 / 30 V | **0.30 / 1.2 ohm per datasheet** (JLC listing says 100 / 300 mohm: listing is wrong) | 0.1 s @ 8 A | 100 A | $0.32 | https://www.lcsc.com/datasheet/lcsc_datasheet_2304140030_Littelfuse-1206L035-30WR_C2153989.pdf (electrical p1, derating p2: 0.15 A at 85 C) |
| LUTE 1812L035/30GR (1812) | C18198328 | Ext | 40 538 | 0.35 / 0.70 / 30 V | 0.20 / 1.7 ohm | 0.1 s @ 8 A | 40 A | $0.046 | https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2309261133_LUTE-1812L035-30GR_C18198328.pdf (p2) |
| BHFUSE BSMD1206-030-30V | C22378342 | Ext | 20 188 | 0.30 / 0.65 / 30 V | 0.25 / 1.7 ohm | 0.1 s @ 8 A | (not read) | - | JLC listing; datasheet family as above |
| TECHFUSE nSMD025-30V | C70070 | Ext | 52 760 | 0.25 / 0.50 / 30 V | 0.35 / 2.5 ohm (**fails R <= 2 ohm**) | 0.08 s @ 8 A | 100 A | $0.044 | https://www.lcsc.com/datasheet/lcsc_datasheet_2304041600_TECHFUSE-nSMD025-30V_C70070.pdf (p2) |
| BHFUSE BSMD1206-025-30V | C883121 | Ext | 10 209 | 0.25 / 0.50 / 30 V | 0.35 / 2.5 ohm (listing; fails R <= 2 ohm) | 0.1 s (listing) | - | - | datasheet not read |

Findings:
- **Best candidate: 0.35 A class, 1206, 30 V** (Bel 0ZCJ0035AF2E, or BHFUSE BSMD1206-035-30V as the cost option, 5x cheaper, 3x the stock). Both meet I_trip = 0.75 A exactly (limit is <=, no margin), R1max 1.2-1.5 ohm (drop at 125 mA <= 0.19 V), 30 V, 8 A trip in 0.1 s.
- **Ihold derates with temperature (design point to flag to Opus):** the 0.25 A parts (Ihold 0.12 A at 85 C in the Littelfuse 1206L025 table, Littelfuse p2) have *less* than the 125 mA worst-case input current at 85 C ambient, so the "Ihold >= 0.25 A" criterion at 23 C is not a 2x margin at the top of the temperature range. The 0.35 A class gives 0.15-0.17 A at 85 C (20-35 % over 125 mA). A 0.5 A part would give more but has Itrip 1.0 A (> 0.75 A limit).
- Vmax 30 V is a rating at Imax; with the TVS conducting, the PTC sees only (Vsupply - about 20 V), so 30 V is adequate.
- Not found on JLC: any Basic PTC; time-to-trip curves in seconds at lower overloads (only the 8 A point is tabulated; the curve plots were not digitised).

## 2. FB1 GZ2012D601TF (C1017, Basic) - Sunlord GZ catalogue, revised 2024/9/10 (12 pages)
Source: https://www.lcsc.com/datasheet/lcsc_datasheet_2310301640_Sunlord-GZ2012D601TF_C1017.pdf (JLC mirror; the PDF is image-only, read by rendering).
- Table p3 (GZ2012 type): impedance 600 ohm +-25 % @ 100 MHz, DCR 0.30 ohm max, rated current 500 mA, thickness 0.85 +-0.2 mm. **Confirms the listing exactly.**
- Impedance plot p10 (GZ2012D601TF), read from the graph (+-10 %): Z about 5 ohm @ 1 MHz, about 15 ohm @ 2 MHz, about 28 ohm @ 5 MHz, about 52 ohm @ 10 MHz; peak about 670 ohm near 130 MHz (R and X cross near 100 MHz; X peak about 350 ohm near 65 MHz). Below about 25 MHz it is almost purely reactive (R about 0), i.e. roughly 0.8 uH.
- Impedance versus DC bias: NOT FOUND (only the zero-bias plot; the derating curve p5 covers temperature only).
- Consequence (interp.): at the 1.1 MHz switching fundamental the bead is only about 5 ohm (inductive), so the pi-filter benefit there is modest; it mainly helps above 20-30 MHz. Its approximately 0.8 uH with C2/C3 (4.6-8.8 uF effective) forms a resonance near 60-80 kHz with Z0 about 0.3 ohm, damped only by bead DCR + C1 ESR; check input stability/peaking in [TEST] (U-11).

## 3. C1 FZ100UF35V90RV0064 (C2992596) - KNSCHA approval sheet p1
Source: https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2204151800_KNSCHA-FZ100UF35V90RV0064_C2992596.pdf (Chinese-language sheet, 4 pages; p1 has the table).
- 100 uF +-20 % (120 Hz, 25 C); rated 35 V DC; surge 44 V DC; tan delta 14 % (120 Hz); leakage 35 uA (2 min, 25 C); 6.3 x 7.7 mm.
- **Impedance max 0.36 ohm @ 100 kHz, 25 C** (the listing calls this ESR; the sheet gives it as impedance); **ripple 280 mA max @ 100 kHz, 105 C**; life 3000 h @ 105 C; -55 to +105 C; FZ "very low impedance" series.
- Ripple frequency correction: 50/60 Hz 0.6, 120 Hz 0.7, 1 kHz 0.85, >= 10 kHz 1.0.
- **Confirms the listing** (35 V, 100 uF, 0.36 ohm, 280 mA). Margin note: 35 V rating vs 29.2 V TVS clamp = 5.8 V; surge rating 44 V.
