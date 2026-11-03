# Connector pinouts, wiring and test points - MCP2515 CAN module Rev A

Nets verified against `kicad/MCP2515_CAN/MCP2515_CAN.kicad_pcb` pad assignments (2026-11-03). Pin 1 of each THT part has the square pad.

## J1 - DC barrel jack (PJ-002AH, 5.5 x 2.1 mm), centre positive
| Pad | Net | Function |
|---|---|---|
| 1 | VIN_RAW | centre pin, **+9 V** (operating 7-16 V; protected, not operated, up to ~18 V) |
| 2 | GND | sleeve |
| 3 | - (unconnected) | switch contact, not used |

Centre-negative adapter: board stays off (D1 blocks; nothing is damaged). Silk: "9V + centre".

## J5 - alternate power terminal (KF128-5.08-2P) - **DNP by default**
| Pin | Net | Function |
|---|---|---|
| 1 | VIN_RAW | **+** (silk "+") |
| 2 | GND | - |

Same net as J1. Use *either* J1 or J5, never two supplies at once.

## J2 - host header (1 x 8, 2.54 mm)
| Pin | Net | Direction (module view) | Level | Notes |
|---|---|---|---|---|
| 1 | VIO | **input (power)** | 3.0-5.5 V | Host logic supply; powers U1 and U2-VIO (about 11 mA). Must be the same voltage as the host's logic |
| 2 | GND | - | 0 V | |
| 3 | SCK | in | VIO | 33 ohm series; <= 10 MHz, SPI mode 0,0 |
| 4 | MOSI (SI) | in | VIO | 33 ohm series |
| 5 | MISO (SO) | out | VIO | tri-state when CS high, 33 ohm at U1 |
| 6 | CS_N | in | VIO | 10 k pull-up to VIO |
| 7 | INT_N | out | VIO | **push-pull, active low**, 33 ohm; do not wire-OR |
| 8 | RST_N | in | VIO | 10 k pull-up + 10 nF; may be left open (power-on reset is on board) |

### Wiring: Arduino Uno / Nano (5 V logic, hardware SPI)
| J2 | Uno / Nano | Note |
|---|---|---|
| 1 VIO | 5V | |
| 2 GND | GND | |
| 3 SCK | D13 | |
| 4 MOSI | D11 | |
| 5 MISO | D12 | |
| 6 CS_N | D10 | any pin; library constructor takes it |
| 7 INT_N | D2 | INT0, optional (can poll) |
| 8 RST_N | not connected or D9 | optional |

### Wiring: ESP32 DevKit (3.3 V logic, VSPI)
| J2 | ESP32 DevKit | Note |
|---|---|---|
| 1 VIO | 3V3 | |
| 2 GND | GND | |
| 3 SCK | GPIO18 | VSPI CLK |
| 4 MOSI | GPIO23 | VSPI MOSI |
| 5 MISO | GPIO19 | VSPI MISO |
| 6 CS_N | GPIO5 | VSPI CS0 |
| 7 INT_N | GPIO4 | any input-capable GPIO (no pull-up needed) |
| 8 RST_N | not connected or GPIO16 | optional |

Do **not** power VIO from 5 V on an ESP32 - the J2 lines would then be 5 V into 3.3 V GPIOs.

### Wiring: STM32 Blue Pill (STM32F103) / Nucleo-64 (3.3 V logic, SPI1)
| J2 | Blue Pill | Nucleo-64 (Arduino header) | Note |
|---|---|---|---|
| 1 VIO | 3V3 | 3V3 | |
| 2 GND | GND | GND | |
| 3 SCK | PA5 (SPI1_SCK) | D13 / PA5 | |
| 4 MOSI | PA7 (SPI1_MOSI) | D11 / PA7 | |
| 5 MISO | PA6 (SPI1_MISO) | D12 / PA6 | |
| 6 CS_N | PA4 (GPIO, software CS) | D10 / PB6 or any GPIO | |
| 7 INT_N | PB0 | D2 / PA10 or any EXTI pin | |
| 8 RST_N | not connected or PB1 | optional | |

Nucleo variants route SPI1 on slightly different Arduino pins - check the board's user manual; PA5/PA6/PA7 is the common case.

## J3, J4 - CAN bus (KF128-5.08-3P), wired in parallel (daisy chain)
| Pin | Net | Function |
|---|---|---|
| 1 | CANH | |
| 2 | CANL | |
| 3 | GND | Signal ground reference (module is **non-isolated**; all nodes must share ground within the CAN common-mode range) |

Use J3 as "in" and J4 as "out" on a bus; both are electrically identical and sit behind D3.

## JP1 - bus termination (2 x 2, 2.54 mm) with two shunts
| Pin | Net | Pin | Net |
|---|---|---|---|
| 1 | CANH | 2 | TERM_H |
| 3 | CANL | 4 | TERM_L |

- **Both shunts fitted** (1-2 and 3-4, horizontal across each row) = 120 ohm split termination (2 x 60 ohm legs, 4.7 nF to GND at the centre).
- **Both removed** = no termination, both legs disconnected.
- Never fit just one shunt: that leaves 60 ohm + 4.7 nF hanging on one line.
- Fit it at the two physical ends of the bus only. Silk: "TERM 120R - fit both".

## SJ1 - 3.3 V to VIO solder jumper (default open)
**Close SJ1 only when no host is driving J2-VIO** (stand-alone bench test; the on-board XC6206 then powers U1/U2-VIO, about 12 mA max). With a host connected the host rail and the LDO would fight. Open again before connecting a host.

## Test points (all D1.5 mm pads)
| TP | Label | Net | Expected value |
|---|---|---|---|
| TP1 | VIN | VIN_PROT (after F1 + D1) | 8.5-8.7 V from a 9.0 V supply (F1 + SS34 drop ~0.3-0.5 V); tracks input minus ~0.4 V |
| TP2 | 5V | +5V | **4.95-5.05 V** (AP63205 fixed 5 V, +-1 %) |
| TP3 | 3V3 | +3V3 (U4 output) | 3.23-3.37 V (XC6206, +-2 %); live even with SJ1 open |
| TP4 | VIO | VIO | = host logic voltage (3.3 V or 5 V); 0 V with no host and SJ1 open; 3.3 V with SJ1 closed |
| TP5 | GND | GND | 0 V (scope-clip ground) |
| TP6 | CLK | CLKOUT (U1 pin 3) | after reset: **2.000 MHz** square wave (16 MHz / 8, CLKEN default on); high-Z/idle once firmware sets CLKEN = 0. Use a counter or a 10x probe (probe C shifts the crystal slightly) |
| TP7 | INT | INT_N (U1 side) | idles at VIO (high); goes low when an interrupt flag is enabled and set |
| TP8 | TXCAN | U1 pin 1 to U2 TXD | idles at VIO (recessive = high); pulses low while transmitting |
| TP9 | RXCAN | U2 RXD to U1 pin 2 | idles at VIO; follows the bus (low = dominant) |
| TP10 | CANH | bus CANH | recessive ~2.5 V; dominant ~3.5 V |
| TP11 | CANL | bus CANL | recessive ~2.5 V; dominant ~1.5 V |
| TP12 | STB | U2 pin 8 | ~0 V (10 k pull-down = Normal mode). Pulled high = standby, bus silent |
