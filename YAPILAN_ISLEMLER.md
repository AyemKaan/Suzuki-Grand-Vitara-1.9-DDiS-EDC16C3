# Suzuki Grand Vitara 1.9 DDiS (F9Q / Bosch EDC16C3-10.25) - Kapsamlı ECU Analizi, Harita Adresleri ve Yapılan Tüm İşlemler Raporu

Bu doküman, **Suzuki Grand Vitara 1.9 DDiS (129 HP / 300 Nm, F9Q Motor, SW: `1037382039P500_V50`, HW: `8200638060`, `EDC16C3-10.25`)** aracı üzerinde yapılan tüm yazılım analizlerini, N75 valfi ve EGR/TVA kök neden çözümlerini, harita adreslerini ve hazırlanan tüm yazılım paketlerinin (`Stage 1`, `Stage 1+`, `Klima 950 RPM` ve `4300 RPM Popcorn`) teknik detaylarını eksiksiz olarak içerir.

---

## 1. Araç, ECU ve Yazılım Kimlik Bilgileri

- **Araç:** Suzuki Grand Vitara 1.9 DDiS (F9Q-B264 motor altyapısı)
- **Fabrika Verisi:** 129.2 HP @ 3750 RPM / 300.0 Nm @ 2000 RPM
- **Motor Kontrol Ünitesi (ECU):** Bosch EDC16C3-10.25 (EDC16C39 / Renault-Suzuki F9Q mimarisi)
- **Yazılım Versiyonu (SW):** `1037382039P500_V50` (`V30_W84`)
- **Donanım Numarası (HW):** `8200638060`
- **Dosya Boyutu:** `1.048.576 Bayt` (1.00 MB — Kess V2 OBD Tam Okuma)
- **Checksum Bölgesi ve Kuralı:**
  - `0x0C0000 .. 0x0FDF7F` aralığındaki 32-bit Big-Endian kelimelerin toplamı (`SumBE32`) her zaman **`0xD01FE500`** olmalıdır.
  - 32-bit sağlama kelimesi **`0x0FDF7C .. 0x0FDF7F`** adresinde yer alır.

---

## 2. Hazırlanan ve Güncellenen Tüm Yazılım Dosyaları

Aşağıdaki tüm yazılım dosyaları bu paket içerisinde yer almaktadır:

| Dosya Adı | Güç / Tork | Kesici (Popcorn) | Checksum (`0x0FDF7C`) | `SumBE32` | Açıklama ve Kullanım Amacı |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **`original_suzuki_fabrika.bin`** | 129 HP / 300 Nm | Fabrika (4800 RPM) | `0xA6A826E2` | `0xD01FE500` | El değmemiş fabrika orijinal referans dosyası (fabrikasyon DPF devre dışı kalibrasyon). |
| **`Suzuki_Grand_Vitara_Stage1_Popcorn_EGR_N75_OK.bin`** | ~158 HP / 355 Nm | **4300 RPM Popcorn** | `0x1CF9D723` | `0xD01FE500` | **Stage 1 + 4300 RPM Hardcut Popcorn** + Komple EGR & TVA OFF + N75 Aktif & Turbo Korumalı. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | Fabrika (4800 RPM) | `0xC8F8793B` | `0xD01FE500` | **Stage 1+ (ÖNERİLEN EN SAĞLIKLI LİMİT)** + Komple EGR & TVA OFF + N75 Aktif & Turbo Korumalı. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Klima950_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | Fabrika (4800 RPM) | `0xC7CC780F` | `0xD01FE500` | **Stage 1+ + Sadece Klima Açıkken 950 RPM (`0x0C1828`, Klima Kapalı 800 RPM)** + EGR/TVA OFF + N75 Korumalı. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Popcorn_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | **4300 RPM Popcorn** | `0xE45C90CC` | `0xD01FE500` | **Stage 1+ (ÖNERİLEN) + 4300 RPM Hardcut Popcorn** + Komple EGR & TVA OFF + N75 Korumalı. |
| **`Suzuki_Grand_Vitara_Stage1_PLUS_Popcorn_Klima950_EGR_N75_OK.bin`** | **~168 HP / 375 Nm** | **4300 RPM Popcorn** | `0xE3308FA0` | `0xD01FE500` | **Stage 1+ + 4300 RPM Popcorn + Sadece Klima Açıkken 950 RPM (`0x0C1828`, Klima Kapalı 800 RPM)** + EGR/TVA OFF + N75 Korumalı. |

