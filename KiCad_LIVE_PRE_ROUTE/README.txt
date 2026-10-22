D03 KiCad LIVE — PRE-ROUTE (Fase 1 seats locked)
Tanggal sync: 2026-07-20

STATUS: GO ROUTE — seats + pad-nets tally BOM / wiring / firmware
  (lihat ../PRE_ROUTE_SENIOR_EE_GATE_BOM_WIRING_FW_2026-07-20.txt)

BUKA
----
  D03-MAIN-CTRL/D03-MAIN-CTRL.kicad_pro
  D03-IO-PWR/D03-IO-PWR.kicad_pro

SEAT LOCK
---------
  U1 = MODULE_ESP32_DEVKITC (USB = atas / -Y)
  U3 = MODULE_MP1584_4P FIXED 5V
  U4/U5/U6 = MODULE_MAX31856_8P
  J_H1 = 2x16 · U12/U13 MCP4725 · U22 PCF8591
  U8 + J_USB = DNP
  MAIN outline ~160x115 mm · tracks MAIN cleared for fresh route

JANGAN
------
  Update PCB from Schematic (schematic may lag)
  Fab dari folder Gerbers/ di atas — itu set STALE (DO_NOT_FAB)

Setelah route + DRC 0: export Gerber baru, timpa PCB-01/PCB-02 Gerbers, hapus DO_NOT_FAB.
