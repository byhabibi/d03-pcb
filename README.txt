D03 Gas Roaster — PAKET PCB MANUFACTURER
Tanggal audit: 2026-07-20 | Electrical v1.5 · Fase 1 · GO ROUTE seats

******************************************************************************
  STATUS
******************************************************************************
  KiCad_LIVE_PRE_ROUTE/  = SEATS BENAR (DevKit / FIXED5V / TC×3) — route di sini
  PCB-01 Gerbers/        = belum diexport (DO_NOT_FAB_STALE.txt)
  PCB-02 Gerbers/        = belum diexport (HOLD_FAB_DRC.txt)
******************************************************************************

MULAI DI SINI
-------------
1. Baca README_Fase1.txt + PRE_ROUTE_SENIOR_EE_GATE_BOM_WIRING_FW_2026-07-20.txt
2. Buka KiCad_LIVE_PRE_ROUTE/*/… .kicad_pro → route
3. schematics/INDEX.html · BOM/D03_BOM_Production.csv
4. Setelah DRC 0: export Gerber baru → timpa PCB-01/PCB-02 · hapus DO_NOT_FAB

ISI PAKET
---------
KiCad_LIVE_PRE_ROUTE/                ★ source route
BOM/ · docs/ · schematics/ · QC/
PCB-01_MAIN-CTRL/Gerbers/            placeholder — export setelah DRC 0
PCB-02_IO-PWR/Gerbers/               placeholder — export setelah DRC 0
PRE_ROUTE_*.txt                      seat gate evidence

BOM LOCK FASE 1
- U1 = ESP32-DevKitC-32E (bukan WROOM)
- U3 = MP1584-FIXED-5V (VIN +24V; LARANG trimpot; QC 4.90–5.10 V)
- U4/U5/U6 = MAX31856 breakout ×3
- U8 + J_USB = DNP
- L1/L4 discrete = TIDAK ada
- K_GAS/K3/K4 = LC1D09BD 24 VDC (IO)

SPESIFIKASI PCB (setelah Gerber baru valid)
- 2 layer FR4 1.6 mm · Cu 1 oz · mask hijau · silk putih · ENIG disarankan

KONTAK ENGINEER sebelum PO fabrikasi.
