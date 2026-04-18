# Analisis Konsistensi Struktur CSV - Produksi Angkutan Udara Niaga Berjadwal Dalam Negeri 2020-2024

## Ringkasan Eksekutif
- **Total File CSV**: 19 file (satu file per perusahaan)
- **Periode Data**: 2020-2024
- **Sumber**: Badan Pusat Statistik (BPS) / Kementerian Perhubungan RI
- **Tanggal Analisis**: April 9, 2026

---

# BAGIAN 1: STRUKTUR TABEL

## 1.1 Gambaran Umum

Setiap file CSV merepresentasikan data produksi satu perusahaan penerbangan dengan struktur **cross-sectional time-series** (panel data).

**Dimensi Data:**
```
19 perusahaan × 5 tahun (2020-2024) × 14 metrik produksi
```

**Orientasi Tabel:**
- Baris = Metrik yang diukur (14 baris)
- Kolom = Tahun pengamatan (2020-2024)

## 1.2 Header/Kolom

Semua 19 file memiliki header yang **IDENTIK**:

```csv
NO,DESCRIPTION,Unit,2020,2021,2022,2023,2024
```

| Kolom | Makna | Satuan | Keterangan |
|-------|-------|--------|------------|
| **NO** | Nomor urut metrik | - | Identifier baris (1-11, dengan sub-baris 9a-9d) |
| **DESCRIPTION** | Nama variabel yang diukur | - | Deskripsi metrik dalam bahasa Inggris |
| **Unit** | Satuan pengukuran | - | Menjelaskan skala: `(000)`, `number`, `ton`, `(%)` |
| **2020** | Nilai metrik tahun 2020 | Sesuai Unit | Bisa berupa angka, waktu, persentase, atau dash (`-`) |
| **2021** | Nilai metrik tahun 2021 | Sesuai Unit | Same as above |
| **2022** | Nilai metrik tahun 2022 | Sesuai Unit | Same as above |
| **2023** | Nilai metrik tahun 2023 | Sesuai Unit | Same as above |
| **2024** | Nilai metrik tahun 2024 | Sesuai Unit | Same as above |

## 1.3 Metrik yang Diukur (Baris)

Setiap file memiliki **14 baris data** dengan struktur berikut:

| No | Metrik | Unit | Arti & Penjelasan |
|----|--------|------|-------------------|
| **1** | Aircraft KM | `(000)` | Total jarak tempuh pesawat (ribuan km). Nilai `289` = 289,000 km |
| **2** | Aircraft Departure | `number` | Jumlah penerbangan (angka mutlak) |
| **3** | Aircraft Hours | `number` | Total jam terbang (format `HH:MM` atau `HH:MM:SS`) |
| **4** | Passenger Carried | `number` | Jumlah penumpang yang diangkut (angka mutlak) |
| **5** | Freight Carried | `ton` | Berat kargo yang diangkut (metrik ton) |
| **6** | Passenger KM | `(000)` | Total jarak penumpang (ribuan passenger-km). Rumus: Penumpang × km |
| **7** | Available Seat KM | `(000)` | Kapasitas kursi tersedia (ribuan seat-km). Rumus: Kursi × km |
| **8** | Passenger L/F | `(%)` | Load Factor penumpang (% okupansi). Rumus: (Pass KM ÷ Seat KM) × 100% |
| **9a** | Ton KM Performed - Passenger | `(000)` | Ton-km dari penumpang (ribuan ton-km) |
| **9b** | Ton KM Performed - Freight | `(000)` | Ton-km dari kargo (ribuan ton-km) |
| **9c** | Ton KM Performed - Mail | `(000)` | Ton-km dari surat (ribuan ton-km) |
| **9d** | Ton KM Performed - Total | `(000)` | Total ton-km (9a + 9b + 9c) |
| **10** | Available Ton KM | `(000)` | Kapasitas angkut berat maksimum (ribuan ton-km) |
| **11** | Weight L/F | `(%)` | Load Factor berat (% utilisasi). Rumus: (Ton KM ÷ Avail Ton KM) × 100% |

