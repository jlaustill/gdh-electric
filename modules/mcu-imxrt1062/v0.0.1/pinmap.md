# mcu-imxrt1062 — Pin & Castellation Map (v0.0.1)

**Part:** NXP MIMXRT1062CVJ5B — 196-MAPBGA (12×12 mm), Cortex-M7 @ 528 MHz, industrial −40/+105 °C (high-temp bin). Same ball-out as the PJRC Teensy 4.x.
**Datasheet:** [`datasheets/MIMXRT1062CVJ5B_C3216692.pdf`](datasheets/MIMXRT1062CVJ5B_C3216692.pdf) (NXP IMXRT1060IEC, industrial).
**Role:** rugged, high-temp, **Teensy-4.1-ecosystem-compatible** general-purpose brain. Spec: `docs/superpowers/specs/2026-07-23-mcu-imxrt1062-rugged-teensy-design.md`.

## Naming philosophy — Teensy numbers **and** raw pads (both)
Unlike `mcu-stm32h725` (raw pin names only), this module's value is Teensy compatibility, so each I/O castellation is identified by its **Teensy pin number** (what `digitalWrite(13)` / `Serial2` / `CAN1` expect). The native **RT1062 pad** and **GPIO port.bit** are documented alongside for register-level transparency — which is what makes the 16-bit parallel port usable. Best of both: ecosystem on top, silicon underneath.

## Locked decisions
| Topic | Decision |
|---|---|
| Form factor | 54 castellations, **27/side**, 2.54 mm pitch (reused `mcu-stm32h725` outline) |
| Pin identity | Teensy 4.1 pin numbers 0–41 + raw RT1062 pad documented |
| USB1 (program) | D− = `USB_OTG1_DN` (ball M8), D+ = `USB_OTG1_DP` (ball L8); on-module USB-C **and** castellated (VBUS-detect mux) |
| Power | clean **5 V** in (VBUS = VIN, same net); OR-ing on-module; 3.3 V generated on-module + exposed |
| Reset | `POR_B` (ball M7), active-low |
| Program | bootloader-chip (MKL02) program-request net — NOT an RT1062 GPIO (see Table 3) |
| Clocks | 24 MHz on `XTALI/XTALO` (P11/N11); 32.768 kHz RTC on `RTC_XTALI/O` (N9/P9) |
| Signature feature | complete 16-bit single-store parallel port on **GPIO6[31:16] = GPIO_AD_B1_00..15**, placed contiguous + ground-flanked on Side A |

## Castellation budget (54)
`42 Teensy I/O (0–41) + 2 USB1 (D±) + 2 Program/Reset + 2 power (5V, 3V3) + 6 GND` = 54.
The 6 GND include **2 "extra" GND** (Cast 7 & 24) flanking the 16-bit parallel window for SSO/ground-bounce return.

## Castellation map

**Side A (Cast 1–27) — USB-C end; carries the 16-bit parallel bus:**

| Cast | Class | Teensy | RT1062 pad | GPIO | Ball | Function / notes |
|---|---|---|---|---|---|---|
| 1 | PWR | | | | | **5V / VIN** (= USB VBUS net) |
| 2 | GND | | | | | GND |
| 3 | RESET | | POR_B | | M7 | active-low reset (→ GND to reset) |
| 4 | PROG | | | | | bootloader program-request net (→ GND for program mode) |
| 5 | USB1 | | USB_OTG1_DN | | M8 | USB1 D− (90 Ω pair) |
| 6 | USB1 | | USB_OTG1_DP | | L8 | USB1 D+ (90 Ω pair) |
| 7 | GND | | | | | **extra GND** (parallel-bus return) |
| 8 | IO | 19 | GPIO_AD_B1_00 | GPIO6.16 | J11 | bus b0 · A5 · Wire SCL0 |
| 9 | IO | 18 | GPIO_AD_B1_01 | GPIO6.17 | K11 | bus b1 · A4 · Wire SDA0 |
| 10 | IO | 14 | GPIO_AD_B1_02 | GPIO6.18 | L11 | bus b2 · A0 · Serial3 TX3 |
| 11 | IO | 15 | GPIO_AD_B1_03 | GPIO6.19 | M12 | bus b3 · A1 · Serial3 RX3 |
| 12 | IO | 40 | GPIO_AD_B1_04 | GPIO6.20 | L12 | bus b4 · A16 |
| 13 | IO | 41 | GPIO_AD_B1_05 | GPIO6.21 | K12 | bus b5 · A17 |
| 14 | IO | 17 | GPIO_AD_B1_06 | GPIO6.22 | J12 | bus b6 · A3 · Wire1 SDA1 · Serial4 TX4 |
| 15 | IO | 16 | GPIO_AD_B1_07 | GPIO6.23 | K10 | bus b7 · A2 · Wire1 SCL1 · Serial4 RX4 |
| 16 | IO | 22 | GPIO_AD_B1_08 | GPIO6.24 | H13 | bus b8 · A8 · **CAN1 TX** |
| 17 | IO | 23 | GPIO_AD_B1_09 | GPIO6.25 | M13 | bus b9 · A9 · **CAN1 RX** |
| 18 | IO | 20 | GPIO_AD_B1_10 | GPIO6.26 | L13 | bus b10 · A6 · Serial5 TX5 |
| 19 | IO | 21 | GPIO_AD_B1_11 | GPIO6.27 | J13 | bus b11 · A7 · Serial5 RX5 |
| 20 | IO | 38 | GPIO_AD_B1_12 | GPIO6.28 | H12 | bus b12 · A14 |
| 21 | IO | 39 | GPIO_AD_B1_13 | GPIO6.29 | H11 | bus b13 · A15 |
| 22 | IO | 26 | GPIO_AD_B1_14 | GPIO6.30 | G12 | bus b14 · A12 · SPI1 MOSI1 |
| 23 | IO | 27 | GPIO_AD_B1_15 | GPIO6.31 | J14 | bus b15 · A13 · SPI1 SCK1 |
| 24 | GND | | | | | **extra GND** (parallel-bus return) |
| 25 | PWR | | | | | **3V3** (module-generated, out) |
| 26 | IO | 24 | GPIO_AD_B0_12 | GPIO6.12 | K14 | A10 · Serial6 TX6 · Wire2 SCL2 |
| 27 | IO | 25 | GPIO_AD_B0_13 | GPIO6.13 | L14 | A11 · Serial6 RX6 · Wire2 SDA2 |

