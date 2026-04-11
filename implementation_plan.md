# Data Warehouse: Korelasi Nilai Rupiah terhadap Angkutan Udara Indonesia (2020-2024)

## Tujuan

Membangun Data Warehouse (Medallion Architecture: Silver & Gold Layer) di **Neon PostgreSQL** untuk menganalisis korelasi antara **nilai tukar IDR/USD** terhadap pergerakan **angkutan udara Indonesia** (2020–2024), divisualisasikan di **Tableau**.

---

## Arsitektur Umum

```mermaid
graph LR
    subgraph "Raw Data (CSV)"
        A["KURS/BI.csv"]
        B["BAB II — Perusahaan"]
        C["BAB IV — Produksi"]
        D["BAB VI — Penumpang/Rute"]
        E["BAB VII — Lalu Lintas Bandara"]
        F["BAB XII — OTP"]
    end

    subgraph "Silver Layer (PostgreSQL Tables)"
        S1["silver_kurs"]
        S2["silver_perusahaan"]
        S3["silver_produksi"]
        S4["silver_penumpang_rute"]
        S5["silver_lalu_lintas_bandara"]
        S6["silver_otp"]
    end

    subgraph "Gold Layer (SQL Views — Star Schema)"
        D1["dim_waktu"]
        D2["dim_bandara"]
        D3["dim_maskapai"]
        D4["dim_rute"]
        D5["dim_tipe_penerbangan"]
        F1["fact_kurs_bulanan"]
        F2["fact_lalu_lintas_bandara"]
        F3["fact_penumpang_rute_bulanan"]
        F4["fact_produksi_maskapai"]
    end

    A --> S1
    B --> S2
    C --> S3
    D --> S4
    E --> S5
    F --> S6
    S1 --> F1
    S2 --> D3
    S3 --> F4
    S4 --> F3
    S5 --> F2
    S5 --> D2
    S6 --> D3
    D1 --> F1 & F2 & F3 & F4
    D3 --> F2 & F3 & F4
```

---

## Phase 1: Silver Layer — DDL (PostgreSQL Tables)

### 1.1 `silver_kurs`

Sumber: `KURS/BI.csv` (1.232 baris, harian, 2020-2024)

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | `SERIAL PRIMARY KEY` | Auto-increment |
| `tanggal` | `DATE NOT NULL` | Tanggal kurs |
| `kurs_jual` | `NUMERIC(12,2)` | Kurs jual USD/IDR |
| `kurs_beli` | `NUMERIC(12,2)` | Kurs beli USD/IDR |
| `kurs_tengah` | `NUMERIC(12,2)` | Rata-rata jual & beli (dihitung) |

**Preprocessing:**
- Parse tanggal dari format `M/D/YYYY 12:00:00 AM` → `DATE`
- Kolom `Nilai` (selalu `1`) dan `NO` di-discard
- Hitung `kurs_tengah = (kurs_jual + kurs_beli) / 2`
- Data sudah bersih (tanpa titik ribuan), langsung cast ke numeric

---

### 1.2 `silver_perusahaan`

Sumber: BAB II — CSV per tahun, 1 file per tahun (2020-2024). Setiap file berisi daftar maskapai yang mendapat izin operasi.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | `SERIAL PRIMARY KEY` | Auto-increment |
| `tahun` | `INTEGER NOT NULL` | Tahun data |
| `no_urut` | `INTEGER` | Nomor urut dalam daftar |
| `nama_maskapai` | `VARCHAR(200) NOT NULL` | Nama maskapai (sudah distandardisasi) |
| `jenis_penerbangan` | `VARCHAR(50)` | Berjadwal / Tidak Berjadwal |
| `jenis_rute` | `VARCHAR(50)` | Dalam Negeri / Luar Negeri |

**Preprocessing:**
- Standardisasi nama maskapai (typo fix):
  - `PT Pelita Air Sevice` → `PT Pelita Air Service`
  - `PT.` vs `PT ` → Semua jadi `PT `
  - Trim spasi ekstra
- Kolom header berubah antar tahun → gunakan positional indexing
- Tahun 2020-2021 ada kolom "Daerah Operasi", 2022-2024 tidak → **discard** kolom ini karena tidak relevan

