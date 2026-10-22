# D03 Fase 1 — Module-on-PCB Optimization

**Versi:** 1.0 — MP1584 **FIXED 5V** locked (no trimpot) · DevKitC · MAX31856 breakout ×3  
**Status:** LOCKED arah desain — **MAIN/IO seats Fase 1 applied 2026-07-20** (placement + pad nets). **Routing = user.** Gerber = HOLD until DRC 0.  
**Referensi:** `D03_Electrical_System_Design.md` v1.5 · `D03_BOM_Production.csv` · `PRE_ROUTE_SENIOR_EE_GATE_BOM_WIRING_FW_2026-07-20.txt`  
**Scope:** Kursi modul di **MAIN** saja — IO-PWR gas / VFD / kontaktor **tetap industrial diskrit**.

---

## 1. Ringkasan keputusan

| Item | Keputusan Fase 1 |
|------|------------------|
| Arsitektur PCB | Tetap **2 board**: `D03-MAIN-CTRL` + `D03-IO-PWR` |
| Mounting modul | **Header 2.54 mm — pluggable** (backplane / module rack) |
| Diganti di MAIN | Buck **FIXED 5V**, ESP32-**DevKitC-32E**, MAX31856 **breakout ×3** |
| Buck 5V | **FIXED step-down 5.0 V only** — **LARANG** adjustable / trimpot |
| VIN buck | **+24 V** dari J9 pin1 (HDR-100-24) — modul harus VIN **≥ 28 V** abs max |
| Tidak diganti | Kontaktor gas/VFD/MOSFET/fuse/EMI di IO; opto; RS485; DFPlayer; PCF; DAC; ADC I2C |
| Tujuan | Junior-friendly: mudah dimengerti, diganti maintenance, QC jelas |

```
IO-PWR (industrial, locked)
  PSU 24V ──J9 pin1──► MAIN U3_MOD (MP1584 FIXED 5V) ──► +5V_SYS
                                              │
                                              ▼
                                    U1_DEVKIT (ESP32-DevKitC)
                         ┌────────────┼────────────┬────────────┐
                         ▼            ▼            ▼            ▼
                   U4_MOD Bean   U5_MOD Env   U6_MOD Motor   Rest (opto/…)
```

---

## 2. Apa yang diganti / tidak diganti

### Diganti (MAIN only)

| Seat silk | RefDes | Was (diskrit) | Becomes (modul) |
|-----------|--------|---------------|-----------------|
| `BUCK 5V FIXED` / `U3_MOD` | U3 | MP1584EN SOT-23-6 + L1/L4 | **Modul buck FIXED 5V** ≥3A, **no pot**, VIN OK @24V |
| `ESP32 DevKit` / `U1_DEVKIT` | U1 | ESP32-WROOM-32E | ESP32-DevKitC-32E dual 1×19 |
| `BEAN TC MODULE` / `U4_MOD` | U4 | MAX31856ASA+ SOIC | Breakout MAX31856 |
| `ENV TC MODULE` / `U5_MOD` | U5 | MAX31856ASA+ SOIC | Breakout MAX31856 |
| `MOTOR TC MODULE` / `U6_MOD` | U6 | MAX31856ASA+ SOIC | Breakout MAX31856 |

### Tidak diganti (Fase 1)

- IO-PWR: PSU1/2, K_GAS / K3 / K4, VFD relays, MOSFET drivers, EMI, MCB  
- MAIN: opto, MAX3485, PCF8574 ×2, MCP4725 ×2, PCF8591, DFPlayer seat  
- Servo LM2596 6V di IO (bukan scope Fase 1 MAIN)

### Implikasi service / refdes (JANGAN copy D01E buta)

| Item | Rule D03 |
|------|----------|
| Flash | USB pada **DevKitC** |
| CP2102 + J_USB | **`U8` + `J_USB` = DNP** — **bukan U11** (U11 = PCF8574 input gas) |
| L1 / L4 discrete | **Dihapus** — inductor di dalam modul |
| U2 MCP1700 | Tetap → +3V3_ANA ke VIN seat U4/U5/U6 |

---

## 3. Spec buck FIXED 5V (locked)

| Parameter | Nilai |
|-----------|--------|
| Tipe | **Fixed 5.0 V** step-down (MP1584-class / Mini360-5V fixed / setara) |
| **Dilarang** | Modul **adjustable** + trimpot |
| Input | **+24 V** (J9-1) — pastikan datasheet modul **VIN max ≥ 28 V** |
| Rating | ≥ **3 A** continuous |
| QC | VOUT vs GND = **4.90 … 5.10 V** @ +24 V in, **tanpa** Nextion dulu |
| Reject | Di luar rentang → **REJECT** — jangan “setel pot” |

### Seat U3_MOD (3 pin)

| Pad | Net |
|-----|-----|
| 1 VIN | +24V_IN (J9 pin1) |
| 2 GND | GND_PWR |
| 3 VOUT | +5V_SYS |

