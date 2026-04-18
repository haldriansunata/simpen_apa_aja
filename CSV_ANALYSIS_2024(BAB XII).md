# CSV Analysis: On Time Performance - Penerbangan Domestik 2024

## 📊 Informasi Umum

| Properti | Nilai |
|----------|-------|
| **Nama File** | `TINGKAT KETEPATAN WAKTU (ON TIME PERFORMANCE) BADAN USAHA ANGKUTAN UDARA NIAGA PENERBANGAN NIAGA BERJADWAL DALAM NEGERI 2024.csv` |
| **Sumber** | Extract dari PDF (Kemenhub/DJPU) |
| **Tahun Data** | 2018 - 2024 |
| **Jumlah Baris** | 15 (14 entitas + 1 baris Total/Rata-rata) |
| **Jumlah Kolom** | 9 |
| **Format Persentase** | String dengan koma desimal & simbol `%` (contoh: `"61,07%"`) |
| **Missing Value** | Ditandai dengan `-` |

---

## 🗂️ Struktur Tabel

### Skema Saat Ini

```
NO (int)
BADAN USAHA (string)
2018 (string - percentage)
2019 (string - percentage)
2020 (string - percentage)
2021 (string - percentage)
2022 (string - percentage)
2023 (string - percentage)
2024 (string - percentage)
```

### Detail Per Kolom

| No | Nama Kolom | Deskripsi | Tipe Data Saat Ini | Tipe Data Rekomendasi (Database) | Nullable | Unique | Contoh Nilai |
|----|-----------|-----------|-------------------|----------------------------------|----------|--------|--------------|
| 1 | `NO` | Nomor urut maskapai | Integer | `INT` | ✅ Yes (ada nilai `-`) | ❌ No | `1`, `2`, ..., `13`, `-` |
| 2 | `BADAN USAHA` | Nama perusahaan maskapai | String | `VARCHAR(100)` | ❌ No | ✅ **Ya** (14 maskapai unik) | `PT. Asi Pudjiastuti` |
| 3 | `2018` | OTP tahun 2018 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"61,07%"`, `-` |
| 4 | `2019` | OTP tahun 2019 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"54,52%"`, `-` |
| 5 | `2020` | OTP tahun 2020 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"28,90%"`, `-` |
| 6 | `2021` | OTP tahun 2021 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"35,32%"`, `-` |
| 7 | `2022` | OTP tahun 2022 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"39,92%"`, `-` |
| 8 | `2023` | OTP tahun 2023 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"56,77%"`, `-` |
| 9 | `2024` | OTP tahun 2024 | String (format: `"XX,XX%"`) | `DECIMAL(5,2)` | ✅ Yes | ❌ No | `"47,03%"`, `-` |

---

## 🔍 Analisis Nilai Unik & Distribusi

### Kolom Kategorikal

#### 1. `NO`
- **Nilai Unik:** 14 (1-13 + `-`)
- **Distinct Values:** `1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, -`
- **Catatan:** Baris "Total/Rata-rata" menggunakan `-` sebagai penanda

#### 2. `BADAN USAHA`
- **Nilai Unik:** 14 maskapai (semua distinct)
- **Distinct Values:**
  1. PT. Asi Pudjiastuti
  2. PT. Batik Indonesia Air
  3. PT. Citilink Indonesia
  4. PT. Garuda Indonesia
  5. PT. Indonesia Airasia
  6. PT. Lion Mentari Airlines
  7. PT. Nam Air
  8. PT. Sriwijaya Air
  9. PT. Transnusa Aviation Mandiri
  10. PT. Trigana Air Service
  11. PT. Wings Abadi Airlines
  12. PT. Super Air Jet
  13. PT. Pelita Air Indonesia
  14. PT. BBN Airlines Indonesia
  15. Total/Rata-rata *(aggregate row)*