> [!NOTE]
> Tabel ini berfungsi sebagai **lookup/dimensi** untuk validasi nama maskapai. Tidak semua file BAB II akan di-load jika strukturnya terlalu berbeda — cukup load yang diperlukan untuk membuat master list maskapai.

---

### 1.3 `silver_produksi`

Sumber: BAB IV — 19 file CSV (satu per maskapai), masing-masing berisi produksi tahunan 2020-2024 per metrik. Orientasi: **pivoted** (kolom = tahun).

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | `SERIAL PRIMARY KEY` | Auto-increment |
| `nama_maskapai` | `VARCHAR(200) NOT NULL` | Dari nama file (distandardisasi) |
| `metrik` | `VARCHAR(100) NOT NULL` | Nama metrik (e.g., `Passenger Carried`, `Available Seat KM`, dll.) |
| `satuan` | `VARCHAR(50)` | Unit satuan (e.g., `number`, `(000)`, `(%)`, `ton`) |
| `tahun` | `INTEGER NOT NULL` | 2020-2024 |
| `nilai` | `NUMERIC(18,2)` | Nilai metrik (NULL jika `-`) |

**Preprocessing:**
- **Unpivot** dari orientasi wide (kolom tahun) ke long (satu baris per metrik per tahun)
- Nama maskapai diambil dari nama file CSV → standardisasi (match dengan `silver_perusahaan`)
- Angka dalam format Indonesia: titik sebagai pemisah ribuan → hapus titik, ganti koma dengan titik desimal
- Nilai `-` → `NULL`
- Format jam `Aircraft Hours` (contoh: `114282:49`) → simpan sebagai string di kolom terpisah atau convert ke numeric hours
- Load Factor `"43,76"` → `43.76`

> [!IMPORTANT]
> BAB IV data bersifat **tahunan** (BUKAN bulanan). Artinya untuk korelasi bulanan, BAB IV hanya menghasilkan **5 data point per maskapai** (2020-2024). Ini akan menjadi enrichment, bukan core fact. Untuk korelasi bulanan yang kuat, kita mengandalkan BAB VI dan BAB VII.

---

### 1.4 `silver_penumpang_rute`

Sumber: BAB VI — 10 file CSV bulanan (domestik + internasional, per tahun). Ini adalah **core data** untuk korelasi bulanan.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | `SERIAL PRIMARY KEY` | Auto-increment |
| `tahun` | `INTEGER NOT NULL` | 2020-2024 |
| `bulan` | `INTEGER NOT NULL` | 1-12 |
| `kategori` | `VARCHAR(20) NOT NULL` | `DOMESTIK` atau `INTERNASIONAL` |
| `rute_kode` | `VARCHAR(20) NOT NULL` | Format: `XXX-YYY` (kode IATA) |
| `kota_asal` | `VARCHAR(100)` | Nama kota asal (jika tersedia, dari 2020-2021) |
| `kota_tujuan` | `VARCHAR(100)` | Nama kota tujuan (jika tersedia, dari 2020-2021) |
| `kode_asal` | `CHAR(3) NOT NULL` | Kode IATA bandara asal |
| `kode_tujuan` | `CHAR(3) NOT NULL` | Kode IATA bandara tujuan |
| `jumlah_penumpang` | `INTEGER` | Jumlah penumpang pada bulan itu |

**Preprocessing (KRUSIAL — paling kompleks):**

1. **Unpivot** dari wide (12 kolom bulan) ke long (satu baris per rute per bulan)
2. **Standardisasi nama kolom rute:**
   - 2020: `RUTE (PP)` → `rute`
   - 2021: `RUTE ( PP)` → `rute`
   - 2022: `RUTE ( PP)` → `rute`
   - 2023: `RUTE` → `rute`
   - 2024: `RUTE PP` → `rute`
