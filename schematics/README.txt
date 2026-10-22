D03 Schematics L0 + L1 + L2 + L3 (SVG only)
==========================================

Mulai: INDEX.html

L0 Blueprint:
  D03_ENGINEERING_BLUEPRINT.svg

L3 manufacturing (13 SVG) — kelas industri:
  L3_01_NET_DEFINITION.svg
  L3_02_DESIGN_RULES.svg          ← Layer + Track width + Via
  L3_03_ROUTING_PRIORITY.svg
  L3_04_DO_NOT.svg                ← termasuk larangan sejajar
  L3_05_FOOTPRINT_ORIENTATION.svg
  L3_06_ASSEMBLY_CONSTRAINTS.svg   ← Keepout antena/buck/TC
  L3_07_GERBER_CHECKLIST.svg
  L3_08_FAB_NOTES.svg
  L3_09_BOM_LOCK.svg
  L3_10_COMMON_MISTAKES.svg
  L3_11_COMPONENT_PROXIMITY.svg   ← Harus dekat / boleh jauh
  L3_12_GND_STRATEGY.svg
  L3_13_FINAL_ROUTE_CHECKLIST.svg ← 24V→5V→GND→SPI→TC→UART→I2C→GPIO→Silk

Regen urutan (pack):
  python scripts/gen_d03_schematics_pack.py

Atau manual:
  python scripts/gen_engineering_blueprint_svg.py
  python scripts/gen_board_schematics_svg.py
  python scripts/gen_board_netmap_l2_svg.py
  python scripts/gen_l3_manufacturing_svg.py   ← terakhir (INDEX penuh)

Truth:
  config.h · D03_Electrical_System_Design.md
  D03_BOM_Production.csv · pcb_nets.py
  D03_Module_Optimization_Fase1.md
