# Controller "brain" — MCU Selection Decision

**Status:** Draft — MCU + package **locked** (§4: `STM32H725RGV3`); doctrine reconciliation + remaining board-design items **open** (§8). Not yet approved for layout.
**Date:** 2026-06-16
**Author:** Joshua Austill
**Repo target:** `modules/<name-TBD>/<version>/` — a planned general-purpose rugged controller ("brain") module that pairs with `crd-mcp2562` and other GDH modules. (Module name is open, §8 O2.)
**License:** CC0 1.0 (per GDH default)

> Records *which MCU the board is built around* and *why*: the field evaluated, the finding that decided it, and the consequences. The MCU choice (§4) is **locked**; everything in §8 is explicitly **not** decided.

---

## 1. Summary & role in the architecture

A general-purpose, rugged, **breadboard-compatible controller "brain"** — the GDH analog of a Teensy 4, built to GDH standards (−40/+125 °C, hand-solderable, anti-landfill). It is **not** application-specific: it is the reusable compute node other GDH modules attach to.

```
   [ protect-12v ] → 3.3V POL ──│              │── CAN-FD ──────── [ crd-mcp2562 ]
                                 │  controller  │── analog ──────── [ wideband-O2 (future) ]
   USB / SWD ── program/debug ──│   "brain"    │── I²C/SPI/UART ── carrier I/O
                                 │ (this board) │
```

**Firmware intent:** a **complete register-level definition written in C-Next**, generated/transcribed from the MCU's **CMSIS-SVD** + reference manual. This makes *SVD quality* a first-class selection criterion, not an afterthought — and motivates a reusable `SVD → C-Next` generator (§8 O6).

---

## 2. Requirement & ranked criteria

Target: a reusable Cortex-M brain. Criteria, ranked:

1. **ARM Cortex-M** with a high-quality, **publicly available CMSIS-SVD** (raw material for the C-Next register definition).
2. **Performance headroom** — Cortex-M7/M85 class preferred; ~150 MHz floor; generous internal flash + RAM.
3. **−40 to +125 °C ambient** operating range; long lifecycle; real multi-distributor stock (anti-landfill).
4. **CAN-FD** — 2–3 native instances (matches the `crd-mcp2562` MCP2562FD transceiver).
5. **Hand-friendly package** — LQFP/TQFP/QFN, **no BGA** (hot-plate / hand-solderable, castellated-module compatible).
6. **Rich, regular peripherals** — many timers, 16+ ADC channels, DAC, USB, optional Ethernet, multiple I²C/SPI/UART.

The AEC-Q100-vs-temperature question these criteria deliberately resolve in favor of the temperature *range* is recorded in §8 O1.

---

## 3. Field evaluated (11 parts, primary-source verified 2026-06-16)

**Tier A — general-purpose, hand-friendly Cortex-M**
| Part (best SKU) | Core / MHz | CAN-FD | AEC-Q100 / max Tₐ | Non-BGA | SVD |
|---|---|---|---|---|---|
| **STM32H725ZGT3** | M7 / 550 | **3** | ✗ industrial / **−40…+125 Tₐ** | LQFP-144 ✓ | ⭐ public pack |
| STM32G474VET3 | M4F / 170 | **3** | ✗ industrial / −40…+125 Tₐ | LQFP-100 ✓ | ⭐ public pack |
| SAMV71Q21B-AAB | M7 / 300 | 2 | **✓ Grade 2** / −40…+105 | LQFP-144 ✓ | ⭐ public pack |
| Renesas RA8M1 | M85 / 400 | 2 | ✗ industrial / +125 *Tⱼ* | LQFP-144 ✓ | ✓ (minor warnings) |
| Renesas RA6M5 | M33 / 200 | 2 | ✗ industrial / −40…+105 | LQFP-100 ✓ | ⭐ public pack |
| Infineon XMC4800 | M4F / 144 | **0 (classic only)** | ✗ industrial / −40…+125 Tₐ | LQFP-144 ✓ | ⭐ public pack |

**Tier B — automotive-native (controls)**
| Part | Core / MHz | CAN-FD | AEC-Q100 / max Tₐ | Non-BGA | SVD |
|---|---|---|---|---|---|
| NXP S32K344 | M7 lockstep / 160 | **6** | **✓ Grade 1** / −40…+125 Tₐ | 172-HDQFP ⚠ hard | ⚠ locked in S32DS |
| Infineon TRAVEO CYT4BF | 2×M7 / 350 | **~10** | **✓ Grade 1** / −40…+125 Tₐ | 176-TEQFP ⚠ hard | ⭐ (verified deep) |
| TI AM2634 | 4×R5F / 400 | 4 | ✓ -Q1 / +125 *Tⱼ* | **BGA only ✗** | ✗ (Cortex-R) |

