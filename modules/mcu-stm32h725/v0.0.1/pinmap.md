# mcu-stm32h725 — Pin & Castellation Plan (v0.0.1, WIP)

**Part:** STM32H725RGV3 — VFQFPN-68, 8×8 mm, Cortex-M7 @ ≤550 MHz, 1 MB flash / 564 KB RAM, 3× FDCAN, −40/+125 °C ambient.
**Datasheet:** [`datasheets/STM32H725RGV3_C5271074.pdf`](datasheets/STM32H725RGV3_C5271074.pdf) (ST DS13311).
**Role:** universal, general-purpose, breadboard-compatible controller "brain" — *not* job-specific. Selection rationale: `docs/superpowers/specs/2026-06-16-mcu-selection-design.md`.

## Locked decisions

| Topic | Decision |
|---|---|
| Form factor | Teensy-4.1 width (~18 mm), ~0.3″ longer; **castellated edges** |
| Pitch | **2.54 mm** (breadboard-native — GDH value prop) |
| Pin count | **54 castellations, 27 per side** (Teensy 4.1 = 24/side; +3/side = +7.6 mm) |
| USB | **Full-Speed on USB-C**, native DFU (no programming chip). D− = **PA11**, D+ = **PA12** |
| Power | 5 V domain (USB-5V **OR** Vin 3.3–5 V), **power-path ORing** (no trace cuts), on-board **≥500 mA 3.3 V regulator** (vs Teensy ~250–350 mA — headroom for the M7 + distributed 3V3 taps). Topology (LDO vs buck) → schematic. Hostile/wide inputs handled upstream by `protect-12v`. |
| Debug/program | SWD broken out (SWDIO = **PA13**, SWCLK = **PA14**), plus **NRST** + **BOOT0**; field-reflash over USB-C (right-to-repair) |
| Clock | **HSE crystal on PH0/PH1** (OSC_IN/OSC_OUT) for clean CAN-FD bit timing (consumes those 2 pins; USB can run crystal-less on HSI48+CRS if ever needed) |

## Pin budget (68-pin package)

`68 total ≈ 46 GPIO + ~22 power/special` — the H7 is power-pin-hungry: multiple VDD/VSS, VCAP (core LDO), VDDA/VSSA/VREF+, VBAT, VDD33USB, VDDLDO, SMPS trio (VDDSMPS/VLXSMPS/VFBSMPS), PDR_ON, NRST, BOOT0.

## Proposed castellation allocation (54 pads)

| Group | Pads | Notes |
|---|---|---|
| GPIO (broken out) | **44** | all bonded GPIO except PH0/PH1 (crystal); includes USB D±, SWD, 3× CAN pairs |
| 3V3 + GND pairs | 6 | **3 distributed [3V3][GND] power pairs** (balanced taps along the edges) |
| Vin + GND | 2 | 5 V power-in pair (input + return) |
| NRST | 1 | |
| BOOT0 | 1 | DFU entry |
| **Total** | **54** | net: 3V3 ×3, GND ×4, Vin ×1, NRST ×1, BOOT0 ×1, GPIO ×44 |

*Crystal-less alternative: frees PH0/PH1 → 46 GPIO, but narrows CAN-FD timing margin. Not recommended for a 3×CAN board.*

## Fixed-function pins (from datasheet pin-definition + AF tables, DS13311)

- **USB-FS:** PA11 (D−), PA12 (D+)
- **SWD:** PA13 (SWDIO), PA14 (SWCLK)
- **HSE:** PH0 (OSC_IN), PH1 (OSC_OUT)
- **3× FDCAN:** FDCAN1 (4 pin-pair options), FDCAN2 (2), FDCAN3 (3) per the datasheet AF map — choose from the VFQFPN-68 pinout / AF table.

## Proposed edge arrangement (principles; exact pins from datasheet)

- **USB-C** connector on one short end; route **PA11/PA12** to it *and* to their castellations (one USB, both places).
- **Power** pads at predictable corners/ends: Vin, 3V3, GND.
- **Debug cluster** grouped (SWDIO, SWCLK, NRST, BOOT0, GND) so a standard pogo/header footprint hits them.
- **3× CAN** TX/RX pairs grouped, so each routes cleanly to a transceiver/carrier (e.g. `crd-mcp2562`).
- **3V3+GND pairs** distributed ~every 9 pads (balanced power taps); remaining GPIO fill, ADC-capable pins kept on a contiguous run where possible.

## Next steps

1. **Pin assignment by hand** from the datasheet pin-definition + alternate-function tables (DS13311): map the VFQFPN-68 bonded GPIO to functions + castellations. (No CubeMX.)
2. Schematic: power-path ORing (e.g. LM66100 ideal-diode OR, or TPS2116 priority mux) → 3.3 V regulator; USB-C (CC 5.1 kΩ ×2, VBUS sense, D± ESD); decoupling per ST AN; HSE crystal; BOOT0/NRST strapping.
3. Castellation footprint + board outline; then EasyEDA deliverables (BOM, PnP, Schematic.svg/pdf, Gerber).
