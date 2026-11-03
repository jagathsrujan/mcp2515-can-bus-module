# JLCPCB stock / class refresh (2026-11-03)

Source: JLCPCB parts API (`POST https://jlcpcb.com/api/overseas-pcb-order/v1/shoppingCart/smtGood/selectSmtComponentList`, keyword = LCSC code), queried 2026-11-03. Stock moves daily - re-check on the day of ordering. Class = `componentLibraryType` (base = Basic, expand = Extended); `preferredComponentFlag` was false for every part (no Preferred-Extended discount applies). Required quantity = per-board qty x 5 boards.

| LCSC | JLC model | Class (now) | Class (BOM_master) | Stock | Qty needed (5 bd) | Unit price @ qty (USD) | Flag |
|---|---|---|---|---|---|---|---|
| C1017 | GZ2012D601TF | Basic | Basic | 487952 | 5 | 0.0201 | ok |
| C12368 | MCP2515-I/SO | Extended | Extended | 896 | 5 | 1.7483 | JLC: THIS PRODUCT IS NO LONGER MANUFACTURE; replacement C153782 |
| C124014 | TCAN1042HGVDR | Extended | Extended | 2253 | 5 | 2.2485 | ok |
| C12891 | CL31A226KAHNNNE | Basic | Basic | 1503113 | 10 | 0.118 | ok |
| C13585 | CL31A106KBHNNNE | Basic | Basic | 3204326 | 10 | 0.1413 | ok |
| C13738 | X322516MLB4SI | Basic | Basic | 350892 | 5 | 0.0937 | ok |
| C14663 | CC0603KRX7R9BB104 | Basic | Basic | 39909204 | 30 | 0.0081 | ok |
| C148219 | SMAJ18A | Extended | Extended | 11106 | 5 | 0.1144 | ok |
| C15849 | CL10A105KB8NNNC | Basic | Basic | 12030232 | 5 | 0.0135 | ok |
| C15850 | CL21A106KAYNNNE | Basic | Basic | 7996240 | 15 | 0.0422 | ok |
| C1634 | CL10C100JB8NNNC | Basic | Basic | 1153830 | 10 | 0.0076 | ok |
| C2071056 | AP63205WU-7 | Extended | Extended | 27120 | 5 | 0.4625 | ok |
| C21189 | 0603WAF0000T5E | Basic | Basic | 20377517 | 15 | 0.0022 | ok |
| C21190 | 0603WAF1001T5E | Basic | Basic | 29830432 | 5 | 0.0017 | ok |
| C22787 | 0603WAF1200T5E | Basic | Basic | 2013689 | 20 | 0.0019 | ok |
| C22843 | 0603WAF1501T5E | Basic | Basic | 3536349 | 5 | 0.002 | ok |
| C2286 | KT-0603R | Basic | Basic | 6960563 | 5 | 0.0077 | ok |
| C23140 | 0603WAF330JT5E | Basic | Basic | 4159515 | 25 | 0.0024 | ok |
| C25804 | 0603WAF1002T5E | Basic | Basic | 46758005 | 15 | 0.0024 | ok |
| C2961147 | PJ-002AH | Extended | Extended | 1247 | 5 | 1.0579 | ok |
| C2992596 | FZ100UF35V90RV0064 | Extended | Extended | 15898 | 5 | 0.053 | ok |
| C474952 | KF128-5.08-2P-AA | Extended | Extended | 56018 | 0 | 0.1931 | ok |
| C474953 | KF128-5.08-3P-AA | Extended | Extended | 6287 | 10 | 0.318 | ok |
| C48427 | SWPA4020S4R7MT | Extended | Extended | 30115 | 5 | 0.0624 | ok |
| C53987 | 0603B472K500NT | Basic | Basic | 836791 | 5 | 0.005 | ok |
| C5446 | XC6206P332MR-G | Basic | Basic | 520539 | 5 | 0.1481 | ok |
| C552488 | PESD2CANFD27V-TR | Extended | Extended | 1595 | 5 | 0.6626 | ok |
| C55629815 | SH-PZ254V-1x8P | Extended | Extended | 1632 | 5 | 0.0617 | ok |
| C55629823 | SH-PZ254V-2x2P | Extended | Extended | 1760 | 5 | 0.0327 | ok |
| C57112 | 0603B103K500NT | Basic | Basic | 5416476 | 5 | 0.0097 | ok |
| C6378052 | 0ZCJ0035AF2E | Extended | Extended | 9782 | 5 | 0.2834 | ok |
| C76584 | ACT45B-510-2P-TL003 | Extended | Extended | 0 | 0 | 0.4839 | STOCK < 50 (DNP part - no impact) |
| C8678 | SS34 | Basic | Basic | 3220419 | 5 | 0.0351 | ok |

Shunt C100114 (hand-fit, not on JLC BOM): stock 363100, class Extended.

## Flags

1. **C12368 MCP2515-I/SO (U1)**: JLC marks it *"This product is no longer manufacture"* (not purchasable for global sourcing, `isBuyComponent`=0) although 896 are in stock today. Pin-compatible replacement listed by JLC: **C153782 MCP2515T-I/SO** (same die, tape-and-reel P/N, same SOIC-18W footprint, Extended, stock 4875, 1.6073 USD @1-9). Recommendation: before ordering, either confirm in the quote that C12368 is still placeable (896 >= 5 needed) or swap the LCSC number to C153782 in the JLC BOM only (no PCB change; MPN differs by the 'T'). Schematic/BOM_master were not modified.
2. **C76584 L2 ACT45B-510-2P**: stock 0 - L2 is DNP, so no effect on the Rev A order; relevant only if a choke is later fitted.
3. No assembled part has stock < 50. Lowest assembled stock: J1 C2961147 (1247), D3 C552488 (1595), J2 C55629815 (1632), J3/J4 C474953 (6287).
4. No Basic/Extended class changes versus BOM_master (all 33 codes identical). XC6206 (C5446) is listed as XC6206P332MR-G (Basic) - same part, RoHS suffix.
5. Extended parts on the assembled BOM (each carries JLC's per-part setup fee, about 3 USD, unverified): C12368, C124014, C2071056, C48427, C148219, C552488, C2961147, C474953, C55629815, C55629823, C6378052, C2992596 = **12**.