- **Catatan:** 
  - Nama maskapai menggunakan prefix `PT.` dengan variasi spacing (ada yang `PT. Asi`, `PT. Garuda`)
  - Tidak ada duplicasi nama maskapai

---

## ⚠️ Potensi Masalah & Saran Pre-Processing

### 1. Format Persentase Tidak Standard

| Properti | Detail |
|----------|--------|
| **Masalah** | Nilai menggunakan koma sebagai desimal (`"61,07%"`) bukan titik (`61.07`) |
| **Dampak** | Database/ETL tools akan gagal parse langsung ke tipe data numerik |
| **Saran** | Lakukan 3 langkah cleaning: (1) Hapus simbol `%`, (2) Ganti koma `,` menjadi titik `.`, (3) Konversi ke numerik (float/decimal) |

**Visualisasi Transformasi:**
```
SEBELUM:  "61,07%"  →  SESUDAH:  61.07
SEBELUM:  "88,78%"  →  SESUDAH:  88.78
SEBELUM:  "52,78%"  →  SESUDAH:  52.78
```

---

### 2. Missing Value Tidak Konsisten

| Properti | Detail |
|----------|--------|
| **Masalah** | Missing value ditandai dengan `-` (string), bukan `NULL` atau kosong |
| **Dampak** | ETL akan gagal saat mencoba convert `-` ke tipe numerik |
| **Saran** | Replace semua nilai `-` dengan `NULL` sebelum konversi ke numerik |

**Data yang Affected:**
| Maskapai | Tahun yang Missing |
|----------|-------------------|
| PT. Transnusa Aviation Mandiri | 2021 |
| PT. Super Air Jet | 2018, 2019, 2020 |
| PT. Pelita Air Indonesia | 2018, 2019, 2020, 2021 |
| PT. BBN Airlines Indonesia | 2018 - 2023 |

**Visualisasi Transformasi:**
```
SEBELUM:  "-"        →  SESUDAH:  NULL
SEBELUM:  "84,18%"   →  SESUDAH:  84.18
```

---

### 3. Wide Format (Kolom Tahun Terpisah)

| Properti | Detail |
|----------|--------|
| **Masalah** | Struktur saat ini adalah **wide format** (1 baris = 1 maskapai, 7 kolom tahun) |
| **Dampak** | • Sulit untuk time-series analysis<br>• Jika ada tahun 2025, harus ALTER TABLE (tambah kolom)<br>• Aggregasi perlu unpivot dulu |
| **Saran** | **Pertimbangkan transformasi ke long format** (1 baris = 1 maskapai + 1 tahun). Ini opsional tergantung kebutuhan analisis. |

**Perbandingan Format:**

```
┌─────────────────────────────────────────────────────────────┐
│ WIDE FORMAT (Saat Ini)                                      │
├──────────────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┤
│ Maskapai     │ 2018 │ 2019 │ 2020 │ 2021 │ 2022 │ 2023 │ 2024 │
├──────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Garuda       │ 88.8 │ 91.4 │ 94.1 │ 92.8 │ 88.1 │ 87.8 │ 83.5 │
│ Citilink     │ 86.2 │ 92.4 │ 90.7 │ 65.9 │ 69.7 │ 86.1 │ 91.3 │
└──────────────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

┌─────────────────────────────────────────────┐
│ LONG FORMAT (Direkomendasikan untuk ETL)    │
├──────────────┬──────┬──────────────┤
│ Maskapai     │ Tahun│ OTP          │
├──────────────┼──────┼──────────────┤
│ Garuda       │ 2018 │ 88.80        │
│ Garuda       │ 2019 │ 91.38        │
│ Garuda       │ 2020 │ 94.11        │
│ Citilink     │ 2018 │ 86.19        │
│ Citilink     │ 2019 │ 92.41        │
└──────────────┴──────┴──────────────┘
```

---

### 4. Baris Aggregate (Total/Rata-rata)