**Anti-references (instincts, tested)**
| Part | Core / MHz | CAN-FD | AEC-Q100 / max Tₐ | Non-BGA | SVD |
|---|---|---|---|---|---|
| i.MX RT1062 (*the* Teensy chip) | M7 / 600 | 1 | ✗ / +125 *Tⱼ* | **BGA only ✗** | ⭐ |
| PIC32MZ…EFH064-E | MIPS / **180 @125 °C** | **0 (classic only)** | **✓ Grade 1** / −40…+125 Tₐ | TQFP-64 ✓ | ✗ (MIPS) |

**Key finding.** The general-purpose parts with the best toolchains/SVD (STM32H7, STM32G4, Renesas RA) are **industrial-grade, not AEC-Q100**. The AEC-Q100 parts are either compromised (SAMV71: +105 °C, 2× CAN-FD) or maker-hostile (S32K3 / TRAVEO: no USB, no DAC, pro-only toolchains, fine-pitch packages; AM263x: BGA + Cortex-R). "Automotive grade −40/+125 °C" conflates two requirements — the −40/+125 °C **ambient range** (a temperature spec) and **AEC-Q100** (a qualification regime). Watch **Tₐ vs Tⱼ**: H723/H743/RA8 quote +125 as *junction*; STM32H**725**/H735, G474-T3, S32K3, TRAVEO give true +125 °C *ambient*.

---

## 4. Decision (locked)

**MCU: STMicroelectronics STM32H725**, Cortex-M7 @ up to 550 MHz, 1 MB flash / 564 KB ECC RAM, 3× FDCAN, −40/+125 °C **ambient** (datasheet, verbatim: *"operate in the −40 to +125 °C ambient temperature range"*).

**Package / anchor SKU — `STM32H725RGV3`** (resolves §8 O3): VFQFPN-68, **8×8 mm**, +125 °C grade, **in stock at LCSC `C5271074`** (~$8–16/unit, qty-dependent). Chosen for minimal footprint with **no BGA fan-out** (perimeter + thermal-pad routing on a low-layer board); 3× FDCAN routability confirmed from the alternate-function map. LQFP fallbacks if more I/O is needed: `STM32H725VGT3` (LQFP-100, 14×14 mm, 67 I/O) or `STM32H725ZGT3` (LQFP-144). Graphics sibling STM32H735 is a drop-in if an LCD / 2-D accelerator is ever wanted.

⚠️ Temperature grade lives in the SKU suffix: `…3` = −40/+125 °C, `…6` = −40/+85 °C. Never value-substitute a `…6` part (e.g. STM32H723ZGT6 / …VGT6) — it fails the −40/+125 °C floor.

---

## 5. Rationale

- **Only general-purpose part hitting all of:** M7-class headroom (550 MHz), **3× native FDCAN** (no external controller), true −40/+125 °C **ambient**, non-BGA LQFP, **best-in-class public CMSIS-SVD** (Open-CMSIS-Pack `STM32H7xx_DFP`, per-device `STM32H725.svd`), and the richest ecosystem (CubeMX/GCC/PlatformIO/Zephyr/STM32duino).
- **Peripheral completeness for a brain:** USB HS+FS, 10/100 Ethernet, DAC, 3× ADC (2× 16-bit), 24 timers. The automotive-native parts (S32K3/TRAVEO) notably **lack USB and DAC** — a poor fit for a Teensy-like board.
- **Internal flash** (1 MB) — no external-flash dependency (the i.MX RT / Teensy weakness for a minimal rugged module).
- **C-Next workflow fit:** ST's public, per-device SVD is the cleanest substrate of the field for hand-generating the register definition.

---

## 6. Rejected alternatives (and why)

- **STM32G474** — ties on CAN-FD (3×) and SVD, but M4F/170 MHz (no M7 headroom) and no Ethernet. The pick *if* cost/analog beat headroom.
- **Microchip SAMV71** — the only AEC-Q100 general-purpose M7, but **+105 °C** (misses the floor), 2× CAN-FD, no Arduino/PlatformIO.
- **Renesas RA8M1 / RA6M5** — excellent silicon + SVD + 15-yr life, but **industrial-grade**; RA8's +125 is *junction*, RA6 tops out at +105 °C.
- **Infineon XMC4800** — disqualified: **classic CAN only** (no FD), 144 MHz.
- **NXP S32K3 / Infineon TRAVEO T2G** — genuinely automotive (AEC-Q100 G1, +125 °C ambient, many CAN-FD) but **no USB, no DAC**, pro-only toolchains, fine-pitch QFP, SVD friction — wrong shape for a general-purpose maker brain.
- **TI AM263x** — confirmed misfit: BGA-only, Cortex-R (no CMSIS-SVD), no internal flash.
- **NXP i.MX RT1062 (the Teensy chip)** — BGA-only + no internal flash + not AEC-Q100: the reason a Teensy can't *itself* be the rugged module.
- **Microchip PIC32MZ EF (original instinct)** — AEC-Q100 + hand-solderable TQFP-64, but **classic CAN (no FD)**, MIPS (no CMSIS-SVD), 180 MHz at +125 °C, XC32 optimization paywall, MIPS legacy at Microchip.