---

## 3. N75 Turbo Valfi Sorunlarının Kök Neden Analizi ve Çözümü

### 3.1. Eski Dosyada N75'in İlk Marşta Hiç Açmama Nedeni
- **Kök Neden:** Eski `org_dtc_off.bin` ve `org_dtc_off_stage1.bin` dosyalarında `0x0F0788` (`150` = 1.5 saniye) ve `0x0F078C` (`700` = 7.0 saniye) adreslerindeki **N75 marş gecikme sayaçları (`PCR_tiStrtDelay`)** yanlışlıkla `0xFFFF` (`65535` = ~11 dakika) yapılmıştı. Ayrıca `0x0EF7EE..0x0EF8FD` aralığındaki **PID regülatör pencere haritaları** `2500` değeriyle ezilmişti.
- **Uygulanan Çözüm:**
  - `0x0F0788..0x0F078D` marş gecikme sayaçları fabrika orijinal değeri olan **`(150, 100, 700)`** haline geri döndürüldü.
  - `0x0EF7EE..0x0EF8FD` PID regülatör pencere haritaları **%100 fabrika orijinal** değerlerine geri döndürüldü.
  - Böylece marşa basıldığı anda 1.5 saniye içinde N75 valfi vakumu çekerek turbo VNT aktüatörünü devreye alır.

### 3.2. `Suzuki EGR OFF...` Dosyasında Marşta Çalışıp Sürerken Kapanma Nedeni
- **Kök Neden:**
  1. DSM hata tablosunda `0x0CAAC1..0x0CAAC2` (`P0243`) ve `0x0CAAF8..0x0CAAFD` (`P0638`) adreslerindeki hata zorlama baytları `0x00` yerine `0x01` yapılmıştı.
  2. `DSM_InhDfp` (Fonksiyon İnhibisyon / Limp Mode) tablosunda (`0x0CC5D8..0x0CCBF7`), `P0243` (`0x0CC9C2..0x0CC9D8`) ve `P0638` (`0x0CCB92..0x0CCBB5`) hata yolları `Fid_PCR_MAX` (Turbo kontrol kilidi) fonksiyonunu tetikliyordu.
  3. Yolda giderken basınç sapması (`PCR_pDvtMax` / `PCR_pDvtMin`) `0x0F043A..0x0F043D` (`400` / `-200`) ve `0x0F0646..0x0F06DD` eşiklerini aştığında ECU Limp Mode'a geçip N75'i `0x0F06FE`, `0x0F0712`, `0x0EFFD4` ve `0x0EFAD4..0x0EFAF3` adreslerindeki **`410` (%5 duty = 0 vakum)** acil durum değerine düşürerek vakumu tamamen kesiyordu.
- **Uygulanan Çözüm (Güncel & Turbo Korumalı):**
  - `0x0CAAC1..0x0CAAC2` ve `0x0CAAF8..0x0CAAFD` zorlama baytları orijinal **`0x00`** değerine döndürüldü.
  - `P0033 / P0034` (`0x0CB454..0x0CB45F`) ve `P0243` (`0x0CB67C, 0x0CB681`) hata sınıfları **`0x00`** yapıldı.
  - `P0033 / P0034` (`BPACD` N75 devre arızası: `0x0CC6BC..0x0CC6C1`, `0x0CC6CA..0x0CC6D0`, `0x0CC6DB..0x0CC6E0`) ve `P0243` (`PCR` basınç sapması: `0x0CC9C2..0x0CC9D8`) Limp Mode (`DSM_InhDfp`) turbo kesme bağlantıları tamamen **`0x00`** yapıldı.
  - `0x0F043A` (`PCR_pDvtMax`) = **`+650 mbar`** ve `0x0F043C` (`PCR_pDvtMin`) = **`-650 mbar`** yapılarak hem yanlış limp-mode önlendi hem de gerçek overboost koruması aktif bırakıldı.
  - 4 basınç sapma eğrisi (`0x0F0646..0x0F0659`, `0x0F0670..0x0F0683`, `0x0F069A..0x0F06AD`, `0x0F06C4..0x0F06D7` — her biri tam 20 bayt) yalnızca dar olan `300–500` hücreleri **`650 / 750 mbar`** olacak şekilde genişletildi; 4. eğriden hemen sonra gelen **`0x0F06D8..0x0F06DD` (`1490, 1500, 1800`)** parametreleri fabrika orijinaline geri alındı.
  - `0x0EFAD4..0x0EFAF3` (`PCR_rMin_CUR`), `0x0EFFD4` (`PCR_rMin_C`), `0x0F06FE` (`PCR_rDflVal`) fabrika **`410` (%5 min/güvenli tahliye)** ve `0x0F0712` fabrika **`414`** değerine döndürüldü; böylece VNT kanatçıkları yüksek devirde `%75`'e kilitlenmeden serbestçe basıncı tahliye edebilir.

