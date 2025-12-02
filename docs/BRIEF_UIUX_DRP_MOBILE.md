# Brief UI/UX DRP Mobile Apps
## Dokumen Panduan Desain Antarmuka Pengguna

---

## Daftar Isi
1. [Latar Belakang & Tujuan](#1-latar-belakang--tujuan)
2. [Prinsip Desain Utama](#2-prinsip-desain-utama)
3. [Struktur Navigasi & Fitur Kunci](#3-struktur-navigasi--fitur-kunci)
4. [Rekomendasi Tambahan](#4-rekomendasi-tambahan)

---

## 1. Latar Belakang & Tujuan

### Gambaran Umum
Aplikasi DRP Mobile digunakan oleh sekitar **35.000 TPP** (PLD, PD, TAPM) untuk melaporkan aktivitas harian yang menjadi dasar pembayaran **Honorarium dan Bantuan Operasional (BOP)**.

### Tujuan Utama
- Memastikan akuntabilitas pelaporan kinerja individu
- Memfasilitasi input data di area sulit sinyal (offline mode)
- Menyederhanakan UX untuk verifikasi berjenjang

### Target Pengguna
TPP dari level Desa (PLD) hingga Pusat (TAPM Pusat)

**Karakteristik Pengguna:**
- Sangat beragam latar belakang
- Sering bekerja di lapangan (desa)
- Koneksi internet terbatas
- Sering terpapar sinar matahari (layar silau)

---

## 2. Prinsip Desain Utama

### 2.1 Compliance-First (Berbasis Kepatuhan)
UI harus mencegah user melakukan kesalahan regulasi secara sistematis.

| Aturan | Implementasi |
|--------|--------------|
| Maksimal jam kerja | Input >12 jam otomatis ditolak |
| Tanggal valid | Input tanggal esok diblokir |

### 2.2 Performance & Stability
- Aplikasi harus ringan
- Mendukung arsitektur **Event Sourcing** untuk beban tinggi
- Memiliki **audit trail** yang sempurna

### 2.3 Transparency
Pengguna harus mengetahui status kinerja mereka secara real-time terhadap target pembayaran:
- Jam Kerja
- Hari Kunjungan

### 2.4 Offline-Adaptive (Adaptif terhadap Sinyal)
- Antarmuka tetap fungsional saat sinyal hilang
- Indikator visual jelas untuk:
  - Data "Menunggu Sinkronisasi"
  - Data "Tersimpan di Server"

---

## 3. Struktur Navigasi & Fitur Kunci

### A. Dashboard Utama (Home Screen)

**Fungsi:** "Alat Monitoring Kepatuhan Pembayaran"

#### Elemen Wajib:

**1. Indikator Kinerja Bulanan (Progress Bar)**
```
JAM KERJA
████████████████░░░░░░░░░░░░░░  105/140 jam (75%)

KUNJUNGAN LAPANGAN
████████████████████░░░░░░░░░░  12/15 hari (80%)
```

| Level | Target Kunjungan |
|-------|------------------|
| PLD/PD | 15 hari/bulan |
| TAPM Kab/Kota | 10 hari/bulan |

**2. Status "Lock" Laporan**
> ⚠️ **PERHATIAN**
> "Laporan akan dikunci permanen pada [Tanggal 3, 00:00]"

**3. Status Verifikasi**
- Indikator apakah laporan bulan lalu sudah diverifikasi
- Disetujui oleh Supervisor (1 jenjang di atas)

**4. Profil Singkat**
- Foto
- Nama
- NIK
- Lokasi Tugas

**5. Status Harian**
- Fokus: "Sudah lapor belum hari ini?"
- Visualisasi sisa jam kerja (Maksimal 12 jam/hari)
- Status sinkronisasi data (Online/Offline)

---

### B. Menu Input Aktivitas (Fitur Inti)

**Prinsip:** Kecepatan dan Kepatuhan

#### Form Input Aktivitas

| Field | Validasi/Aturan |
|-------|-----------------|
| **Tanggal** | Date picker DISABLE tanggal masa depan (besok/lusa tidak bisa dipilih) |
| **Jam Kerja** | Maksimal 12 jam per hari. Jika >12 jam → otomatis ditolak/dipotong dengan notifikasi peringatan |
| **Jenis Aktivitas** | Dropdown/Toggle: "Kunjungan Lapangan" atau "Non-Kunjungan Lapangan". User bisa pilih >1 subjek dampingan |
| **Deskripsi Kegiatan** | Text area luas mencakup: Proses, Masalah, Inovasi |
| **Action Buttons** | "Simpan sebagai Draft" dan "Kirim" (Submit) |

#### Fitur Kunjungan Lapangan (Geotagging)

Jika memilih "Kunjungan Lapangan":
- UI **WAJIB** meminta akses kamera/lokasi
- **Kamera:** Built-in viewfinder (BUKAN upload galeri)
- **Sistem Tagging:** Mekanisme tagging lokasi/desa
  - Jam aktivitas pada tanggal sama terakumulasi benar
  - Menghindari duplikasi jam
- **Bukti Foto:** Capture langsung dengan watermark koordinat
- Foto disimpan di object storage terpisah (meringankan app)

#### Mode Offline

```
🟠 ANDA SEDANG OFFLINE

Data tersimpan di perangkat Anda.
Tekan tombol di bawah saat koneksi tersedia.

[    🔄 SINKRONISASI SEKARANG    ]

⚠️ Wajib sinkronisasi minimal 1x per bulan
```

**Elemen Visual Mode Offline:**
- Banner peringatan oranye
- Tombol "Coba Lagi"
- Status "Menunggu Sinyal"
- Tombol "Sinkronisasi" menyala saat koneksi tersedia

#### Indikator Status Laporan

| Status | Visualisasi |
|--------|-------------|
| Draft | 📝 Abu-abu |
| Pending Sync | 🔄 Oranye (berkedip) |
| Menunggu Validasi | ⏳ Kuning |
| Disetujui | ✅ Hijau |
| Ditolak | ❌ Merah |
| Terkunci | 🔒 Abu-abu gelap + ikon gembok |

---

### C. Riwayat & Edit Laporan

#### 1. Immutability Tanggal
- Pada menu Edit, field "Tanggal" **HARUS DIKUNCI** (disabled)
- User **TIDAK BOLEH** mengubah tanggal aktivitas yang sudah disimpan

#### 2. Audit Trail/History

Menampilkan riwayat status laporan menggunakan pendekatan story:

```
📋 RIWAYAT LAPORAN

● Dibuat         │ 25 Nov 2025, 14:30 │ Oleh: User
↓
● Diedit         │ 25 Nov 2025, 16:45 │ Oleh: User
↓
● Diverifikasi   │ 28 Nov 2025, 09:00 │ Oleh: Supervisor
↓
🔒 Dikunci       │ 03 Des 2025, 00:00 │ Otomatis
```

#### 3. Locked State
- Laporan yang sudah melewati tanggal 3 bulan berikutnya
- Status: "Terkunci" (Locked)
- **TIDAK ADA** tombol Edit/Hapus

---

### D. Fitur Administrasi (Cuti & Profil)

#### Menu Cuti

**1. Rekap Visual**
```
SISA CUTI TAHUNAN

Kuota      :  12 hari
Terpakai   :   4 hari
─────────────────────
SISA       :   8 hari

████████████████░░░░░░░░  8/12 hari tersisa
```

**2. Form Pengajuan Cuti Online**
- Tanggal Mulai
- Tanggal Selesai
- Jenis Cuti
- Alasan/Keterangan
- Upload Dokumen Pendukung (jika diperlukan)

---

### E. Fitur Evaluasi Kinerja (EVKIN) - Triwulanan

> **CATATAN:** Menu TERPISAH dari pelaporan harian (siklus 3 bulanan)

#### Tampilan Utama

```
EVALUASI KINERJA TRIWULAN III/2025

┌──────────────────────────────────────┐
│       NILAI AKHIR KINERJA            │
│                                      │
│                A                     │
│           (Sangat Baik)              │
└──────────────────────────────────────┘
```

#### Rincian Penilaian

| Penilai | Bobot | Nilai | Kontribusi |
|---------|-------|-------|------------|
| Kepala P3MD | 30% | 85 | 25.5 |
| PPK | 60% | 88 | 52.8 |
| Pengguna Layanan | 10% | 90 | 9.0 |
| **TOTAL** | **100%** | - | **87.3** |

> ⚠️ **CATATAN:**
> Nilai telah diubah oleh PPK (Override Authority)
> Nilai sebelumnya: 82.5 → Nilai setelah override: 87.3

---

## 4. Rekomendasi Tambahan

Berdasarkan karakteristik pengguna (bekerja di lapangan, sinar matahari, koneksi terbatas):

| No | Rekomendasi | Alasan |
|----|-------------|--------|
| 1 | **High Contrast Mode** | Layar silau terkena matahari |
| 2 | **Large Touch Targets** | Min 48dp untuk penggunaan di lapangan |
| 3 | **Haptic Feedback** | Konfirmasi input tanpa melihat layar |
| 4 | **Progressive Disclosure** | Info esensial dulu, detail kemudian |
| 5 | **Error Prevention** | Konfirmasi sebelum submit final |
| 6 | **Simple Typography** | Font besar, mudah dibaca |
| 7 | **Minimal Animation** | Hemat baterai & performa |

---

## Informasi Dokumen

| Atribut | Nilai |
|---------|-------|
| **Tanggal** | Desember 2025 |
| **Versi** | 1.0 |
| **Status** | Draft |

---

*Dokumen ini disusun sebagai panduan pengembangan UI/UX untuk Aplikasi DRP Mobile.*