# Space Concordia Robotics — Electrical Intro Task

CAN bus node built in KiCad. An STM32G030 microcontroller toggles an LED, communicating with an MCP25625 CAN controller/transceiver over SPI.

## Overview

The board takes 5V, GND, CANH and CANL from a 4-pin connector and implements a complete CAN node with power regulation, decoupling, bus termination and surge protection.

### v2 (current)

<img width="300"  alt="image" src="https://github.com/user-attachments/assets/5f7144a8-4e8d-4801-be72-d1532ff0eceb" />


### v1

<img width="300" alt="v1 schematic" src="https://github.com/user-attachments/assets/c9766e0a-2c51-46aa-9c16-4e3cf3652109" />

## Changes in v2

Reviewed by the division's Electrical Department Manager. Changes made in response:

- **Net labels throughout** instead of long wire runs, with the schematic split into labelled sections (power, CAN, SWD, decoupling).
- **Edited the MCP25625 library symbol** so power pins sit on the top edge and grounds on the bottom. The stock KiCad symbol had them reversed, which made the sheet read inconsistently against the STM32.
- **NRST circuit.** Added a momentary switch from NRST to GND. C7 now does real work — debouncing the switch and setting the reset pulse width via the internal pull-up ($\tau = 40\text{k} \times 100\text{nF} \approx 4$ ms).
- **SWD header (J2).** The board previously had no way to be programmed. Five pins: 3V3, SWDIO (PA13), SWCLK (PA14), NRST, GND. NRST is included so a programmer can connect under reset.
- **Fixed J1.** The connector pins were not actually wired to anything in v1.
- **LED moved to PA4.** PA2 has no timer channel; PA4 has TIM14_CH1 and was free since chip select is driven from PA1 in software. This allows hardware PWM for brightness control and blink patterns.

## Components

| Ref | Part | Function |
|---|---|---|
| U1 | MCP25625 | CAN controller + transceiver |
| U2 | STM32G030F6Px | Microcontroller |
| U3 | AP2127-3.3 | 3.3V LDO |
| D1 | LED | Output indicator |
| D2 | PESD1CAN | Dual bidirectional TVS |
| SW1 | Tactile switch | Manual reset |
| J1 | 4-pin connector | 5V, GND, CANH, CANL |
| J2 | 5-pin header | SWD programming/debug |

## Design decisions

**Two power rails.** VDDA is tied to 5V because the CAN transceiver needs the headroom to drive the differential swing. The STM32 runs at 2.0–3.6V, so an AP2127 LDO generates 3.3V on-board. MCP VDD and VIO sit on 3.3V to match the MCU, which removes the need for level shifters on SPI.

**LDO over a switching regulator.** Total load is around 25 mA, so dissipation is roughly 42 mW. Not worth an inductor and switching noise next to a CAN transceiver.

**Decoupling.** 100 nF X7R on every supply pin, plus 1 µF at the LDO input and output per its datasheet.

**Bus termination.** 120 Ω across CANH/CANL, assuming this node sits at an end of the bus. Should be DNP if used mid-bus.

**Surge protection.** PESD1CAN dual TVS at the connector. Bidirectional, 24V standoff, 11 pF so it doesn't load the differential pair.

**LED drive.** Sourcing, active high. The STM32G030 specifies both $V_{OL}$ and $V_{OH}$ at 0.4 V (Table 50), so sourcing and sinking give identical current (~4.1 mA at 220 Ω). The usual advice to sink LED current comes from older 5V parts with asymmetric drive and doesn't apply here. See `/docs` for the full analysis.

## Pin assignments

Verified against STM32G030 datasheet Table 12.

| Function | STM32 | TSSOP20 pin | MCP25625 |
|---|---|---|---|
| SCK | PA5 | 12 | SCK (14) |
| MISO | PA6 | 13 | SO (16) |
| MOSI | PA7 | 14 | SI (15) |
| CS | PA1 | 8 | CS (17) |
| STBY | PA0 | 7 | STBY (5) |
| LED | PA4 | 11 | — |
| SWDIO | PA13 | 18 | — |
| SWCLK | PA14 | 19 | — |

## Known gaps

- No bulk capacitor at the 5V input. Worth adding 10 µF in a next revision since the supply arrives over cable.
- OSC pins left unconnected, per the task instructions.
- D1 has no part number selected. Calculations assume a red LED at $V_f = 2$ V.

## Files

- `/kicad` — schematic and project files
- `/docs` — design notes, LED drive analysis, datasheet references
