# Testing and commissioning - MCP2515 CAN module Rev A

Equipment: bench supply (current-limited, 0-30 V), DMM, oscilloscope (>= 100 MHz, 10x probes, spring ground tip), frequency counter (optional), 2 modules, host MCU (Arduino Uno/Nano, ESP32, or STM32), 120 ohm test resistor, jumper wires. Pin and test-point references: [connector_pinouts.md](connector_pinouts.md). Expected values marked **[CALC]** are estimates from the phase documents; record real measurements.

Board fitted as shipped: J5 and L2 not fitted, R16/R17 = 0 ohm, SJ1 open, JP1 shunts hand-fitted (bring-up step 5 removes/fits them).

## 0. Visual / AOI inspection (power off)
- [ ] Compare against `docs/pcb/render_top.png` and the schematic: no missing parts, tombstones, bridges (especially U1 SOIC-18, U2 SOIC-8, U3 TSOT-26).
- [ ] **Polarity / pin-1 checks** (the CPL rotations were never proven): U1, U2 pin-1 dots at pad 1; U3 (FB pin 1, SW pin 5); U4 SOT-23; D3 (SOT-23); **D1/D2 cathode bands**: D1 cathode towards C1/FB1 (VIN_PROT), D2 cathode to VIN_PROT; **C1 negative stripe to GND side**; D4 LED cathode to GND; Y1 straight.
- [ ] Through-hole parts seated and soldered: J1, J2, J3, J4, JP1; J5 and L2 absent; R16/R17 present (0 ohm).
- [ ] JP1 has two shunts for step 5 (initially fit both). SJ1 open (no solder bridge).
- [ ] No solder balls under the buck inductor; mask/silk legible; fiducials intact.

## 1. Unpowered resistance checks (DMM, no supply, no host)
Allow readings of capacitor-charging nets to settle for 5 s.

| Measure | Expected | Fail if |
|---|---|---|
| J1 centre to sleeve (VIN_RAW - GND), red on VIN | > 10 kohm and rising (D1 forward, C1/C2/C3 charging, then buck/TVS high-Z); black on VIN: open (D1 blocks) | < 1 kohm |
| +5V - GND (TP2 - TP5) | > 10 kohm, rising (LED/R1 path is below the diode turn-on at DMM voltage) | < 100 ohm |
| VIO - GND (TP4 - TP5) | > 100 kohm / OL (only IC ESD diodes and decoupling; pull-ups go to VIO, not GND) | < 1 kohm |
| +3V3 - GND (TP3) | > 10 kohm | < 100 ohm |
| CANH - CANL at J3 pins 1-2, **JP1 fitted** | **120 ohm +-2 % (117.6-122.4)** [CALC: 2 x (120 ohm 1 % || 120 ohm 1 %)] | outside range |
| CANH - CANL, **JP1 shunts removed** | > 1 Mohm / OL | any low value (a leg is still connected) |
| CANH - GND, CANL - GND | OL (4.7 nF split cap only, no DC path) | < 100 kohm |
| STB (TP12) - GND | 10 kohm +-1 % | other |
| J1 pin continuity vs J5 pad 1 | same net (VIN_RAW) | open |

## 2. First power-up (current-limited)
1. Bench supply: **9.0 V, current limit 100 mA**, leads to J1 (centre +). No host, JP1 fitted, SJ1 open.
2. Switch on. Supply should stay in constant-voltage mode (CC LED off).
3. Idle current expected **about 2-5 mA** at 9 V [CALC: LED 2.1 mA + TCAN recessive up to 1.5 mA at 5 V, referred through the buck at about 75-85 % PFM efficiency]. Accept 1-10 mA. Stop and power off if > 20 mA, or the supply hits the 100 mA limit.
4. Power LED (D4) lit. Nothing hot after 30 s (finger / thermal camera on U3, L1, D1).
5. Raise to 12 V and 16 V (limit still 100 mA): +5V stays in range; current stays about 2-5 mA. Return to 9 V.

## 3. Rail measurements (9 V in, no host)
| Test point | Expected | Pass |
|---|---|---|
| TP1 VIN | 8.5-8.7 V | |
| TP2 5V | **4.95-5.05 V** | |
| TP3 3V3 | 3.23-3.37 V | |
| TP4 VIO | 0 V (no host, SJ1 open) | |
| TP12 STB | < 0.2 V | |
| TP10 / TP11 CANH / CANL | ~0 V while VIO is absent (U2 in protected mode, bus high-Z) | |