**Side B (Cast 28–54) — digital / CAN / SPI side:**

| Cast | Class | Teensy | RT1062 pad | GPIO | Ball | Function / notes |
|---|---|---|---|---|---|---|
| 28 | GND | | | | | GND |
| 29 | IO | 0 | GPIO_AD_B0_03 | GPIO6.3 | G11 | Serial1 RX1 · **CAN2 RX** |
| 30 | IO | 1 | GPIO_AD_B0_02 | GPIO6.2 | M11 | Serial1 TX1 · **CAN2 TX** |
| 31 | IO | 2 | GPIO_EMC_04 | GPIO9.4 | F2 | PWM |
| 32 | IO | 3 | GPIO_EMC_05 | GPIO9.5 | G5 | PWM |
| 33 | IO | 4 | GPIO_EMC_06 | GPIO9.6 | H5 | PWM |
| 34 | IO | 5 | GPIO_EMC_08 | GPIO9.8 | H3 | PWM |
| 35 | GND | | | | | GND |
| 36 | IO | 6 | GPIO_B0_10 | GPIO7.10 | D9 | PWM · FlexIO |
| 37 | IO | 7 | GPIO_B1_01 | GPIO7.17 | B11 | Serial2 RX2 · FlexIO |
| 38 | IO | 8 | GPIO_B1_00 | GPIO7.16 | A11 | Serial2 TX2 · FlexIO |
| 39 | IO | 9 | GPIO_B0_11 | GPIO7.11 | A10 | PWM · FlexIO |
| 40 | IO | 10 | GPIO_B0_00 | GPIO7.0 | D7 | SPI CS0 |
| 41 | IO | 11 | GPIO_B0_02 | GPIO7.2 | E8 | SPI MOSI |
| 42 | IO | 12 | GPIO_B0_01 | GPIO7.1 | E7 | SPI MISO |
| 43 | IO | 13 | GPIO_B0_03 | GPIO7.3 | D8 | SPI SCK · onboard LED |
| 44 | GND | | | | | GND |
| 45 | IO | 28 | GPIO_EMC_32 | GPIO8.18 | D5 | Serial7 RX7 |
| 46 | IO | 29 | GPIO_EMC_31 | GPIO9.31 | C5 | Serial7 TX7 |
| 47 | IO | 30 | GPIO_EMC_37 | GPIO8.23 | E4 | **CAN3 RX (CAN-FD)** |
| 48 | IO | 31 | GPIO_EMC_36 | GPIO8.22 | C3 | **CAN3 TX (CAN-FD)** |
| 49 | IO | 32 | GPIO_B0_12 | GPIO7.12 | C10 | FlexIO · PWM |
| 50 | IO | 33 | GPIO_EMC_07 | GPIO9.7 | H4 | PWM |
| 51 | IO | 34 | GPIO_B1_13 | GPIO7.29 | D14 | Serial8 RX8 · SPI2 MISO |
| 52 | IO | 35 | GPIO_B1_12 | GPIO7.28 | D13 | Serial8 TX8 · SPI2 MOSI |
| 53 | IO | 36 | GPIO_B1_02 | GPIO7.18 | C11 | SPI2 CS · FlexIO |
| 54 | IO | 37 | GPIO_B1_03 | GPIO7.19 | D11 | SPI2 SCK · FlexIO |

