# Kamus Data & API Specification
## DRP Mobile Apps - Data Dictionary & API Documentation

---

## Daftar Isi
1. [Kamus Data (Data Dictionary)](#1-kamus-data-data-dictionary)
2. [API Endpoints Specification](#2-api-endpoints-specification)
3. [Request & Response Examples](#3-request--response-examples)
4. [Error Codes](#4-error-codes)
5. [Authentication & Authorization](#5-authentication--authorization)

---

## 1. Kamus Data (Data Dictionary)

### 1.1 Entitas: User (TPP)

| Field Name | Type | Length | Nullable | Description | Constraint |
|------------|------|--------|----------|-------------|------------|
| `user_id` | UUID | 36 | NO | Primary Key | PK |
| `nik` | VARCHAR | 16 | NO | Nomor Induk Kependudukan | UNIQUE, INDEX |
| `nip` | VARCHAR | 18 | YES | Nomor Induk Pegawai | UNIQUE |
| `nama_lengkap` | VARCHAR | 255 | NO | Nama lengkap TPP | - |
| `email` | VARCHAR | 100 | YES | Email pengguna | UNIQUE |
| `phone` | VARCHAR | 15 | YES | Nomor telepon | - |
| `password_hash` | VARCHAR | 255 | NO | Hashed password | - |
| `foto_profil_url` | TEXT | - | YES | URL foto profil | - |
| `level_tpp` | ENUM | - | NO | Level TPP | PLD, PD, TAPM_KAB, TAPM_PROV, TAPM_PUSAT |
| `status_aktif` | BOOLEAN | 1 | NO | Status kepegawaian | DEFAULT TRUE |
| `lokasi_tugas_id` | UUID | 36 | NO | FK ke tabel lokasi_tugas | FK |
| `supervisor_id` | UUID | 36 | YES | FK ke user (atasan langsung) | FK, SELF-REFERENCE |
| `created_at` | TIMESTAMP | - | NO | Waktu pembuatan | DEFAULT CURRENT_TIMESTAMP |
| `updated_at` | TIMESTAMP | - | NO | Waktu update terakhir | ON UPDATE CURRENT_TIMESTAMP |
| `last_login_at` | TIMESTAMP | - | YES | Login terakhir | - |

---

### 1.2 Entitas: Aktivitas (Activity Report)

| Field Name | Type | Length | Nullable | Description | Constraint |
|------------|------|--------|----------|-------------|------------|
| `aktivitas_id` | UUID | 36 | NO | Primary Key | PK |
| `user_id` | UUID | 36 | NO | FK ke user | FK, INDEX |
| `tanggal_aktivitas` | DATE | - | NO | Tanggal kegiatan | INDEX |
| `jenis_aktivitas` | ENUM | - | NO | Jenis aktivitas | KUNJUNGAN_LAPANGAN, NON_KUNJUNGAN |
| `jam_kerja` | DECIMAL | 4,2 | NO | Durasi jam kerja | CHECK (jam_kerja <= 12.00) |
| `deskripsi` | TEXT | - | NO | Deskripsi kegiatan | MIN 50 chars |
| `lokasi_id` | UUID | 36 | YES | FK ke lokasi (desa/kelurahan) | FK |
| `subjek_dampingan` | JSON | - | YES | Array subjek dampingan | - |
| `foto_bukti_url` | TEXT | - | YES | URL foto geotagging | REQUIRED if KUNJUNGAN |
| `koordinat_lat` | DECIMAL | 10,8 | YES | Latitude | REQUIRED if KUNJUNGAN |
| `koordinat_lng` | DECIMAL | 11,8 | YES | Longitude | REQUIRED if KUNJUNGAN |
| `akurasi_gps` | DECIMAL | 5,2 | YES | Akurasi GPS (meter) | - |
| `status` | ENUM | - | NO | Status laporan | DRAFT, PENDING_SYNC, SUBMITTED, APPROVED, REJECTED, LOCKED |
| `is_offline_created` | BOOLEAN | 1 | NO | Dibuat offline? | DEFAULT FALSE |
| `sync_at` | TIMESTAMP | - | YES | Waktu sinkronisasi | - |
| `verified_by` | UUID | 36 | YES | FK ke user (verifikator) | FK |
| `verified_at` | TIMESTAMP | - | YES | Waktu verifikasi | - |
| `rejection_reason` | TEXT | - | YES | Alasan penolakan | - |
| `locked_at` | TIMESTAMP | - | YES | Waktu penguncian | AUTO: tanggal 3 bulan berikutnya |
| `created_at` | TIMESTAMP | - | NO | Waktu pembuatan | DEFAULT CURRENT_TIMESTAMP |
| `updated_at` | TIMESTAMP | - | NO | Waktu update terakhir | ON UPDATE CURRENT_TIMESTAMP |

**Business Rules:**
- `tanggal_aktivitas` tidak boleh lebih dari hari ini
- Jika `jenis_aktivitas` = KUNJUNGAN_LAPANGAN, maka `foto_bukti_url`, `koordinat_lat`, `koordinat_lng` WAJIB diisi
- Total `jam_kerja` per `user_id` per `tanggal_aktivitas` maksimal 12 jam
- Data otomatis `locked_at` pada tanggal 3 pukul 00:00 bulan berikutnya

---

### 1.3 Entitas: Subjek Dampingan

| Field Name | Type | Length | Nullable | Description | Constraint |
|------------|------|--------|----------|-------------|------------|
| `subjek_id` | UUID | 36 | NO | Primary Key | PK |
| `nama_subjek` | VARCHAR | 255 | NO | Nama kelompok/lembaga | - |
| `jenis_subjek` | ENUM | - | NO | Jenis subjek | BUMDES, POSYANDU, KELOMPOK_TANI, KARANG_TARUNA, PKK, LAINNYA |
| `lokasi_id` | UUID | 36 | NO | FK ke lokasi | FK |
| `kontak` | VARCHAR | 15 | YES | Nomor kontak | - |
| `created_at` | TIMESTAMP | - | NO | Waktu pembuatan | DEFAULT CURRENT_TIMESTAMP |

---

### 1.4 Entitas: Lokasi Tugas

| Field Name | Type | Length | Nullable | Description | Constraint |
|------------|------|--------|----------|-------------|------------|
| `lokasi_id` | UUID | 36 | NO | Primary Key | PK |
| `kode_wilayah` | VARCHAR | 13 | NO | Kode Kemendagri | UNIQUE |
| `provinsi` | VARCHAR | 100 | NO | Nama provinsi | - |
| `kabupaten_kota` | VARCHAR | 100 | NO | Nama kabupaten/kota | - |
| `kecamatan` | VARCHAR | 100 | YES | Nama kecamatan | - |
| `desa_kelurahan` | VARCHAR | 100 | YES | Nama desa/kelurahan | - |
| `created_at` | TIMESTAMP | - | NO | Waktu pembuatan | DEFAULT CURRENT_TIMESTAMP |

---

### 1.5 Entitas: Pengajuan Cuti

| Field Name | Type | Length | Nullable | Description | Constraint |
|------------|------|--------|----------|-------------|------------|
| `cuti_id` | UUID | 36 | NO | Primary Key | PK |
| `user_id` | UUID | 36 | NO | FK ke user | FK, INDEX |
| `jenis_cuti` | ENUM | - | NO | Jenis cuti | TAHUNAN, SAKIT, MELAHIRKAN, ALASAN_PENTING |
| `tanggal_mulai` | DATE | - | NO | Tanggal mulai cuti | - |
| `tanggal_selesai` | DATE | - | NO | Tanggal selesai cuti | - |
| `durasi_hari` | INT | - | NO | Jumlah hari | AUTO CALCULATED |
| `alasan` | TEXT | - | NO | Alasan pengajuan | - |
| `dokumen_pendukung_url` | TEXT | - | YES | URL dokumen (PDF/JPG) | - |
| `status` | ENUM | - | NO | Status pengajuan | PENDING, APPROVED, REJECTED |
| `approved_by` | UUID | 36 | YES | FK ke user (approver) | FK |
| `approved_at` | TIMESTAMP | - | YES | Waktu persetujuan | - |
| `rejection_reason` | TEXT | - | YES | Alasan penolakan | - |
| `created_at` | TIMESTAMP | - | NO | Waktu pengajuan | DEFAULT CURRENT_TIMESTAMP |
| `updated_at` | TIMESTAMP | - | NO | Waktu update | ON UPDATE CURRENT_TIMESTAMP |

**Business Rules:**
- Kuota cuti tahunan: 12 hari per tahun
- `tanggal_mulai` dan `tanggal_selesai` tidak boleh overlap dengan cuti lain yang sudah APPROVED
- `durasi_hari` = COUNT(working_days) antara `tanggal_mulai` dan `tanggal_selesai`

---

### 1.6 Entitas: Evaluasi Kinerja (EVKIN)

| Field Name | Type | Length | Nullable | Description | Constraint |
|------------|------|--------|----------|-------------|------------|
| `evkin_id` | UUID | 36 | NO | Primary Key | PK |
| `user_id` | UUID | 36 | NO | FK ke user | FK, INDEX |
| `tahun` | INT | 4 | NO | Tahun evaluasi | - |
| `triwulan` | INT | 1 | NO | Triwulan (1-4) | CHECK (triwulan BETWEEN 1 AND 4) |
| `nilai_p3md` | DECIMAL | 5,2 | YES | Nilai dari Kepala P3MD | 0-100 |
| `nilai_ppk` | DECIMAL | 5,2 | YES | Nilai dari PPK | 0-100 |
| `nilai_pengguna` | DECIMAL | 5,2 | YES | Nilai dari Pengguna Layanan | 0-100 |
| `nilai_akhir` | DECIMAL | 5,2 | YES | Nilai akhir (weighted) | AUTO CALCULATED |
| `grade` | ENUM | - | YES | Grade kinerja | A, B, C |
| `is_override` | BOOLEAN | 1 | NO | Diubah oleh PPK? | DEFAULT FALSE |
| `override_reason` | TEXT | - | YES | Alasan override | REQUIRED if is_override |
| `original_nilai` | DECIMAL | 5,2 | YES | Nilai sebelum override | - |
| `created_at` | TIMESTAMP | - | NO | Waktu pembuatan | DEFAULT CURRENT_TIMESTAMP |
| `updated_at` | TIMESTAMP | - | NO | Waktu update | ON UPDATE CURRENT_TIMESTAMP |

**Business Rules:**
- `nilai_akhir` = (nilai_p3md × 0.30) + (nilai_ppk × 0.60) + (nilai_pengguna × 0.10)
- Grade: A: nilai_akhir >= 85, B: 70 <= nilai_akhir < 85, C: nilai_akhir < 70

---