3. **Standardisasi format rute:**
   - 2020: `Jakarta (CGK)-Denpasar (DPS)` → extract `CGK` dan `DPS`, format `CGK-DPS`
   - 2021: `Jakarta (CGK) - Denpasar (DPS)` → extract `CGK` dan `DPS`, format `CGK-DPS`
   - 2022-2024: Sudah dalam format `CGK-DPS` → split on `-`
   - Regex untuk 2020: `(\w+)\s*\((\w{3})\)\s*-\s*(\w+)\s*\((\w{3})\)` — capture city name & IATA code
   - Regex untuk 2021: sama tapi dengan ` - ` (spasi sebelum dan sesudah dash)
4. **Standardisasi nama kolom bulan:**
   - Deteksi posisi kolom bulan (kolom ke-3 sampai ke-14), **abaikan nama header** — gunakan positional
   - Map posisi → bulan 1-12
5. **Standardisasi format angka:**
   - 2020 & 2024: Integer langsung (beberapa float `.0`) → `int(float(x))`
   - 2021 & 2023: Float dengan `.0` → `int(float(x))`
   - 2022: Titik sebagai pemisah ribuan → `x.replace('.', '')` lalu `int()`
   - Sel kosong / `""` → `NULL`
   - 2024: Angka `0` bisa berarti missing → biarkan sebagai `0` (conservative approach)
6. **Hapus baris non-data:**
   - Baris `Total` di akhir file → skip (NO bukan angka atau `RUTE` berisi `Total`)
   - Baris `* Rute Codeshare...` (INT 2020) → skip
   - Baris dengan `KARGO` di kolom rute → skip
7. **Simpan nama kota** (2020-2021) sebagai kolom opsional

> [!WARNING]
> BAB VI adalah dataset paling berantakan. Setiap tahun memiliki 3-4 perbedaan format. Script Python HARUS punya logic per-tahun (`if year == 2020 ... elif year == 2021 ...`).

---

### 1.5 `silver_lalu_lintas_bandara`

Sumber: BAB VII — 1 file CSV gabungan `DATA LALU LINTAS ANGKUTAN UDARA DI BANDAR UDARA TAHUN 2020 - 2024.csv` (1.283 baris).

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | `SERIAL PRIMARY KEY` | Auto-increment |
| `propinsi_code` | `INTEGER` | Kode urut provinsi (internal DJPU) |
| `propinsi_name` | `VARCHAR(100)` | Nama provinsi |
| `airport_code` | `VARCHAR(5)` | Kode urut bandara (internal DJPU, BUKAN IATA) |
| `airport_name` | `VARCHAR(200)` | Nama bandara + kota + (DOM/INT) |
| `tipe_penerbangan` | `VARCHAR(20)` | `DOMESTIK` / `INTERNASIONAL` (extract dari `airport_name`) |
| `nama_bandara_bersih` | `VARCHAR(200)` | Nama bandara tanpa suffix (DOM/INT) |
| `kota` | `VARCHAR(100)` | Nama kota (extract dari `airport_name`) |
| `tahun` | `INTEGER NOT NULL` | 2020-2024 |
| `pesawat_dtg` | `INTEGER` | Pesawat datang |
| `pesawat_brk` | `INTEGER` | Pesawat berangkat |
| `penumpang_dtg` | `INTEGER` | Penumpang datang |
| `penumpang_brk` | `INTEGER` | Penumpang berangkat |
| `penumpang_total` | `INTEGER` | Total penumpang |
| `penumpang_transit` | `INTEGER` | Penumpang transit |
| `bagasi_dtg` | `INTEGER` | Bagasi datang (kg) |
| `bagasi_brk` | `INTEGER` | Bagasi berangkat (kg) |
| `bagasi_total` | `INTEGER` | Total bagasi |
| `barang_dtg` | `INTEGER` | Barang datang (kg) |
| `barang_brk` | `INTEGER` | Barang berangkat (kg) |
| `barang_total` | `INTEGER` | Total barang |
| `pos_dtg` | `INTEGER` | Pos datang (kg) |
| `pos_brk` | `INTEGER` | Pos berangkat (kg) |
| `pos_total` | `INTEGER` | Total pos |
| `keterangan` | `VARCHAR(50)` | E.g., "Data 12 Bln" |

**Preprocessing:**
- Format angka: titik sebagai pemisah ribuan untuk sebagian besar data → `x.replace('.', '')`
  - **TAPI** perhatikan: mulai dari baris ±385 format berubah menjadi **integer biasa** tanpa titik — perlu deteksi adaptif
