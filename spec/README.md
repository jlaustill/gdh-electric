# GDH Electric — Module Specification

This is the standard every GDH Electric module conforms to.

GDH Electric sells **proven, breadboard-compatible, production-ready reference boards**.

---

## Part 1 — Design Philosophy (why we build this way)

### 1. Mission
Bridge a gap no one else fills: sell proven, breadboard-compatible reference boards that are ready for production, so people who just want a board that works can buy one instead of designing their own.

### 2. Right to repair & perpetuity
- Anyone who buys GDH hardware has the full right to repair it.
- If GDH ceases to exist, hardware already sold stays valuable to whoever owns it, in perpetuity, for as long as it makes sense.
- Anyone is free to fabricate their own board from the published design files. GDH's value is being there for people who would rather buy a proven one and focus their time elsewhere.

### 3. Breadboard-compatible & production-ready
Modules expose standard, hand-friendly interfaces (e.g. castellated pads / 0.1"-compatible breakouts) so they drop onto a breadboard for prototyping, yet are designed for reflow and production use without redesign. The goal is to make modules that are re-usable to minimize landfill waste, save customers money in the long run, and provide the maximum value possible. By using castellated edges instead of 2.54mm pins modules can be easily desoldered on a hot plate vs having to deal with pins that are much more difficult to desolder and often times result in lifted pads and traces, burnt boards, etc.

### 4. Robust by default
Boards favor real-world durability over minimal BOMs. Where the application warrants it, they include proper protection and quality parts. Example: the `crd-mcp2562` CAN transceiver carries layered line protection (ESD/TVS, common-mode choke, transient blocking units, and gas-discharge tubes) and uses AEC-Q200-grade parts exclusively. At a minimum, every module must be rated for -40C to 125C and any components that aren't up to the task will not be used.

---

## Part 2 — Module Conformance (what every module must include)

### 1. Repository layout
Each module lives at `modules/<name>/<version>/`. The name is stable; each fabricated revision is its own version folder — a module *is* a versioned snapshot, and it's version must be silk screened onto the PCB as such.

### 2. Required deliverables
Every module version folder contains:
- `BOM.csv` — bill of materials
- `PickAndPlace.csv` — component placement
- `Schematic.pdf` — human-readable schematic
- `Gerber.zip` — fabrication outputs
- `datasheets/` — one datasheet per orderable part

### 3. BOM & PickAndPlace conventions
- **UTF-8** encoded (EasyEDA exports UTF-16; convert on import).
- **Exclude `CASTELLATED_PAD` rows** — board-edge copper is neither orderable nor placeable, so it belongs in neither the BOM nor the pick-and-place.
- Keep manufacturer annotations as exported (including non-Latin text).
- Every value sits in its correct column (watch for part numbers EasyEDA leaves stranded in the `Name` field).
- The BOM carries a `Datasheet` column linking each part to its file under `datasheets/`.

### 4. Datasheets
- One PDF per orderable part (mechanical-only footprints excluded).
- Named `<ManufacturerPart>_<SupplierCode>.pdf` (e.g. `MCP2562FD-E-SN_C124016.pdf`), with `/` replaced by `-`.
- Sourced from the supplier or manufacturer and verified to match the part.

### 5. Licensing
Design files are dedicated to the public domain under **CC0 1.0 Universal**, consistent with the right-to-repair and perpetuity goals above.

### 6. Reserved for future revisions
The following are intentionally not yet specified; they will be added as conventions settle:
- **Electrical & protection minimums** — baseline protection and derating expectations by board class.
- **Footprint & pin conventions** — pad pitch, pin ordering, silkscreen and labeling rules.
- **Validation & test requirements** — what must be verified before a module version is published.
- **Mechanical & dimensional standards** — board outline, mounting, connector placement.