### 3.3. VNT / Bypass Regülasyonu Güvenliği
- **5 Adet Ana VNT / N75 Haritası (`0x0EA334`, `0x0EA578`, `0x0EA7BC`, `0x0EAA14`, `0x0EB33E`):** %100 Fabrika orijinal bırakıldı.
- **PID Kapalı Çevrim Kontrolü (`0x0EF7CA..0x0EFAD3`) ve Min/Max Sınırları (`0x0EFB02..0x0EFFD3`):** %100 Fabrika orijinal bırakıldı.
- **Sonuç:** N75 valfi sabit açık kalmaz; düşük devirde vakum çekerek turboyu doldurur (`%75`), basınç yükseldikçe vakumu kısarak (`%55 -> %40 -> %33`) fazla basıncı tahliye (bypass) eder ve turboyu korur.

---

## 4. Komple EGR ve TVA (Boğaz Kelebeği / P0638) İptali

| Bölge / Harita | Başlangıç – Bitiş Adresi | Boyut | Orijinal Değer | Yeni Değer | Açıklama |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Ana EGR Histerezis 1** | `0x0C7084 .. 0x0C709F` | 28 Bayt | `400 .. 9000` | **`0`** | EGR açılma üst eşiği sıfırlandı. |
| **Ana EGR Histerezis 2** | `0x0C70BE .. 0x0C70D9` | 28 Bayt | `2700 .. 3550` | **`0`** | EGR açılma alt eşiği sıfırlandı. |
| **Yardımcı EGR Histerezis** | `0x0C6F18 .. 0x0C6F77` | 96 Bayt | `400 .. 9000` | **`0`** | Tüm yardımcı EGR eğrileri sıfırlandı. |
| **Rejenerasyon EGR Eğrisi** | `0x0C7008 .. 0x0C7023` | 28 Bayt | `2000 .. 3500` | **`0`** | Eksen (`0x0C7004..0x0C7007`) korundu, veriler sıfırlandı. |
| **Açık Çevrim EGR Haritası 1** | `0x0C69B2 .. 0x0C6BB1` | 512 Bayt | `6717` | **`8192` (%0)** | 16x16 EGR valf haritası tamamen kapatıldı. |
| **Açık Çevrim EGR Haritası 2** | `0x0C6BF6 .. 0x0C6DF5` | 512 Bayt | `6963` | **`8192` (%0)** | 16x16 EGR valf haritası tamamen kapatıldı. |
| **EGR DTC Sınıfları** | `0x0CB4D7 .. 0x0CB509` | 10 Adet | `3 / 13 / 16` | **`0`** | P0400, P0401, P0402, P0403, P0409, P0489, P0490 kapatıldı. |
| **EGR Limp Bağlantıları** | `0x0CC786 .. 0x0CC80B` | 38 Bayt | `Fid_...` | **`0`** | EGR kaynaklı tork/turbo kısıtlamaları kaldırıldı. |
| **TVA (P0638) DTC Sınıfları** | `0x0CB758..0x0CB76C` | 7 Adet | `3 / 6 / 12 / 13` | **`0`** | Boğaz kelebeği (TVA) arıza kodları kapatıldı (`P0651` 5V sensör referans hattı `0x0CB740..0x0CB741` teşhis güvenliği için açık bırakıldı). |
| **TVA Limp Bağlantıları** | `0x0CCB92 .. 0x0CCBB5` | 36 Bayt | `Fid_...` | **`0`** | TVA arızasında N75/tork kilitlemesi kaldırıldı. |

