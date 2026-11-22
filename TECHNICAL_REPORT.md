# Suzuki Grand Vitara 1.9 DDiS (F9Q / Bosch EDC16C3-10.25) — Full ECU Analysis, Map Addresses & Calibration Report

This document provides the complete technical analysis, hex map addresses, N75/VNT and EGR/TVA root-cause fixes, and verified calibration details for the **Suzuki Grand Vitara 1.9 DDiS (`F9Q` engine, 129 HP / 300 Nm stock, SW `1037382039P500_V50`, HW `8200638060`, Bosch `EDC16C3-10.25`)** tuning package (`Stage 1`, `Stage 1+`, `A/C 950 RPM`, and `4300 RPM Popcorn`).

---

## 1. Vehicle, ECU & Checksum Identification

- **Vehicle:** Suzuki Grand Vitara 1.9 DDiS (`F9Q-B264`, 5-speed manual, 4x4)
- **Factory Output:** 129.2 HP @ 3750 RPM / 300.0 Nm @ 2000 RPM
- **ECU:** Bosch `EDC16C3-10.25` (Renault-Suzuki F9Q architecture)
- **Software Version (SW):** `1037382039P500_V50` (`V30_W84`)
- **Hardware Number (HW):** `8200638060`
- **File Size:** `1,048,576 bytes` (1.00 MB — Kess V2 OBD full read)
- **Checksum Region & Rule:**
  - The 32-bit Big-Endian word sum `SumBE32(0x0C0000 .. 0x0FDF7F)` must always equal **`0xD01FE500`**.
  - The 4-byte checksum word resides at **`0x0FDF7C .. 0x0FDF7F`**.

---

## 2. Verified Binary Inventory

| File Name | Power / Torque | Rev Limiter | Checksum (`0x0FDF7C`) | `SumBE32` | Description & Intended Use |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **`original_suzuki_fabrika.bin`** | 129 HP / 300 Nm | Stock (4800 RPM) | `0xA6A826E2` | `0xD01FE500` | Untouched factory reference binary (factory DPF-off calibration). |
| **`org.bin`** | 129 HP / 300 Nm | Stock (4800 RPM) | `0xB32F5EFA` | `0xD01FE500` | Earlier stock OBD read. |
| **`Suzuki_Grand_Vitara_Stage1_Popcorn_EGR_N75_OK.bin`** | ~158 HP / 355 Nm | **4300 RPM Popcorn** | `0x1CF9D723` | `0xD01FE500` | Mild **Stage 1 + 4300 RPM Hardcut Popcorn** + Full EGR & TVA OFF + N75 Active & Protected. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | Stock (4800 RPM) | `0xC8F8793B` | `0xD01FE500` | **Stage 1+ (Recommended Daily Limit)** + Full EGR & TVA OFF + N75 Active & Protected. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Klima950_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | Stock (4800 RPM) | `0xC7CC780F` | `0xD01FE500` | **Stage 1+ + A/C-ON 950 RPM Idle (`0x0C1828`, A/C OFF stays 800 RPM)** + EGR/TVA OFF + N75 Protected. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Popcorn_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | **4300 RPM Popcorn** | `0xE45C90CC` | `0xD01FE500` | **Stage 1+ + 4300 RPM Hardcut Popcorn** + Full EGR & TVA OFF + N75 Protected. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Popcorn_Klima950_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | **4300 RPM Popcorn** | `0xE3308FA0` | `0xD01FE500` | **Stage 1+ + 4300 RPM Popcorn + A/C-ON 950 RPM Idle (`0x0C1828`, A/C OFF 800 RPM)** + EGR/TVA OFF + N75 Protected. |

---

## 3. N75 Turbo Solenoid Root-Cause Analysis & Fixes

