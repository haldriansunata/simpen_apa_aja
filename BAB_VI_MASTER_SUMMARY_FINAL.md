# BAB VI Master Summary: Penumpang Per Rute (2020-2024)

## 📊 Overview

Dokumen ini adalah **master summary final** untuk BAB VI — Penumpang Per Rute, yang membandingkan **struktur tabel, format, penamaan kolom, dan tipe data** dari kelima tahun (2020-2024).

Tujuannya adalah memberi gambaran jelas tentang **apa saja yang harus ditangani** sebelum data dimasukan ke database.

---

## 🔍 Perbandingan Struktur Tabel Antar Tahun

### 1. File Bulanan (Domestik & Internasional)

#### Jumlah Kolom

| Tahun | Jumlah Kolom | Kolom ID | Kolom Rute | Kolom Bulan | Kolom Total |
|-------|-------------|----------|------------|-------------|-------------|
| **2020** | 18 | `NO` | `RUTE (PP)` | 12 kolom | 3 kolom |
| **2021** | 18 | `NO` | `RUTE ( PP)` | 12 kolom | 3 kolom |
| **2022** | 15 | `NO` | `RUTE ( PP)` | 12 kolom | 1 kolom |
| **2023** | 15 | `NO` | `RUTE` | 12 kolom | 1 kolom |
| **2024** | 15 | `NO` | `RUTE PP` | 12 kolom | 1 kolom |

**Ketidakonsistenan:**
- ⚠️ 2020-2021 punya 18 kolom, 2022-2024 punya 15 kolom
- ⚠️ Nama kolom rute berbeda setiap tahun
- ⚠️ 2020-2021 punya 3 kolom total (tahun sekarang + 2 tahun sebelumnya), 2022-2024 hanya 1 kolom total

---

#### Nama Kolom Rute

| Tahun | Nama Kolom | Contoh Isi |
|-------|-----------|------------|
| **2020** | `RUTE (PP)` | `Jakarta (CGK)-Denpasar (DPS)` |
| **2021** | `RUTE ( PP)` | `Jakarta (CGK) - Denpasar (DPS)` |
| **2022** | `RUTE ( PP)` | `CGK-DPS` |
| **2023** | `RUTE` | `CGK-DPS` |
| **2024** | `RUTE PP` | `CGK-DPS` |

**Yang harus ditangani:**
- ⚠️ Nama kolom berbeda di setiap tahun
- ⚠️ Format isi berubah dari nama kota lengkap menjadi hanya kode IATA (mulai 2022)
- ⚠️ Spasi tidak konsisten (2020 tanpa spasi di kurung, 2021 ada spasi)

---

#### Nama Kolom Bulan

| Tahun | Bahasa | Contoh Nama Kolom |
|-------|--------|------------------|
| **2020** | Inggris | `Jan-20`, `Feb-20`, `Mar-20`, `Apr-20`, `May-20`, `Jun-20`, `Jul-20`, `Aug-20`, `Sep-20`, `Oct-20`, `Nov-20`, `Dec-20` |
| **2021** | Indonesia | `Jan-21`, `Feb-21`, `Mar-21`, `Apr-21`, `Mei-21`, `Jun-21`, `Jul-21`, `Agu-21`, `Sep-21`, `Okt-21`, `Nov-21`, `Des-21` |
| **2022** | Inggris | `Jan-22`, `Feb-22`, `Mar-22`, `Apr-22`, `May-22`, `Jun-22`, `Jul-22`, `Aug-22`, `Sep-22`, `Oct-22`, `Nov-22`, `Dec-22` |
| **2023** | Indonesia | `Jan-23`, `Feb-23`, `Mar-23`, `Apr-23`, `Mei-23`, `Jun-23`, `Jul-23`, `Agu-23`, `Sep-23`, `Okt-23`, `Nov-23`, `Des-23` |
| **2024** | Indonesia | `Jan-24`, `Feb-24`, `Mar-24`, `Apr-24`, `May-24`, `Jun-24`, `Jul-24`, `Aug-24`, `Sep-24`, `Oct-24`, `Nov-24`, `Des-24` |