- Extract `tipe_penerbangan` dari `airport_name`: jika mengandung `(DOM)` / `(DOMESTIK)` → `DOMESTIK`, jika `(INT)` / `(INTERNASIONAL)` → `INTERNASIONAL`, jika tidak ada → `DOMESTIK` (default untuk bandara kecil)
- Extract `nama_bandara_bersih` dan `kota` dari `airport_name` menggunakan pola `NAMA BANDARA - KOTA (DOM/INT)`
- `propinsi_code` dan `airport_code` adalah **nomor urut internal** (bukan kode resmi), tetap disimpan untuk referensi
- Kolom `keterangan` berisi informasi cakupan data (e.g., "Data 12 Bln", "Data 5 bln") — disimpan apa adanya
- Nilai `-` → `NULL`

> [!NOTE]
> Data BAB VII bersifat **tahunan per bandara** (bukan bulanan). Meskipun demikian, data ini sangat kaya karena mencakup total pergerakan pesawat, penumpang, bagasi, barang, dan pos per bandara per tahun. Untuk korelasi bulanan, kita akan mengandalkan BAB VI. BAB VII berfungsi sebagai dimensi bandara dan fakta tahunan.

---

### 1.6 `silver_otp`

Sumber: BAB XII — 1 file CSV `TINGKAT KETEPATAN WAKTU ... 2024.csv` (16 baris)

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | `SERIAL PRIMARY KEY` | Auto-increment |
| `nama_maskapai` | `VARCHAR(200) NOT NULL` | Nama maskapai (distandardisasi) |
| `tahun` | `INTEGER NOT NULL` | 2018-2024 |
| `otp_persen` | `NUMERIC(5,2)` | Persentase OTP |

**Preprocessing:**
- **Unpivot** dari wide (kolom tahun) ke long
- Angka `"61,07%"` → `61.07` (hapus `"`, ganti `,` → `.`, hapus `%`)
- Nilai `-` → `NULL`
- Baris "Total/Rata-rata" → discard (atau simpan terpisah)
- Standardisasi nama maskapai (match dengan master)
- File ini mencakup **2018-2024 (7 tahun)**, kita hanya filter **2020-2024**

---

## Phase 2: Python ETL Scripts

### Struktur File

```
d:\Kuliah\projek_dw\
├── etl/
│   ├── __init__.py
│   ├── config.py              # .env loader, DB connection string
│   ├── utils.py               # Shared parsing utilities
│   ├── 01_load_kurs.py        # KURS/BI.csv → silver_kurs
│   ├── 02_load_perusahaan.py  # BAB II → silver_perusahaan
│   ├── 03_load_produksi.py    # BAB IV → silver_produksi
│   ├── 04_load_penumpang.py   # BAB VI → silver_penumpang_rute
│   ├── 05_load_bandara.py     # BAB VII → silver_lalu_lintas_bandara
│   ├── 06_load_otp.py         # BAB XII → silver_otp
│   ├── 07_create_gold.py      # Create Gold Layer Views
│   └── run_all.py             # Orchestrator
├── sql/
│   ├── 01_silver_ddl.sql      # CREATE TABLE statements
│   └── 02_gold_views.sql      # CREATE VIEW statements
├── .env                        # DATABASE_URL=postgresql://...
└── requirements.txt            # pandas, sqlalchemy, python-dotenv, psycopg2-binary
```

### Dependensi

```
pandas>=2.0
sqlalchemy>=2.0
python-dotenv
psycopg2-binary
```

### Shared Utilities (`utils.py`)

```python
# Fungsi utama yang digunakan berulang:

def parse_indonesian_number(val):
    """Mengubah angka format Indonesia (titik = ribuan, koma = desimal) ke float."""
    # Handles: "1.234.567" → 1234567, "43,76" → 43.76

def standardize_maskapai_name(name):
    """Mapping nama maskapai ke bentuk standar."""
    # Fix typos, PT. → PT, trim spaces

def extract_iata_from_route_2020(route_str):
    """Parse 'Jakarta (CGK)-Denpasar (DPS)' → ('CGK', 'DPS', 'Jakarta', 'Denpasar')"""

def extract_iata_from_route_2021(route_str):
    """Parse 'Jakarta (CGK) - Denpasar (DPS)' → ('CGK', 'DPS', 'Jakarta', 'Denpasar')"""

def extract_iata_from_route_code(route_str):
    """Parse 'CGK-DPS' → ('CGK', 'DPS', None, None)"""
```

