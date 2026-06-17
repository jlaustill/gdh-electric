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

## Castellation pinout — RAW pin names, no aliasing (design decision)

**Every castellation is silkscreened with its native STM32 pin name (`PA8`, `PB3`, …) — NOT Arduino-style sequential numbers and NOT a single fixed function.** The board exposes the silicon; the integrator reads the DS13311 alternate-function table and chooses what each pin does (UART/SPI/I2C/ADC/FDCAN/timer/…). Rationale: transparency + no abstraction lock-in — the Teensy frustration of "only CAN1 is labeled, reverse-engineer the rest from registers" is designed out here: *all* CAN-capable pins are exposed by name and the integrator picks. So functions are **not** pre-assigned (e.g. FDCAN3 = PB3/PB4 **or** PA8/PA15 — both exposed, user decides).

### 54-pad budget
44 GPIO (all bonded I/O except PH0/PH1 = HSE crystal) + 3×3V3 + 4×GND + 1×Vin + 1×NRST + 1×BOOT0.
GPIO: PA0–PA15 (16), PB0–PB10 + PB12–PB15 (15), PC0/PC1/PC4–PC7/PC9–PC12/PC14/PC15 (12), PD2 (1).

### FDCAN-capable pins (reference for integrators; NOT pre-wired)
| Bus | RX (bonded) | TX (bonded) |
|---|---|---|
| FDCAN1 | PB8 *(or PA11=USB)* | PB9 *(or PA12=USB)* |
| FDCAN2 | PB12, PB5 | PB13, PB6 |
| FDCAN3 | PB3, PA8 | PB4, PA15 |

## Fixed-function pins (from datasheet pin-definition + AF tables, DS13311)

- **USB-FS:** PA11 (D−), PA12 (D+)
- **SWD:** PA13 (SWDIO), PA14 (SWCLK)
- **HSE:** PH0 (OSC_IN), PH1 (OSC_OUT)
- **3× FDCAN:** FDCAN1 (4 pin-pair options), FDCAN2 (2), FDCAN3 (3) per the datasheet AF map — choose from the VFQFPN-68 pinout / AF table.

## Proposed edge order (27/side, PORT-grouped so any CAN pair lands adjacent — tweakable)

**Side A (USB-C end →):** `VIN GND PA0 PA1 PA2 PA3 PA4 PA5 PA6 PA7 3V3 GND PA8 PA9 PA10 PA11 PA12 PA13 PA14 PA15 3V3 GND PC0 PC1 PC4 PC5 NRST`

**Side B:** `BOOT0 GND PB0 PB1 PB2 PB3 PB4 PB5 PB6 PB7 PB8 PB9 PB10 3V3 PB12 PB13 PB14 PB15 PC6 PC7 PC9 PC10 PC11 PC12 PC14 PC15 PD2`

- PA11/PA12 also wire to the USB-C (D−/D+); PA13/PA14 = SWD — each still reaches its castellation (one signal, both places).
- Silk = raw pin name; the *net* may carry a functional name (USB_DM/DP, SWDIO/SWCLK) but the pad **label** is the pin name.
- Port-grouped means every FDCAN pair (PB8/9, PB12/13, PB3/4, PB5/6) is physically adjacent → clean transceiver/carrier routing.
- ADC-capable pins (PA0–PA7, PB0/PB1, PC0–PC5) sit on contiguous runs.

## Next steps

1. **Pin assignment by hand** from the datasheet pin-definition + alternate-function tables (DS13311): map the VFQFPN-68 bonded GPIO to functions + castellations. (No CubeMX.)
2. Schematic: power-path ORing (e.g. LM66100 ideal-diode OR, or TPS2116 priority mux) → 3.3 V regulator; USB-C (CC 5.1 kΩ ×2, VBUS sense, D± ESD); decoupling per ST AN; HSE crystal; BOOT0/NRST strapping.
3. Castellation footprint + board outline; then EasyEDA deliverables (BOM, PnP, Schematic.svg/pdf, Gerber).