**Catatan Unit `(000)`:**
- Nilai yang tercatat **sudah dibagi 1,000**
- Untuk mendapatkan nilai sebenarnya: **Kalikan dengan 1,000**
- Contoh: `289` → 289 × 1,000 = **289,000**

---

# BAGIAN 2: ANALISIS KONSISTENSI

## 2.1 Yang KONSISTEN ✅

| Aspek | Detail | Status |
|-------|--------|--------|
| **Header** | Semua file memiliki header yang sama persis | ✅ Konsisten |
| **Jumlah Baris** | Setiap file memiliki 14 baris data | ✅ Konsisten |
| **Urutan Metrik** | Baris 1-11 (dengan sub-baris 9a-9d) selalu sama | ✅ Konsisten |
| **Kolom Unit** | Nilai unit seragam di semua file (lihat tabel 2.2) | ✅ Konsisten |
| **Nama Metrik** | Deskripsi dalam bahasa Inggris identik | ✅ Konsisten |

### Tabel Unit per Metrik (Konsisten)

| Baris | Metrik | Unit |
|-------|--------|------|
| 1 | Aircraft KM | `(000)` |
| 2 | Aircraft Departure | `number` |
| 3 | Aircraft Hours | `number` |
| 4 | Passenger Carried | `number` |
| 5 | Freight Carried | `ton` |
| 6 | Passenger KM | `(000)` |
| 7 | Available Seat KM | `(000)` |
| 8 | Passenger L/F | `(%)` |
| 9a-9d | Ton KM Performed | `(000)` |
| 10 | Available Ton KM | `(000)` |
| 11 | Weight L/F | `(%)` |

## 2.2 Yang TIDAK KONSISTEN ❌

| Aspek | Masalah | Tingkat |
|-------|---------|---------|
| **Format Angka Ribuan** | Titik (`.`) sebagai pemisah ribuan, kadang tunggal kadang ganda | 🔴 KRITIS |
| **Format Waktu** | 4 format berbeda (`HH:MM`, `HH:MM:SS`, `H:MM`, integer) | 🟠 TINGGI |
| **Format Persentase** | Koma dalam quotes vs integer murni | 🟡 SEDANG |
| **Data Kosong** | Dash (`-`) vs nol (`0`) vs `-,00` | 🟢 RENDAH |

### Detail Inkonsistensi

#### A. Format Angka Ribuan

| Format | Contoh | Arti | File |
|--------|--------|------|------|
| Titik tunggal | `1.661` | **1,661** | PT ASI PUJIASTUTI, PT NAM AIR, dll |
| Titik ganda | `6.123.017` | **6,123,017** | PT BATIK AIR, PT CITILINK, dll |
| Tanpa pemisah | `289` | **289** | Semua file (untuk angka < 1,000) |

**Kesimpulan:** Titik adalah **PEMISAH RIBUAN** (format Indonesia), BUKAN desimal.

**Bukti:**
- Pedoman Publikasi BPS menggunakan titik untuk pemisah ribuan
- Logika bisnis: `3.740` penumpang = 3,740 orang ✅ (bukan 3.74 orang ❌)

#### B. Format Waktu (Aircraft Hours - Baris 3)

| Format | Contoh | File yang Menggunakan |
|--------|--------|----------------------|
| `HH:MM` | `1221:26` | PT ASI PUJIASTUTI |
| `HH:MM:SS` | `167383:21:00` | PT BATIK AIR, PT CITILINK, PT GARUDA, dll |
| `H:MM` | `0:00` | PT BBN AIRLINES, PT PELITA AIR SERVICE |
| Integer | `0` | PT CARDIG AIR (2024) |

**Masalah:** Jam bisa melebihi 24 (karena ini akumulasi tahunan), tapi format tidak seragam.

#### C. Format Persentase (Load Factor - Baris 8 & 11)

| Format | Contoh | File |
|--------|--------|------|
| Koma + Quotes | `"20,28"` | Mayoritas (standar Indonesia) |
| Integer murni | `80`, `84`, `57` | PT SUPER AIR JET |
| Integer tanpa desimal | `75`, `74`, `77` | PT NAM AIR, PT SRIWIJAYA, PT TRI MG |
| Format aneh | `"-,00"` | PT RUSKY AERO |