---

## Phase 3: Gold Layer — Star Schema (SQL Views)

### 3.1 Dimension Views

#### `dim_waktu`

```sql
-- Generated dari range 2020-01 s/d 2024-12 (60 baris)
CREATE VIEW gold.dim_waktu AS
SELECT
    (tahun * 100 + bulan) AS waktu_id,  -- e.g., 202001
    tahun,
    bulan,
    CASE bulan
        WHEN 1 THEN 'Januari' WHEN 2 THEN 'Februari' ...
    END AS nama_bulan,
    CASE WHEN bulan <= 6 THEN 1 ELSE 2 END AS semester,
    CEIL(bulan / 3.0) AS kuartal
FROM generate_series(2020, 2024) AS tahun
CROSS JOIN generate_series(1, 12) AS bulan;
```

#### `dim_bandara`

```sql
CREATE VIEW gold.dim_bandara AS
SELECT DISTINCT
    propinsi_code,
    propinsi_name,
    airport_code,
    nama_bandara_bersih AS nama_bandara,
    kota,
    tipe_penerbangan
FROM silver_lalu_lintas_bandara;
```

#### `dim_maskapai`

```sql
CREATE VIEW gold.dim_maskapai AS
SELECT DISTINCT nama_maskapai
FROM (
    SELECT nama_maskapai FROM silver_perusahaan
    UNION
    SELECT nama_maskapai FROM silver_produksi
    UNION
    SELECT nama_maskapai FROM silver_otp
) AS combined;
```

#### `dim_rute`

```sql
CREATE VIEW gold.dim_rute AS
SELECT DISTINCT
    rute_kode,
    kode_asal,
    kode_tujuan,
    MAX(kota_asal) AS kota_asal,  -- from 2020-2021 where available
    MAX(kota_tujuan) AS kota_tujuan,
    kategori
FROM silver_penumpang_rute
GROUP BY rute_kode, kode_asal, kode_tujuan, kategori;
```

#### `dim_tipe_penerbangan`

```sql
CREATE VIEW gold.dim_tipe_penerbangan AS
SELECT 'DOMESTIK' AS tipe_id, 'Dalam Negeri' AS deskripsi
UNION ALL
SELECT 'INTERNASIONAL', 'Luar Negeri';
```

---

### 3.2 Fact Views

#### `fact_kurs_bulanan` ⭐ (Core untuk korelasi)

```sql
CREATE VIEW gold.fact_kurs_bulanan AS
SELECT
    (EXTRACT(YEAR FROM tanggal) * 100 + EXTRACT(MONTH FROM tanggal))::INT AS waktu_id,
    EXTRACT(YEAR FROM tanggal)::INT AS tahun,
    EXTRACT(MONTH FROM tanggal)::INT AS bulan,
    ROUND(AVG(kurs_tengah), 2) AS avg_kurs_tengah,
    ROUND(AVG(kurs_jual), 2) AS avg_kurs_jual,
    ROUND(AVG(kurs_beli), 2) AS avg_kurs_beli,
    ROUND(MIN(kurs_tengah), 2) AS min_kurs_tengah,
    ROUND(MAX(kurs_tengah), 2) AS max_kurs_tengah,
    COUNT(*) AS jumlah_hari_trading
FROM silver_kurs
WHERE EXTRACT(YEAR FROM tanggal) BETWEEN 2020 AND 2024
GROUP BY EXTRACT(YEAR FROM tanggal), EXTRACT(MONTH FROM tanggal);
```

#### `fact_penumpang_rute_bulanan` ⭐ (Core untuk korelasi)

```sql
CREATE VIEW gold.fact_penumpang_rute_bulanan AS
SELECT
    (tahun * 100 + bulan) AS waktu_id,
    tahun,
    bulan,
    kategori,
    rute_kode,
    kode_asal,
    kode_tujuan,
    jumlah_penumpang
FROM silver_penumpang_rute
WHERE jumlah_penumpang IS NOT NULL;
```