| Properti | Detail |
|----------|--------|
| **Masalah** | Baris terakhir `Total/Rata-rata` adalah hasil kalkulasi, bukan data mentah |
| **Dampak** | Jika dimuat ke database, bisa menyebabkan double-counting saat aggregasi (misal: SUM dari semua maskapai + total = salah) |
| **Saran** | **Pisahkan atau beri penanda:** Tambahkan kolom flag `is_aggregate` (TRUE/FALSE), atau pisahkan ke tabel/view tersendiri |

**Visualisasi:**
```
┌────┬────────────────────────┬──────┬───────────────┐
│ NO │ BADAN USAHA            │ 2024 │ is_aggregate  │
├────┼────────────────────────┼──────┼───────────────┤
│ 1  │ PT. Asi Pudjiastuti    │ 47.03│ FALSE         │
│ 2  │ PT. Batik Indonesia Air│ 74.62│ FALSE         │
│ .. │ ...                    │ ...  │ ...           │
│ -  │ Total/Rata-rata        │ 76.80│ TRUE ⚠️       │
└────┴────────────────────────┴──────┴───────────────┘
```

---

### 5. Nama Kolom Tahun (Numeric Column Names)

| Properti | Detail |
|----------|--------|
| **Masalah** | Nama kolom `2018`, `2019`, dll. adalah angka murni |
| **Dampak** | Beberapa database tidak support column name dimulai angka tanpa quoting khusus. Query SQL jadi rumit: `"2018"` atau `[2018]` |
| **Saran** | **Rename dengan prefix** yang deskriptif, contoh: `otp_2018`, `otp_2019`, atau `year_2018`, `year_2019` |

**Visualisasi Rename:**
```
SEBELUM:  2018, 2019, 2020, 2021, 2022, 2023, 2024
SESUDAH:  otp_2018, otp_2019, otp_2020, otp_2021, otp_2022, otp_2023, otp_2024
```

---

### 6. Tidak Ada Primary Key Eksplisit

| Properti | Detail |
|----------|--------|
| **Masalah** | Tidak ada kolom yang bisa jadi primary key unik |
| **Dampak** | Sulit untuk operasi UPSERT (update/insert), UPDATE, atau DELETE spesifik record |
| **Saran** | **Generate surrogate key** (auto-increment ID atau UUID) saat load ke database. Kolom `NO` tidak bisa diandalkan karena ada nilai `-` |

**Contoh Surrogate Key:**
```
┌─────────────┬─────────────────────────────┬──────┐
│ maskapai_id │ nama_maskapai               │ 2024 │
├─────────────┼─────────────────────────────┼──────┤
│ 1           │ PT. Asi Pudjiastuti         │ 47.03│
│ 2           │ PT. Batik Indonesia Air     │ 74.62│
│ 3           │ PT. Citilink Indonesia      │ 91.27│
│ ...         │ ...                         │ ...  │
│ 14          │ PT. BBN Airlines Indonesia  │ 90.14│
└─────────────┴─────────────────────────────┴──────┘
```

---

### 7. Inkonsistensi Penulisan Nama Maskapai

| Properti | Detail |
|----------|--------|
| **Masalah** | Semua nama pakai prefix `PT.` tapi ada variasi spacing & capitalization |
| **Dampak** | Sulit join dengan tabel referensi maskapai jika format tidak match (misal: `"PT. Garuda Indonesia"` vs `"Garuda Indonesia"`) |
| **Saran** | **Standardisasi format nama:** Trim whitespace, consistent capitalization (Title Case), dan pertimbangkan apakah prefix `PT.` perlu dipertahankan atau dipisah ke kolom tersendiri |

**Contoh Inkonsistensi Potensial:**
```
┌─────────────────────────────────────────┐
│ Format Saat Ini:                        │
│  - PT. Asi Pudjiastuti                  │
│  - PT. Batik Indonesia Air              │
│  - PT. Indonesia Airasia                │
├─────────────────────────────────────────┤
│ Kemungkinan Variasi di Sumber Lain:     │
│  - Garuda Indonesia (tanpa PT.)         │
│  - PT  Garuda Indonesia (double space)  │
│  - pt. garuda indonesia (lowercase)     │
└─────────────────────────────────────────┘
```