### 3.1. Startup N75 Inactivity Fix
- **Root Cause (in legacy files):** The N75 startup delay timers (`PCR_tiStrtDelay`) at `0x0F0788` (`150` = 1.5 s) and `0x0F078C` (`700` = 7.0 s) had been corrupted to `0xFFFF` (~11 min), and PID window maps (`0x0EF7EE..0x0EF8FD`) had been overwritten with `2500`.
- **Applied Fix:**
  - Restored `0x0F0788..0x0F078D` to factory **`(150, 100, 700)`**.
  - Restored `0x0EF7EE..0x0EF8FD` PID window maps to **100% factory stock**.
  - Result: N75 pulls vacuum cleanly ~1.5 s after startup.

### 3.2. Driving Limp-Mode & VNT Overboost Protection Fix
- **Root Cause:**
  1. DSM force-error bytes at `0x0CAAC1..0x0CAAC2` (`P0243`) and `0x0CAAF8..0x0CAAFD` (`P0638`) had been set to `0x01`.
  2. `DSM_InhDfp` inhibition blocks for `P0243` (`0x0CC9C2..0x0CC9D8`), `P0033`/`P0034` (`BPACD`: `0x0CC6BC..0x0CC6E0`), and `P0638` (`0x0CCB92..0x0CCBB5`) triggered `Fid_PCR_MAX` (turbo shutoff limp mode).
  3. An earlier workaround had clamped `PCR_rMin_CUR` (`0x0EFAD4..0x0EFAF3`), `PCR_rMin_C` (`0x0EFFD4`), `PCR_rDflVal` (`0x0F06FE`), and `0x0F0712` to `6144` (`75%` duty), preventing VNT vanes from opening to bleed boost at high RPM, and had overwritten 6 bytes past Deviation Curve 4 (`0x0F06D8..0x0F06DD`).
- **Applied Fix (Verified Safe):**
  - Restored `0x0CAAC1..0x0CAAC2` and `0x0CAAF8..0x0CAAFD` to factory **`0x00`**.
  - Disabled fault classes for `P0033 / P0034` (`0x0CB454..0x0CB45F`) and `P0243` (`0x0CB67C`, `0x0CB681`) -> **`0x00`**.
  - Zeroed all turbo-cut `DSM_InhDfp` blocks for `P0033 / P0034` (`0x0CC6BC..0x0CC6C1`, `0x0CC6CA..0x0CC6D0`, `0x0CC6DB..0x0CC6E0`) and `P0243` (`0x0CC9C2..0x0CC9D8`) so minor exhaust/boost leaks never trigger limp-mode turbo shutoff.
  - Set `PCR_pDvtMax` (`0x0F043A`) = **`+650 mbar`** and `PCR_pDvtMin` (`0x0F043C`) = **`-650 mbar`**; widened the 4 deviation curves (`0x0F0646..0x0F06D7`) floor to **`650 / 750 mbar`**; restored the overwritten 6 bytes at **`0x0F06D8..0x0F06DD` (`1490, 1500, 1800`)** to factory stock.
  - Restored `0x0EFAD4..0x0EFAF3`, `0x0EFFD4`, `0x0F06FE` to factory **`410` (`5%` min duty)** and `0x0F0712` to factory **`414`**, allowing unrestricted VNT boost bleed at high load/RPM.

### 3.3. VNT Base Control Safety
- **5x Main VNT / N75 Maps (`0x0EA334`, `0x0EA578`, `0x0EA7BC`, `0x0EAA14`, `0x0EB33E`):** 100% factory stock.
- **Closed-Loop PID (`0x0EF7CA..0x0EFAD3`) & Min/Max Limits (`0x0EFB02..0x0EFFD3`):** 100% factory stock.

---

## 4. Complete EGR, TVA (`P0638`) & DPF Status