#### `fact_penumpang_agregat_bulanan` ⭐⭐ (Paling penting untuk Tableau)

```sql
-- Agregasi total penumpang per bulan, siap di-join dengan kurs
CREATE VIEW gold.fact_penumpang_agregat_bulanan AS
SELECT
    (tahun * 100 + bulan) AS waktu_id,
    tahun,
    bulan,
    kategori,
    SUM(jumlah_penumpang) AS total_penumpang,
    COUNT(DISTINCT rute_kode) AS jumlah_rute_aktif
FROM silver_penumpang_rute
WHERE jumlah_penumpang IS NOT NULL AND jumlah_penumpang > 0
GROUP BY tahun, bulan, kategori;
```

#### `fact_lalu_lintas_bandara`

```sql
CREATE VIEW gold.fact_lalu_lintas_bandara AS
SELECT
    (tahun * 100 + 1) AS waktu_id,  -- Tahunan → assign ke bulan 1
    tahun,
    propinsi_code,
    propinsi_name,
    airport_code,
    nama_bandara_bersih AS nama_bandara,
    tipe_penerbangan,
    penumpang_total,
    pesawat_dtg + pesawat_brk AS pesawat_total,
    barang_total,
    pos_total,
    bagasi_total
FROM silver_lalu_lintas_bandara;
```

#### `fact_produksi_maskapai`

```sql
CREATE VIEW gold.fact_produksi_maskapai AS
SELECT
    (tahun * 100 + 1) AS waktu_id,
    tahun,
    nama_maskapai,
    MAX(CASE WHEN metrik = 'Passenger Carried' THEN nilai END) AS penumpang_diangkut,
    MAX(CASE WHEN metrik = 'Freight Carried' THEN nilai END) AS freight_ton,
    MAX(CASE WHEN metrik = 'Passenger L/F' THEN nilai END) AS load_factor_persen,
    MAX(CASE WHEN metrik = 'Aircraft Departure' THEN nilai END) AS jumlah_penerbangan,
    MAX(CASE WHEN metrik = 'Available Seat KM' THEN nilai END) AS ask_ribuan
FROM silver_produksi
GROUP BY tahun, nama_maskapai;
```

---

### 3.3 View Analitik Utama — Korelasi Bulanan ⭐⭐⭐

```sql
-- View utama untuk Tableau: Korelasi Kurs vs Penumpang per Bulan
CREATE VIEW gold.analisis_korelasi_bulanan AS
SELECT
    k.waktu_id,
    k.tahun,
    k.bulan,
    k.avg_kurs_tengah,
    k.min_kurs_tengah,
    k.max_kurs_tengah,
    k.jumlah_hari_trading,
    COALESCE(dom.total_penumpang, 0) AS penumpang_domestik,
    COALESCE(dom.jumlah_rute_aktif, 0) AS rute_domestik_aktif,
    COALESCE(intl.total_penumpang, 0) AS penumpang_internasional,
    COALESCE(intl.jumlah_rute_aktif, 0) AS rute_internasional_aktif,
    COALESCE(dom.total_penumpang, 0) + COALESCE(intl.total_penumpang, 0) AS penumpang_total
FROM gold.fact_kurs_bulanan k
LEFT JOIN gold.fact_penumpang_agregat_bulanan dom
    ON k.waktu_id = dom.waktu_id AND dom.kategori = 'DOMESTIK'
LEFT JOIN gold.fact_penumpang_agregat_bulanan intl
    ON k.waktu_id = intl.waktu_id AND intl.kategori = 'INTERNASIONAL';
```

> Ini menghasilkan **60 baris** (Januari 2020 — Desember 2024), masing-masing berisi rata-rata kurs bulanan bersamaan dengan total penumpang domestik & internasional. **Inilah inti dari analisis korelasi.**

---

## Phase 4: Urutan Eksekusi