**+5V ripple measurement method**: scope on TP2 with the probe's ground spring in the nearby TP5 (no long ground lead), AC-coupled, 20 mV/div, 20 us/div, 100 MHz bandwidth (do not use the 20 MHz limit alone - also look at the switching spike with full bandwidth). Measure (a) idle / light load (PFM, burst-like ripple, 2-10 mA), (b) with a dummy load of 50 ohm to GND on TP2 (about 100 mA) while power is off before connecting, (c) later with the bus transmitting continuously. Pass: < 50 mV p-p in each case (Phase 3 U-08 limit); record values. If > 50 mV p-p at light load, add the 0 ohm/bead option on U2 VCC noted in phase 3.

## 4. Reverse-polarity test (before applying a host)
1. Supply off. Swap the leads (centre -), 9 V, **limit 100 mA**.
2. Switch on: supply current < 1 mA (SS34 leakage), LED stays off, TP2 = 0 V, nothing warms.
3. Restore correct polarity, confirm normal start-up (step 2) again.

## 5. Over-voltage test, 18-20 V and above (optional, use a spare board)
Risk: stresses D2 and F1; do only short steps.
1. Supply 9 V, limit 300 mA. Raise slowly to 16 V (normal) then 18 V: board runs, +5V unchanged, input current barely rises.
2. 18 V -> 20 V: TVS starts leaking (current rises from µA to tens of mA above ~20 V; breakdown 20.0-22.1 V). Stay at 20 V for a few seconds only.
3. Optional hard fault: 24 V, limit 500 mA, a **maximum 5 s**: expect the TVS to conduct, F1 to heat and trip (input current collapses to a few mA hold-off) and +5V to drop; remove power and allow 1-2 min for F1 to reset. Never exceed 30 V (F1 rating).
4. After the test repeat steps 0-3 to check the board still works. Expect a sacrificial board to be possible.

## 6. Host hookup and SPI check
Wire per [connector_pinouts.md](connector_pinouts.md) (Uno/Nano 5 V: VIO = 5V; ESP32/STM32: VIO = 3V3). SJ1 open. Power 9 V and the host.
- [ ] TP4 VIO equals the host logic voltage; TP7 INT = VIO; TP8/TP9 = VIO.
- [ ] Module supply current at 9 V unchanged (about 2-5 mA); host supplies about 6-11 mA on VIO.

**Register read-back** (SPI mode 0, start at 1 MHz, scope CS/SCK/MISO if it fails):
1. CS low, send `0xC0` (RESET), CS high, wait 10 ms.
2. Read `CANSTAT` (0x0E): CS low, `0x03`, `0x0E`, read byte, CS high. **Expect 0x80** (Configuration mode).
3. Read `CANCTRL` (0x0F): expect 0x87.
4. Write/read CNF1 (0x2A) with 0x55 then 0xAA: values read back exactly.
Fail causes: 0x00 or 0xFF -> wiring, VIO missing, CS polarity, MISO swapped, crystal not running (see 7).

## 7. Crystal frequency check via CLKOUT
After a reset CLKOUT is enabled at FOSC/8. Measure TP6 with a counter (high-Z input) or a 10x probe: **2.000 MHz**, i.e. 16.000000 MHz / 8. Pass: within +-100 ppm (1.99980-2.00020 MHz) for function; goal +-20 ppm (Phase 4) - if the error is larger, check C15/C16 (10 pF C0G) and bench-trim them. Note the probe adds 8-12 pF to a 2 MHz pin, not to the crystal, so reading is valid. Then disable CLKOUT in firmware (CLKEN = 0) and confirm TP6 goes quiet.

## 8. Loopback test (no bus, no second node)
1. Reset; set CNF for the target rate (e.g. 500 kbit/s: 0x00 / 0xAE / 0x01).
2. Set CANCTRL REQOP = loopback (`0x40`): read CANSTAT -> **0x40**.
3. Load TXB0 (ID 0x123, DLC 2, data 0xAA 0x55) and request transmit. Expect `TXB0CTRL.TXREQ` clears, `CANINTF.TX0IF` and `RX0IF` set, INT (TP7) goes low if enabled, RXB0 contains ID 0x123 and data 0xAA 0x55.
4. TP8 (TXCAN) pulses with the frame; the bus pins stay quiet in loopback. Return to normal mode (`CANCTRL = 0x00`).