**Yang harus ditangani:**
- ⚠️ Bahasa bergantian antara Inggris (2020, 2022) dan Indonesia (2021, 2023, 2024)
- ⚠️ Penulisan berbeda: Mei/Mei, Agustus/Agu, Oktober/Okt, Desember/Des

---

#### Nama Kolom Total

| Tahun | Nama Kolom Total | Jumlah Kolom |
|-------|-----------------|--------------|
| **2020** | `TOTAL 2020`, `TOTAL 2019`, `TOTAL 2018` | 3 |
| **2021** | `TOTAL 2021`, `TOTAL 2020`, `TOTAL 2019` | 3 |
| **2022** | `TOTAL 2022` | 1 |
| **2023** | `TOTAL 2023` | 1 |
| **2024** | `TOTAL 2024` | 1 |

**Yang harus ditangani:**
- ⚠️ 2020-2021 punya kolom komparasi tahun sebelumnya, 2022-2024 tidak ada

---

### 2. File Ranking (Domestik & Internasional)

#### Jumlah Kolom

| Tahun | Jumlah Kolom | Kolom ID | Kolom Rute | Kolom Metrik |
|-------|-------------|----------|------------|--------------|
| **2020** | 8 | `NO` | `RUTE (PP)` | 6 kolom |
| **2021** | 8 | `NO` | `RUTE ( PP)` | 6 kolom |
| **2022** | 8 | `NO` | `RUTE ( PP)` | 6 kolom |
| **2023** | 8 | `NO` | `RUTE` | 6 kolom |
| **2024** | 8 | `NO` | `RUTE PP` | 6 kolom |

---

#### Nama Kolom Ranking

| Nama Kolom | 2020 | 2021 | 2022 | 2023 | 2024 |
|-----------|------|------|------|------|------|
| **Nomor** | `NO` | `NO` | `NO` | `NO` | `NO` |
| **Rute** | `RUTE (PP)` | `RUTE ( PP)` | `RUTE ( PP)` | `RUTE` | `RUTE PP` |
| **Penerbangan** | `JUMLAH PENERBANGAN` | Sama | Sama | Sama | Sama |
| **Penumpang** | `JUMLAH PENUMPANG` | Sama | Sama | Sama | Sama |
| **Kapasitas** | `KAPASITAS SEAT` | Sama | Sama | Sama | Sama |
| **Barang** | `JUMLAH BARANG (Kg)` | `JUMLAH BARANG` | `JUMLAH BARANG` | `JUMLAH BARANG` | `JUMLAH BARANG KG` |
| **Pos** | `JUMLAH POS` | Sama | Sama | Sama | `JUMLAH POS KG` |
| **Load Factor** | `L/F` | Sama | Sama | Sama | `LF %` |

**Yang harus ditangani:**
- ⚠️ Nama kolom rute berubah setiap tahun
- ⚠️ 2024: nama kolom barang dan pos berubah (tambah "KG" tanpa kurung)
- ⚠️ 2024: nama kolom Load Factor berubah dari `L/F` menjadi `LF %`

---

## 📋 Arti Setiap Kolom dengan Deskripsi

### File Bulanan

| Nama Kolom | Arti | Deskripsi |
|-----------|------|-----------|
| `NO` | Nomor Urut | Nomor urut rute berdasarkan ranking jumlah penumpang (dari terbesar ke terkecil) |
| `RUTE ...` | Rute Penerbangan | Rute penerbangan pulang-pergi dalam format "Kota Asal (Kode IATA)-Kota Tujuan (Kode IATA)" atau hanya "KODE-KODE" |
| `Jan-XX` s/d `Dec-XX` | Jumlah Penumpang Bulanan | Jumlah penumpang yang mengangkut pada rute tersebut di masing-masing bulan |
| `TOTAL XXXX` | Total Tahunan | Akumulasi total penumpang selama satu tahun (penjumlahan dari 12 bulan) |
| `TOTAL XXXX` (tambahan) | Total Tahun Sebelumnya | Total penumpang tahun sebelumnya untuk perbandingan (hanya ada di 2020-2021) |

