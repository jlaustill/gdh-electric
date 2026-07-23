# mcu-imxrt1062 Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up the `mcu-imxrt1062` module: KiCad 9 project scaffold on the reused 54-castellation outline, every part sourced (LCSC code + symbol/footprint/3D + verified datasheet), and the definitive 54-pin castellation map — so the schematic plan (Plan 2) can start with all parts and nets defined.

**Architecture:** This is milestone 1 of 3 (Foundation → Schematic → Layout/outputs). Foundation produces *inputs*, not a routed board: populated KiCad libraries, a verified BOM skeleton, and `pinmap.md`. No schematic or layout work happens here beyond copying the physical outline.

**Tech Stack:** KiCad 9 (`kicad-cli` for checks/exports), `easyeda2kicad` (LCSC → symbol/footprint/3D), LCSC product API (datasheets/availability), `pdftotext` (datasheet verification). Design spec: `docs/superpowers/specs/2026-07-23-mcu-imxrt1062-rugged-teensy-design.md`.

## Global Constraints

- KiCad **9** (matches `mcu-stm32h725`; this is the repo's one KiCad module).
- Fab = **PCBWay**. **Every part needs an LCSC code EXCEPT the PJRC MKL02Z32 bootloader chip** (customer-supplied, mail-in assembly).
- Castellated SMT module: **B.Cu component-free** (routing OK); all castellation pads get `(property pad_prop_castellated)`.
- Part generation: `easyeda2kicad --full --lcsc_id=C<code> --output <dir>` (symbol+footprint+3D from LCSC's verified geometry).
- Datasheets: name `<ManufacturerPart>_<LCSC>.pdf` with `/`→`-`; store in `modules/mcu-imxrt1062/v0.0.1/datasheets/`. Verify each: first bytes are `%PDF` **and** `pdftotext -l 2 <f> -` contains the part number.
- LCSC datasheet/availability lookup (browser UA required): `curl -A "Mozilla/5.0" "https://wmsc.lcsc.com/ftps/wm/product/detail?productCode=C<code>"` → JSON `pdfUrl` / stock fields.
- Module folder: `modules/mcu-imxrt1062/v0.0.1/`. **License: CC0 1.0.**
- Physical outline is **copied verbatim** from `mcu-stm32h725 v0.0.1` (54 castellations, 27/edge × 2) — do not redraw it.
- MCU is **MIMXRT1062CVJ5B** (industrial C-grade). Pinout is identical to the Teensy 4.x / consumer grade, so all Teensy pin-mapping references apply.

---

### Task 1: Environment — confirm/install easyeda2kicad

**Files:** none (tooling only)

**Interfaces:**
- Produces: a working `easyeda2kicad` on PATH for all later sourcing tasks.

- [ ] **Step 1: Check whether easyeda2kicad is available**

Run: `easyeda2kicad --version 2>/dev/null || echo MISSING`
Expected: either a version string, or `MISSING`.

- [ ] **Step 2: Install if missing (pipx preferred, pip fallback)**

Run: `pipx install easyeda2kicad || pip install --user easyeda2kicad`

- [ ] **Step 3: Verify it runs**

Run: `easyeda2kicad --version`
Expected: prints a version (e.g. `easyeda2kicad 0.8.x`). Do not proceed until this succeeds.

- [ ] **Step 4: Confirm kicad-cli is present (used for later checks/exports)**

Run: `kicad-cli version`
Expected: prints a KiCad 9.x version.

---

### Task 2: Scaffold the module directory + copy the physical outline

**Files:**
- Create: `modules/mcu-imxrt1062/v0.0.1/` (dir)
- Create: `modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_pro`
- Create: `modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_pcb`
- Create: `modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_sch`
- Create: `modules/mcu-imxrt1062/v0.0.1/datasheets/` (dir)
- Create: `modules/mcu-imxrt1062/v0.0.1/LICENSE` (CC0 text)
- Template from: `modules/mcu-stm32h725/v0.0.1/*.kicad_pcb` (outline only)

**Interfaces:**
- Produces: an openable KiCad 9 project whose `.kicad_pcb` contains ONLY the Edge.Cuts outline + 54 empty castellation pad positions, no STM32 parts.

- [ ] **Step 1: Create directory structure**

```bash
cd /home/linux/code/gdh-electric
mkdir -p modules/mcu-imxrt1062/v0.0.1/datasheets
```

- [ ] **Step 2: Copy the H725 board, then strip it to the outline**

Copy `modules/mcu-stm32h725/v0.0.1/mcu-stm32h725.kicad_pcb` to the new path, then (pcbnew CLOSED, editing sexp directly per repo convention) delete every `(footprint …)` block and every net except the outline, keeping all `(gr_line …)`/`(gr_rect …)` on `Edge.Cuts` and the 54 castellation pad *positions* if they are board-level pads. Rename the internal `(general)`/title references to `mcu-imxrt1062`.

```bash
cp modules/mcu-stm32h725/v0.0.1/mcu-stm32h725.kicad_pcb \
   modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_pcb
```

- [ ] **Step 3: Create the project + empty schematic files**

Create `mcu-imxrt1062.kicad_pro` and an empty `mcu-imxrt1062.kicad_sch` (copy the H725 `.kicad_pro`, change the project name field; create a blank schematic sheet). Also copy `fp-lib-table` and `sym-lib-table` from the H725 module and repoint any project-relative library paths to `mcu-imxrt1062`.

- [ ] **Step 4: Add the CC0 LICENSE file**

```bash
curl -sL https://creativecommons.org/publicdomain/zero/1.0/legalcode.txt \
  -o modules/mcu-imxrt1062/v0.0.1/LICENSE
head -c 4 modules/mcu-imxrt1062/v0.0.1/LICENSE   # sanity: non-empty
```

- [ ] **Step 5: Verify Edge.Cuts + castellation count survived the strip**

Run:
```bash
grep -c 'pad_prop_castellated' modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_pcb
```
Expected: `54` (matches the source outline). Then confirm `(`/`)` balance:
```bash
python3 -c "s=open('modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_pcb').read(); print('BALANCED' if s.count('(')==s.count(')') else 'UNBALANCED')"
```
Expected: `BALANCED`.

- [ ] **Step 6: Verify the project opens headless (ERC run, expected to be trivially clean/empty)**

Run: `kicad-cli sch erc --exit-code-violations modules/mcu-imxrt1062/v0.0.1/mcu-imxrt1062.kicad_sch; echo "exit=$?"`
Expected: runs without crashing (empty schematic → no violations, `exit=0`).

- [ ] **Step 7: Commit**

```bash
git add modules/mcu-imxrt1062/v0.0.1/
git commit -m "scaffold mcu-imxrt1062 module on the 54-castellation outline"
```

---

### Task 3: Lock the BOM skeleton (manufacturer parts → LCSC codes)

**Files:**
- Create: `modules/mcu-imxrt1062/v0.0.1/bom-skeleton.md`

**Interfaces:**
- Produces: `bom-skeleton.md` — a table `RefDes | Function | Manufacturer Part | LCSC code | Package | Datasheet filename`, with an LCSC code confirmed in-stock for every row except the bootloader chip. Consumed by Tasks 4–7.

**Baseline part choices (specified — this task's job is to confirm LCSC codes + stock, not to re-select):**

| RefDes | Function | Manufacturer Part | Notes |
|---|---|---|---|
| U1 | MCU | NXP **MIMXRT1062CVJ5B** | industrial C-grade, 196-MAPBGA |
| U2 | Bootloader | **MKL02Z32VFG4** (PJRC pre-programmed) | QFN-16; **NO LCSC** — customer-supplied |
| U3 | QSPI flash 16MB | Winbond **W25Q128JVSIQ** | FlexSPI1 |
| U4 | QSPI PSRAM 8MB | APMemory **APS6404L-3SQR-ZR** | FlexSPI2 (Teensy 4.1 part) |
| J1 | USB-C receptacle 2.0 16P | **TYPE-C-31-M-12** (or equivalent 16P 2.0) | on-module USB1 |
| U5 | USB 2.0 HS 2:1 mux | TI **TS3USB221ARSER** | selects USB-C vs castellated D+/D− |
| D1 | USB ESD array | ST **USBLC6-2SC6** | D+/D−/VBUS ESD |
| U6 | Priority power mux | TI **TPS2116DMQR** | 5V USB vs 5V VIN OR-ing |
| U7 | 3.3V buck | TI **TLV62569DBVR** | 5V→3.3V, high-temp headroom vs LDO |
| L1 | Core DCDC inductor 2.2µH | (shielded power inductor, ≥1A) | RT1062 internal DCDC |
| Y1 | Crystal 24 MHz | (RT1062 main clock) | matches Teensy |
| Y2 | RTC crystal 32.768 kHz | (RT1062 RTC) | matches Teensy 4.1 |
| D2 | VBUS TVS 5V | (low-cap 5V TVS) | VBUS over-voltage |

- [ ] **Step 1: Create `bom-skeleton.md` with the baseline table above (LCSC column empty)**

Write the table verbatim into `modules/mcu-imxrt1062/v0.0.1/bom-skeleton.md`, LCSC column marked `?` for each row (except U2 = `N/A — PJRC`).

- [ ] **Step 2: Look up the LCSC code + stock for each manufacturer part**

For each row (skip U2), find the LCSC code (search LCSC for the manufacturer part), then confirm stock:
```bash
# after finding a candidate code C<code>:
curl -sA "Mozilla/5.0" "https://wmsc.lcsc.com/ftps/wm/product/detail?productCode=C<code>" \
  | python3 -c "import sys,json; d=json.load(sys.stdin)['result']; print(d['productModel'], '| stock', d.get('stockNumber'), '| pdf', bool(d.get('pdfUrl')))"
```
Expected: prints the matching manufacturer model, a non-zero stock number, and `pdf True`. Record the code in the table.

- [ ] **Step 3: Flag any part with zero stock / no LCSC match**

If a baseline part is unavailable, record the closest in-stock alternative in the same package in a `Substitution notes` section (per `gdh-part-sourcing-rule`: LCSC availability is the only hard requirement; footprint/symbol/3D may come from anywhere). Do NOT silently swap packages that break the layout — note it for review.

- [ ] **Step 4: Verify every non-bootloader row has a code**

Run:
```bash
grep -E '^\| (U|J|D|L|Y)[0-9]' modules/mcu-imxrt1062/v0.0.1/bom-skeleton.md | grep -c ' C[0-9]'
```
Expected: count = number of rows minus 1 (U2 excluded). Confirm the only `N/A` is U2.

- [ ] **Step 5: Commit**

```bash
git add modules/mcu-imxrt1062/v0.0.1/bom-skeleton.md
git commit -m "lock mcu-imxrt1062 BOM skeleton with confirmed LCSC codes"
```

---

### Task 4: Source the MCU + memory (U1, U3, U4) — libs + datasheets

**Files:**
- Create: `modules/mcu-imxrt1062/v0.0.1/kicad/` symbol/footprint/3D libs (easyeda2kicad output)
- Create: `modules/mcu-imxrt1062/v0.0.1/datasheets/{MIMXRT1062CVJ5B,W25Q128JVSIQ,APS6404L-3SQR-ZR}_C<code>.pdf`

**Interfaces:**
- Consumes: LCSC codes from `bom-skeleton.md` (Task 3).
- Produces: KiCad symbol+footprint+3D for U1/U3/U4 registered in the module libraries; three verified datasheets.

- [ ] **Step 1: Generate symbol/footprint/3D for each of U1, U3, U4**

```bash
cd modules/mcu-imxrt1062/v0.0.1
for CODE in <U1_code> <U3_code> <U4_code>; do
  easyeda2kicad --full --lcsc_id=$CODE --output kicad/mcu-imxrt1062
done
```
Expected: creates/updates `kicad/mcu-imxrt1062.kicad_sym`, `.pretty/`, `.3dshapes/`.

- [ ] **Step 2: Verify the MCU footprint pad count matches a 196-ball BGA**

Run:
```bash
grep -c '(pad ' kicad/mcu-imxrt1062.pretty/*MIMXRT1062*.kicad_mod
```
Expected: `196` (± any mechanical pads). If wildly off, the wrong LCSC part was pulled — stop and recheck the code.

- [ ] **Step 3: Fetch + verify each datasheet**

For each part, pull `pdfUrl` from the LCSC API (Global Constraints command), download to `datasheets/<Part>_C<code>.pdf`, then:
```bash
head -c 4 datasheets/<Part>_C<code>.pdf          # expect: %PDF
pdftotext -l 2 datasheets/<Part>_C<code>.pdf - | grep -i "<part number>"   # expect: a match
```
Expected: `%PDF` and a part-number hit for all three.

- [ ] **Step 4: Commit**

```bash
git add modules/mcu-imxrt1062/v0.0.1/kicad modules/mcu-imxrt1062/v0.0.1/datasheets
git commit -m "source mcu-imxrt1062 MCU + flash + PSRAM (libs + datasheets)"
```

---

### Task 5: Source the USB1 subsystem (J1, U5, D1) — libs + datasheets

**Files:**
- Modify: module KiCad libs (append J1/U5/D1)
- Create: `datasheets/{USB-C,TS3USB221,USBLC6-2SC6}_C<code>.pdf`

**Interfaces:**
- Consumes: LCSC codes (Task 3).
- Produces: symbol+footprint+3D for the USB-C receptacle, HS mux, and ESD array; three verified datasheets.

- [ ] **Step 1: Generate libs for J1, U5, D1**

```bash
cd modules/mcu-imxrt1062/v0.0.1
for CODE in <J1_code> <U5_code> <D1_code>; do
  easyeda2kicad --full --lcsc_id=$CODE --output kicad/mcu-imxrt1062
done
```

- [ ] **Step 2: Verify the USB-C footprint has the USB-2.0 pin set**

Run:
```bash
grep -oE '\(pad "[A-Z0-9]+"' kicad/mcu-imxrt1062.pretty/*[Tt]ype*[Cc]*.kicad_mod | sort -u | head -40
```
Expected: includes VBUS, GND, D+, D−, CC1/CC2, SHIELD pads (a 16-pin 2.0 receptacle). If it shows the full 24-pin USB-3 set, that's fine but note it for layout.

- [ ] **Step 3: Fetch + verify the three datasheets** (same method as Task 4, Step 3)

Expected: `%PDF` + part-number hit for each.

- [ ] **Step 4: Commit**

```bash
git add modules/mcu-imxrt1062/v0.0.1/kicad modules/mcu-imxrt1062/v0.0.1/datasheets
git commit -m "source mcu-imxrt1062 USB1 subsystem (USB-C, HS mux, ESD)"
```

---

### Task 6: Source power + clocks + protection (U6, U7, L1, Y1, Y2, D2) — libs + datasheets

**Files:**
- Modify: module KiCad libs (append U6/U7/L1/Y1/Y2/D2)
- Create: `datasheets/{TPS2116,TLV62569,<inductor>,<24MHz xtal>,<32k xtal>,<TVS>}_C<code>.pdf`

**Interfaces:**
- Consumes: LCSC codes (Task 3).
- Produces: symbol+footprint+3D for the power/clock/protection parts; verified datasheets.

- [ ] **Step 1: Generate libs for U6, U7, L1, Y1, Y2, D2**

```bash
cd modules/mcu-imxrt1062/v0.0.1
for CODE in <U6_code> <U7_code> <L1_code> <Y1_code> <Y2_code> <D2_code>; do
  easyeda2kicad --full --lcsc_id=$CODE --output kicad/mcu-imxrt1062
done
```

- [ ] **Step 2: Verify each generated symbol exists**

Run:
```bash
for NAME in TPS2116 TLV62569 <inductor> <xtal24> <xtal32k> <tvs>; do
  grep -q "$NAME" kicad/mcu-imxrt1062.kicad_sym && echo "$NAME OK" || echo "$NAME MISSING"
done
```
Expected: all `OK`.

- [ ] **Step 3: Fetch + verify datasheets** (same method as Task 4, Step 3)

Expected: `%PDF` + part-number hit for each.

- [ ] **Step 4: Commit**

```bash
git add modules/mcu-imxrt1062/v0.0.1/kicad modules/mcu-imxrt1062/v0.0.1/datasheets
git commit -m "source mcu-imxrt1062 power, clock, and protection parts"
```

---

### Task 7: Author the definitive 54-pin castellation map (`pinmap.md`)

**Files:**
- Create: `modules/mcu-imxrt1062/v0.0.1/pinmap.md`
- Template from: `modules/mcu-stm32h725/v0.0.1/pinmap.md`
- Reference: `datasheets/MIMXRT1062CVJ5B_*.pdf` (IOMUX/ball map), PJRC Teensy 4.1 pinout

**Interfaces:**
- Consumes: the MCU datasheet (Task 4).
- Produces: `pinmap.md` — the authoritative table `Castellation# | RT1062 ball | Signal/Teensy-pin | Class`, used by Plan 2 (schematic net assignment).

**Required contents (from the spec):** all **Teensy-4.1 I/O pins 0–41**; **Program** + **Reset**; **USB1 D+/D−**; power/ground; **2× extra GND**. The 16-bit parallel window (`GPIO6[31:16]` = `GPIO_AD_B1_00..15`) must be fully present — pins 14,15,16,17,18,19,20,21,22,23,26,27,38,39,40,41.

- [ ] **Step 1: Extract the RT1062 ball map from the datasheet**

Run: `pdftotext -layout datasheets/MIMXRT1062CVJ5B_*.pdf - | less` and locate the ball-assignment + IOMUX tables. Record, for each needed signal, its BGA ball (e.g. `GPIO_AD_B1_08` → ball, → `GPIO6_IO24` → Teensy pin 22).

- [ ] **Step 2: Build the castellation table (54 rows)**

Fill `pinmap.md` with one row per castellation. Assign in this priority order until 54 are used:
  1. USB1 D+/D− (2)
  2. Program, Reset (2)
  3. Teensy pins 0–41 (42) — including all 16 parallel-window pins
  4. Power: 5V/VIN (=VBUS), 3.3V-out (4)
  5. Ground: base GND + **2× extra GND** (remaining, ~4)
Verify the running total lands on exactly 54.

- [ ] **Step 3: Mark the 16-bit parallel window explicitly**

Add a `## 16-bit parallel port (GPIO6[31:16])` section listing the 16 castellation numbers carrying `AD_B1_00..15`, with the CAN1/I2C0/I2C1/Serial3/analog overlap trade noted (CAN2 on 0/1 and CAN3 on 30/31 remain free).

- [ ] **Step 4: Validate — 54 unique castellations, no double-assigned RT1062 ball, window complete**

Run:
```bash
python3 - <<'PY'
import re
rows=[l for l in open('modules/mcu-imxrt1062/v0.0.1/pinmap.md') if re.match(r'^\|\s*\d+\s*\|', l)]
nums=[int(l.split('|')[1]) for l in rows]
balls=[l.split('|')[2].strip() for l in rows if l.split('|')[2].strip() not in ('GND','')]
print('rows', len(rows), '| unique cast#', len(set(nums)), '| dup balls', len(balls)-len(set(balls)))
window={'14','15','16','17','18','19','20','21','22','23','26','27','38','39','40','41'}
teensy=set(re.findall(r'pin (\d+)', open('modules/mcu-imxrt1062/v0.0.1/pinmap.md').read()))
print('window present:', window<=teensy)
PY
```
Expected: `rows 54 | unique cast# 54 | dup balls 0` and `window present: True`.

- [ ] **Step 5: Commit**

```bash
git add modules/mcu-imxrt1062/v0.0.1/pinmap.md
git commit -m "author definitive 54-pin castellation map for mcu-imxrt1062"
```

---

### Task 8: Foundation gate — availability + completeness sweep

**Files:**
- Create: `modules/mcu-imxrt1062/v0.0.1/report.txt` (sourcing summary)

**Interfaces:**
- Consumes: everything from Tasks 3–7.
- Produces: a pass/fail sourcing report; green-lights Plan 2 (Schematic).

- [ ] **Step 1: Re-confirm live stock for every LCSC code**

For each code in `bom-skeleton.md`, re-run the LCSC stock query (Global Constraints). Write `part | code | stock | OK/LOW` lines into `report.txt`. Any zero-stock part → flag for review before Plan 2.

- [ ] **Step 2: Confirm every part has symbol + footprint + 3D + datasheet**

Run:
```bash
cd modules/mcu-imxrt1062/v0.0.1
echo "symbols:"; grep -c '(symbol ' kicad/mcu-imxrt1062.kicad_sym
echo "footprints:"; ls kicad/mcu-imxrt1062.pretty/*.kicad_mod | wc -l
echo "3d:"; ls kicad/mcu-imxrt1062.3dshapes/ | wc -l
echo "datasheets:"; ls datasheets/*.pdf | wc -l
```
Expected: each count ≥ the number of orderable parts (12; U2 has no LCSC datasheet but still needs a footprint/symbol — confirm a generic QFN-16 exists for it).

- [ ] **Step 3: Confirm the bootloader chip (U2) has a footprint despite no LCSC**

Run: `grep -qi 'QFN.*16\|MKL02' kicad/mcu-imxrt1062.kicad_sym && echo "U2 symbol OK" || echo "U2 MISSING — create generic QFN-16"`
Expected: `U2 symbol OK` (if missing, create a generic QFN-16 3×3 0.5mm symbol/footprint — it is customer-supplied but still placed).

- [ ] **Step 4: Write the foundation summary + commit**

Summarize in `report.txt`: parts sourced, any substitutions, pinmap validated, open risks carried into Plan 2. Then:
```bash
git add modules/mcu-imxrt1062/v0.0.1/report.txt
git commit -m "mcu-imxrt1062 foundation gate: sourcing + pinmap verified"
```

---

## Self-Review

**Spec coverage (Foundation-relevant sections):**
- MCU (industrial C-grade) → Task 3/4 ✓
- Bootloader chip, no-LCSC/customer-supplied → Task 3 (row U2), Task 8 Step 3 ✓
- 16MB flash + 8MB PSRAM → Task 3/4 ✓
- USB1 dual-access parts (USB-C, HS mux, ESD) → Task 5 ✓
- Power OR-ing + reverse-polarity + rails → Task 6 (U6/U7/L1) ✓
- Clocks (24MHz, 32.768kHz) → Task 6 ✓
- VBUS OVP → Task 6 (D2) ✓
- 54-castellation outline reuse → Task 2 ✓
- 54-pin map incl. 16-bit window + Program/Reset + 2×GND → Task 7 ✓
- CC0 license → Task 2 ✓
- PCBWay/LCSC availability rule → Task 3/8 ✓
- *Deferred to Plan 2/3 (correctly not here):* schematic capture, layout, USB stub mitigation, ERC/DRC, Gerber/BOM/PickAndPlace/SVG exports.

**Placeholder scan:** `<code>`/`<Un_code>` tokens are deliberate lookup outputs from Task 3, not placeholders — each is resolved by a concrete command with expected output. No "TBD/handle appropriately" steps.

**Type consistency:** RefDes (U1–U7, J1, D1–D2, L1, Y1–Y2) are used consistently across Tasks 3–8. `pinmap.md`/`bom-skeleton.md`/`report.txt` filenames are stable across references.

---

## Next milestones (not in this plan)
- **Plan 2 — Schematic:** power tree, RT1062 core + decoupling, FlexSPI1 flash / FlexSPI2 PSRAM, USB1 dual-access + mux + ESD, bootloader wiring, Program/Reset, net assignment from `pinmap.md`. Gate: `kicad-cli sch erc` clean.
- **Plan 3 — Layout + outputs:** placement (B.Cu component-free, USB-C hard against castellation breakout), HS routing, ground pour/stitching, DRC, then Gerber/BOM/PickAndPlace/SVG exports. Gate: `kicad-cli pcb drc` clean + exports present.