## 9. Two-node bus test
Two modules A and B, each with its own host (or one host with two chip-selects), common ground. Both modules powered 9 V.
1. Connect A.J3 to B.J3: pin 1 - pin 1 (CANH), pin 2 - pin 2 (CANL), pin 3 - pin 3 (GND), twisted pair, < 1 m.
2. Termination: **JP1 fitted (both shunts) on both modules** (bus then reads 60 ohm CANH-CANL, unpowered). Measure with the DMM first: 60 ohm +-3 %.
3. Both nodes set to the same rate; node A sends a frame every 100 ms, node B receives and prints/echoes it.
4. Run at **125 k, 500 k and 1 M**; mark pass if >= 1000 frames received with no errors (`EFLG` clean, RX error counters 0).
5. Scope CANH and CANL (two probes, same ground, AC-coupled differential math channel) at TP10/TP11 on the transmitting node:

| Rate | Bit time | Expected |
|---|---|---|
| 125 kbit/s | 8 us | recessive CANH = CANL = 2.5 V (+-0.2 V); dominant CANH about 3.5 V, CANL about 1.5 V |
| 500 kbit/s | 2 us | same levels, clean edges (<= 50-100 ns), no ringing > 10 % |
| 1 Mbit/s | 1 us | same levels; slight edge rounding, no overshoot beyond about 4 V / below about 1 V |

**VOD (dominant differential, CANH - CANL) must be 1.5-3.0 V** into the 60 ohm bus [DS TCAN1042 Table 7.7]; typical about 2.0 V. Recessive differential about 0 V (|V| < 0.05 V). Record the sample point position (RXCAN TP9 transitions) vs the TXCAN edge: loop delay about 110-175 ns.
6. **Termination check**: remove one module's shunts (single termination, 120 ohm): the bus still works at 125 k-500 k with slightly more ringing; remove both: errors and ringing at 1 M expected (confirms JP1 has effect). Refit both shunts before continuing.
7. Disconnect the B node mid-run: A transmits with no ACK -> error frames and rising TEC, no damage; reconnect, errors clear.

## 10. STB check
- TP12 reads about 0 V in normal operation (done in 3).
- Optional: briefly touch TP12 to TP4 (VIO): transceiver enters standby, the receiver stops receiving frames (RX count stops); release and normal operation resumes. Do not leave TP12 floating or shorted.

## 11. ESD test (optional)
Only with a calibrated IEC 61000-4-2 gun: apply +-2, +-4, +-8 kV contact to the J3 pin 1/2 terminals and to GND, module powered and transmitting. Pass: no latch-up, no permanent damage, bus recovers by itself (error counters may increment during the event). Known risk: the 19-26 mm unclamped trace between J3/J4 and D3 (phase 7 R-1b). Do not do this on a board you need.

## 12. Sign-off table (record per board: serial ______ , date ______ , tester ______ )

| # | Test | Criterion | Measured | Pass | Fail |
|---|---|---|---|---|---|
| 0 | Visual / AOI + polarity | step 0 all ticked | | [ ] | [ ] |
| 1a | VIN - GND, +5V - GND, VIO - GND | no shorts (>10 k / >100 k) | | [ ] | [ ] |
| 1b | CANH - CANL, JP1 on | 120 ohm +-2 % | | [ ] | [ ] |
| 1c | CANH - CANL, JP1 off | > 1 Mohm | | [ ] | [ ] |
| 2 | First power, 9 V / 100 mA limit | idle 1-10 mA, LED on | | [ ] | [ ] |
| 3a | +5V (TP2) | 4.95-5.05 V | | [ ] | [ ] |
| 3b | +3V3 (TP3), VIN (TP1) | 3.23-3.37 V; 8.5-8.7 V | | [ ] | [ ] |
| 3c | +5V ripple idle / loaded | < 50 mV p-p | | [ ] | [ ] |
| 4 | Reverse polarity | < 1 mA, no damage | | [ ] | [ ] |
| 5 | 18-20 V (and 24 V optional) | runs to 18 V; PTC trips at 24 V | | [ ] | [ ] |
| 6a | SPI reset + CANSTAT | 0x80 | | [ ] | [ ] |
| 6b | CANCTRL / CNF read-back | 0x87; written values | | [ ] | [ ] |
| 7 | CLKOUT frequency | 2.000 MHz +-100 ppm | | [ ] | [ ] |
| 8 | Loopback | RX frame identical, CANSTAT 0x40 | | [ ] | [ ] |
| 9a | Two-node 125 k | >= 1000 frames, 0 errors | | [ ] | [ ] |
| 9b | Two-node 500 k | >= 1000 frames, 0 errors | | [ ] | [ ] |
| 9c | Two-node 1 M | >= 1000 frames, 0 errors | | [ ] | [ ] |
| 9d | VOD dominant | 1.5-3.0 V | | [ ] | [ ] |
| 9e | Termination effect | single/none worse than both | | [ ] | [ ] |
| 10 | STB | ~0 V; standby stops RX | | [ ] | [ ] |
| 11 | ESD (optional) | no damage | | [ ] | [ ] |