---

## 5. Performans Haritaları ve Adresleri (Stage 1+ Odaklı)

### 5.1. Sürücü İstek Haritası (Drivers Wish) — `0x0C11F8` (`11x10`)
- **Adres:** `0x0C11F8` (Boyutlar: `11x10`, Y=RPM `0x0C11FC..0x0C1211`, X=Pedal `0x0C1212..0x0C1225`, Veri: `0x0C1226..0x0C1301`)
- **Yapılan İşlem:** `%25` ile `%100` gaz pedalı sütunları (`1500 - 4500 RPM` arası) pürüzsüzleştirilerek `%100` gazda `405 - 515 Nm` tork talebi oluşturacak şekilde kalibre edildi.

### 5.2. Ana Tork Sınırlayıcılar (Torque Limiters) — `0x0D3358` (`25x1`) & `0x0D403C` (`18x1`)
Stage 1+ dosyasında F9Q motorun kol yataklarını ve çift kütleli volanını korumak için alt devirler yumuşak, orta ve üst devirler güçlü ayarlanmıştır:

| Devir (RPM) | Fabrika Tork (Nm) | Fabrika Güç (HP) | **Stage 1+ Tork (Nm)** | **Stage 1+ Güç (HP)** |
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
| **4250 / 4300** | 200.0 Nm | 121.0 HP | **252.0 Nm (Düz) / 285.0 Nm (Popcorn)** | **152.5 HP / 174.5 HP** |

### 5.3. Şanzıman Ana Giriş Tork Tavanı (`0x0D7820`) ve Vites Bazlı Tork Sınırlayıcılar (`0x0D7822 .. 0x0D7996`)
- **Adresler:** `0x0D7820` (`Gearbx_trqInMax_C` — Şanzıman Ana Giriş / Varsayılan Tork Tavanı), `0x0D7822` (1. Vites), `0x0D7860` (2. Vites), `0x0D789E` (3. Vites), `0x0D78DC` (4. Vites), `0x0D791A` (5. Vites), `0x0D7958` (6. Vites), `0x0D7996` (Geri Vites).
- **Yapılan İşlem (Stage 1+):** `0x0D7820` (`Gearbx_trqInMax_C`) fabrika `335 Nm` (`3350`) değerinden **`385 Nm` (`3850`)** değerine (Stage 1 dosyasında `360 Nm` / `3600` değerine) yükseltildi; 1. vites ve Geri vites **`315 Nm` (`3150`)**, 2-3-4-5-6. vitesler ise **`385 Nm` (`3850`)** olarak ayarlandı.

### 5.4. Tork – Yakıt Çevrim Haritası (Torque to IQ / NM to IQ) — `0x0D628A` (`16x16`)
- **Adres:** `0x0D628A` (Y=RPM `750..4500`, X=Tork `0..3800`)
- **Yapılan İşlem (Stage 1+):** `320 - 380 Nm` sütunları yeniden kalibre edilerek `375 Nm` tork talebinde **`70.0 – 71.5 mg/str`** (`7000 - 7150`) yakıt püskürtülmesi sağlandı.

### 5.5. Duman Sınırlayıcı (Lambda / Smoke Limiter) Haritaları (5 Adet `16x16`)
- **Adresler:** `0x0D5688`, `0x0D58CC`, `0x0D5B10`, `0x0D5D54`, `0x0D5F98`
- **Yapılan İşlem:**
  - Turbo dolmadan önceki düşük hava akışı bölgesi (`250 – 500 mg/str` hava) fabrika orijinal (`1.22 – 1.45 Lambda`) bırakıldı (alt devirde kara dumanı önlemek için).
  - Turbo dolduktan sonraki yüksek hava akışı bölgesi (`>= 535 mg/str` hava ve `1500 - 4250 RPM`), **Stage 1+** dosyasında **`1100` (`1.10 Lambda` = `15.95:1 AFR` — Dumansız Temiz Yanma)** değerine ayarlandı.

