# D03 Harness Pinout — Referensi Produksi

**Versi:** 1.3 — Fase 1 (pump/cool → U11 P4/P5)  
**Referensi:** `D03_Electrical_System_Design.md` v1.5 · `D03_Module_Optimization_Fase1.md`

---

## H1 — Panel → MAIN (J_H1 2×16 = 32 pin)

| Pin | Label | Warna | Ke | Catatan |
|-----|-------|-------|-----|---------|
| 1 | H1-01 | COKLAT | Prox PRL12 OUT → GPIO14 | NPN sinking, pull-up 10k→3V3 |
| 2 | H1-02 | MERAH | Prox PRL12 +24V | Dari IO bus 24V via Fused tap |
| 3 | H1-03 | BIRU | Prox PRL12 0V | GND_PWR |
| 4 | H1-04 | PUTIH/MERAH | Btn Air → U14 opto | |
| 5 | H1-05 | PUTIH/BIRU | Btn Drum → U15 opto | |
| 6 | H1-06 | PUTIH/HIJAU | Btn Burner → U16 opto | |
| 7 | H1-07 | PUTIH/KUNING | Btn Cooling → U17 → **U11 P5** | Fase1; bukan GPIO38 |
| 8 | H1-08 | PUTIH/ORANYE | Btn Pump → U18 → **U11 P4** | Fase1; bukan GPIO37; LED 12V dari IO PSU2 |
| 9 | H1-09 | ABU | E-Stop aux → U11 P1 via opto | Hardware NC terpisah H3 |
| 10 | H1-10 | MERAH MUDA | Pot Air wiper → GPIO36 | |
| 11 | H1-11 | HIJAU MUDA | Pot Burner wiper → GPIO39 | |
| 12 | H1-12 | UNGU | Pot Drum wiper → U22 AIN0 | |
| 13 | H1-13 | ORANYE | Pot +3.3V (shared) | Dari MAIN LDO |
| 14 | H1-14 | HITAM | Pot GND (shared) | GND_SIG |
| 15 | H1-15 | MERAH | Nextion 5V | Dari MAIN U3 buck |
| 16 | H1-16 | HITAM | Nextion GND | |
| 17 | H1-17 | HIJAU | Nextion TX → **GPIO32** RX | MCU RX |
| 18 | H1-18 | BIRU | Nextion RX → **GPIO33** TX | MCU TX |
| 19 | H1-19 | KUNING | RS485 A → J4 | **Internal only** — user Artisan = J_USB_ART (bukan H1) |
| 20 | H1-20 | HIJAU MUDA | RS485 B → J4 | Internal / jumper U_USB485; prefer CBL_RS485 direct → J4 |
| 21 | H1-21 | HITAM | RS485 GND | Internal |
| 22 | H1-22 | MERAH | Pressure +5V | Dari MAIN 5V |
| 23 | H1-23 | COKLAT | Pressure sig → GPIO34 | |
| 24 | H1-24 | HITAM | Pressure GND | |
| 25 | H1-25 | MERAH | MQ-6 heater +5V | **5V** dari MAIN (bukan 12V) |
| 26 | H1-26 | COKLAT | MQ-6 sig → GPIO35 | |
| 27 | H1-27 | HITAM | MQ-6 GND | |
| 28 | H1-28 | PUTIH | Pump LED +12V | Dari IO PSU2 LRS-35-12 |
| 29 | H1-29 | MERAH MUDA | Speaker + | U9 DFPlayer SPK+ |
| 30 | H1-30 | PUTIH | Speaker − | U9 DFPlayer SPK− |
| 31–32 | SPARE | — | J_COLOR DNP | Future color sensor |

---

## H1L — Panel lamps → IO (J_H1L 1×6)

Harness **CBL_H1L** terpisah dari H1 (panel depan ↔ IO box). Lampu = **S2TR-P3WABD** LED block 12 VDC.

