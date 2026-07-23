# mcu-imxrt1062 — Rugged High-Temp Teensy-Compatible Brain Module

**Status:** Design (approved 2026-07-23)
**Replaces:** `mcu-stm32h725` as GDH's general-purpose brain module
**License:** CC0 1.0 (public domain, per GDH default)

## Purpose

A production-ready, ruggedized, high-temperature module that is **binary-compatible
with the PJRC Teensy 4.x software ecosystem** (Teensyduino / Teensy Loader) while
being built for GDH's world: sealed, high-vibration, wide-temperature, CAN-centric
deployments.

The motivating insight: the pleasant part of "using a Teensy" is **PJRC's ecosystem**
(bootloader, core library, community), not the i.MXRT silicon itself. So GDH's
value-add is **not** novel circuitry — it is ruggedness and form-factor around a
proven, blessed design. PJRC explicitly supports this: they sell a pre-programmed
bootloader chip for exactly this purpose. There is **zero from-scratch firmware** to
write; the module boots and programs like a Teensy.

This is GDH doctrine in its purest form: ruggedize a proven open design; decide
on-module vs carrier by *"will a future integrator be forced to supply it, or is it
silently omittable?"*

## Why this over the STM32H725

The H725 path required building the entire software stack from scratch and living in
the STM32 tooling ecosystem, which is the specific friction we are escaping. The
Teensy path inherits a mature, loved ecosystem for free. The H725 module effort is
**superseded**, not extended. The physical module outline (54-castellation footprint)
is reused.

## Core Decisions

### MCU
- **NXP MIMXRT1062, industrial "C" grade** — −40/+105 °C, 528 MHz. Same silicon as the
  Teensy 4.x consumer "D" grade (600 MHz, 0/+95 °C), binned for temperature. Pinout
  identical to Teensy 4.x. Neither grade is AEC-Q100 automotive-qualified; this is an
  industrial-grade Cortex-M7, consistent with the temp-grade-vs-AEC-Q100 doctrine.
- **No on-die flash** — all code lives in external QSPI (see Memory).

### Ecosystem / bootloader
- **PJRC MKL02Z32 bootloader chip** (`IC_MKL02Z32_QFN16`), pre-programmed, purchased
  from PJRC (verify current source — PJRC vs SparkFun distribution — at BOM time).
- This is the **one part with no LCSC code.** It is handled via **PCBWay's
  mail-in / customer-supplied-component assembly flow** (previously used for the
  MicroMod standoffs on the translator boards). Adds a few weeks; no technical blocker.
- Result: fully Teensy Loader / Teensyduino compatible.

### Form factor
- **Castellated SMT module**, **54 castellations**, GDH-defined pinout.
- Same physical outline as `mcu-stm32h725 v0.0.1` (27 castellations per long edge × 2 =
  54; i.e. 3 pitches longer per edge than a Teensy 4.1's 24/edge → **6 extra
  castellations** vs a Teensy-4.1 edge count).
- Reflows flat onto a carrier → **B.Cu must stay component-free** (all parts top-side;
  B.Cu routing is fine). Castellation pads marked `(property pad_prop_castellated)`.

### Memory
- **16 MB QSPI flash** on FlexSPI1 + **8 MB PSRAM** on FlexSPI2, both **populated**.
- Anti-landfill / not-MVP: bake in the capability rather than force a future integrator
  to discover they are short on room.
- Ecosystem-compatible: extra flash beyond the stock 8 MB is exposed as a **LittleFS**
  filesystem; PSRAM is the standard Teensy 4.1 `EXTMEM` (APS6404L-class part).

### USB1 — device / programming port (the value prop)
USB1 is the device/programming interface and is exposed at **two physical access
points for the same logical port**, so the module is **always reprogrammable** — on a
bench bare, or sealed inside a carrier:
- **On-module USB-C** (bench / breadboard).
- **Castellated D+/D−** (a carrier routes USB1 out to its own connector when the module
  is potted/enclosed).
- Only one side is ever connected at a time.

Implementation requirements (all three):
1. **USB-C placed hard against the castellation breakout** to minimize both stub
   lengths on the 480 Mbps High-Speed differential pair (avoid the T-junction stub that
   wrecks HS signal integrity).
2. **VBUS-detect USB switch** to electrically select whichever side is live (keys off
   the on-module USB-C VBUS presence, which is distinct from the always-present 5V rail).
3. **Port protection** — ESD/TVS array on D+/D−, over-voltage protection on VBUS
   (these lines leave the board into the real world).

### USB2
- **Not broken out** in this revision. The second USB controller is available in
  silicon but its castellations were reallocated (see Pinout). A future revision could
  expose it as a carrier host port.

### Power — module is voltage-agnostic (clean 5 V in)
- The module accepts **clean 5 V** and does **not** commit to any input voltage domain.
  Input voltage is entirely a deployment detail, so a module-baked wide-input converter
  would presume a use-case the module cannot know.
- Two 5 V sources, one at a time, always available:
  - **USB-C VBUS** (bench).
  - **Carrier 5 V** via castellation. Because VIN is clean 5 V, **VBUS and VIN are the
    same net** — VBUS costs no extra castellation.
