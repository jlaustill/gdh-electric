mcu-imxrt1062 v0.0.1 — Foundation report (Plan 1 gate, 2026-07-23)
==================================================================

STATUS: PASS — foundation complete, green-lit for Plan 2 (Schematic).

SOURCING (live LCSC stock re-confirmed 2026-07-23)
  U1  MIMXRT1062CVJ5B  C3216692   stock 35     * distributor/mail-in (Digikey 350, Production)
  U3  W25Q128JVSIQ     C97521     stock 74360
  U4  APS6404L-3SQR-ZR C3040877   stock 1647
  J1  TYPE-C-31-M-12   C165948    stock 282870
  U5  TS3USB221ARSER   C128396    stock 3535
  D1  USBLC6-2SC6      C7519      stock 29750
  U6  TPS2116DRLR      C3235557   stock 14193   (sub from DMQR/VSSOP-8; SOT-583)
  U7  TLV62569DBVR     C141836    stock 98665
  L1  ZNR3015ST2R2M    C42429102  stock 5690
  Y1  X322524MSB4SI    C15643     stock 27330
  Y2  Q13FC1350000400  C32346     stock 293975
  D2  ESD5B5.0ST1G     C93623     stock 170020
  U2  MKL02Z32 (PJRC)  N/A                       * customer-supplied / mail-in (no LCSC)

LIBRARY COMPLETENESS (kicad/mcu-imxrt1062.*)
  symbols     12/12 orderable parts
  footprints  13   (12 parts + Castellation_1x27 strip)
  3D models   23   (all parts; U4 APS6404L STEP 404'd at LCSC, WRL present)
  datasheets  12/12 orderable parts (%PDF + part-number verified)

PINMAP
  pinmap.md validated: 54 unique castellations (1..54 complete);
  budget 42 IO + 2 USB1 + 1 PROG + 1 RESET + 2 PWR + 6 GND;
  all Teensy pins 0..41 present; 16-bit GPIO6[31:16] window contiguous
  on Cast 8..23, GND-flanked (7,24); no duplicate balls.

CARRY-FORWARD TO PLAN 2 (Schematic)
  1. U2 MKL02Z32 bootloader chip: create symbol + QFN-16 (3x3, 0.5mm)
     footprint (KiCad standard lib or a blank-MKL02 LCSC part for geometry);
     fetch the MKL02Z32 datasheet (NXP). Customer-supplied, but must be
     placed and wired (RT1062 boot/reset, USB, program-request net).
  2. U4 APS6404L STEP model (cosmetic) — source from AP Memory if a
     mechanical 3D is wanted; WRL suffices for rendering.

RISKS
  - U1 high-temp MCU is distributor-sourced (LCSC trickle). Confirmed
    procurable (Digikey 350, Production) — mail-in to PCBWay like the
    bootloader. Re-check availability before a production run.