#### D. Representasi Data Kosong

| Format | Contoh | File |
|--------|--------|------|
| Dash | `-` | Mayoritas file |
| Nol | `0` | PT CARDIG AIR, PT SUPER AIR JET |
| Nol desimal | `0,00` | PT TRI MG AIRLINES |

---

# BAGIAN 3: DAFTAR FILE & KARAKTERISTIK

| No | Nama Perusahaan | Tahun Aktif | Catatan Khusus |
|----|-----------------|-------------|----------------|
| 1 | PT ASI PUJIASTUTI | 2020-2024 | Format standar |
| 2 | PT BATIK AIR | 2020-2024 | Waktu `HH:MM:SS`, angka juta 2 titik |
| 3 | PT BBN AIRLINES | 2024 only | Data hanya tersedia di 2024 |
| 4 | PT CARDIG AIR | 2020-2023 | 2024 semua kosong, waktu=`0` |
| 5 | PT CITILINK INDONESIA | 2020-2024 | Waktu `HH:MM:SS` |
| 6 | PT GARUDA INDONESIA | 2020-2024 | Waktu `HH:MM:SS` |
| 7 | PT INDONESIA AIRASIA | 2020-2024 | Ada nilai `0` di Mail |
| 8 | PT LION MENTARI AIRLINES | 2020-2024 | Waktu `HH:MM:SS` |
| 9 | PT MY INDO AIRLINES | 2020-2024 | Cargo only (penumpang semua `-`) |
| 10 | PT NAM AIR | 2020-2024 | Persentase 2024 tanpa desimal |
| 11 | PT PELITA AIR SERVICE | 2021-2023 | 2020 & 2024 kosong |
| 12 | PT RUSKY AERO | 2023-2024 | Format `"-,00"` tidak standar |
| 13 | PT SRIWIJAYA AIR | 2020-2024 | Persentase 2024 tanpa desimal |
| 14 | PT SUPER AIR JET | 2021-2024 | **Semua persentase integer** |
| 15 | PT TRANSNUSA | 2020, 2022-2024 | 2021 kosong, Weight LF >100% |
| 16 | PT TRAVEL EXPRESS | 2020 only | Hanya ada data di 2020 |
| 17 | PT TRI MG AIRLINES | 2020-2024 | Persentase mix format |
| 18 | PT TRIGANA AIR SERVICE | 2020-2024 | Waktu `HH:MM:SS` |
| 19 | WINGS ABADI AIRLINES | 2020-2024 | Format standar |

---

# BAGIAN 4: ANOMALI DATA

| File | Metrik | Tahun | Nilai | Masalah |
|------|--------|-------|-------|---------|
| PT TRANSNUSA | Weight L/F | 2022 | `114,13` | Load factor > 100% (mungkin overbooking) |
| PT RUSKY AERO | Passenger L/F | 2023-2024 | `-,00` | Format tidak standar (seharusnya `-`) |

---

# BAGIAN 5: REKOMENDASI STRUKTUR DATABASE

## 5.1 Format yang Harus Diperbaiki Sebelum Masuk Database

Sebelum data CSV digabungkan ke database, ada **empat masalah format** yang wajib dinormalisasi:

### Masalah 1: Pemisah Ribuan dengan Titik (`.`)

**Masalah:**
- Format Indonesia menggunakan titik untuk memisahkan ribuan
- `1.661` = **seribu enam ratus enam puluh satu** (bukan 1.661 desimal)
- `6.123.017` = **enam juta seratus dua puluh tiga ribu tujuh belas**

**Yang harus dilakukan:**
- **Hapus semua titik** dari angka
- `1.661` → `1661`
- `6.123.017` → `6123017`

### Masalah 2: Desimal dengan Koma (`,`)

**Masalah:**
- Format Indonesia menggunakan koma untuk desimal: `"20,28"`
- Harus diubah ke format standar database (titik desimal)

**Yang harus dilakukan:**
- Ganti koma dengan titik
- `"20,28"` → `20.28`
- `"79,62"` → `79.62`