| Region / Map | Address Range | Size | Stock Value | Modified Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Main EGR Hysteresis 1** | `0x0C7084 .. 0x0C709F` | 28 B | `400 .. 9000` | **`0`** | Upper EGR activation threshold zeroed. |
| **Main EGR Hysteresis 2** | `0x0C70BE .. 0x0C70D9` | 28 B | `2700 .. 3550` | **`0`** | Lower EGR activation threshold zeroed. |
| **Aux EGR Hysteresis** | `0x0C6F18 .. 0x0C6F77` | 96 B | `400 .. 9000` | **`0`** | Auxiliary EGR hysteresis curves zeroed. |
| **Regen EGR Curve** | `0x0C7008 .. 0x0C7023` | 28 B | `2000 .. 3500` | **`0`** | Axis (`0x0C7004..0x0C7007`) preserved, values zeroed. |
| **Open-Loop EGR Map 1** | `0x0C69B2 .. 0x0C6BB1` | 512 B | `6717` | **`8192` (`0%`)** | 16x16 EGR valve duty map fully closed. |
| **Open-Loop EGR Map 2** | `0x0C6BF6 .. 0x0C6DF5` | 512 B | `6963` | **`8192` (`0%`)** | 16x16 EGR valve duty map fully closed. |
| **EGR DTC Classes** | `0x0CB4D7 .. 0x0CB509` | 10 entries | `8 / 9 / 10` | **`0`** | `P0403`, `P0409`, `P0486`, `P0487`, `P2141`, `P2142`, `P2413` disabled. |
| **EGR Limp Inhibitions** | `0x0CC786 .. 0x0CC80B` | 38 B | `Fid_...` | **`0`** | Torque/turbo restrictions from EGR removed. |
| **TVA (`P0638`) DTC Classes** | `0x0CB758 .. 0x0CB76C` | 7 entries | `3 / 6 / 12 / 13` | **`0`** | Throttle Valve Actuator (`P0638`) disabled; `P0651` 5V Ref B (`0x0CB740..0x0CB741`) kept active (`0x10 0x10`). |
| **TVA Limp Inhibitions** | `0x0CCB92 .. 0x0CCBB5` | 36 B | `Fid_...` | **`0`** | Torque/turbo lock on TVA error removed. |

- **Factory DPF Status (`original_suzuki_fabrika.bin`):** All 34 `PFlt` DTC paths (`DFP 165..198`) and `PFltCD` differential pressure / temperature sensor curves are already factory-disabled in the base OEM software. Pre-turbo EGT sensor (`P242A`/`P242B`) remains active.

---

## 5. Performance Calibration (Stage 1+ Focus)

### 5.1. Driver Wish (`AccPed`) — `0x0C11F8` (`11x10`)
- **Address:** `0x0C11F8` (`11x10`, RPM `0x0C11FC..0x0C1211`, Pedal `0x0C1212..0x0C1225`, Data `0x0C1226..0x0C1301`).
- **Calibration:** Smoothed `25%–100%` pedal columns (`1500–4500 RPM`) requesting `405–515 Nm` at `100%` throttle.

### 5.2. Main Torque Limiters — `0x0D3358` (`25x1`) & `0x0D403C` (`18x1`)
Calibrated with soft low-RPM delivery to protect `F9Q` conrod bearings and dual-mass flywheel:

| Engine Speed (RPM) | Stock Torque (Nm) | Stock Power (HP) | **Stage 1+ Torque (Nm)** | **Stage 1+ Power (HP)** |
| :---: | :---: | :---: | :---: | :---: |
| **1000** | 132.0 Nm | 18.8 HP | **133.0 Nm** | **18.9 HP** |
| **1250** | 160.0 Nm | 28.5 HP | **168.0 Nm** | **29.9 HP** |
| **1500** | 222.0 Nm | 47.4 HP | **250.0 Nm** | **53.4 HP** |
| **1750** | 275.0 Nm | 68.5 HP | **325.0 Nm** | **81.0 HP** |
| **2000** | **300.0 Nm** | 85.4 HP | **375.0 Nm** | **106.8 HP** |
| **2250** | 299.0 Nm | 95.8 HP | **375.0 Nm** | **120.1 HP** |
| **2500** | 297.0 Nm | 105.7 HP | **372.0 Nm** | **132.4 HP** |
| **2750** | 294.0 Nm | 115.1 HP | **366.0 Nm** | **143.3 HP** |
| **3000** | 283.0 Nm | 120.9 HP | **355.0 Nm** | **151.6 HP** |
| **3250** | 270.0 Nm | 124.9 HP | **342.0 Nm** | **158.3 HP** |
| **3500** | 255.0 Nm | 127.1 HP | **326.0 Nm** | **162.5 HP** |
| **3750** | 242.0 Nm | **129.2 HP** | **308.0 Nm** | **164.4 HP** |
| **4000** | 220.0 Nm | 125.3 HP | **285.0 Nm** | **162.3 HP** |
| **4250 / 4300** | 200.0 Nm | 121.0 HP | **252.0 Nm (Plain) / 285.0 Nm (Popcorn)** | **152.5 HP / 174.5 HP** |

