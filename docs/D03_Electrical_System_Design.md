# Panduan Desain Elektronik — Hartanzah Danish 03 Gas (D03)

**Versi:** 1.5 — Fase 1 **module seats** (MP1584 FIXED 5V · DevKitC · MAX31856×3 breakout)  
**Status:** Fungsi gas **locked**; MAIN seats Fase 1 **locked di dokumen/BOM** — Gerber seats masih TBD  
**Modul:** lihat `D03_Module_Optimization_Fase1.md` v1.0


---

## DAFTAR ISI

1. [Ringkasan mesin](#1-ringkasan-mesin)
2. [Aturan safety gas & E-Stop](#2-aturan-safety-gas--e-stop)
3. [Diagram aliran daya](#3-diagram-aliran-daya)
4. [Dua PCB & designator](#4-dua-pcb--designator)
5. [Tabel I/O](#5-tabel-io)
6. [Pin ESP32 & ekspander](#6-pin-esp32--ekspander)
7. [Thermocouple & sensor analog](#7-thermocouple--sensor-analog)
8. [VFD AT1 & servo burner](#8-vfd-at1--servo-burner)
9. [Harness kabel](#9-harness-kabel)
10. [Logika HMI & status](#10-logika-hmi--status)
11. [Hitungan daya & proteksi](#11-hitungan-daya--proteksi)
12. [Modbus / Artisan](#12-modbus--artisan)
13. [Checklist produksi](#13-checklist-produksi)
14. [Keputusan terkunci](#14-keputusan-terkunci)

---

## 1. Ringkasan mesin

| Parameter | Nilai |
|-----------|-------|
| Bahan bakar | **LPG** (gas) — solenoid VX2120 ×2 + modul GX103 |
| Listrik | 220 V AC 1-fasa 50 Hz (L + N + PE) |
| Kapasitas batch | **3 kg** |
| HMI | Nextion **NX8048K070-011C** kapasitif 10" |
| Komunikasi roasting | RS485 Modbus RTU + profil **Danish Series.aset** |
| Sertifikasi target | **CE** (gas appliance + EMC + LVD) |

### Beban motor & drive

| Beban | Motor | Daya | Drive |
|-------|-------|------|-------|
| Drum | NRMV 040 | 550 W | **VFD AT1-2200** (2.2 kW class) |
| Cyclone / exhaust fan | Centrifugal | 135 W | **VFD AT1-1500** (1.5 kW class) |
| Agitator cooling bin | NRMV 040 | 550 W | **Kontaktor K3** — putaran tetap |
| Fan cooling bin | Centrifugal | 135 W | **Kontaktor K4** — on/off |
| Burner valve | Servo RDS3225 | — | **PWM** 50 Hz standar |
| Water suppression | Pump DC | 12 V 5 A | Relay Q_PUMP |

| PCB | File KiCad (rencana) | Isi |
|-----|----------------------|-----|
| PCB-01 | `D03-MAIN-CTRL` | **Module seats:** DevKitC, FIXED 5V buck, MAX31856×3 + RS485/opto/DFPlayer |
| PCB-02 | `D03-IO-PWR` | PSU 24V, kontaktor gas/motor, VFD interface — **industrial locked** (Fase 1 tidak diganti modul) |

---

## 2. Aturan safety gas & E-Stop

### 2.1 Prinsip utama

> **E-Stop memutus semua daya berbahaya di hardware** — gas, VFD, agitator, dan supply kontrol — bukan hanya software.

### 2.2 Rantai gas (hardware)

```
L ──► [SW Main S2SR-S3WA] ──► [F_MAIN MCB] ──► [E-Stop NC #1] ──► [F_GAS] ──► [K_GAS] ──► VX2120 #1 + #2 + GX103
                                                      │
                                    [GX103 flame fail NC] ── seri di jalur 24 VDC coil K_GAS
```

### 2.2b Isolasi sinyal GX103 (M5 — resolved)

GX103 modul burner menyediakan **kontak kering (dry contact)** untuk flame failure — **tidak** langsung ke GPIO.

| Lapisan | Implementasi |
|---------|--------------|
| **Hardware gas cut** | Kontak NC GX103 **seri** di rantai 24 VDC → coil K_GAS (sebelum Q_GAS driver) |
| **Status ke MCU** | Opto PC817 **U_GX** → U11 PCF8574 P0 (interlock hardware) |
| **Burner ON feedback** | Relay dry modul burner → opto **U_BURN** → U11 **P2** |
| **Flame sense feedback** | Relay dry modul burner → opto **U_FLAME** → U11 **P3** |
| **Verifikasi** | Uji continuity kontak GX103 saat flame OK vs fail sebelum commissioning |

```
+24V ──► E-Stop NC#2 ──► GX103 NC (flame OK=closed) ──► Q_GAS ──► K_GAS coil ──► GND
                              │
                         (fail=open) → gas OFF hardware
```

| Komponen | Fungsi |
|----------|--------|
| **K_GAS** LC1D09 **24 VDC coil** | Master gas enable — OFF saat E-Stop, flame fail, atau firmware cut |
| **VX2120-08E2 ×2** | Solenoid gas 220 VAC (main + pilot/secondary per piping) |
| **GX103** | Modul burner 220 VAC — kontak **flame failure** NC ke safety chain |
| **F_GAS** 6 A | Fuse cabang gas |

### 2.3 E-Stop — **WAJIB 2× NC** (audit B1)

> ⚠️ `S2ER-E4RAD` bawaan = **1× NO saja** — **TIDAK CUKUP**.  
> Gunakan **`S2ER-E4R2BD`** (2 NC) **atau** `S2ER-E4RAD` + 2× blok NC **SA-CB**.

| NC | Fungsi |
|----|--------|
| **NC #1** | Memutus **L AC** ke cabang gas + VFD + agitator + PSU kontrol |
| **NC #2** | Memutus **24 VDC** bus coil (K_GAS, K3, K4, REL_VFD_D/F, driver IO) |

Firmware **tidak boleh** menjadi satu-satunya lapisan untuk mematikan gas.

### 2.4 Interlock gas (firmware + hardware)

| Kondisi | Aksi |
|---------|------|
| MQ-6 > threshold | Alarm + **K_GAS OFF** + speaker track 1 |
| Tekanan LPG **&lt; 2,0 kPa** | Low pressure HMI `p5` + **K_GAS OFF** + speaker track 2 |
| GX103 flame fail | Hardware NC buka + firmware `p4` (P0 + P3) |
| E-Stop | Semua OFF |
| Burner OFF button | K_GAS OFF |

### 2.5 Safe shutdown drum

- **Pemicu:** operator menekan tombol **Drum OFF** saat **ET (Environment TC) ≥ 50 °C**
- **Bukan** dipicu oleh STOP roast HMI atau matikan burner saja
- Saat aktif (`p6` ON): drum **tetap RUN** (cooldown) + DFPlayer track 3 tiap 30 s
- Setelah **ET &lt; 50 °C**: drum OFF otomatis, `p6` OFF; operator harus **lepas** tombol drum sebelum ON lagi
- Jika ET sudah &lt; 50 °C saat Drum OFF → drum OFF normal (tanpa safe shutdown)

### 2.6 Motor overheat

- **TC Drum motor &gt; 80 °C** → status alarm HMI `p7` + speaker track 4 tiap 5 s
- **TC Drum motor &gt; 90 °C** → **stop drum** paksa (firmware) + alarm tetap

---

## 3. Diagram aliran daya

### 3.1 Distribusi L mains (setelah SW main + F_MAIN 32A)

```
L ──► [SW Main S2SR-S3WA] ──► F_MAIN 32A ──► E-Stop NC#1 ──┬──► F_GAS 6A ──► K_GAS ──► Solenoid + GX103
                                    ├──► F_VFD_D 10A ──► VFD Drum (AT1-2200)
                                    ├──► F_VFD_F 6A  ──► VFD Fan   (AT1-1500)
                                    ├──► F_MOT 10A   ──► K3 ──► Agitator 550W
                                    ├──► F_MOT2 6A   ──► K4 ──► Cooling fan 135W
                                    └──► F_CTRL 6A ──► EMI ──► PSU 24V

N ──► SW Main ──► semua beban N
PE ──► chassis 1 titik ──► J_PE + VFD ground + TC shield 1 titik
```

### 3.2 Bus kontrol 24 VDC (PSU IO)

```
PSU HDR-100-24 ──► F_24V 4A ──► E-Stop NC#2 ──► +24V bus
  ├──► Coil K_GAS, K3, K4 + REL_VFD_D/F (via driver IO)
  ├──► LRS-35-12 ──► F_LED 500mA ──► +12V_LAMP bus (H1-28 pump + H1L panel button lamps)
  ├──► Buck 6V 3A ──► Servo RDS3225
  └──► IDC pin +24V ke MAIN (buck 5V/3V3 → DFPlayer U9)
```

### 3.3 VFD interface (IO → field)

Setiap AT1 (built-in filter, V/F, PWM):

| Sinyal | Sumber | Catatan |
|--------|--------|---------|
| **L/N** | Setelah F_VFD_* + E-Stop | 220 V single-phase in |
| **FOR / RUN** | REL_VFD_D / REL_VFD_F (G5V-2 dry) | Drum / Fan run command — **bukan** kontaktor beban |
| **AI speed** | MCP4725 0–3,3 V → IO U20 op-amp → **0–10 V** | Dari MAIN via IDC |
| **COM** | VFD common | Referensi AI |

Panel pot **bisa** parallel ke AI (manual override) melalui selector jumper — default **MCU control** untuk Artisan sync.

---

## 4. Dua PCB & designator

### PCB-01 MAIN — `D03-MAIN-CTRL`

| RefDes | Komponen | Fungsi |
|--------|----------|--------|
| U1 / `U1_DEVKIT` | ESP32-DevKitC-32E | MCU pluggable — flash USB DevKit |
| U2 | MCP1700-3.3 | LDO analog → VIN seat TC ×3 |
| U3 / `U3_MOD` | **MP1584 FIXED 5V** modul | Buck **24V→5V** (J9-1) — **LARANG** trimpot; QC 4.90–5.10 V |
| U4/U5/U6 `_MOD` | MAX31856 **breakout** ×3 | TC Bean / Env / Drum motor — pluggable |
| U7 | MAX3485 | RS485 Modbus |
| U8 | CP2102N | **DNP Fase 1** (konflik UART0 vs DevKit) |
| U9 | DFPlayer Mini | Audio |
| U10 | PCF8574AP @0x20 | RS485 DE, relay logic, flags |
| U11 | PCF8574AP @0x21 | Input: GX103 P0, E-Stop P1, burner ON P2, flame FB P3 |
| U12, U13 | MCP4725 ×2 | Setpoint 0–3,3 V → IDC → **IO op-amp 0–10 V** |
| U22 | PCF8591 @0x48 | **ADC I2C** — Pot Drum (bebas GPIO32) |
| U14–U21 | PC817 ×8 | Opto panel buttons (lihat §6.4) |
| U_GX | PC817 | Opto GX103 interlock → U11 P0 |
| U_BURN | PC817 | Opto burner ON relay → U11 P2 |
| U_FLAME | PC817 | Opto flame sense relay → U11 P3 |
| Q1 | BC847C | RS485 DE inverter |
| J1–J3 | PCC-SMP-K | TC panel 3 channel |
| J4 | Phoenix 3p | Landing RS485 dari **U_USB485** (bukan colokan user) |
| J_USB_ART | Neutrik NAUSB-W | Socket USB user → Artisan (FIELD) |
| U_USB485 | Waveshare USB-RS485 iso | Di dalam box → J4 |
| J_H1 | Header 2×16 | Harness panel H1 |
| J_USB | USB-C | Optional/DNP spare (flash utama = DevKit) |
| J9 | IDC 2×10 | Ke PCB IO (+24V pin1 → U3_MOD VIN) |
| J_COLOR | 1×4 | Color sensor (DNP v1) |
| — | L1/L4 inductor diskrit | **Dihapus** Fase 1 (inductor di modul buck) |

### PCB-02 IO — `D03-IO-PWR`

| RefDes | Komponen | Fungsi |
|--------|----------|--------|
| PSU1 | HDR-100-24 | 24 VDC 4.2 A control |
| PSU2 | LRS-35-12 | 12 V aux (pump LED, panel) |
| K_GAS | LC1D09BD 24VDC | Gas master 220 VAC load + **LAEN11N** aux NO → lamp burner |
| K3 | LC1D09BD | Agitator motor power + **LAEN11N** aux NO → lamp cooling |
| K4 | LC1D09BD | Cooling bin fan power |
| REL_VFD_D, REL_VFD_F | G5V-2-DC24 ×2 | VFD FOR dry contact (bukan LC1D09) |
| Q_GAS…Q_PUMP | **AO3400** ×6 (SOT-23) | **Coil driver** dari IDC (logic-level) |
| U20 | **LM358** + Rf/Rg 20k/10k | **0–10 V buffer** VFD AI (VDD +12V_AUX) |
| D2–D6, D_GAS | SS34 / 1N5819 / 1N4007 | Flyback kontaktor/relay/pump |
| F_MAIN…F_CTRL | MCB/Fuse | Proteksi cabang |
| EMI1 | FMAB filter | Cabang PSU saja |
| J1–J8 | Terminal | Field power |
| J9 | IDC 2×10 | Ke MAIN |
| J_PE | Stud M4 | Earth |

### J9 IDC Pinout (D03)

| Pin | Sinyal | Arah | Fungsi |
|-----|--------|------|--------|
| 1 | +24V | IO → MAIN | Supply buck MAIN |
| 2 | GND_PWR | Bersama | Power return |
| 3 | NC | — | — |
| 4 | GND_SIG | Bersama | Signal GND |
| 5 | GAS_EN | MAIN → IO | K_GAS driver |
| 6 | VFD_D_RUN | MAIN → IO | Drum VFD FOR |
| 7 | VFD_F_RUN | MAIN → IO | Fan VFD FOR |
| 8 | AGIT_EN | MAIN → IO | K3 agitator |
| 9 | COOL_FAN_EN | MAIN → IO | K4 cooling fan |
| 10 | PUMP_EN | MAIN → IO | Water pump |
| 11 | DAC_D_0-3V3 | MAIN → IO | Drum speed ref (buffer → 0–10 V) |
| 12 | DAC_F_0-3V3 | MAIN → IO | Fan speed ref (buffer → 0–10 V) |
| 13 | +5V | MAIN → IO | Sense (opsional) |
| 14–20 | SPARE | — | — |

> Detail pin H1/H2A/H2B/H3: lihat **`D03_Harness_Pinout.md`**

---

## 5. Tabel I/O

### INPUT

| # | Nama | Hardware | Tujuan |
|---|------|----------|--------|
| 1 | TC Bean | K-type 3 m | **U4_MOD** breakout |
| 2 | TC Environment | K-type 3 m | **U5_MOD** breakout |
| 3 | TC Drum motor | K-type 3 m | **U6_MOD** breakout |
| 4 | Drum RPM | PRL12-4DN proximity | GPIO14 via opto |
| 5 | Gas pressure | 0–12 bar 5 V sensor | GPIO34 ADC — alarm **&lt; 2,0 kPa** |
| 6 | Gas leak | MQ-6 | GPIO35 ADC + heater |
| 7 | Flame interlock | GX103 NC dry contact | U11 P0 (opto U_GX) |
| 8 | Burner ON feedback | REL_BR_ON dry relay | U11 P2 (opto U_BURN) |
| 9 | Flame sense feedback | REL_BR_FL dry relay | U11 P3 (opto U_FLAME) |
| 10 | Main power | S2SR-S3WA | Mains (bukan GPIO) |
| 11 | Btn Air | **S2TR-P3WABD** | GPIO25 via U14 + lamp 12V via REL_VFD_F NO |
| 12 | Btn Drum | **S2TR-P3WABD** | GPIO26 via U15 + lamp 12V via REL_VFD_D NO |
| 13 | Btn Burner | **S2TR-P3WABD** | GPIO27 via U16 + lamp 12V via K_GAS aux NO |
| 14 | Btn Cooling bin | **S2TR-P3WABD** | **U11 P5** via U17 + lamp 12V via K3 aux NO (Fase1; bukan GPIO38) |
| 15 | Btn Water pump | LED switch 12 V | **U11 P4** via U18 (Fase1; bukan GPIO37) |
| 16 | Pot Air | 10 kΩ | GPIO36 ADC |
| 17 | Pot Burner | 10 kΩ | GPIO39 ADC |
| 18 | Pot Drum | 10 kΩ | **U22 PCF8591 AIN0** (I2C) |
| 19 | E-Stop | **S2ER-E4R2BD** 2NC | NC hardware + U11 P1 sense |
| 20 | RS485 | Modbus RTU 9600 | U7 |
| 21 | Color sensor | Virtual Nextion | DNP v1 |

### OUTPUT

| # | Nama | Driver | Beban |
|---|------|--------|-------|
| 1 | Drum motor | VFD AT1 + DAC | NRMV 550 W |
| 2 | Cyclone fan | VFD AT1 + DAC | 135 W |
| 3 | Agitator | K3 | NRMV 550 W fixed |
| 4 | Cooling fan | K4 | 135 W fixed |
| 5 | Gas train | K_GAS | VX2120 ×2 + GX103 |
| 6 | Burner modulate | Servo PWM | RDS3225 |
| 7 | Water pump | Q_PUMP | 12 V 5 A |
| 8 | Speaker | DFPlayer | 3 W 8 Ω |

---

## 6. Pin ESP32 & ekspander

### 6.1 GPIO ESP32 — **Map v1.2** (Fase 1 DevKit + U11 pump/cool)

| GPIO | Fungsi | Catatan |
|------|--------|---------|
| 1, 3 | UART0 | Flash/monitor USB **DevKit** (U8 CP2102 = DNP Fase 1) |
| 4 | DFPlayer TX | SoftSerial |
| 5, 15, 0 | SPI CS | TC Bean / Env / Drum motor |
| 13 | **Servo PWM** LEDC | RDS3225 — duty=0 @ boot |
| 14 | Proximity PRL12 | NPN + pull-up |
| 16, 17 | UART1 RS485 | GPIO17 TX, GPIO16 RX (pola D01E) |
| 18, 19, 23 | SPI CLK/MISO/MOSI | 3× MAX31856 |
| 21, 22 | I2C | MCP4725 + PCF8574 ×2 + PCF8591 |
| 25–27 | Btn Air / Drum / Burner | PC817 |
| 32, 33 | UART2 Nextion | GPIO32 RX, GPIO33 TX (pola D01E) |
| 34 | ADC | Gas pressure |
| 35 | ADC | MQ-6 |
| 36 | ADC | Pot Air |
| 37 | — | **NC Fase 1** — DevKitC tidak expose; tombol pump → **U11 P4** |
| 38 | — | **NC Fase 1** — DevKitC tidak expose; tombol cool → **U11 P5** |
| 39 | ADC | Pot Burner |

**SPI CS TC Drum motor:** GPIO **0** (pull-up 10k — jangan pull low saat boot).  
**Fase 1 DevKit:** lihat `D03_Module_Optimization_Fase1.md` — flash USB DevKit; **U8 DNP**.

**Pot Drum:** PCF8591 AIN0 — menghindari bentrok UART2 Nextion di GPIO32/33.

### 6.1b PCF8591 @0x48

| Channel | Fungsi |
|---------|--------|
| AIN0 | Pot Drum panel |
| AIN1–AIN3 | Spare |

### 6.1c PCF8574 @0x21 (input)

| Pin | Fungsi |
|-----|--------|
| P0 | GX103 flame interlock (opto U_GX) — hardware NC chain |
| P1 | E-Stop aux sense |
| P2 | **BURNER_ON** relay feedback modul burner (opto U_BURN) |
| P3 | **FLAME_FB** relay feedback modul burner (opto U_FLAME) |
| P4 | **Btn Water pump** (opto U18) — Fase 1 (bukan GPIO37) |
| P5 | **Btn Cooling bin** (opto U17) — Fase 1 (bukan GPIO38) |
| P6–P7 | Spare |

### 6.2 PCF8574 @0x20

| Pin | Fungsi |
|-----|--------|
| P0 | GAS_EN → IDC pin 5 |
| P1 | VFD_D_RUN → IDC pin 6 |
| P2 | VFD_F_RUN → IDC pin 7 |
| P3 | AGIT_EN → IDC pin 8 |
| P4 | COOL_FAN_EN → IDC pin 9 |
| P5 | PUMP_EN → IDC pin 10 |
| P6 | RS485 DE | Via Q1 inverter |
| P7 | DFPlayer BUSY in |

### 6.4 Opto panel (U14–U21)

| RefDes | Input | GPIO / tujuan |
|--------|-------|---------------|
| U14 | Btn Air | GPIO25 |
| U15 | Btn Drum | GPIO26 |
| U16 | Btn Burner | GPIO27 |
| U17 | Btn Cooling | **U11 P5** (Fase 1) |
| U18 | Btn Water pump | **U11 P4** (Fase 1) |
| U19 | Spare | — |
| U20 | Spare | — |
| U21 | Spare | — |

### 6.4b Lampu tombol panel — S2TR-P3WABD (12 VDC)

Model **S2TR-P3WABD** = double push + **LED block DC 12–30 V**. Kontak switch (1NO+1NC) tetap ke opto U14–U17; **lampu terpisah** via harness **H1L** ke IO box.

**Prinsip:** lampu tengah nyala saat **output RUN** (bukan hanya saat jari menekan tombol hijau).

| Tombol | LED X1 ← | LED X2 → | Sumber RUN (NO aux) |
|--------|----------|----------|---------------------|
| **Air** | H1L-2 | H1L-6 GND | **REL_VFD_F** NO |
| **Drum** | H1L-3 | H1L-6 GND | **REL_VFD_D** NO |
| **Burner** | H1L-4 | H1L-6 GND | **K_GAS** LAEN11N aux NO |
| **Cooling** | H1L-5 | H1L-6 GND | **K3** LAEN11N aux NO |

**Bus 12 V (IO PSU2):**

```
LRS-35-12 ──► F_LED 500mA ──► +12V_LAMP ──┬── H1-28 (pump LED)
                                           ├── H1L-1 (common, opsional)
                                           └── COM semua NO aux (VFD relay + kontaktor)
```

**Per lampu (contoh Drum di IO box):**

```
+12V_LAMP ── REL_VFD_D NO ── H1L-3 ──► Panel SW_DRUM LED X1
Panel SW_DRUM LED X2 ── H1L-6 ──► GND (IO PSU2)
```

Saat **REL_VFD_D** energize (drum RUN) → NO close → lampu hijau nyala. Safe shutdown: drum hold → lampu **tetap nyala** sampai drum OFF.

**E-Stop:** NC#2 matikan 24 V → LRS-35-12 off → **semua lampu panel mati** (konsisten safety).

**Harness tambahan:** `CBL_H1L` 6×22 AWG, konektor **J_H1L** 1×6 (panel ↔ IO box, paralel dengan H1 ke MAIN).

### 6.5 Alamat I2C

| IC | Alamat | Fungsi |
|----|--------|--------|
| U10 PCF8574 | 0x20 | Output → IDC |
| U11 PCF8574 | 0x21 | Input GX103 P0, E-Stop P1, burner P2, flame FB P3 |
| U12 MCP4725 | 0x60 | Drum DAC (A0=GND) |
| U13 MCP4725 | 0x61 | Fan DAC (A0=VCC) |
| U22 PCF8591 | 0x48 | Pot Drum AIN0 |

### 6.3 UART

| Port | Baud | Device |
|------|------|--------|
| UART0 | 115200 | USB DevKit (U8 DNP) |
| UART1 | 9600 | RS485 | GPIO17 TX, GPIO16 RX |
| UART2 | 115200 | Nextion | GPIO33 TX, GPIO32 RX |
| SoftSerial | 9600 | DFPlayer |

---

## 7. Thermocouple & sensor analog

### 7.1 Thermocouple (sama ketatnya D01E)

1. Probe **ungrounded**, kabel **shielded 3 m**
2. Shield → GND **1 titik** per connector J1/J2/J3
3. Jarak **≥ 80 mm** dari kabel VFD, contactor, dan gas solenoid
4. Per channel: ferrite bead + 100 pF C0G + LDO U2 shared
5. **Jangan** routing TC parallel bundle dengan 220 V motor

| Channel | Posisi | IC |
|---------|--------|-----|
| Bean | Zona biji drum | U4 |
| Environment | Exhaust/chamber (ET) | U5 |
| Drum motor | Bearing motor drum | U6 |

### 7.2 MQ-6

- Heater **5 V** dari MAIN buck U3 (via H1-25) — **bukan 12 V**
- Warmup 24–48 h first use (firmware)
- ADC dengan divider + RC filter
- Threshold kalibrasi di field (bukan hardcode produksi)

### 7.3 Pressure 0–12 bar (5 V)

- Divider presisi 1% ke ADC 3.3 V
- Firmware: `kPa = bar × 100` dari ADC (sensor max 12 bar)
- **Alarm low:** **&lt; 2,0 kPa** (`GAS_PRESS_LOW_KPA`)
- **Kerja normal:** 2,0–2,3 kPa (commissioning verifikasi di field)
- HMI `n9`: nilai **Xfloat** kPa (val = kPa×10, vvs0=2 vvs1=1); Modbus 30004: ×0,1 bar

---

## 8. VFD AT1 & servo burner

### 8.1 AT1-1500 / AT1-2200

| Item | Drum VFD | Fan VFD |
|------|----------|---------|
| Model | AT1-2200 | AT1-1500 |
| Motor | 550 W | 135 W |
| Input | 1φ 220 V | 1φ 220 V |
| Output | 3φ 220 V Δ | 3φ 220 V Δ |
| Motor wiring | **Delta 220 V** jika ex-380Y | sama |
| Control | FOR + AI 0–10 V | FOR + AI 0–10 V |
| Filter | Built-in | Built-in |

**Catatan produksi:** Motor 550 W NRMV — konfirmasi nameplate (Δ/Y) sebelum commissioning.

### 8.2 Servo RDS3225

- Supply **6 V 3 A** buck dedicated (jangan dari 3.3 V MCU)
- Sinyal **50 Hz PWM** 1–2 ms pulse (LEDC)
- Pot Burner panel → ADC scaling → HMI `n1` 0–100% → servo travel capped by **`SERVO_TRAVEL_MAX_PCT`** (default 20) di `firmware/.../config.h`

---

## 9. Harness kabel

### H2A — TC probe → MAIN (terpisah dari panel)

| Label | Ke |
|-------|-----|
| H2A-01..03 | Bean → MAIN J1 |
| H2A-04..06 | Environment → MAIN J2 |
| H2A-07..09 | Drum motor → MAIN J3 |

### H1 — Panel → MAIN

Lihat **`D03_Harness_Wiring_Diagram.md`** (lengkap) atau **`D03_Harness_Pinout.md`** (ringkas).

### H2B — IO → Mesin

| Label | Ukuran | Ke |
|-------|--------|-----|
| H2B-01..03 | 2.5 mm² | VFD Drum L/N/PE |
| H2B-04..06 | 1.5 mm² | VFD Fan L/N/PE |
| H2B-07..09 | 2.5 mm² | Agitator K3 |
| H2B-10..12 | 1.5 mm² | Cooling fan K4 |
| H2B-13..15 | 1.5 mm² | Gas solenoid + GX103 |
| H2B-16 | 1.0 mm² | Water pump 12 V |

### H3 — Power Safety (wajib)

| Label | Fungsi |
|-------|--------|
| H3-01 | L → SW Main → F_MAIN |
| H3-02 | N mains |
| H3-03 | PE |
| H3-04 | E-Stop NC#1 → distribusi L |
| H3-05 | E-Stop NC#2 → 24 V coil bus |

---

## 10. Logika HMI & status

Firmware: `status_control.cpp` + `gas_safety.cpp`. HMI mirror: `vis pX,1/0` (bukan `.pic`).

| HMI | Label | Sumber | Audio |
|-----|-------|--------|-------|
| `p2` | DRUM | VFD drum RUN (`REL_VFD_D` + feedback) | — |
| `p1` | BURNER | Relay ON modul burner → U11 P2 | — |
| `p3` | GAS LEAK | MQ-6 ADC ≥ threshold | DF track 1 / 3 s |
| `p4` | FLAME FAILURE | GX103 P0 + relay flame P3 | **Tidak ada** |
| `p5` | LOW PRESSURE | Pressure **&lt; 2,0 kPa** | DF track 2 / 3 s |
| `p6` | SAFE SHUTDOWN | Drum OFF + ET ≥ 50 °C | DF track 3 / 30 s |
| `p7` | MOTOR OVERHEAT | TC motor **&gt; 80 °C** (stop drum **&gt; 90 °C**) | DF track 4 / 5 s |
| `p8` | AIR FLOW | Fan VFD RUN | — |
| `p9` | BEAN LOADER | — | **Fase berikutnya** |
| `p10` | DESTONER | — | **Fase berikutnya** |

**DFPlayer (U9, GPIO4):** generate `scripts/gen_status_audio.py` → `0001.mp3`…`0004.mp3` di microSD.

Detail teks suara: `D03_HMI_DEVNOTE.md` §8.4.

### 10.1 Cooling bin (gabungan)

Tombol **Cooling Bin ON** → firmware set **AGIT_EN + COOL_FAN_EN** bersamaan (K3 + K4).

---

## 11. Hitungan daya & proteksi

| Beban | P≈ | Proteksi |
|-------|-----|----------|
| Drum VFD + motor | 550 W | F_VFD_D 10 A |
| Fan VFD | 135 W | F_VFD_F 6 A |
| Agitator | 550 W | F_MOT 10 A, K3 |
| Cool fan | 135 W | F_MOT2 6 A, K4 |
| Gas + GX103 | < 100 W | F_GAS 6 A |
| Control 24 V | ~3 A peak | F_24V 4 A, PSU **HDR-100-24** |
| **Total connected** | ~1.5 kW | **F_MAIN 32 A** |

Starting current: VFD limits inrush — **jangan** DOL agitator bersamaan drum tanpa interlock firmware.

---

## 12. Modbus / Artisan

### Port fisik user (Opsi B — locked, sama D01E)

```
Laptop → CBL_USB_USER → J_USB_ART (panel) → U_USB485 → CBL_RS485 → J4 → U7 MAX3485 → UART1
```

User **hanya 1 kabel USB**. Tanpa dongle. Detail: `wiring/04_USB_Artisan_Panel.md`.

| Item | Locked |
|------|--------|
| Socket panel | **J_USB_ART** Neutrik NAUSB-W |
| Isolator | **U_USB485** Waveshare SKU26547, 120Ω ON |
| PCB landing | **J4** A/B/GND |
| Bukan Artisan | USB DevKit, `J_USB`/`U8` DNP |

File: **`Danish Series.aset`** (sama nama, disalin ke folder D03)

| Parameter | Nilai aset | Firmware D03 |
|-----------|------------|--------------|
| Main device id | 19 (Arduino TC4) | Serial/Modbus path via **U_USB485** COM |
| Extra Modbus | 19200, devices 28/32/44 | Map Air/Burner/SV |
| Slider cmd | `AIR;{}`, `BURN;{}` | Write holding reg burner % + fan % |
| Capacity | 0.2 → set **3.0 kg** | Update di aset saat commissioning |

**Register map:** `D03_Modbus_Register_Map.md` — wajib implementasi firmware.

---

## 13. Checklist produksi

- [ ] Schematic KiCad hierarchical MAIN + IO
- [ ] ERC 0 kedua board
- [ ] PCB layout DRC 0
- [ ] Gas interlock uji E-Stop putus solenoid
- [ ] Flame fail uji GX103
- [ ] MQ-6 + pressure kalibrasi
- [ ] VFD commissioning Delta wiring
- [ ] TC noise test dengan VFD running
- [ ] **Panel lamp** Air/Drum/Burner/Cooling nyala saat RUN (H1L + S2TR-P3WABD)
- [ ] Artisan connect test
- [ ] CE technical file gas + electrical

---

## 14. Keputusan terkunci

| Item | Keputusan |
|------|-----------|
| Mains | 220 V 1φ 50 Hz |
| Batch | 3 kg |
| VFD count | **2** — drum + cyclone fan only |
| Agitator | Fixed speed contactor, **no VFD** |
| E-Stop | Putus **semua** termasuk gas |
| Air flow indicator | Fan ON = green |
| PCB | Split MAIN + IO seperti D01E |
| Artisan profile | Danish Series.aset |
| Artisan port fisik | **Opsi B:** J_USB_ART → U_USB485 → J4 (1× USB user) |
| GX103 flame | 220 VAC kontak → interlock hardware |
| Panel push buttons | **S2TR-P3WABD** 12 VDC + lamp RUN via H1L |
| Servo | PWM standar RDS3225 |
| Safe shutdown | Drum OFF + ET ≥ 50 °C → hold RUN → ET &lt; 50 °C → OFF |
| Tekanan gas alarm | **&lt; 2,0 kPa** |
| Motor overheat | Warn 80 °C / stop drum 90 °C |
| Burner status HMI | Relay feedback modul burner (bukan infer K_GAS) |
| Bean loader / destoner | Fase berikutnya (`p9`/`p10`) |
| Fase 1 module seats | MP1584 **FIXED 5V** + DevKitC + MAX31856 breakout **×3** — pluggable |
| Flash firmware Fase 1 | USB **DevKitC**; **U8**/J_USB = DNP |
| Buck 5V | **FIXED 5.0 V only** · VIN dari **+24 V** · QC **4.90–5.10 V** · LARANG trimpot |
| Tombol pump / cool | **U11 P4 / P5** (bukan GPIO37/38) |
| IO-PWR | Tetap diskrit industrial — tidak diganti modul di Fase 1 |

---

## FILE PROYEK (rencana)

| File | Isi |
|------|-----|
| `D03_Electrical_System_Design.md` | Dokumen ini |
| `D03_Module_Optimization_Fase1.md` | Seats FIXED 5V / DevKit / TC×3 |
| `D03_IO.txt` | Ringkasan I/O terkunci |
| `D03_BOM_Production.csv` | BOM produksi |
| `D03_Modbus_Register_Map.md` | Register Modbus + Artisan |
| `D03_Harness_Pinout.md` | Pin harness H1/H2A/H2B/H3 |
| `D03_Harness_Wiring_Diagram.md` | **Wiring diagram lengkap** |
| `D03_Harness_Wiring_Diagram.pdf` | PDF produksi (gen script) |
| `D03_CE_Technical_File_Template.md` | CE gas + electrical |
| `D03_Paranoid_Audit_Report.md` | Audit final PASS + addendum v2.1 |
| `D03_DEVNOTE.md` | Handoff notes |
| `Danish Series.aset` | Profil Artisan |
| `D03 Notes.txt` | Catatan mekanik burner |
| `KiCad/D03-*` | Fase implementasi PCB |

---