---

## 7. Consequences & risks

- **Industrial-grade, not AEC-Q100 — by conscious choice** (§8 O1). Survivability at −40/+125 °C ambient is met; what is foregone is the formal automotive *qualification* (stress suite / PPAP / change-control), which matters only if a downstream product must itself be automotive-certified.
- **Sourcing risk:** the +125 °C suffix-3 SKU (H725ZGT3) is **thin-stocked and ~$16** (LCSC ~19 pcs) vs the +85 °C H723ZGT6 (~$6.63, ~5000 pcs). For a "buyable proven board," verify multi-distributor availability and **qualify a second source / SKU** (e.g. H735ZGT3) before any production run.
- **VFQFPN-68 (8×8 mm) package realities:** the exposed thermal pad must tie to a ground pour via a thermal-via array (550 MHz M7 heat in a small body); 0.4 mm pitch → stencil + reflow/hot-plate assembly (no LQFP-style hand-dragging); ~50 I/O fits 3× CAN-FD + USB + analog/GPIO, with Ethernet (if ever) relegated to a carrier (needs ~9 pins + external PHY). Clock can derate well below 550 MHz (≥150 MHz floor) for ample Tⱼ margin at +125 °C ambient.

---

## 8. Open items (NOT decided)

- **O1 — Doctrine reconciliation (important).** The `protect-12v` doc (§7) states "**AEC-Q100 (ICs)** grade parts only," but `spec/README.md` mandates only **−40/+125 °C** and lists electrical/protection minimums as *reserved/TBD* (§6). This decision sets a precedent: **for complex digital ICs (MCUs), the −40/+125 °C ambient range is the hard floor; AEC-Q100 is preferred-where-practical, not mandatory** — because a rigid AEC-Q100-on-MCU rule eliminates every well-toolchained general-purpose Cortex-M (§3). Per the spec's own meta-rule ("if applying a rule would strip [the goal], reject the rule"), relax the rule, not the silicon. **Action:** reconcile the `protect-12v` bullet and fold this into the spec's reserved electrical-minimums section.
- **O2 — Module name** (the `crd-mcp2562`-style identifier for this board).
- **O3 — Package / pin count — RESOLVED:** VFQFPN-68 (`STM32H725RGV3`, 8×8 mm) for minimal footprint + no BGA fan-out. LQFP-100 (`VGT3`) / LQFP-144 (`ZGT3`) remain fallbacks if more I/O is needed.
- **O4 — Programming / boot** — USB DFU (built-in ST system bootloader) and/or an SWD header; on-module vs carrier.
- **O5 — Board-design essentials** — power input (pairs with `protect-12v` → 3.3V POL), decoupling, HSE oscillator, BOOT/NRST strapping, castellated pad map (gated on the forthcoming GDH footprint conventions, `spec/README.md` §6).
- **O6 — Firmware tooling track** — build the reusable `SVD → C-Next register-definition generator` (parallel to the board; benefits every future GDH ARM module).

---

## 9. Conformance note (`spec/README.md`)

Meets the −40/+125 °C floor (ambient). Castellated / hand-solderable LQFP. CC0. Standard deliverables on fabrication. **Recorded deviation:** industrial-grade MCU (not AEC-Q100) — see §7 / O1.

---

## 10. Sources (primary)

- STM32H725xE/G datasheet **DS13311** — saved in-repo: `modules/mcu-stm32h725/v0.0.1/datasheets/STM32H725RGV3_C5271074.pdf` (273 pp). Verified from its ordering tables: 8 packages incl. VFQFPN-68 (8×8 mm, 0.4 mm) & WLCSP-115, **no LQFP-64**; temp legend `3 = −40/+125 °C ambient`, `6 = −40/+85 °C`; pin codes R=68 / V=100·115 / Z=144; package codes T=LQFP / V=VFQFPN / Y=WLCSP.
- Orderable & verified in stock: **STM32H725RGV3** (VFQFPN-68, +125 °C) — LCSC **C5271074**.
- STM32H723 datasheet **DS13313** (suffix-6 = +85 °C ambient — the non-compliant value SKU).
- CMSIS-SVD: `github.com/Open-CMSIS-Pack/STM32H7xx_DFP` → `CMSIS/SVD/STM32H725.svd`.
- ST Product Longevity Program (10-year commitment tier).
- Full 11-part comparison data: this session's research sweep (2026-06-16).