### Masalah 3: Format Waktu Tidak Seragam

**Masalah:**
- Ada 4 format berbeda: `HH:MM`, `HH:MM:SS`, `H:MM`, dan integer `0`
- Contoh: `1221:26` vs `167383:21:00` vs `0:00` vs `0`

**Yang harus dilakukan:**
- Konversi semua ke **total menit** (integer)
- `1221:26` → 1221 × 60 + 26 = **73,286 menit**
- `167383:21:00` → 167383 × 60 + 21 = **10,043,001 menit**
- `0:00` atau `0` → **NULL** (tidak ada data)

### Masalah 4: Representasi Data Kosong

**Masalah:**
- Ada yang pakai dash `-`, ada yang pakai `0`, ada yang `0,00`
- Secara semantik berbeda: `-` = tidak beroperasi, `0` = beroperasi tapi nol

**Yang harus dilakukan:**
- Seragamkan menjadi **NULL** untuk semua yang tidak ada data
- Kecuali jika konteks bisnisnya memang harus dibedakan

## 5.2 Struktur Tabel yang Disarankan

### Konsep: Satu Tabel Besar (Wide Table)

Alih-alih menyimpan 19 file terpisah, gabungkan semua menjadi **satu tabel** dimana:
- **1 baris** = data 1 perusahaan untuk 1 tahun tertentu
- Total baris maksimal: 19 perusahaan × 5 tahun = **95 baris**

### Visualisasi Struktur Tabel

```
┌─────────────────────────────────────────────────────────────────────────┐
│               production_airline_production                              │
├──────────┬──────────┬──────────────┬──────────────┬──────────┬──────────┤
│ id       │ airline_ │ year         │ aircraft_km  │ aircraft │ aircraft │
│ (auto)   │ name     │ (INT)        │ (DECIMAL)    │ departures│ hours_   │
│          │ (VARCHAR)│              │              │ (INT)    │ minutes  │
│          │          │              │              │          │ (INT)    │
├──────────┼──────────┼──────────────┼──────────────┼──────────┼──────────┤
│  1       │ BATIK    │ 2020         │ 67540        │ 58027    │ NULL     │
│          │ AIR      │              │              │          │          │
├──────────┼──────────┼──────────────┼──────────────┼──────────┼──────────┤
│  2       │ BATIK    │ 2021         │ 71032        │ 59326    │ NULL     │
│          │ AIR      │              │              │          │          │
├──────────┼──────────┼──────────────┼──────────────┼──────────┼──────────┤
│  3       │ ASI      │ 2020         │ 289          │ 1661     │ 73286    │
│          │ PUJIASTUTI│             │              │          │          │
├──────────┼──────────┼──────────────┼──────────────┼──────────┼──────────┤
│  4       │ CITILINK │ 2024         │ 72006        │ 84779    │ NULL     │
│          │          │              │              │          │          │
└──────────┴──────────┴──────────────┴──────────────┴──────────┴──────────┘
```

### Detail Kolom

| Nama Kolom | Tipe Data | Asal | Keterangan |
|------------|-----------|------|------------|
| `id` | INT (auto increment) | System | Primary key |
| `airline_name` | VARCHAR(255) | Nama file | Nama perusahaan |
| `year` | INT | Header CSV | Tahun data |
| `aircraft_km` | DECIMAL(15,2) | Baris 1 | Unit (000), simpan nilai asli |
| `aircraft_departures` | INT | Baris 2 | Jumlah penerbangan |
| `aircraft_hours_minutes` | INT | Baris 3 | **Sudah dikonversi ke menit** |
| `passengers_carried` | INT | Baris 4 | Jumlah penumpang |
| `freight_carried_ton` | DECIMAL(15,2) | Baris 5 | Ton kargo |
| `passenger_km` | DECIMAL(15,2) | Baris 6 | Unit (000) |
| `available_seat_km` | DECIMAL(15,2) | Baris 7 | Unit (000) |
| `passenger_load_factor` | DECIMAL(5,2) | Baris 8 | Persen (sudah ganti koma→titik) |
| `ton_km_passenger` | DECIMAL(15,2) | Baris 9a | Unit (000) |
| `ton_km_freight` | DECIMAL(15,2) | Baris 9b | Unit (000) |
| `ton_km_mail` | DECIMAL(15,2) | Baris 9c | Unit (000) |
| `ton_km_total` | DECIMAL(15,2) | Baris 9d | Unit (000) |
| `available_ton_km` | DECIMAL(15,2) | Baris 10 | Unit (000) |
| `weight_load_factor` | DECIMAL(5,2) | Baris 11 | Persen (sudah ganti koma→titik) |