---

## 📐 Rekomendasi Skema Database (Gambaran)

### Opsi A: Wide Format (Seperti CSV Asli)

**Konsep:** Satu tabel dengan kolom tahun melebar (horizontal)

**Struktur Logis:**
```
┌─────────────┬──────────────┬────────┬────────┬──────┬────────┐
│ ID (PK)     │ Nama         │ 2018   │ 2019   │ ...  │ 2024   │
├─────────────┼──────────────┼────────┼────────┼──────┼────────┤
│ 1           │ Garuda       │ 88.80  │ 91.38  │ ...  │ 83.47  │
│ 2           │ Citilink     │ 86.19  │ 92.41  │ ...  │ 91.27  │
│ ...         │ ...          │ ...    │ ...    │ ...  │ ...    │
└─────────────┴──────────────┴────────┴────────┴──────┴────────┘
```

**Kolom yang Dibutuhkan:**
- **Primary Key** (auto-generated ID)
- **Nama Maskapai** (VARCHAR)
- **Kolom per Tahun** (DECIMAL): otp_2018, otp_2019, ..., otp_2024
- **Flag Aggregate** (BOOLEAN): untuk tandai baris Total/Rata-rata
- **Timestamp** (opsional): kapan data di-load

**Kelebihan:** 
- ✅ Simple, sesuai format sumber
- ✅ Query langsung tanpa JOIN
- ✅ Mudah dipahami user non-teknis

**Kekurangan:**
- ❌ Sulit extend (tambah tahun 2025 = ALTER TABLE)
- ❌ Tidak optimal untuk time-series analysis
- ❌ Row jadi lebar (banyak kolom)

**Kapan Pakai Opsi Ini?**
→ Jika data hanya untuk **reporting static** dan tidak akan sering bertambah tahun

---

### Opsi B: Long Format (Recommended)

**Konsep:** Dua tabel terpisah (Master + Fakta) dengan struktur vertikal

**Tabel 1: Master Maskapai (Dimension Table)**
```
┌─────────────┬──────────────────────────┬─────────────┐
│ ID (PK)     │ Nama Maskapai            │ Created At  │
├─────────────┼──────────────────────────┼─────────────┤
│ 1           │ PT. Asi Pudjiastuti      │ 2026-04-10  │
│ 2           │ PT. Batik Indonesia Air  │ 2026-04-10  │
│ ...         │ ...                      │ ...         │
└─────────────┴──────────────────────────┴─────────────┘
```

**Tabel 2: Fakta OTP (Fact Table)**
```
┌──────────┬─────────────┬──────┬───────────────┐
│ ID (PK)  │ Maskapai ID │ Tahun│ OTP (%)       │
├──────────┼─────────────┼──────┼───────────────┤
│ 1        │ 1           │ 2018 │ 61.07         │
│ 2        │ 1           │ 2019 │ 54.52         │
│ 3        │ 1           │ 2020 │ 28.90         │
│ ...      │ ...         │ ...  │ ...           │
│ 8        │ 2           │ 2018 │ 88.78         │
└──────────┴─────────────┴──────┴───────────────┘
```

**Struktur Logis:**
- **Tabel Master (dim_maskapai):**
  - Primary Key (ID unik per maskapai)
  - Nama Maskapai (unik, tidak boleh duplikat)
  - Metadata (opsional: created_at, updated_at)

- **Tabel Fakta (fact_otp):**
  - Primary Key (ID unik per record)
  - Foreign Key (link ke master maskapai)
  - Tahun (INT): 2018, 2019, ..., 2024
  - OTP Percentage (DECIMAL): nilai yang sudah di-clean
  - **Unique Constraint:** 1 maskapai hanya boleh punya 1 nilai per tahun