### 5.6. Turbo Basınç Hedef Haritaları (6 Adet `16x16`) ve Basınç Sınırlayıcılar (SVBL)
- **Hedef Harita Adresleri:** `0x0EC986`, `0x0ECBCA`, `0x0ECE0E`, `0x0ED066`, `0x0ED732`, `0x0ED98A` (`16x16`)
- **Basınç Sınırlayıcı Adresleri:** `0x0ED2AA` (`16x16`), `0x0EC326` (`10x10`), `0x0EC41A` (`10x10`), `0x0EC522` (`10x10`), `0x0EC618` (`10x8`), `0x0EC6E0` (`10x8`), `0x0EC7A8` (`10x8`), `0x0EC870` (`10x8`), `0x0EC938` (`1x1` SVBL).
- **Yapılan İşlem (Stage 1+):**
  - Fabrika pik basıncı `2510 mbar` (`1.51 bar`) iken, **Stage 1+** dosyasında `2500 - 3500 RPM` tam yükte **`2550 mbar` (`1.55 bar`)** yapıldı.
  - Üst devirlerde (`4000 - 4250 RPM`) Garrett `GT1749V` turbonun mil hızını korumak için basınç kademeli olarak **`2450 mbar` (`1.45 bar`) -> `2380 mbar` (`1.38 bar`)** seviyesine düşürüldü.
  - Tüm basınç limitörleri (`0x0ED2AA`, `0x0EC6E0`, `0x0EC7A8`, `0x0EC938`) **`2680 – 2700 mbar`** seviyesine çıkarılarak `150 mbar` güvenlik marjı bırakıldı.

### 5.7. Rail Basıncı Haritaları ve Limitörleri
- **Adresler:** `0x0F4128` (`16x16`), `0x0F4390` (`16x16`), `0x0F472E` (`16x16`), `0x0F4972` (`16x16`), `0x0F4C7E` (`16x8` Tavan Limitör).
- **Yapılan İşlem (Stage 1+):** Fabrika maksimum rail basıncı tavanı olan **`1600 bar` (`16000`) aşılmadı** (CP3 pompa ve rail emniyet valfi korundu); yalnızca orta-yüksek yük geçiş bölgeleri `+30 bar` iyileştirilerek daha iyi atomizasyon sağlandı.

### 5.8. Enjeksiyon Avansı (SOI - Start of Injection) Haritaları (9 Adet `16x16`)
- **Adresler:** `0x0D9534`, `0x0D9778`, `0x0D99BC`, `0x0D9C00`, `0x0DBBF8`, `0x0DBE3C`, `0x0DC080`, `0x0DC9B8`, `0x0DCC10` ve SOI Limitörleri `0x0DA2CC`, `0x0DC74C`, `0x0DC880` (`16x8`).
- **Yapılan İşlem (Stage 1+):** `2250 RPM` üzerinde ve `>= 45 mg/str` tam yükte **`+1.0°` ile `+1.4°` (`+45` ile `+65` ham birim)** güvenli avans eklenerek yanmanın silindir içinde tamamlanması ve egzoz sıcaklığının (EGT) düşük kalması sağlandı.

---

## 6. 4300 RPM Popcorn (Hardcut Rev Limiter) Kalibrasyonu

Popcorn (kesicili) dosyalarda yakıtın `4300 RPM`'de bıçak gibi kesilmesi için 3 harita senkronize edildi:
1. **Tork Limitörü 1 (`0x0D3358`):**
   - 15. nokta (`0x0D3378` / `0x0D33AA`): **`4300 RPM` = Tam Tork (`285 Nm`)**
   - 16. nokta (`0x0D337A` / `0x0D33AC`): **`4305 RPM` = `0 Nm`**
   - 17. nokta (`0x0D337C` / `0x0D33AE`): **`4500 RPM` = `0 Nm`**
2. **Tork Limitörü 2 (`0x0D403C`):**
   - 15. nokta (`0x0D405C` / `0x0D4080`): **`4300 RPM` = Tam Tork (`285 Nm`)**
   - 16. nokta (`0x0D405E` / `0x0D4082`): **`4305 RPM` = `0 Nm`**