### Contoh Isi Tabel (Setelah Digabung)

```
┌────┬─────────────┬──────┬─────────────┬─────────────────┬───────────────────┬──────────────────┬────────────┐
│ id │ airline_    │ year │ aircraft_km │ aircraft_       │ aircraft_hours_   │ passengers_      │ freight_   │
│    │ name        │      │             │ departures      │ minutes           │ carried          │ carried_ton│
├────┼─────────────┼──────┼─────────────┼─────────────────┼───────────────────┼──────────────────┼────────────┤
│ 1  │ ASI PUJI... │ 2020 │ 289         │ 1661            │ 73286             │ 3740             │ 11         │
│ 2  │ ASI PUJI... │ 2021 │ 276         │ 1768            │ 78631             │ 9463             │ 39         │
│ 3  │ ASI PUJI... │ 2022 │ 383         │ 2100            │ 102458            │ 12429            │ 20         │
│ 4  │ ASI PUJI... │ 2023 │ 528         │ 2920            │ 147414            │ 15592            │ 24         │
│ 5  │ ASI PUJI... │ 2024 │ 641         │ 3605            │ 176285            │ 8498             │ 93         │
├────┼─────────────┼──────┼─────────────┼─────────────────┼───────────────────┼──────────────────┼────────────┤
│ 6  │ BATIK AIR │ 2020 │ 67540       │ 58027           │ NULL              │ 6123017          │ 45149      │
│ 7  │ BATIK AIR │ 2021 │ 71032       │ 59326           │ NULL              │ 7260547          │ 52852      │
│ ...│ ...        │ ...  │ ...         │ ...             │ ...               │ ...              │ ...        │
├────┼─────────────┼──────┼─────────────┼─────────────────┼───────────────────┼──────────────────┼────────────┤
│ 90 │ BBN AIR    │ 2024 │ 209         │ 367             │ 40209             │ 34423            │ 204        │
│    │ (hanya     │      │             │                 │                   │                  │            │
│    │ ada 2024)  │      │             │                 │                   │                  │            │
└────┴─────────────┴──────┴─────────────┴─────────────────┴───────────────────┴──────────────────┴────────────┘
```

## 5.3 Alur Transformasi Data

```
┌──────────────────────────────────────────────────────────────┐
│  19 File CSV Terpisah                                        │
│  (Format asli BPS dengan titik/koma Indonesia)               │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│  STEP 1: Baca & Parse                                        │
│  - Extract nama perusahaan dari nama file                    │
│  - Baca header (NO, DESCRIPTION, Unit, 2020-2024)            │
│  - Baca 14 baris metrik                                      │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│  STEP 2: Bersihkan Format                                    │
│  ✅ Hapus titik dari angka ribuan:                           │
│     `1.661` → `1661`                                         │
│     `6.123.017` → `6123017`                                  │
│                                                              │
│  ✅ Ganti koma desimal dengan titik:                         │
│     `"20,28"` → `20.28`                                      │
│                                                              │
│  ✅ Konversi waktu ke menit:                                 │
│     `1221:26` → `73286`                                      │
│     `167383:21:00` → `10043001`                              │
│                                                              │
│  ✅ Ubah data kosong ke NULL:                                │
│     `-` → `NULL`                                             │
│     `0` → `NULL` (kecuali memang nol)                        │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│  STEP 3: Pivot Data                                          │
│  Dari: 1 file = 14 baris × 5 kolom tahun                     │
│  Menjadi: 5 baris × 16 kolom metrik (satu baris per tahun)   │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│  STEP 4: Gabung Semua File                                   │
│  19 file × 5 tahun = maksimal 95 baris                       │
│  (beberapa baris NULL jika data tidak tersedia)              │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│  1 Tabel Database: production_airline_production             │
│  Siap untuk query, analisis, dan reporting                   │
└──────────────────────────────────────────────────────────────┘
```