Release rule: tests 0-4, 6-9 pass on the first 3 boards before building more; any failure -> note the symptom, power off and see the debug hints in each section.

## Appendix A - Minimal Arduino sketch (autowp/arduino-mcp2515, 16 MHz crystal)
Library: https://github.com/autowp/arduino-mcp2515 (Install "mcp2515" by autowp). Uno/Nano wiring as in connector_pinouts.md, CS = D10.

```cpp
#include <SPI.h>
#include <mcp2515.h>

MCP2515 mcp2515(10);                  // CS = D10, default 10 MHz SPI

void setup() {
  Serial.begin(115200);
  SPI.begin();
  mcp2515.reset();                    // SPI RESET 0xC0
  mcp2515.setBitrate(CAN_500KBPS, MCP_16MHZ);   // 16 MHz crystal; CAN_125KBPS / CAN_250KBPS / CAN_1000KBPS also valid
  // mcp2515.setLoopbackMode();       // test 8 (no bus needed)
  mcp2515.setNormalMode();
}

void loop() {
  struct can_frame tx;
  tx.can_id = 0x123; tx.can_dlc = 2; tx.data[0] = 0xAA; tx.data[1] = 0x55;
  mcp2515.sendMessage(&tx);
  struct can_frame rx;
  if (mcp2515.readMessage(&rx) == MCP2515::ERROR_OK) {
    Serial.print(rx.can_id, HEX); Serial.print(' ');
    for (int i = 0; i < rx.can_dlc; i++) { Serial.print(rx.data[i], HEX); Serial.print(' '); }
    Serial.println();
  }
  delay(100);
}
```
Alternative with mcp_can (coryjfowler): `MCP_CAN CAN0(10); CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_16MHZ);` then `CAN0.setMode(MCP_NORMAL);` and `CAN0.sendMsgBuf(0x123, 0, 2, data);`. Neither library disables CLKOUT for you; use the raw `CANCTRL` bit modify of Appendix C if you want TP6 quiet.

## Appendix B - Minimal ESP32 sketch (VSPI: SCK 18, MISO 19, MOSI 23, CS 5)
```cpp
#include <SPI.h>
#include <mcp2515.h>

SPIClass vspi(VSPI);
MCP2515 mcp2515(5, 10000000, &vspi);  // CS = GPIO5, 10 MHz SPI (use 4 MHz for first tests)

void setup() {
  Serial.begin(115200);
  vspi.begin(18, 19, 23, 5);          // SCK, MISO, MOSI, SS
  mcp2515.reset();
  mcp2515.setBitrate(CAN_500KBPS, MCP_16MHZ);
  mcp2515.setNormalMode();            // or setLoopbackMode() for test 8
}

void loop() {
  struct can_frame tx;
  tx.can_id = 0x123; tx.can_dlc = 2; tx.data[0] = 0xAA; tx.data[1] = 0x55;
  mcp2515.sendMessage(&tx);
  struct can_frame rx;
  if (mcp2515.readMessage(&rx) == MCP2515::ERROR_OK) Serial.printf("RX %03X dlc=%d\n", rx.can_id, rx.can_dlc);
  delay(100);
}
```
STM32 (Arduino core, Blue Pill): use the same sketch with `SPI.begin()` (SPI1: PA5/PA6/PA7) and `MCP2515 mcp2515(PA4);`.

## Appendix C - Raw register sequence (any MCU), bit timing from phase 4
```
RESET:        0xC0                          then delay >= 10 ms
Read CANSTAT: 0x03 0x0E -> 0x80             (Configuration mode)
Write CNF3,CNF2,CNF1 (0x28..0x2A): 0x02 0x28 <CNF3> <CNF2> <CNF1>   e.g. 500k: 0x01 0xAE 0x00
Set mode:     BIT MODIFY 0x05 0x0F 0xE0 <mode>   (0x00 normal, 0x40 loopback, 0x60 listen-only)
Disable CLKOUT: BIT MODIFY 0x05 0x0F 0x04 0x00
Keep CS high within 47 us after TXREQ / RTS (errata #5).
```
Library presets differ from the phase 4 table in the segment split (valid timing, different sample point): mixed networks should use the same values on every node.
