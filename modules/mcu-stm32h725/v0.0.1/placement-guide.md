# mcu-stm32h725 — Component Placement Guide

Placement-criticality reference for the PCB layout: how close each support
component must sit to its target pin, and why. Use it to decide where to spend
placement precision when routing.

> **Reference designators below are board-side (post-F8).** KiCad's *Update PCB
> from Schematic* re-annotated some parts, so a handful differ from the
> schematic. Key parts by function: **MCU = U3**, buck = U2, OR-mux = U4,
> USB protection = U1, USB-C = J1, crystal = Y1, castellation edges = J2/J3.

## The one rule

> **The tighter the current loop a part carries, the closer it must sit.**
>
> SMPS switch node ▸ supply decoupling ▸ bulk caps ▸ quasi-static pins.

A long loop on a switching or decoupling net adds parasitic inductance →
radiated EMI + regulator instability. Quasi-static pins (NRST, BOOT0) carry no
high-frequency current, so distance barely matters.

---

## 🔴 Critical — hug the MCU pin (≤ 2–3 mm)

| Ref | Value | Function | Ideal max | Why |
|---|---|---|---|---|
| **L1** | 2.2 µH | **SMPS inductor** (`VLXSMPS → L1 → VCORE`) | ≤ 2–3 mm, **minimal loop area** | Live switching node — *the* most critical net on the board. Short, fat traces; keep the loop tiny. |
| C3 | 220 pF | SMPS switch-node filter (Cfilt) | ≤ 2 mm | Snubs the `VLXSMPS` switch node. |
| C4 | 10 µF | SMPS output cap (VCORE / VFBSMPS) | ≤ 3 mm | Closes the SMPS loop; stabilizes VCORE. |
| C6, C7 | 100 nF | VCAP / VCORE decoupling | ≤ 2 mm | At the VCAP pins. |
| C9, C11, C12, C13, C14, C18 | 100 nF | VDD high-freq decoupling | ≤ 2 mm | **One per VDD pin** — each hugs its own pin. |
| C17 | 100 nF | VDDA high-freq decoupling | ≤ 2 mm | At the VDDA pin. |
| C19, C20 | 27 pF | HSE crystal load caps | ≤ 3 mm | Short, symmetric traces to OSC_IN / OSC_OUT. |

## 🟠 Important (≤ 5 mm)

| Ref | Value | Function | Ideal max | Why |
|---|---|---|---|---|
| C1, C15 | 4.7 µF | VDD bulk reservoir | ≤ 5 mm | Backs up the 100 nF's; near the MCU power entry. |
| C16 | 1 µF | VDDA bulk | ≤ 5 mm | Near the VDDA pin. |
| FB1 | ferrite (600 Ω@100 MHz) | VDDA filter bead | ≤ 5 mm | Forms the VDDA LC filter with C16/C17. |
| Y1 | 25 MHz | HSE crystal | ≤ 5 mm, short OSC | Keep the oscillator loop tight + away from noisy/switching nets. |

## 🟢 Relaxed (≤ 10–15 mm — don't sweat it)

| Ref | Value | Function | Why it's relaxed |
|---|---|---|---|
| C21 | 100 nF | NRST reset / noise filter | NRST is quasi-static; cap only filters glitches. |
| R6 | 10 k | BOOT0 pulldown | Static strap; position irrelevant. |
| (VBAT cap) | 100 nF | VBAT decoupling | VBAT tied to +3V3; low criticality. |

## ⚙️ Power block — place near their *own* IC, not the MCU

| Ref | Value | Function | Target | Ideal max |
|---|---|---|---|---|
| C5 | 22 µF | Buck output cap | buck U2 | ≤ 3 mm |
| L2 | 2.2 µH | Buck inductor (`SW → L2 → +3V3`) | buck U2 | ≤ 3 mm, tight loop |
| C2, C10 | 10 µF | Buck input / 5 V rail | buck U2 input | ≤ 3 mm |
| R1, R2 | 453 k / 100 k | Buck feedback divider | buck U2 FB | ≤ 3 mm to FB pin |
| R3 | 47 k | OR-mux PR1 priority-set | mux U4 | ≤ 5 mm |
| R4, R5 | 5.1 k | USB-C CC pulldowns | USB-C J1 | ≤ 5 mm |
| C8 | 10 µF | VBUS reservoir | USB protect U1 / J1 | ≤ 5 mm |

---

## Routing notes

- **4-layer stackup:** F.Cu (signal) · In1.Cu (GND plane) · In2.Cu (+3V3 plane) · B.Cu (signal).
- The MCU's power/ground pads are **SMD (top only)** — drop a **via per pad** down to the inner planes. The castellation power pads are through-hole and reach the planes directly.
- Castellations carry **raw MCU pin names**; the pin→pad map is crossing-free per edge and optimal for the centered MCU (re-run the optimizer if the MCU moves).
- The open right side of the board is **routing channel** for the right-edge castellations — not wasted space.

---

*Generated 2026-06-18. Distances are guidelines for a 550 MHz Cortex-M7 + on-chip SMPS; tighten the 🔴 rows first, especially the SMPS `VLXSMPS→L1→VCORE` loop.*