## 5.4 Kenapa Struktur Ini Bagus?

### ✅ Kelebihan

1. **Mudah dibaca** - Satu baris = satu perusahaan untuk satu tahun
2. **Mudah dianalisis** - Semua metrik sudah dalam kolom terpisah
3. **Mudah filtering** - Bisa filter berdasarkan tahun, perusahaan, atau nilai tertentu
4. **Cocok untuk reporting** - Langsung bisa export ke Excel/BI tool tanpa pivot lagi
5. **Tidak perlu JOIN** - Semua data sudah dalam satu tabel

### ⚠️ Catatan Penting

1. **Banyak NULL**: Perusahaan yang data tidak lengkap (misal PT BBN hanya ada 2024) akan punya banyak NULL
2. **Skala Unit `(000)`**: Harus didokumentasikan bahwa kolom seperti `aircraft_km`, `passenger_km`, dll nilainya **masih dalam ribuan**. Jika ingin nilai sebenarnya, kalikan dengan 1,000.
3. **Load Factor > 100%**: Data seperti PT TRANSNUSA 2022 (114.13%) perlu dicatat sebagai anomali, jangan di-correct otomatis.

## 5.5 Alternatif: Kalau Mau Lebih Fleksibel

Jika suatu saat ingin menambahkan:
- Tahun baru (2025, 2026, dst)
- Metrik baru
- Data dari sumber lain

Maka bisa pakai struktur **3 tabel terpisah**:

```
Tabel 1: airlines
┌────────────┬─────────────────────┐
│ airline_id │ airline_name        │
├────────────┼─────────────────────┤
│     1      │ ASI PUJIASTUTI      │
│     2      │ BATIK AIR           │
│    ...     │ ...                 │
└────────────┴─────────────────────┘

Tabel 2: metrics (daftar metrik)
┌───────────┬──────────────────────┬─────────┐
│ metric_id │ metric_name          │ unit    │
├───────────┼──────────────────────┼─────────┤
│     1     │ Aircraft KM          │ (000)   │
│     2     │ Aircraft Departure   │ number  │
│    ...    │ ...                  │ ...     │
└───────────┴──────────────────────┴─────────┘

Tabel 3: production_data (data aktual)
┌────┬────────────┬───────────┬──────┬───────┐
│ id │ airline_id │ metric_id │ year │ value │
├────┼────────────┼───────────┼──────┼───────┤
│ 1  │     1      │     1     │ 2020 │  289  │
│ 2  │     1      │     2     │ 2020 │ 1661  │
│ 3  │     1      │     1     │ 2021 │  276  │
│ ...│    ...     │    ...    │ ...  │  ...  │
└────┴────────────┴───────────┴──────┴───────┘
```

**Kelebihan**: Lebih fleksibel, mudah tambah metrik/tahun baru  
**Kekurangan**: Query lebih rumit, harus JOIN 3 tabel

## 5.6 Rekomendasi Final

**Untuk use case ini (19 file, 5 tahun, metrik tetap):**
- ✅ **Pakai Wide Table** (struktur 5.2) - lebih simpel dan praktis
- ✅ **Simpan nilai raw** dari CSV di kolom terpisah untuk audit (opsional)
- ✅ **Dokumentasikan** bahwa unit `(000)` berarti nilai dalam ribuan
- ✅ **Handle NULL dengan benar** - jangan ganti dengan 0

**Untuk skalabilitas jangka panjang:**
- Pertimbangkan struktur 3 tabel (alternatif di atas)

---

**Generator**: Qwen Code Analysis  
**Tanggal**: April 9, 2026  
**Sumber Referensi**: 
- Pedoman Pembuatan Publikasi BPS
- CEIC Data (Indonesia Airline Production)
- Kementerian Perhubungan RI