### 5.3. Gearbox Master Torque Cap (`0x0D7820`) & Gear Limiters (`0x0D7822 .. 0x0D7996`)
- **Addresses:** `0x0D7820` (`Gearbx_trqInMax_C` master/default gearbox torque cap), `0x0D7822` (1st), `0x0D7860` (2nd), `0x0D789E` (3rd), `0x0D78DC` (4th), `0x0D791A` (5th), `0x0D7958` (unused 6th slot), `0x0D7996` (Reverse).
- **Calibration (Stage 1+):** Raised `0x0D7820` (`Gearbx_trqInMax_C`) from stock `335 Nm` (`3350`) to **`385 Nm` (`3850`)** (`360 Nm` on Stage 1) so full torque is available even if vehicle speed / `v/n` gear detection defaults to `0`. 1st & Reverse set to **`315 Nm`**, gears 2–5 set to **`385 Nm`**.

### 5.4. Torque-to-IQ Conversion (`NM2IQ`) — `0x0D628A` (`16x16`)
- Recalibrated `320–380 Nm` columns to deliver **`70.0 – 71.5 mg/str`** at `375 Nm`.

### 5.5. Smoke Limiters (5x Lambda Maps) — `0x0D5688`, `0x0D58CC`, `0x0D5B10`, `0x0D5D54`, `0x0D5F98`
- Low-airflow pre-boost cells (`250–500 mg/str`) kept factory stock (`1.22–1.45 Lambda`) for zero low-RPM smoke.
- High-airflow full-boost cells (`>= 535 mg/str`, `1500–4250 RPM`) set to **`1.10 Lambda` (`15.95:1 AFR`)** for clean, smoke-free combustion.

### 5.6. Turbo Boost Targets (6x `16x16`) & Boost Limiters (`SVBL`)
- **Target Maps:** `0x0EC986`, `0x0ECBCA`, `0x0ECE0E`, `0x0ED066`, `0x0ED732`, `0x0ED98A`.
- **Limiters:** `0x0ED2AA`, `0x0EC326`, `0x0EC41A`, `0x0EC522`, `0x0EC618`, `0x0EC6E0`, `0x0EC7A8`, `0x0EC870`, `0x0EC938` (`SVBL`).
- **Calibration (Stage 1+):** Stock `1.51 bar` (`2510 mbar`) raised gently to **`1.55 bar` (`2550 mbar`)** at `2500–3500 RPM`, tapering safely to **`1.45 -> 1.38 bar`** at `4000–4250 RPM` to protect the Garrett `GT1749V` turbo shaft. Boost limiters and SVBL set to **`2680–2700 mbar`**.

### 5.7. Rail Pressure & Start of Injection (SOI)
- **Rail Maps (`0x0F4128`, `0x0F4390`, `0x0F472E`, `0x0F4972`, `0x0F4C7E`):** Factory **`1600 bar` (`16000`)** peak ceiling is **never exceeded** (mid-load +30 bar smoothing only).
- **9x SOI Maps (`0x0D9534 .. 0x0DCC10`):** Full-load (`2250+ RPM`, `>= 45 mg/str`) advanced by **`+1.0° to +1.4°`** within the stock `22°` SOI limiters to keep EGT low.

---

## 6. 4300 RPM Popcorn Hardcut & A/C-ON 950 RPM Idle