| Pin | Label | Warna | Ke (Panel) | Ke (IO) |
|-----|-------|-------|------------|---------|
| 1 | H1L-1 | PUTIH | +12V_LAMP bus (opsional tie) | LRS-35-12 via **F_LED 500mA** |
| 2 | H1L-2 | KUNING | SW_AIR LED **X1** | **REL_VFD_F** NO |
| 3 | H1L-3 | HIJAU | SW_DRUM LED **X1** | **REL_VFD_D** NO |
| 4 | H1L-4 | MERAH MUDA | SW_BURN LED **X1** | **K_GAS** LAEN11N aux NO |
| 5 | H1L-5 | BIRU MUDA | SW_COOL LED **X1** | **K3** LAEN11N aux NO |
| 6 | H1L-6 | HITAM | LED **X2** semua tombol (GND) | PSU2 GND |

**Wiring tiap tombol (panel):**

```
H1L-n (switched +12V) ──► LED X1
LED X2 ──► H1L-6 GND
```

Kontak switch (NO hijau/merah) tetap lewat **H1** ke MAIN opto — tidak berubah.

---

## H2A — TC probe → MAIN (shielded, terpisah)

| Label | Warna TC | Ke MAIN |
|-------|----------|---------|
| H2A-01..03 | COKLAT/PUTIH/HIJAU+shield | J1 Bean |
| H2A-04..06 | COKLAT/PUTIH/HIJAU+shield | J2 Environment |
| H2A-07..09 | COKLAT/PUTIH/HIJAU+shield | J3 Drum motor |

Shield → GND **1 titik** di connector PCB. Jangan bundle dengan 220 V motor.

---

## H2B — IO → Field

| Label | mm² | Ke |
|-------|-----|-----|
| H2B-01..03 | 2.5 | VFD Drum L/N/PE |
| H2B-04..06 | 1.5 | VFD Fan L/N/PE |
| H2B-07..09 | 2.5 | Agitator K3 |
| H2B-10..12 | 1.5 | Cooling fan K4 |
| H2B-13..15 | 1.5 | Gas solenoid + GX103 220 VAC |
| H2B-16 | 1.0 | Water pump 12 V + / − |
| H2B-17 | 0.5 | Servo RDS3225 PWM → GPIO13 (via MAIN harness tap) |
| H2B-18..19 | 0.5 | Servo +6 V / GND dari IO buck |
| H2B-20 | 0.5 | REL_BR_ON dry NO → U_BURN opto (burner ON) |
| H2B-21 | 0.5 | REL_BR_FL dry NO → U_FLAME opto (flame sense) |

**Burner feedback (GX103 modul):** pasang 2 relay dry 220 VAC (REL_BR_ON, REL_BR_FL) di modul burner. Kontak NO ke opto U_BURN/U_FLAME di MAIN → U11 P2/P3. Common relay ke GND_SIG; jangan parallel dengan GX103 NC interlock (P0).

---

## H3 — Power safety

| Label | Fungsi |
|-------|--------|
| H3-01 | L → SW_MAIN → F_MAIN |
| H3-02 | N mains |
| H3-03 | PE → J_PE |
| H3-04 | E-Stop NC#1 → distribusi L |
| H3-05 | E-Stop NC#2 → +24 V coil bus |

---

## J9 IDC (MAIN ↔ IO)

| Pin | Net | Level |
|-----|-----|-------|
| 1 | +24V | Power |
| 2 | GND_PWR | Power |
| 4 | GND_SIG | Signal |
| 5 | GAS_EN | 3.3 V logic |
| 6 | VFD_D_RUN | 3.3 V logic |
| 7 | VFD_F_RUN | 3.3 V logic |
| 8 | AGIT_EN | 3.3 V logic |
| 9 | COOL_FAN_EN | 3.3 V logic |
| 10 | PUMP_EN | 3.3 V logic |
| 11 | DAC_D_0-3V3 | Analog ref |
| 12 | DAC_F_0-3V3 | Analog ref |

---

## Pot manual override (jumper JP1/JP2 di IO)

| Jumper | Posisi A | Posisi B |
|--------|----------|----------|
| JP1 Drum AI | MCU (default) | Panel pot parallel |
| JP2 Fan AI | MCU (default) | Panel pot parallel |

---