---

### File Ranking

| Nama Kolom | Arti | Deskripsi |
|-----------|------|-----------|
| `NO` | Nomor Urut | Nomor urut rute berdasarkan ranking jumlah penumpang (dari terbesar ke terkecil) |
| `RUTE ...` | Rute Penerbangan | Rute penerbangan pulang-pergi dalam format "Kota Asal (Kode IATA)-Kota Tujuan (Kode IATA)" atau hanya "KODE-KODE" |
| `JUMLAH PENERBANGAN` | Frekuensi Penerbangan | Jumlah total penerbangan pulang-pergi yang beroperasi pada rute tersebut selama satu tahun |
| `JUMLAH PENUMPANG` | Volume Penumpang | Jumlah total penumpang yang mengangkut pada rute tersebut selama satu tahun |
| `KAPASITAS SEAT` | Kapasitas Kursi Tersedia | Jumlah total kursi yang tersedia dari semua penerbangan pada rute tersebut selama satu tahun |
| `JUMLAH BARANG ...` | Volume Kargo | Total berat barang atau kargo yang diangkut pada rute tersebut selama satu tahun (satuan: kilogram) |
| `JUMLAH POS ...` | Volume Pos | Total berat pos yang diangkut pada rute tersebut selama satu tahun (satuan: kilogram) |
| `L/F` atau `LF %` | Load Factor | Persentase tingkat okupansi atau keterisian pesawat, dihitung dari (Jumlah Penumpang ÷ Kapasitas Seat) × 100% |

---

## ⚠️ Ringkasan Ketidakkonsistenan yang Harus Ditangani

### Format Data

| Aspek | 2020 | 2021 | 2022 | 2023 | 2024 | Keterangan |
|-------|------|------|------|------|------|------------|
| **Format Angka Penumpang** | Integer | Float (`.0`) | Float (titik = ribuan) | Float (`.0`) | Integer | Setiap tahun berbeda |
| **Format Load Factor** | `"XX,X%"` (koma + quotes) | `"XX,X%"` (koma + quotes) | `"XX,X%"` (koma + quotes) | `XX.X%` (titik, tanpa quotes) | `XX%` (tanpa desimal, tanpa quotes) | 4 format berbeda |
| **Missing Value** | Sel kosong | Sel kosong | Sel kosong | Sel kosong | Angka `0` | 2024 berbeda |
| **Error Excel** | Tidak ada | Tidak ada | Tidak ada | Tidak ada | `#DIV/0!` | Hanya di 2024 |

---

### Penamaan Kolom

| Kolom | 2020 | 2021 | 2022 | 2023 | 2024 | Masalah |
|-------|------|------|------|------|------|---------|
| **Rute (Bulanan)** | `RUTE (PP)` | `RUTE ( PP)` | `RUTE ( PP)` | `RUTE` | `RUTE PP` | Berubah setiap tahun |
| **Rute (Ranking)** | `RUTE (PP)` | `RUTE ( PP)` | `RUTE ( PP)` | `RUTE` | `RUTE PP` | Berubah setiap tahun |
| **Bulan 5** | `May-20` | `Mei-21` | `May-22` | `Mei-23` | `May-24` | Inggris vs Indonesia |
| **Bulan 8** | `Aug-20` | `Agu-21` | `Aug-22` | `Agu-23` | `Aug-24` | Inggris vs Indonesia |
| **Bulan 10** | `Oct-20` | `Okt-21` | `Oct-22` | `Okt-23` | `Oct-24` | Inggris vs Indonesia |
| **Bulan 12** | `Dec-20` | `Des-21` | `Dec-22` | `Des-23` | `Dec-24` | Inggris vs Indonesia |
| **Barang (Ranking)** | `JUMLAH BARANG (Kg)` | `JUMLAH BARANG` | `JUMLAH BARANG` | `JUMLAH BARANG` | `JUMLAH BARANG KG` | 2024 berubah |
| **Pos (Ranking)** | `JUMLAH POS` | `JUMLAH POS` | `JUMLAH POS` | `JUMLAH POS` | `JUMLAH POS KG` | 2024 berubah |
| **Load Factor (Ranking)** | `L/F` | `L/F` | `L/F` | `L/F` | `LF %` | 2024 berubah |

