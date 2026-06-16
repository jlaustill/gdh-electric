# `protect-12v` — Module Design Spec

**Status:** Draft — contains open decisions (see §6). Not yet approved for layout.
**Date:** 2026-06-06
**Author:** Joshua Austill
**Repo target:** `modules/protect-12v/<version>/` (per `spec/README.md`)
**License:** CC0 1.0 (per GDH default)

> This document records decisions that are **locked** (§2) and decisions that are **still open** (§6). Nothing in §6 has been chosen. Do not treat any open item as decided.

---

## 1. Summary & role in the architecture

`protect-12v` is a rugged **power-input protection front-end** module: the board-ized, GDH-conformant lift of the OCT "Stage 1" power-protection section. It takes a hostile automotive 12V input and presents a **protected pass-through bus** to downstream point-of-load (POL) regulators.

It sits at the front of an **Intermediate Bus Architecture (IBA)**:

```
  hostile input            protected bus (unregulated)        point-of-load
  (12V nominal,    ┌───────────────────────┐   6.5–18V    ┌── 3.3V POL ── MCU
   ~8V…spikes) ───>│   protect-12v          │── window ──>├── 5V  POL ── transceivers
                   │   (this module)        │  or it      └── (more POLs as needed)
                   └───────────────────────┘  disconnects
  Stage 1: survive the input                  Stage 2: make the voltages
```

**Why this module exists / market gap:** off-the-shelf POL modules (e.g. Pololu) are good and cheap but carry **no input protection** — they assume a benign supply. `protect-12v` fills exactly that gap. GDH does **not** reinvent the POL; it supplies the protected bus that makes any POL safe to use on a real vehicle. The intermediate-bus interface is regulator-agnostic: off-the-shelf POLs today, optional GDH castellated POLs later, with no change to this module or the carrier.

---

## 2. Locked decisions (confirmed)

| # | Decision | Value |
|---|----------|-------|
| L1 | Downstream regulator safety posture | POLs are **pure POL** and assume a clean bus (like `crd` assumes clean 5V). Standalone-input protection is the integrator's responsibility; regulation failing is *obvious*, not silently omittable. |
| L2 | Bus contract | **Protected pass-through, UNREGULATED.** Output = the input rail, conditioned/protected, guaranteed within a **~6.5–18V window or disconnected**. Downstream POLs handle the wide range. No conversion stage in this module. |
| L3 | Voltage-class scope | **12V automotive nominal.** Topology and footprint designed so a future **`protect-24v`** sibling is a drop-in (same outline + pad map; only threshold resistors and TVS change). |
| L4 | Continuous current ceiling | **~3A continuous** (≈4–5A trip). Sized so the protected bus can feed several POLs plus buck-inefficiency headroom. |
| L5 | POL strategy (for now) | **No GDH POL module in this scope.** Use off-the-shelf POLs (e.g. Pololu) downstream. POL granularity (single-rail vs multi-rail vs adjustable) is **deferred**, not decided. |
| L6 | Safety boundary | The module's **entire** safety contribution is its **`FAULT#` power-health pin**. Power is binary good/bad. **No watchdog, no actuator safe-state, no functional-safety claim** — those are separate concerns with separate owners (see memory: GDH power/safety boundary). |

---

## 3. Protection function set (the lift from OCT Stage 1)

The module is a chain of protection stages. Functions are **locked** (these are the protections the module exists to provide); specific parts/values are **baseline-from-OCT** and finalized at schematic/layout, except where they depend on an open decision in §6.

| Stage | Function | OCT baseline part | Notes |
|-------|----------|-------------------|-------|
| EMI filter | Differential/HF noise suppression | Ferrite bead `HCB4532MF-681T40` (1812, 680Ω@100MHz) + caps | Value to confirm for 3A copper. |
| Overcurrent | Resettable fuse | OCT used `1210L100` (1A hold) | **Must be re-selected for ~3A hold / ~4–5A trip**, AEC-Q200, −40/+125 °C. Part TBD. |
| Surge / load dump | TVS clamp | `SMCJ26CA` class (1500W) | Standoff must sit above the OV threshold; sized to ISO 7637-2 / ISO 16750-2 load dump. Final standoff/clamp TBD. |
| Reverse polarity + UV + OV | Disconnect (ideal-diode + window) | **Depends on §6 D1** | OCT baseline: `LTC4365` controller + `SI9945AEY` dual N-FET. See open decision. |
| Decoupling | Input/output bulk + ceramic | OCT input 470µF + ceramics | On-module: enough for stable protection + TVS response. Large hold-up bulk → carrier. Values TBD. |

**UV/OV window:** target ~6.5V / ~18V (per L2), set by a 1% resistor divider on-module (OCT used 1.82M / 97.6k / 54.9k). Exact thresholds finalized in design; these are the values that make `protect-24v` a resistor swap (L3).

---

## 4. Interface contract

**Castellated edge** (per GDH spec — reflow + breadboard friendly, hot-plate desolderable). Power pads are paralleled because this module carries real current, unlike signal-level `crd`.