**Kelebihan:**
- ✅ Mudah extend (tambah tahun 2025 = INSERT data baru, bukan ALTER TABLE)
- ✅ Optimal untuk time-series & trend analysis
- ✅ Normalized (tidak ada data redundancy)
- ✅ Skalabel untuk data besar

**Kekurangan:**
- ❌ Perlu JOIN untuk query lengkap (tapi ini best practice)
- ❌ Baris lebih banyak (14 maskapai × 7 tahun = 98 rows)
- ❌ Sedikit lebih kompleks untuk user awam

**Kapan Pakai Opsi Ini?**
→ Jika data akan dipakai untuk **analytics, dashboard, atau akan bertambah terus**

---

### 📊 Perbandingan Kedua Opsi

| Aspek | Wide Format | Long Format |
|-------|-------------|-------------|
| **Jumlah Tabel** | 1 | 2 (Master + Fakta) |
| **Jumlah Baris** | 14-15 | 98+ (14 maskapai × 7 tahun) |
| **Jumlah Kolom** | 9+ | 3-4 per tabel |
| **Tambah Tahun Baru** | ALTER TABLE (struktural) | INSERT biasa |
| **Query Trend/Time-Series** | Kompleks (unpivot dulu) | Simple (GROUP BY tahun) |
| **Cocok untuk** | Reporting statis | Analytics & BI |
| **Kompleksitas** | Rendah | Medium |

---

### 🎯 Rekomendasi Final

**Untuk Use Case Data Warehouse:**

```
Pakai LONG FORMAT (Opsi B) karena:
  ✅ Data warehouse biasanya untuk analytics
  ✅ Mudah agregasi per tahun atau per maskapai
  ✅ Skalabel jika ada data tahun baru
  ✅ Sesuai konsep star schema (Dimension + Fact)
```

**Alur ETL Sederhana:**
```
CSV Mentah → Pre-Processing → Load ke Staging → Transform → Load ke Production
     ↓            ↓              ↓              ↓            ↓
  (Parsing    (Clean %,      (Tabel       (Unpivot,    (Tabel final
   CSV)        NULL, ID)      Sementara)   Join)        siap pakai)
```

---

## 🎯 Kesimpulan & Next Steps

### Masalah Kritikal (Harus Ditangani)
1. ✅ Format persentase (`"61,07%"` → `61.07`)
2. ✅ Missing value (`-` → `NULL`)
3. ✅ Kolom tahun perlu rename (hindari numeric-only names)
4. ✅ Baris aggregate perlu isolasi

### Masalah Struktural (Pertimbangkan Transformasi)
1. ⚠️ Wide → Long format (tergantung use case)
2. ⚠️ Primary key & surrogate ID
3. ⚠️ Standardisasi nama maskapai

### Next Steps untuk File Ini
- [ ] Tentukan format target (wide vs long)
- [ ] Buat script pre-processing (Python/pandas)
- [ ] Validasi hasil cleaning (cek min/max/avg vs sumber)
- [ ] Jika ada CSV tahun lain, cek konsistensi struktur
- [ ] Load ke staging table → transform → production table

---

## 📝 Metadata Tambahan

| Properti | Nilai |
|----------|-------|
| **Analysis Date** | 2026-04-10 |
| **Analyzed By** | Data Engineer (AI Assistant) |
| **File Size** | ~1.5 KB |
| **Encoding** | UTF-8 (asumsi, perlu verifikasi) |
| **Delimiter** | `,` (comma) |
| **Quote Character** | `"` (double quote) |
| **Line Terminator** | `\r\n` (Windows-style, asumsi) |

---

> **Catatan:** Dokumen ini hanya fokus pada file CSV 2024. Untuk analisis komparatif antar tahun atau integrasi multi-file, diperlukan dokumen terpisah.