---

### Format Isi Kolom Rute

| Tahun | Format | Contoh | Keterangan |
|-------|--------|--------|------------|
| **2020** | Nama Kota (Kode IATA)-Nama Kota (Kode IATA) | `Jakarta (CGK)-Denpasar (DPS)` | Nama kota lengkap + IATA dalam kurung, tanpa spasi di sekitar dash |
| **2021** | Nama Kota (Kode IATA) - Nama Kota (Kode IATA) | `Jakarta (CGK) - Denpasar (DPS)` | Nama kota lengkap + IATA dalam kurung, **ada spasi** di sekitar dash |
| **2022** | Kode IATA Asal-Kode IATA Tujuan | `CGK-DPS` | Hanya kode IATA, tanpa nama kota |
| **2023** | Kode IATA Asal-Kode IATA Tujuan | `CGK-DPS` | Hanya kode IATA, tanpa nama kota |
| **2024** | Kode IATA Asal-Kode IATA Tujuan | `CGK-DPS` | Hanya kode IATA, tanpa nama kota |

**Yang harus ditangani:**
- ⚠️ 2020-2021 pakai nama kota lengkap, 2022-2024 hanya kode IATA
- ⚠️ 2020 tanpa spasi, 2021 ada spasi di sekitar dash
- ⚠️ Perlu mapping table untuk menyamakan format antar tahun

---

## 📊 Jumlah Baris Data per Tahun

### File Domestik

| Tahun | File Bulanan | File Ranking | Jumlah Rute |
|-------|-------------|--------------|-------------|
| **2020** | 412 baris | 412 baris | 410 rute |
| **2021** | 381 baris | 381 baris | 379 rute |
| **2022** | 377 baris | 377 baris | 374 rute |
| **2023** | 306 baris | 306 baris | 303 rute |
| **2024** | 317 baris | 318 baris | 314-315 rute |

### File Internasional

| Tahun | File Bulanan | File Ranking | Jumlah Rute |
|-------|-------------|--------------|-------------|
| **2020** | 160 baris | 161 baris | ~147 rute |
| **2021** | 148 baris | 148 baris | ~145 rute |
| **2022** | 136 baris | 136 baris | 133 rute |
| **2023** | 128 baris | 128 baris | 125 rute |
| **2024** | 137 baris | 137 baris | 134 rute |

---

## 🎯 Apa yang Harus Ditangani Sebelum Masuk Database

### 1. Standarisasi Format Data

| Yang Harus Dilakukan | Detail |
|---------------------|--------|
| **Standarisasi format angka penumpang** | Setiap tahun perlu penanganan berbeda: 2020 langsung integer, 2021/2023 hapus `.0` di belakang, 2022 hapus titik sebagai pemisah ribuan, 2024 langsung integer |
| **Standarisasi Load Factor** | 2020-2022: hapus tanda kutip dan ganti koma dengan titik. 2023: hapus tanda persen. 2024: hapus tanda persen, nilai `#DIV/0!` diganti kosong |
| **Standarisasi missing value** | 2020-2023: ganti sel kosong dengan NULL. 2024: ganti angka `0` dengan NULL |

### 2. Standarisasi Penamaan Kolom

