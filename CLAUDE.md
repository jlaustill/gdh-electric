# gdh-electric

Open-hardware **module library** (not a software project — no build/test).
Each module lives in `modules/<name>/<version>/` with EasyEDA outputs
(`BOM.csv`, `PickAndPlace.csv`, `Schematic.pdf`, `Gerber.zip`) plus a `datasheets/` folder.

## Licensing
- Design files are dedicated to the public domain under **CC0 1.0** (right-to-repair / perpetual-value philosophy). Default new modules to CC0.

## EasyEDA export quirk
- `BOM.csv` / `PickAndPlace.csv` export as **UTF-16LE, tab-delimited**, unquoted header, double-quoted data fields.
- Convert to UTF-8 before editing: `iconv -f UTF-16 -t UTF-8 file.csv`
- EasyEDA's **PDF** schematic export is low-fidelity (mangled/overlapping labels). Export **SVG** instead (primary artifact — crisp, renders in-browser on GitHub), then generate the PDF from it: `inkscape Schematic.svg --export-type=pdf --export-filename=Schematic.pdf` (or `cairosvg`).

## BOM / PickAndPlace conventions
- Exclude `CASTELLATED_PAD` rows — board-edge copper, not orderable/placeable parts.
- Keep Chinese manufacturer annotations (e.g. `KEMET(基美)`) as-is.
- BOM carries a `Datasheet` column linking each part to `datasheets/<MfgPart>_<LCSC>.pdf`.
- Watch for stranded part numbers: EasyEDA sometimes leaves `Manufacturer Part` empty with the value only in `Name`.

## Datasheets
- Source via LCSC: `curl -A "<browser-UA>" "https://wmsc.lcsc.com/ftps/wm/product/detail?productCode=C<code>"` → JSON `pdfUrl`.
  (The `/wmsc/product/detail` path 404s — use `/ftps/wm/product/detail`; a browser User-Agent is required.)
- Naming: `<ManufacturerPart>_<LCSCcode>.pdf`, with `/` → `-`.
- When LCSC `pdfUrl` is empty, fall back to the manufacturer (e.g. Kyocera AVX: `https://spicat.kyocera-avx.com/datasheet/<part>`). Microchip 403s automated requests.
- Verify each download: header is `%PDF`, and `pdftotext -l 2 file.pdf -` contains the part number.

## Git
- `.claude/settings.local.json` is gitignored. Remote: `github.com/jlaustill/gdh-electric` (public).