| Signal | Direction | Pads | Notes |
|--------|-----------|------|-------|
| `VIN` | in | 2–3 paralleled | Raw protected input. |
| `GND` | — | 2–3 paralleled | Star-ground on the carrier. |
| `VBUS` (protected out) | out | 2–3 paralleled | The protected, unregulated bus (L2). |
| `FAULT#` | out | 1 | Open-drain, active-low power-health (L6). |
| `EN` / `SHDN#` | in | 1 | **Open — see §6 D3.** |

**Bus guarantee to consumers:** "`VBUS` is the input rail with reverse polarity blocked, surge clamped, and UV/OV disconnect — it is either inside ~6.5–18V or it is off. It is NOT regulated. Size your POL for this window."

---

## 5. Block diagram

```
 VIN ──[Ferrite/EMI]──[PTC ~3A]──[TVS clamp]──[ Reverse + UV/OV disconnect ]──┬── VBUS
                                                        │  (§6 D1 topology)    │
                                                        └────────► FAULT# ─────┘
                                                                   (open-drain)
 GND ─────────────────────────────────────────────────────────────────────────── GND
```

---

## 6. Open decisions (choices to be made — NOT decided)

> These are recorded as choices on purpose. None is selected.

### D1 — Protection-IC topology
The reverse + UV/OV disconnect can be built three ways:

- **A — Lift OCT verbatim: `LTC4365` controller + `SI9945AEY` dual N-FET.** Bench-proven on OCT; native 60V UV/OV/reverse incl. transient OV ride-through; spans to 60V so it also covers the future 24V sibling. Highest BOM cost (~$5.56 controller + dual FET). Note: the LTC4365 is a *controller, not an eFuse* — it has no current limit of its own; the PTC fuse bounds the 3A.
- **B — Modern automotive eFuse (single IC, e.g. TI TPS1663 / Infineon PROFET-class).** Bundles OV/UV/current-limit/inrush; fewer parts, often cheaper. But most eFuses don't block reverse natively (needs an added back-to-back FET / ideal-diode), narrower 60V load-dump selection, and re-validation of behavior already proven on OCT.
- **C — Split: dedicated reverse ideal-diode controller (e.g. `LM74700`) + separate OV/UV supervisor.** Cheap simple blocks, lowest reverse loss; but most ICs / largest board, loses the single-part transient-OV ride-through.

*D1 also determines the pass-element (the `SI9945` baseline belongs to A) and influences fuse/TVS choices.*

### D1a — Fault response (only if A): auto-restart vs latch-off
OCT used `LTC4365HTS8-1` (auto-restart when fault clears). The latching variant (`-2`) holds off until power-cycled. Not decided.

### D2 — 7.5V VPW rail placement
The OCT `ZTP7192Y` 7.5V boost exists only for J1850 VPW transmit levels. **Proposed** to live on the future `pci` module (intrinsic to that function), not as a general PSU module. **Not confirmed.**

### D3 — Expose `EN`/`SHDN#` pad?
Whether the carrier can disable the module (and whether the pad pulls up on-module so floating = enabled). Not decided.

### D4 — Specific part numbers & values
PTC fuse (~3A), TVS standoff/clamp, input/output cap values, and final pass-FET are **to be selected**, several gated on D1. None assumed.

### D5 — Footprint, pinout order, board outline, silkscreen
Per `spec/README.md` §6 these GDH-wide conventions are still being settled. `protect-12v`'s outline/pad map must be fixed against those once defined, and must satisfy the L3 drop-in-sibling requirement.

---

## 7. Conformance to the GDH module spec (`spec/README.md`)

- Castellated edges; reflow + breadboard compatible; hot-plate desolderable.
- **−40 °C to +125 °C minimum**; AEC-Q200 (passives) / AEC-Q100 (ICs) grade parts only.
- Robust-by-default: protection is the product, not optional polish.
- Deliverables in the version folder: `BOM.csv` (UTF-8, `Datasheet` column, no `CASTELLATED_PAD` rows), `PickAndPlace.csv`, `Schematic.svg` (primary) + `Schematic.pdf` (generated from SVG), `Gerber.zip`, `datasheets/` (one per orderable part).
- Version silkscreened on the PCB.

---

## 8. Validation (proposed — to confirm against forthcoming GDH validation conventions)

Before publishing a version, verify:
1. **Reverse polarity:** apply −14V; confirm no forward current, no damage.
2. **Undervoltage:** ramp input down; confirm disconnect near the UV threshold and `FAULT#` asserts.
3. **Overvoltage:** ramp input up; confirm disconnect near the OV threshold and `FAULT#` asserts.
4. **Surge / load dump:** apply ISO 7637-2 pulses and ISO 16750-2 load dump; confirm survival and clamp behavior.
5. **Continuous current:** 3A through `VBUS`; measure pass-element and castellation temperature rise; confirm margin.
6. **Fault recovery:** confirm `FAULT#` deasserts and the bus restores per the D1a behavior chosen.
7. **Quiescent current** at nominal 12V.

---

## 9. Explicitly out of scope

- **Voltage regulation / POLs** — off-the-shelf downstream (L5).
- **7.5V VPW boost** — proposed to `pci` module (D2).
- **Watchdog, MCU-liveness, actuator safe-state, any functional-safety / ASIL claim** — separate concern, separate owner (L6).
- **24V / 48V variants** — future siblings; this version is 12V only (L3).
```