| Yang Harus Dilakukan | Detail |
|---------------------|--------|
| **Kolom Rute** | Rename semua kolom rute ke nama yang sama untuk semua tahun (misalnya: `rute`) |
| **Kolom Bulan** | Rename semua kolom bulan ke format seragam (misalnya: `bulan_01`, `bulan_02`, sampai `bulan_12` atau `m01_YYYY`, `m02_YYYY`, dst) |
| **Kolom Total** | Rename ke nama seragam (misalnya: `total_tahun_ini`). Kolom tambahan di 2020-2021 bisa diabaikan atau dipisah sebagai data historis |
| **Kolom Barang** | Rename `JUMLAH BARANG (Kg)` dan `JUMLAH BARANG KG` ke nama yang sama |
| **Kolom Pos** | Rename `JUMLAH POS` dan `JUMLAH POS KG` ke nama yang sama |
| **Kolom Load Factor** | Rename `L/F` dan `LF %` ke nama yang sama |

### 3. Standarisasi Format Isi

| Yang Harus Dilakukan | Detail |
|---------------------|--------|
| **Kolom Rute** | 2020-2021 pakai nama kota + kode IATA, 2022-2024 hanya kode IATA. Perlu mapping table untuk menyamakan format. Pilih salah satu: semua jadi kode IATA saja, atau semua jadi nama kota + kode IATA |
| **Baris Total** | Semua file punya baris Total di akhir file. Baris ini harus diberi penanda atau dipisah agar tidak tercampur dengan data rute biasa |
| **Footer** | File internasional 2020 punya baris footer `* Rute Codeshare Niaga Berjadwal Luar Negeri` yang bukan data. Harus dihapus |

### 4. Penanganan Data Unik per Tahun

| Tahun | Yang Perlu Ditangani |
|-------|---------------------|
| **2022** | Format angka menggunakan titik sebagai pemisah ribuan (berbeda dari tahun lain yang pakai format biasa) |
| **2024** | Ada nilai `#DIV/0!` (error Excel) di kolom Load Factor. Nama kolom barang, pos, dan Load Factor berubah. Missing value menggunakan angka `0` bukan sel kosong |

---

## 💡 Opsi Penggabungan Data ke Database

### Opsi A: Data Bulanan dan Ranking Dipisah

**Struktur:**
- **Tabel 1:** Data Bulanan (semua tahun digabung, format lebar atau panjang)
- **Tabel 2:** Data Ranking (semua tahun digabung, format lebar atau panjang)

**Kelebihan:** Struktur jelas sesuai jenis data, mudah dipahami
**Kekurangan:** Perlu 2 tabel terpisah

---

### Opsi B: Data Domestik dan Internasional Dipisah

**Struktur:**
- **Tabel 1:** Domestik Bulanan
- **Tabel 2:** Domestik Ranking
- **Tabel 3:** Internasional Bulanan
- **Tabel 4:** Internasional Ranking

**Kelebihan:** Pemisahan jelas berdasarkan kategori rute
**Kekurangan:** Lebih banyak tabel yang harus dikelola

---

### Opsi C: Satu Tabel per Jenis Data dengan Penanda Kategori

**Struktur:**
- **Tabel 1:** Data Bulanan (ada kolom `kategori` = domestik atau internasional)
- **Tabel 2:** Data Ranking (ada kolom `kategori` = domestik atau internasional)

**Kelebihan:** Tabel lebih sedikit
**Kekurangan:** Perlu kolom tambahan untuk penanda kategori

---

## 📝 Metadata Master

| Properti | Nilai |
|----------|-------|
| **Tanggal Analisis** | 2026-04-10 |
| **Scope** | BAB VI — Penumpang Per Rute (2020-2024) |
| **Jumlah File CSV** | 20 file (5 tahun × 4 file per tahun) |
| **Jumlah File Dokumentasi** | 25 file analisis (5 tahun × 5 file) + 1 master summary |
| **Total Baris Data** | Sekitar 4,300+ baris (termasuk header dan baris Total) |
| **Rentang Jumlah Rute Domestik** | 303 sampai 410 rute |
| **Rentang Jumlah Rute Internasional** | 125 sampai 147 rute |

---

> **Catatan:** Dokumen ini merangkum perbandingan struktur tabel, format, penamaan kolom, dan tipe data dari BAB VI tahun 2020-2024. Fokus adalah mengidentifikasi ketidakkonsistenan yang harus ditangani sebelum data dimasukan ke database.