### 6.1. 4300 RPM Popcorn Hardcut (`Popcorn` Variants)
Synchronized across 3 maps with a sharp `5 RPM` cut (`4300 -> 4305 RPM`):
1. **Torque Limiter 1 (`0x0D3358`):** `4300 RPM` = Full Torque (`285 Nm`), `4305 RPM` = `0 Nm`, `4500 RPM` = `0 Nm`.
2. **Torque Limiter 2 (`0x0D403C`):** `4300 RPM` = Full Torque (`285 Nm`), `4305 RPM` = `0 Nm`.
3. **Driver Wish (`0x0C11F8`):** Final two RPM rows (`0x0C120E`, `0x0C1210`) set to `4300 RPM` and `4305 RPM`, with all `4305 RPM` pedal columns zeroed (`0`).

### 6.2. A/C-ON 950 RPM Idle (`Klima950` Variants)
- The Suzuki Grand Vitara 1.9 DDiS A/C main switch (`ACCD_numDioInMnSwt_C = Pin 4`, `0x0C119C`), refrigerant pressure sensor (`ACCD_numAdc_C = Pin 23`, `0x0C117C`), and compressor clutch relay (`ACCD_numDioOut_C = Pin 3`, `0x0C1100`) are directly wired to the Bosch EDC16C3 ECU.
- Only **`ACCtl_nMin_CUR` (`0x0C1828..0x0C182F`)** is changed from factory `[800, 800, 800, 800]` to **`[950, 950, 950, 950] RPM`**.
- When A/C is OFF, engine idle remains factory **`800 RPM`**. The compressor undershoot shutoff (`0x0C17A2`) remains factory **`[780, 750] RPM`**, ensuring uninterrupted cooling at idle.

---

## 7. Byte-Level Audit & Bugfix Summary

All tuned binaries were audited byte-for-byte against `original_suzuki_fabrika.bin`:
1. **N75 Min Duty Lock Fixed:** Restored `0x0EFAD4..0x0EFAF3`, `0x0EFFD4`, `0x0F06FE` from `6144` (`75%`) to factory `410` (`5%`) and `0x0F0712` to factory `414`.
2. **Curve 4 Overwrite & Deviation Bounds Fixed:** Restored `0x0F06D8..0x0F06DD` (`1490, 1500, 1800`); set `PCR_pDvtMax/Min` to `±650 mbar` and deviation curves floor to `650 / 750 mbar`.
3. **`P0651` 5V Ref B Restored:** Re-enabled `0x0CB740..0x0CB741` (`0x10 0x10`) for sensor diagnostic safety.
4. **Missing `BPACD` (`P0033`/`P0034`) Turbo-Cut Inhibitions Zeroed:** Cleared `0x0CC6BC..0x0CC6C1`, `0x0CC6CA..0x0CC6D0`, `0x0CC6DB..0x0CC6E0` across all tuned files.
5. **Stage 1 Boost Limiters Fixed:** Raised `0x0EC6E0` and `0x0EC7A8` from `2500` to `2600 mbar` in `Stage1_Popcorn`.
6. **Master Gearbox Cap (`0x0D7820`) Fixed & Speed-Based Launch Control Removed:** Road testing confirmed that when the ECU receives `0 km/h` (`VSSCD_vDfl_C = 0` at `0x0F6194`, e.g., when vehicle speed / Cruise Control signal is absent), speed-based `0–5 km/h` Launch Control maps (`0x0D3D32`, `0x0D3E1E`, `0x0D3F1E`) stay active while driving and `Gearbx` falls back to `0x0D7820` (`Gearbx_trqInMax_C`). Launch Control files were removed and `0x0D7820` was raised from `335 Nm` to `385 Nm` (`360 Nm` on Stage 1).
7. **Cranking, Start Fuel & Glow Plug Regions Verified 100% Stock:** `StSys` (`0x0F5A00..0x0F6100`), Rail start (`0x0F3800..0x0F4127`), and `GlwCD`/`GlwCtl` (`0x0D7A00..0x0D8200`) have **0 bytes modified** vs. `original_suzuki_fabrika.bin`.
