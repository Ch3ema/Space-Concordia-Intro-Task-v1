# Space Concordia Robotics — Electrical Intro Task

CAN bus node built in KiCad. An STM32G030 microcontroller toggles an LED, communicating with an MCP25625 CAN controller/transceiver over SPI.

## Overview

The board takes 5V, GND, CANH and CANL from a 4-pin connector and implements a complete CAN node with power regulation, decoupling, bus termination and surge protection.

<img width="300"  alt="image" src="https://github.com/user-attachments/assets/c9766e0a-2c51-46aa-9c16-4e3cf3652109" />


## Components

| Ref | Part | Function |
|---|---|---|
| U1 | MCP25625 | CAN controller + transceiver |
| U2 | STM32G030F6Px | Microcontroller |
| U3 | AP2127-3.3 | 3.3V LDO |
| D1 | LED | Output indicator |
| D2 | PESD1CAN | Dual bidirectional TVS |
| J1 | 4-pin connector | 5V, GND, CANH, CANL |

## Design decisions

**Two power rails.** VDDA is tied to 5V because the CAN transceiver needs the headroom to drive the differential swing. The STM32 runs at 2.0–3.6V, so an AP2127 LDO generates 3.3V on-board. MCP VDD and VIO sit on 3.3V to match the MCU, which removes the need for level shifters on SPI.

**LDO over a switching regulator.** Total load is around 25 mA, so dissipation is roughly 42 mW. Not worth an inductor and switching noise next to a CAN transceiver.

**Decoupling.** 100 nF X7R on every supply pin, plus 1 µF at the LDO input and output per its datasheet.

**Bus termination.** 120 Ω across CANH/CANL, assuming this node sits at an end of the bus. Should be DNP if used mid-bus.

**Surge protection.** PESD1CAN dual TVS at the connector. Bidirectional, 24V standoff, 11 pF so it doesn't load the differential pair.

## Pin assignments

Verified against STM32G030 datasheet Table 12.

| Function | STM32 | TSSOP20 pin | MCP25625 |
|---|---|---|---|
| SCK | PA5 | 12 | SCK (14) |
| MISO | PA6 | 13 | SO (16) |
| MOSI | PA7 | 14 | SI (15) |
| CS | PA1 | 8 | CS (17) |
| STBY | PA0 | 7 | STBY (5) |
| LED | PA2 | 9 | — |

## Known gaps

- No bulk capacitor at the 5V input. Worth adding 10 µF in a next revision since the supply arrives over cable.
- OSC pins left unconnected, per the task instructions.

## Files

- `/kicad` — schematic and project files
- `/docs` — design notes and datasheet references
