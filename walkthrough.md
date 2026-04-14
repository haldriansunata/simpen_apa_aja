# Walkthrough — ETL Pipeline Complete (Fase 1 + Fase 2)

## Summary

Pipeline menghasilkan **9 file CSV bersih** siap Tableau:
- **Fase 1 Core** (5 files): Untuk analisis korelasi kurs vs penumpang
- **Fase 2 Enrichment** (4 files): Untuk konteks tambahan (produksi, OTP, bandara)

## Output Files

### `output/fase1_core/`

| File | Rows | PK/FK |
|------|------|-------|
| `dim_waktu.csv` | 60 | PK: `waktu_id` |
| `dim_rute.csv` | 653 | PK: `rute_id` |
| `fact_kurs_bulanan.csv` | 60 | FK: `waktu_id` |
| `fact_penumpang_rute_bulanan.csv` | 19,650 | FK: `waktu_id`, `rute_id` |
| `fact_penumpang_agregat_bulanan.csv` | 120 | FK: `waktu_id` |

### `output/fase2_enrichment/`

| File | Rows | PK/FK |
|------|------|-------|
| `dim_maskapai.csv` | 19 | PK: `maskapai_id` |
| `fact_produksi_maskapai.csv` | 79 | FK: `maskapai_id` |
| `fact_otp_maskapai.csv` | 62 | FK: `maskapai_id` |
| `fact_lalu_lintas_bandara.csv` | 1,281 | FK: `tahun` |

## ETL Scripts

| Script | Input | Output | Time |
|--------|-------|--------|------|
| [01_dim_waktu.py](file:///d:/Kuliah/projek_dw/etl/01_dim_waktu.py) | Generated | 1 file | <1s |
| [03_fact_kurs.py](file:///d:/Kuliah/projek_dw/etl/03_fact_kurs.py) | 1 CSV | 1 file | <1s |
| [04_fact_penumpang.py](file:///d:/Kuliah/projek_dw/etl/04_fact_penumpang.py) | 10 CSV | 3 files | ~3s |
| [02_dim_maskapai.py](file:///d:/Kuliah/projek_dw/etl/02_dim_maskapai.py) | Filenames + 2 CSV | 1 file | <1s |
| [05_fact_enrichment.py](file:///d:/Kuliah/projek_dw/etl/05_fact_enrichment.py) | 21 CSV | 3 files | ~2s |
| [run_all.py](file:///d:/Kuliah/projek_dw/etl/run_all.py) | Orchestrator | All 9 files | ~7s |

## Shared Libraries

| File | Purpose |
|------|---------|
| [config.py](file:///d:/Kuliah/projek_dw/etl/config.py) | Paths & constants |
| [utils.py](file:///d:/Kuliah/projek_dw/etl/utils.py) | Parsing functions (angka Indonesia, rute IATA, PP normalization, maskapai standardization) |

## Key Technical Decisions

1. **BI.csv**: `skiprows=2` — double header row
2. **Route PP**: `sorted()` on IATA codes → alphabetical normalization, zero duplicates
3. **Route edge cases**: Bare IATA codes (`KXB`, `TRT`), codeshare asterisks (`TPE*`)
4. **BAB VII angka parsing**: Per-value detection (not row-based) — checks if dots create 3-digit segments
5. **BAB VII bandara tanpa dash**: Default to DOMESTIK (bandara kecil hanya domestik)
6. **Maskapai mapping**: 40+ name variants mapped to 19 unique maskapai across BAB II/IV/XII
7. **2024 zero values**: Kept as `0` (conservative, not converted to NULL)

## Verification Results

### Fase 1

| Check | Expected | Actual | ✅ |
|-------|----------|--------|---|
| dim_waktu rows | 60 | 60 | ✅ |
| fact_kurs rows | 60 | 60 | ✅ |
| Kurs 2020 range | 13K-16K | 13,732-15,867 | ✅ |
| DOM 2020 total | ~35.4M | 35,394,235 | ✅ |
| INT 2024 total | ~36.2M | 36,164,923 | ✅ |
| Agregat rows | 120 | 120 | ✅ |
| PP violations | 0 | 0 | ✅ |

### Fase 2

| Check | Expected | Actual | ✅ |
|-------|----------|--------|---|
| dim_maskapai unique | ~19 | 19 | ✅ |
| fact_produksi rows | ≤95 | 79 | ✅ |
| fact_otp rows | ≤70 | 62 | ✅ |
| fact_bandara rows | ~1280 | 1,281 | ✅ |
| Bandara UNKNOWN | 0 | 0 | ✅ |
| Top maskapai 2024 | Lion Air | Lion Air (15.7M) | ✅ |
| Top OTP 2024 | Pelita Air | Pelita Air (94.33%) | ✅ |

## How to Run

```powershell
cd d:\Kuliah\projek_dw
$env:PYTHONIOENCODING='utf-8'

# Run everything:
python etl/run_all.py

# Or individually:
python etl/01_dim_waktu.py
python etl/03_fact_kurs.py
python etl/04_fact_penumpang.py
python etl/02_dim_maskapai.py
python etl/05_fact_enrichment.py
```

## Next Steps

Load ke Tableau Desktop:
1. **Connect** → Text File → Load semua CSV dari `output/fase1_core/`
2. **Join**: `fact_kurs_bulanan.waktu_id = dim_waktu.waktu_id`
3. **Join**: `fact_penumpang_agregat_bulanan.waktu_id = dim_waktu.waktu_id`
4. **Visualisasi**: Dual-axis time series (kurs + penumpang) & scatter plot korelasi
