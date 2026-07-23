# mcu-imxrt1062 BOM skeleton

Locked 2026-07-23. Every part verified against the LCSC detail API
(`productModel` / `stockNumber` / `pdfUrl`) unless marked **mail-in**.

**Sourcing model:** this module is PCBWay-fabbed with a **mail-in / customer-supplied**
assembly step (already required for the PJRC bootloader chip), so two parts are
distributor-sourced rather than LCSC-assembled. All others are LCSC-orderable.

| RefDes | Function | Manufacturer Part | LCSC | Package | Stock | Datasheet source |
|--------|----------|-------------------|------|---------|-------|------------------|
| U1 | MCU (high-temp C-grade) | NXP **MIMXRT1062CVJ5B** | C3216692 *(trickle 35)* — **mail-in** | LFBGA-196 (12×12) | Digikey 350, Production, ~$16.61 | NXP (no LCSC pdf) |
| U2 | Bootloader (pre-programmed) | **MKL02Z32VFG4** (PJRC) | **N/A — PJRC, mail-in** | QFN-16 (3×3) | PJRC store | PJRC / NXP |
| U3 | QSPI flash 16MB (FlexSPI1) | Winbond **W25Q128JVSIQ** | C97521 | SOIC-8 208mil | 74,900 | LCSC |
| U4 | QSPI PSRAM 8MB (FlexSPI2) | APMemory **APS6404L-3SQR-ZR** | C3040877 | USON-8 | 1,647 | AP Memory (no LCSC pdf) |
| J1 | USB-C receptacle 2.0 16P | **TYPE-C-31-M-12** | C165948 | SMD 16-pin | 283,010 | LCSC |
| U5 | USB 2.0 HS 2:1 mux | TI **TS3USB221ARSER** | C128396 | UQFN-10 | 3,535 | LCSC |
| D1 | USB ESD array | ST **USBLC6-2SC6** | C7519 | SOT-23-6 | 29,750 | LCSC |
| U6 | Priority power mux | TI **TPS2116DRLR** | C3235557 | SOT-583 | 14,193 | LCSC |
| U7 | 3.3V buck | TI **TLV62569DBVR** | C141836 | SOT-23-5 | 98,675 | LCSC |
| L1 | Core DCDC inductor 2.2µH | **ZNR3015ST2R2M** | C42429102 | 3×3mm shielded, 2A | 5,690 | LCSC |
| Y1 | Crystal 24 MHz | **X322524MSB4SI** | C15643 | SMD3225-4P, 20pF | 27,330 | LCSC |
| Y2 | RTC crystal 32.768 kHz | Epson **Q13FC1350000400** | C32346 | SMD3215-2P, 12.5pF | 293,975 | LCSC |
| D2 | VBUS TVS 5V low-cap | onsemi **ESD5B5.0ST1G** | C93623 | SOD-523 | 170,020 | LCSC |

## Substitution notes
- **U6:** `TPS2116DMQR` (VSSOP-8) is not carried by LCSC (and TI's mainstream package
  is SOT-583, not VSSOP-8). Adopted **TPS2116DRLR (SOT-583)** — same silicon, same
  2-input priority-mux function. Footprint comes from easyeda2kicad for C3235557.
- **U1:** kept the industrial high-temp `CVJ5B` (the project's whole premise). LCSC
  stock is a trickle (35), so it is **distributor-sourced (Digikey/Mouser/Arrow) +
  mail-in to PCBWay**, exactly like the bootloader chip. easyeda2kicad can still
  generate its symbol/footprint/3D from LCSC code C3216692 (the part exists in LCSC's
  DB even at low stock).

## Package-note corrections vs the plan baseline
- U7 `TLV62569DBVR` = SOT-23-**5** (DBV = 5-pin), not SOT-23-6.
- U4 `APS6404L-3SQR-ZR` real package = **USON-8** (the standard Teensy 4.1 PSRAM part).

## Alternatives on file (if a primary goes short)
- L1: `YNR5020-2R2N` (C341020) — 2.2µH 2.9A, 5×5mm, more headroom.