| Step | File | Deskripsi | Dependen |
|------|-----|-----------|----------|
| 1 | `sql/01_silver_ddl.sql` | CREATE SCHEMA + CREATE TABLE Silver Layer | - |
| 2 | `etl/01_load_kurs.py` | Load KURS/BI.csv | Step 1 |
| 3 | `etl/02_load_perusahaan.py` | Load BAB II | Step 1 |
| 4 | `etl/03_load_produksi.py` | Load BAB IV | Step 1 |
| 5 | `etl/04_load_penumpang.py` | Load BAB VI (paling kompleks) | Step 1 |
| 6 | `etl/05_load_bandara.py` | Load BAB VII | Step 1 |
| 7 | `etl/06_load_otp.py` | Load BAB XII | Step 1 |
| 8 | `sql/02_gold_views.sql` | CREATE SCHEMA gold + Views | Step 2-7 |

---

## Phase 5: Verifikasi

### Automated Verification

| Test | Query | Expected |
|------|-------|----------|
| Kurs completeness | `SELECT COUNT(*) FROM silver_kurs WHERE EXTRACT(YEAR FROM tanggal) BETWEEN 2020 AND 2024` | ~1.230 baris |
| Kurs monthly | `SELECT COUNT(DISTINCT waktu_id) FROM gold.fact_kurs_bulanan` | 60 |
| Penumpang DOM | `SELECT COUNT(*) FROM silver_penumpang_rute WHERE kategori = 'DOMESTIK'` | > 20.000 baris |
| Penumpang INT | `SELECT COUNT(*) FROM silver_penumpang_rute WHERE kategori = 'INTERNASIONAL'` | > 5.000 baris |
| Korelasi view | `SELECT COUNT(*) FROM gold.analisis_korelasi_bulanan` | 60 baris |
| No NULL kurs | `SELECT COUNT(*) FROM gold.fact_kurs_bulanan WHERE avg_kurs_tengah IS NULL` | 0 |
| Bandara | `SELECT COUNT(*) FROM silver_lalu_lintas_bandara` | ~1.280 baris |

### Manual Verification

- Spot-check 5 kurs harian terhadap BI.csv asli
- Spot-check total penumpang bulanan terhadap baris `Total` di CSV BAB VI
- Preview `gold.analisis_korelasi_bulanan` di Tableau → scatter plot Kurs vs Penumpang

---

## Open Questions

> [!IMPORTANT]
> **Q1: Schema PostgreSQL** — Apakah kamu ingin menggunakan **schema terpisah** (`silver.kurs`, `gold.fact_kurs_bulanan`) atau semua di schema `public` dengan prefix (`silver_kurs`, `gold_fact_kurs_bulanan`)? Neon PostgreSQL mendukung keduanya. Saya merekomendasikan schema terpisah untuk organisasi yang lebih bersih.

> [!IMPORTANT]
> **Q2: Koneksi Neon** — Apakah kamu sudah memiliki connection string Neon PostgreSQL (`postgresql://user:pass@host/db`)? Saya akan membutuhkan ini dimasukkan ke file `.env` sebelum menjalankan ETL.

> [!NOTE]
> **Q3: BAB II Scope** — Dari riset saya, data BAB II (Perusahaan) hanya berfungsi sebagai master list nama maskapai. Apakah kamu setuju bahwa kita cukup membuat **mapping dict** di Python (bukan full table loading) untuk standardisasi nama maskapai? Atau tetap ingin tabel silver_perusahaan diisi lengkap untuk dokumentasi?

> [!NOTE]
> **Q4: BAB VII KOMPOSISI** — Selain file utama lalu lintas bandara, di folder BAB VII ada 5 file `KOMPOSISI PENUMPANG BERDASARKAN PENGELOLA BANDAR UDARA TAHUN 20XX.csv`. File ini sangat kecil (3-4 baris), berisi persentase penumpang per pengelola (PT Angkasa Pura I, II, UPT, dll.). Apakah ingin dimasukkan? Saya merekomendasikan **skip** karena data terlalu agregat dan tidak relevan untuk korelasi rute-level.

> [!NOTE]
> **Q5: Tableau Connection** — Setelah Gold Layer selesai, apakah kamu akan menghubungkan Tableau langsung ke Neon PostgreSQL via native connector, atau perlu panduan setup?