## 16-bit parallel port — GPIO6[31:16] (`GPIO_AD_B1_00..15`)
Cast **8–23** carry the full window, contiguous and in **bit order 0→15** (Cast 8 = bit16/LSB of the halfword … Cast 23 = bit31/MSB), flanked by GND at Cast 7 & 24.

- **Write:** store to the high halfword of `GPIO6_DR`. **Read:** `(GPIO6_DR >> 16) & 0xFFFF`.
- **Trade (mutually exclusive with the bus while active):** CAN1 (22/23), I2C0 (18/19), I2C1 (16/17), Serial3 (14/15), Serial4 (16/17), Serial5 (20/21), SPI1 (26/27), and analog A0–A9 / A12–A17.
- **Survives:** CAN2 (0/1), CAN3/CAN-FD (30/31), primary SPI (10–13), Wire2 (24/25), Serial1/2/6/7/8, and analog A10/A11 (pins 24/25 are in GPIO6[15:0], outside the window).

## CAN reference (Teensy fixed muxing)
| Bus | RX pin | TX pin | Castellations |
|---|---|---|---|
| CAN2 | 0 | 1 | 29 / 30 |
| CAN1 | 23 | 22 | 17 / 16 |
| CAN3 (CAN-FD) | 30 | 31 | 47 / 48 |

## Supply / special balls (for schematic — Plan 2)
| Signal | Ball(s) | Notes |
|---|---|---|
| USB_OTG1_DP / DN | L8 / M8 | USB1 programming port |
| USB_OTG1_VBUS | N6 | VBUS sense |
| USB_OTG2_DP / DN | P7 / N7 | USB2 (not broken out this rev) |
| POR_B | M7 | reset input, 100 K PU |
| ONOFF | M6 | SNVS on/off (not reset) |
| TEST_MODE | K6 | tie GND |
| VDD_SOC_IN | F6 F7 F8 F9 G6 G9 H6 H9 J9 | core (DCDC out) |
| DCDC_IN / _IN_Q | L1 L2 / K4 | DCDC input (VDD_HIGH) |
| DCDC_LP | M1 M2 | DCDC switch node → L1 inductor |
| DCDC_GND | N1 N2 | DCDC power ground |
| DCDC_PSWITCH | K3 | RC to VDD_SNVS on startup |
| DCDC_SENSE | J5 | Kelvin to VDD_SOC_IN |
| VDD_HIGH_IN | P12 | 3.3 V main |
| VDD_HIGH_CAP | P8 | decap only |
| VDD_SNVS_IN | M9 | RTC/SNVS always-on (coin cell) |
| VDD_SNVS_CAP / VDD_USB_CAP | M10 / K8 | decap only |
| NVCC_GPIO | E9 F10 J10 | 3.3 V I/O (AD_B0/AD_B1/B0/B1) |
| NVCC_EMC | E6 F5 | 3.3 V I/O (EMC) |
| NVCC_SD0 / SD1 | J6 / K5 | 3.3 V I/O (SD_B0/SD_B1) |
| NVCC_PLL | P10 | decap (1.1 V) |
| VDDA_ADC_3P3 | N14 | ferrite/filter from 3.3 V |
| GPANAIO | N10 | leave unconnected |
| RTC_XTALI / O | N9 / P9 | 32.768 kHz |
| XTALI / O | P11 / N11 | 24 MHz |
| VSS (grounds) | A1 A14 B5 B10 E2 E13 G7 G8 H7 H8 J7 J8 K2 K13 L9 N5 N8 P1 P14 | to ground plane |

## Program / Reset semantics (Table 3)
The RT1062 has **no user "program pin"** — programming is mediated by the on-board **MKL02Z32 bootloader chip**. The Program button grounds the MKL02's program-request input, which reboots the RT1062 into its ROM bootloader.
- **PROG castellation (Cast 4)** → the bootloader-chip program-request net (same net the on-board Program button pulls low). Momentary-to-GND enters bootloader. **Not** an RT1062 GPIO, **not** POR_B.
- **RESET castellation (Cast 3)** → `POR_B` (ball M7), active-low. Momentary-to-GND resets the RT1062.
- Both are carrier-accessible so a sealed module stays programmable/recoverable.

## Provisional edge order — tweakable in layout (Plan 3)
The Side-A / Side-B split above is the recommended order; the bus-contiguous placement (Cast 8–23) and its GND flanks (7, 24) are the one part worth holding fixed — it's what makes the 16-bit port routable as a single ribbon off one edge.