3. **Sürücü İstek Haritası (`0x0C11F8`):**
   - Y ekseni (RPM) son iki satırı (`0x0C120E`, `0x0C1210`) **`4300 RPM`** ve **`4305 RPM`** yapıldı; `4305 RPM` satırındaki tüm pedal sütunları **`0`** yapıldı.
   - `5 RPM` aralığında (`4300 -> 4305 RPM`) yakıt tamamen kesildiği için araç `4300 RPM`'de net ve seri **Popcorn (Hardcut)** kesiciye girer.

---

## 7. Yazılım Denetimi ve Düzeltilen Hatalar (20.11.2026)

Klasördeki tüm modifiyeli `.bin` dosyaları fabrika referansı (`original_suzuki_fabrika.bin`) ile bayt bayt karşılaştırılarak şu hatalar giderildi ve tüm checksum'lar (`SumBE32 = 0xD01FE500`) yeniden doğrulandı:

1. **N75 Min Duty Kilidi ve Turbo Koruma Düzeltmesi (tüm dosyalar):** `0x0EFAD4..0x0EFAF3` (`PCR_rMin_CUR`), `0x0EFFD4` (`PCR_rMin_C`), `0x0F06FE` (`PCR_rDflVal`) `6144` (%75) değerinden fabrika **`410` (%5)** değerine, `0x0F0712` fabrika **`414`** değerine döndürüldü. Böylece VNT kanatçıklarının yüksek devirde %75'e kilitlenmesi önlendi.
2. **`0x0F06D8..0x0F06DD` Taşma ve Basınç Sapma Eşiği Düzeltmesi (tüm dosyalar):** 4. sapma eğrisi sonrasındaki 6 baytlık taşma (`[1490, 1500, 1800]`) orijinaline alındı; `PCR_pDvtMax/Min` `±3500` yerine **`±650 mbar`**, 4 sapma eğrisinin alt limitleri **`650 / 750 mbar`** yapılarak turbo overboost koruması geri kazandırıldı.
3. **`P0651` (5V Sensör Referans Hattı B) Geri Açıldı (tüm dosyalar):** `0x0CB740..0x0CB741` orijinal `0x10 0x10` değerine döndürüldü.
4. **Launch Control İptali ve Şanzıman Ana Tork Tavanı (`0x0D7820`) Düzeltmesi:** Suzuki Grand Vitara `EDC16C3-10.25` beyninde fabrika kalibrasyonunda hız sensörü hata yolu (`P0500`, `DFP 228/229`) ve debriyaj müşürü (`P0830`, `DFP 44`) devre dışı (`P0000`) olduğundan ve varsayılan hız `VSSCD_vDfl_C = 0` (`0x0F6194`) olarak kaldığından, ECU hız bazlı haritalarda (`0x0D3D32`, `0x0D3E1E`, `0x0D3F1E`) araç hareket halindeyken de `0 km/s` sütununu okumakta ve 2750 RPM Launch Control kesicisi sürüş esnasında da devrede kalmaktadır. Bu nedenle Launch Control dosyaları kaldırılmış; ayrıca `v = 0` durumunda devreye giren şanzıman ana tork tavanı `0x0D7820` (`Gearbx_trqInMax_C`) `3350` (`335 Nm`) değerinden **`3850` (`385 Nm`)** değerine (Stage 1'de `3600` / `360 Nm`) yükseltilerek 375 Nm torkun kısıtlanması tamamen engellenmiştir.
5. **Düz Stage 1 Boost Limitörü Düzeltildi (`Stage1_Popcorn`):** `0x0EC6E0` ve `0x0EC7A8` limitörleri `2500 -> 2600 mbar` yapıldı.
6. **N75 (`BPACD` — `P0033 / P0034`) Eksik `DSM_InhDfp` Turbo Kesme Bağlantıları Kapatıldı (tüm dosyalar):** Önceki yazılımlarda `P0033 / P0034` hata sınıfları kapatılmış ama turboyu kesen `DSM_InhDfp` blokları (`0x0CC6BC..0x0CC6C1`, `0x0CC6CA..0x0CC6D0`, `0x0CC6DB..0x0CC6E0`) açık unutulmuştu; tamamı sıfırlanarak N75 devre/sapma kaynaklı turbo kapanması kökten engellendi.
