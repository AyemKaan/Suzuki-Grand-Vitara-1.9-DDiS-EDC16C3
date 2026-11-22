# Suzuki Grand Vitara 1.9 DDiS (F9Q / Bosch EDC16C3-10.25) — ECU Yazılım Paketi

**Araç / Motor:** Suzuki Grand Vitara 1.9 DDiS (`F9Q`, 5 ileri manuel, 129 HP / 300 Nm fabrika)  
**ECU:** Bosch `EDC16C3-10.25`  
**SW:** `1037382039P500_V50` | **HW:** `8200638060`  
**Okuma / Yazma:** 1.048.576 Bayt (1 MB — Kess V2 OBD)  
**Checksum Doğrulaması:** `SumBE32(0x0C0000..0x0FDF7F) == 0xD01FE500` (tüm dosyalar doğrulanmıştır)

---

## Paket İçeriği

| Dosya | Güç / Tork | Rölanti / Klima | Kesici | Checksum (`0x0FDF7C`) | Açıklama |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **`original_suzuki_fabrika.bin`** | 129 HP / 300 Nm | 800 RPM | 4800 RPM | `0xA6A826E2` | El değmemiş fabrika orijinal referans yazılımı |
| **`org.bin`** | 129 HP / 300 Nm | 800 RPM | 4800 RPM | `0xB32F5EFA` | Önceki stok/orijinal ECU okuması |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | 800 RPM | 4800 RPM | `0xC8F8793B` | **Önerilen (En Sağlıklı Düz)** — Stage 1+, EGR/TVA OFF, N75 Korumalı |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Klima950_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | **800 / Klima: 950 RPM** | 4800 RPM | `0xC7CC780F` | **Önerilen (Düz + Klima 950)** — Sadece klima açıkken 950 RPM |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Popcorn_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | 800 RPM | **4300 RPM Popcorn** | `0xE45C90CC` | Stage 1+ + 4300 RPM Hardcut Popcorn |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Popcorn_Klima950_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | **800 / Klima: 950 RPM** | **4300 RPM Popcorn** | `0xE3308FA0` | Stage 1+ + 4300 RPM Popcorn + Klima açıkken 950 RPM |
| **`Suzuki_Grand_Vitara_Stage1_Popcorn_EGR_N75_OK.bin`** | ~158 HP / 355 Nm | 800 RPM | **4300 RPM Popcorn** | `0x1CF9D723` | Yumuşak Stage 1 + 4300 RPM Popcorn |

## Rapor ve Dokümantasyon
- **[YAPILAN_ISLEMLER.md](./YAPILAN_ISLEMLER.md):** Tüm harita adresleri, N75/VNT kök neden analizi, EGR/TVA iptali, Klima 950 RPM (`0x0C1828`), şanzıman ana tork tavanı (`0x0D7820`) ve doğrulama detayları.
- **[Suzuki_Grand_Vitara_1.9_DDiS_Dyno_Guc_Egrisi_ve_Raporu.pdf](./Suzuki_Grand_Vitara_1.9_DDiS_Dyno_Guc_Egrisi_ve_Raporu.pdf):** Fabrika vs. Stage 1 vs. Stage 1+ güç/tork eğrileri ve teknik özet raporu.

---

> **Uyarı:** Bu dosyalar yalnızca `SW: 1037382039P500_V50` / `HW: 8200638060` (`Bosch EDC16C3-10.25`) donanım/yazılım numarasına sahip Suzuki Grand Vitara 1.9 DDiS araçlar içindir. Yazma işlemi öncesinde aracın kendi orijinal yazılım yedeğini almanız önerilir.