- On-module **ideal-diode / priority OR-ing controller** between the two sources (never
  back-feed), giving **reverse-polarity protection** on the input leg for free — the
  same "two sources, one load, never fight" design as the VBUS-detect USB switch.
- The module generates **3.3 V + the RT1062 core/DDR rails internally** — these are
  silently-omittable essentials and are non-negotiably on-module regardless of input.
- **Wide-input conversion (12 V / 24 V / 48 V), reverse-polarity, and load-dump
  survival are the CARRIER's responsibility**, sized to that deployment. Trade
  accepted: every rugged carrier re-solves 5 V conversion — correct, because each
  carrier's power needs genuinely differ.

### Ruggedness split (summary)
- **Module owns:** high-temp silicon, sealed castellated form, USB port protection,
  reverse-polarity on the 5 V input, internal rail generation.
- **Carrier owns:** the power domain (wide-input buck, load-dump/transient protection),
  and any application peripherals (Ethernet PHY, sensors, etc.).

## Pinout (54 castellations)

Budget (exact assignment is a layout deliverable — see Open Items):
- **Teensy-4.1-equivalent I/O**, pins **0–41** (42 signals) — on the 4.1, pins 33–41
  are edge pins (not bottom pads; that is a Teensy 4.0 trait), so a 4.1-equivalent set
  naturally includes 38–41.
- **Program + Reset** (2) — exposed on castellations so a **carrier can trigger
  programming/reset when the module is sealed** and the onboard button is unreachable.
  This closes the "always reprogrammable when sealed" loop: carrier-side USB1 can
  reflash a *running* board; Program/Reset lets it recover a *halted/bricked* one.
- **USB1 D+/D−** (2).
- **Power / ground** (~8), including **2× extra GND** castellations (see below).

### Feature: free 16-bit single-store parallel port
- **`GPIO6[31:16]` — the `GPIO_AD_B1_00..15` pad group — is a complete, contiguous
  16-bit window**, and on a Teensy-4.1 I/O set **all 16 bits are already broken out**:
  pins **14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 26, 27, 38, 39, 40, 41**.
- Single-store access: write the high halfword of `GPIO6_DR`; read `GPIO6_DR >> 16`.
- **Costs 0 extra castellations.** This fixes a long-standing Teensy 4.x gap (no
  complete contiguous 16-bit port broken out for true parallel I/O).
- **The memory config protects it rather than harming it:** flash (FlexSPI1) sits on
  GPIO9/`SD_B1` and PSRAM (FlexSPI2) on GPIO8/`EMC`, which is exactly why those ports
  are dead for a parallel window — but `AD_B1` is unrelated to FlexSPI, so GPIO6 is
  pristine. GPIO7 is too fragmented (~8–10 pads short).

### Trade: parallel-mode pin overlap
Driving `GPIO6[31:16]` as a bus consumes the default functions on those pins while
active (the pads remain ordinary fast GPIO, so they are fully usable *as* the bus):
- **CAN1 (22/23)** — parallel-mode and CAN1 are mutually exclusive. **CAN2 (0/1) and
  CAN3 (30/31, CAN-FD) remain available**, so two CAN controllers (incl. FD) survive
  even while the bus runs.
- I2C0 (18/19), I2C1 (16/17), Serial3 (14/15), analog A0–A9 and A12–A17.
No hard conflicts — USB, flash, and PSRAM do not intrude on `AD_B1`.

### Reclaimed pins → 2× GND
The 2 castellations originally earmarked to "complete" the 16-bit port are freed (the
port is already complete). They are assigned to **extra GND**: a 16-bit port switches
16 outputs simultaneously, which is the classic source of **ground bounce (SSO
noise)** — a stout ground return is the single best thing for making the feature clean
at speed. The reclaimed pins and the parallel bus are the same design story.

### Not on castellations
- **USB2**, **Ethernet RMII**, **SDIO** — each needs more signals than are available,
  and they are use-case-specific. A carrier adds an Ethernet PHY (over SPI/RMII) or SD
  as needed. On a stock Teensy 4.1 these live off the primary pin rows anyway.

## Fabrication / Assembly
- **Fab:** PCBWay (confirm against the JLCPCB note carried over in the old
  `mcu-stm32h725` CLAUDE.md — this module uses PCBWay).
- **Assembly:** standard SMT for all LCSC-sourceable parts; **mail-in / customer-supplied**
  for the PJRC MKL02Z32 bootloader chip.

## Open Items (for the plan / layout phase)
1. **Exact 54-pin castellation assignment** — full RT1062-pad → Teensy-pin → castellation
   map, guaranteeing pins 0–41 (incl. the 16 parallel-window pins), Program, Reset,
   USB1 D+/D−, power, and 2× extra GND all fit.
2. **Part selection:** VBUS-detect USB switch; ideal-diode/priority OR-ing controller;
   ESD/TVS arrays; 16 MB flash; 8 MB PSRAM; 3.3 V + core regulation matching the RT1062
   power sequence.
3. **USB1 stub-mitigation layout** — USB-C hard against the castellation breakout;
   HS pair routing.
4. **Confirm fab = PCBWay** (vs the inherited JLCPCB note).
5. **Confirm bootloader-chip current source** — PJRC vs SparkFun distribution.
6. **Module name / version** — proposed `modules/mcu-imxrt1062/v0.0.1/`.