Bypass: C dekat VIN & VOUT tetap di board.

---

## 4. GPIO / DevKit — map v1.2 + remap wajib

Firmware pin map **utama tetap** (`config.h` v1.2).  
**Pengecualian Fase 1 (wajib):** DevKitC-32E **tidak** mengeluarkan **GPIO37 / GPIO38** di header.

| Fungsi | Was | Fase 1 (locked) |
|--------|-----|-----------------|
| Btn Water pump | GPIO37 via U18 | **U11 P4** (tetap lewat opto U18 → expander) |
| Btn Cooling bin | GPIO38 via U17 | **U11 P5** (tetap lewat opto U17 → expander) |

Firmware harus baca cooling/pump dari **U11**, bukan `digitalRead(37/38)`.  
GPIO37/38 di BOM/net KiCad lama → **NC / jangan route ke DevKit**.

### DevKitC pad → net (USB di atas / −Y) — selain remap di atas

| Net | GPIO | Pad DevKit (approx) |
|-----|------|---------------------|
| +5V_SYS | — | **L19 `5V`** |
| GND | — | L14, R1, R7 |
| POT_AIR | 36 | L3 (VP) |
| POT_BURNER | 39 | L4 (VN) |
| GAS_PRESS | 34 | L5 |
| MQ6 | 35 | L6 |
| UART2_RX Nextion | 32 | L7 |
| UART2_TX Nextion | 33 | L8 |
| BTN_AIR | 25 | L9 |
| BTN_DRUM | 26 | L10 |
| BTN_BURNER | 27 | L11 |
| PROX_RPM | 14 | L12 |
| SERVO_PWM | 13 | L15 |
| SPI_MOSI | 23 | R2 |
| I2C_SCL | 22 | R3 |
| UART0 | 1/3 | R4/R5 — **DevKit USB only** |
| I2C_SDA | 21 | R6 |
| SPI_MISO | 19 | R8 |
| SPI_SCK | 18 | R9 |
| SPI_CS_BEAN | 5 | R10 |
| UART1_TX RS485 | 17 | R11 |
| UART1_RX RS485 | 16 | R12 |
| DFPLAYER_TX | 4 | R13 |
| SPI_CS_MOTOR | 0 | R14 — **pull-up 10k; hati-hati boot** |
| SPI_CS_ENV | 15 | R16 |

**Jangan** kawat L1 `3V3` DevKit ke rail board.

### MAX31856 seat U4 / U5 / U6 — pin 1…8

| Pad | Nama modul | Net |
|-----|------------|-----|
| 1 | VIN | +3V3_ANA |
| 2 | 3Vo | NC |
| 3 | GND | GND_SIG |
| 4 | DO | SPI_MISO |
| 5 | CS | CS Bean / Env / Motor |
| 6 | CLK | SPI_SCK |
| 7 | DIN | SPI_MOSI |
| 8 | DRDY | NC |

TC: H2A → **J1/J2/J3** → T+/T− modul.

---

## 5. Mekanik / ukuran board

Outline MAIN sekarang **130×95 mm** mungkin sempit untuk DevKit + buck + **3×** breakout + standoff.  
Fase 1 boleh **perbesar outline** atau susun zonasi ketat (lihat L2 placement — TBD). Retensi nylon M2.5 **wajib**.

---

## 6. Checklist QC junior

### Incoming
- [ ] Buck = **FIXED 5V**, **no trimpot**, VIN rated **≥28 V**
- [ ] DevKitC-32E; MAX31856 breakout **×3**

### Board
- [ ] Silk: `BUCK 5V FIXED`, `ESP32 DevKit`, `BEAN/ENV/MOTOR TC MODULE`
- [ ] Continuity J9-1 → U3 VIN; VOUT → +5V_SYS
- [ ] U8/J_USB **tidak** dipopulasi bersamaan DevKit

### Power
- [ ] VOUT **4.90…5.10 V** @ 24 V in
- [ ] Reject jika di luar rentang

### Functional
- [ ] Flash via USB DevKit
- [ ] 3× TC baca; swap modul tanpa solder
- [ ] Tombol pump/cool lewat **U11 P4/P5**

---

## 7. Delivery / Gerber

| Item | Status |
|------|--------|
| BOM + dokumen ini | Truth Fase 1 |
| Gerber MAIN lama (WROOM / MP1584EN / L4 / SOIC×3) | **STALE — jangan fab** |
| IO-PWR Gerber | Industrial; tunggu DRC 0 engineer |
| Junior L0–L3 SVG | Menyusul (pola D01E) |

---

## 8. File terkait

| File | Peran |
|------|-------|
| `D03_Module_Optimization_Fase1.md` | Dokumen ini |
| `D03_BOM_Production.csv` | BOM seats |
| `D03_Electrical_System_Design.md` | Spec sistem v1.5+ |
| `D03_IO.txt` | Ringkasan locked v1.4+ |
| `firmware/.../config.h` | Update baca U11 P4/P5 |
