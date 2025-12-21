# Kamus Data dan API Documentation

## Daftar Isi
- [1. Entity: User](#1-entity-user)
- [2. Entity: Aktivitas](#2-entity-aktivitas)
- [3. Entity: Subjek Dampingan](#3-entity-subjek-dampingan)
- [4. Entity: Lokasi Tugas](#4-entity-lokasi-tugas)
- [5. Entity: Pengajuan Cuti](#5-entity-pengajuan-cuti)
- [6. Entity: Evaluasi Kinerja](#6-entity-evaluasi-kinerja)
- [7. Entity: Sertifikat Pelatihan](#7-entity-sertifikat-pelatihan)
- [8. Entity: Audit Trail](#8-entity-audit-trail)
- [9. Entity: Kinerja Bulanan](#9-entity-kinerja-bulanan)
- [10. API Endpoints](#10-api-endpoints)

---

## 1. Entity: User

### Tabel: `users`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier untuk user | Auto-generated UUID v4 |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email address user | Valid email format, lowercase |
| password_hash | VARCHAR(255) | NOT NULL | Hashed password | Minimum 8 karakter, harus include huruf besar, kecil, angka, dan simbol |
| nama_lengkap | VARCHAR(255) | NOT NULL | Nama lengkap user | Minimal 3 karakter |
| nip | VARCHAR(50) | UNIQUE, NULLABLE | Nomor Induk Pegawai | Format: 18 digit untuk PNS |
| role | ENUM | NOT NULL | Role user dalam sistem | Values: 'admin', 'pendamping', 'supervisor', 'koordinator' |
| unit_kerja | VARCHAR(255) | NULLABLE | Unit kerja user | - |
| jabatan | VARCHAR(255) | NULLABLE | Jabatan user | - |
| no_telepon | VARCHAR(20) | NULLABLE | Nomor telepon user | Format: +62xxx atau 08xxx |
| alamat | TEXT | NULLABLE | Alamat lengkap user | - |
| foto_profil_url | VARCHAR(500) | NULLABLE | URL foto profil | Valid URL format |
| status | ENUM | NOT NULL, DEFAULT 'aktif' | Status user | Values: 'aktif', 'nonaktif', 'suspended' |
| email_verified_at | TIMESTAMP | NULLABLE | Waktu verifikasi email | - |
| last_login_at | TIMESTAMP | NULLABLE | Waktu login terakhir | Auto-update on login |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- UNIQUE INDEX: `email`
- UNIQUE INDEX: `nip`
- INDEX: `role`
- INDEX: `status`
- INDEX: `unit_kerja`

### Business Rules
1. Email harus unik dan valid format
2. Password minimal 8 karakter dengan kombinasi huruf besar, kecil, angka, dan simbol
3. NIP harus unik jika diisi (18 digit untuk PNS)
4. Role menentukan hak akses dalam sistem
5. Status 'nonaktif' atau 'suspended' tidak bisa login
6. User dengan role 'admin' memiliki akses penuh
7. Soft delete tidak digunakan, gunakan status 'nonaktif'

---

## 2. Entity: Aktivitas

### Tabel: `aktivitas`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier aktivitas | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID pendamping yang melakukan aktivitas | - |
| subjek_dampingan_id | UUID | FOREIGN KEY (subjek_dampingan.id), NOT NULL | ID subjek dampingan | - |
| lokasi_tugas_id | UUID | FOREIGN KEY (lokasi_tugas.id), NOT NULL | ID lokasi tugas | - |
| tanggal_aktivitas | DATE | NOT NULL | Tanggal aktivitas dilakukan | Tidak boleh tanggal masa depan |
| waktu_mulai | TIME | NOT NULL | Waktu mulai aktivitas | - |
| waktu_selesai | TIME | NOT NULL | Waktu selesai aktivitas | Harus lebih besar dari waktu_mulai |
| jenis_aktivitas | ENUM | NOT NULL | Jenis aktivitas | Values: 'kunjungan', 'konseling', 'pelatihan', 'monitoring', 'evaluasi', 'rapat_koordinasi', 'lainnya' |
| kategori_dampingan | ENUM | NOT NULL | Kategori dampingan | Values: 'individu', 'kelompok', 'komunitas' |
| jumlah_peserta | INTEGER | NOT NULL, DEFAULT 1 | Jumlah peserta yang didampingi | Minimal 1 |
| deskripsi_aktivitas | TEXT | NOT NULL | Deskripsi detail aktivitas | Minimal 20 karakter |
| tujuan_aktivitas | TEXT | NOT NULL | Tujuan dari aktivitas | Minimal 10 karakter |
| hasil_capaian | TEXT | NULLABLE | Hasil yang dicapai dari aktivitas | - |
| kendala | TEXT | NULLABLE | Kendala yang dihadapi | - |
| solusi | TEXT | NULLABLE | Solusi yang diterapkan | - |
| tindak_lanjut | TEXT | NULLABLE | Rencana tindak lanjut | - |
| latitude | DECIMAL(10,8) | NULLABLE | Koordinat latitude lokasi | Range: -90 to 90 |
| longitude | DECIMAL(11,8) | NULLABLE | Koordinat longitude lokasi | Range: -180 to 180 |
| foto_dokumentasi | JSON | NULLABLE | Array URL foto dokumentasi | Valid JSON array of URLs |
| dokumen_pendukung | JSON | NULLABLE | Array URL dokumen pendukung | Valid JSON array of URLs |
| status_verifikasi | ENUM | NOT NULL, DEFAULT 'draft' | Status verifikasi aktivitas | Values: 'draft', 'submitted', 'verified', 'rejected' |
| verified_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang memverifikasi | Harus role supervisor/koordinator |
| verified_at | TIMESTAMP | NULLABLE | Waktu verifikasi | - |
| catatan_verifikasi | TEXT | NULLABLE | Catatan dari verifikator | - |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- INDEX: `user_id`
- INDEX: `subjek_dampingan_id`
- INDEX: `lokasi_tugas_id`
- INDEX: `tanggal_aktivitas`
- INDEX: `jenis_aktivitas`
- INDEX: `status_verifikasi`
- COMPOSITE INDEX: `(user_id, tanggal_aktivitas)`
- COMPOSITE INDEX: `(subjek_dampingan_id, tanggal_aktivitas)`

### Business Rules
1. Tanggal aktivitas tidak boleh di masa depan
2. Waktu selesai harus lebih besar dari waktu mulai
3. Durasi aktivitas maksimal 24 jam
4. Jumlah peserta minimal 1
5. Deskripsi aktivitas minimal 20 karakter
6. Status 'draft' dapat diedit oleh pendamping
7. Status 'submitted' menunggu verifikasi
8. Status 'verified' tidak dapat diubah kecuali oleh admin
9. Status 'rejected' dapat dikembalikan ke draft untuk revisi
10. Verifikasi hanya dapat dilakukan oleh supervisor/koordinator
11. Foto dokumentasi maksimal 5 file per aktivitas
12. Ukuran file foto maksimal 5MB per file

---

## 3. Entity: Subjek Dampingan

### Tabel: `subjek_dampingan`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier subjek dampingan | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID pendamping yang bertanggung jawab | - |
| nama | VARCHAR(255) | NOT NULL | Nama subjek dampingan | Minimal 3 karakter |
| nik | VARCHAR(16) | UNIQUE, NULLABLE | NIK subjek dampingan | 16 digit untuk individu |
| jenis_subjek | ENUM | NOT NULL | Jenis subjek dampingan | Values: 'individu', 'kelompok', 'organisasi', 'komunitas' |
| kategori | ENUM | NOT NULL | Kategori dampingan | Values: 'pmks', 'psks', 'bpnt', 'pkh', 'lainnya' |
| sub_kategori | VARCHAR(100) | NULLABLE | Sub kategori dampingan | - |
| tanggal_lahir | DATE | NULLABLE | Tanggal lahir (untuk individu) | Tidak boleh tanggal masa depan |
| jenis_kelamin | ENUM | NULLABLE | Jenis kelamin (untuk individu) | Values: 'L', 'P' |
| alamat_lengkap | TEXT | NOT NULL | Alamat lengkap subjek | - |
| kelurahan_desa | VARCHAR(100) | NOT NULL | Kelurahan/Desa | - |
| kecamatan | VARCHAR(100) | NOT NULL | Kecamatan | - |
| kabupaten_kota | VARCHAR(100) | NOT NULL | Kabupaten/Kota | - |
| provinsi | VARCHAR(100) | NOT NULL | Provinsi | - |
| kode_pos | VARCHAR(10) | NULLABLE | Kode pos | - |
| no_telepon | VARCHAR(20) | NULLABLE | Nomor telepon | Format: +62xxx atau 08xxx |
| email | VARCHAR(255) | NULLABLE | Email subjek dampingan | Valid email format |
| pekerjaan | VARCHAR(100) | NULLABLE | Pekerjaan (untuk individu) | - |
| pendidikan_terakhir | ENUM | NULLABLE | Pendidikan terakhir | Values: 'tidak_sekolah', 'sd', 'smp', 'sma', 'diploma', 'sarjana', 'pascasarjana' |
| status_perkawinan | ENUM | NULLABLE | Status perkawinan | Values: 'belum_kawin', 'kawin', 'cerai_hidup', 'cerai_mati' |
| jumlah_anggota | INTEGER | NULLABLE | Jumlah anggota (untuk kelompok/organisasi) | Minimal 1 |
| nama_ketua | VARCHAR(255) | NULLABLE | Nama ketua (untuk kelompok/organisasi) | - |
| tahun_berdiri | INTEGER | NULLABLE | Tahun berdiri (untuk organisasi) | 1900 - current year |
| legalitas | VARCHAR(255) | NULLABLE | Status legalitas (untuk organisasi) | - |
| kondisi_ekonomi | ENUM | NULLABLE | Kondisi ekonomi | Values: 'sangat_miskin', 'miskin', 'rentan_miskin', 'tidak_miskin' |
| kondisi_kesehatan | TEXT | NULLABLE | Kondisi kesehatan | - |
| kondisi_sosial | TEXT | NULLABLE | Kondisi sosial | - |
| kebutuhan_utama | TEXT | NULLABLE | Kebutuhan utama | - |
| foto_subjek_url | VARCHAR(500) | NULLABLE | URL foto subjek | Valid URL format |
| dokumen_identitas_url | JSON | NULLABLE | Array URL dokumen identitas | Valid JSON array of URLs |
| status | ENUM | NOT NULL, DEFAULT 'aktif' | Status subjek dampingan | Values: 'aktif', 'nonaktif', 'lulus', 'pindah', 'meninggal' |
| tanggal_mulai_dampingan | DATE | NOT NULL | Tanggal mulai pendampingan | - |
| tanggal_selesai_dampingan | DATE | NULLABLE | Tanggal selesai pendampingan | Harus lebih besar dari tanggal_mulai |
| catatan | TEXT | NULLABLE | Catatan tambahan | - |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- UNIQUE INDEX: `nik`
- INDEX: `user_id`
- INDEX: `jenis_subjek`
- INDEX: `kategori`
- INDEX: `status`
- INDEX: `kabupaten_kota`
- INDEX: `kecamatan`
- COMPOSITE INDEX: `(user_id, status)`

### Business Rules
1. NIK harus 16 digit dan unik jika diisi
2. Jenis subjek menentukan field yang wajib diisi
3. Untuk individu: tanggal_lahir, jenis_kelamin wajib diisi
4. Untuk kelompok/organisasi: jumlah_anggota, nama_ketua wajib diisi
5. Tanggal lahir tidak boleh di masa depan
6. Tanggal selesai dampingan harus lebih besar dari tanggal mulai
7. Status 'lulus' berarti subjek sudah mandiri
8. Status 'pindah' berarti pindah ke pendamping lain
9. Perubahan status harus dicatat dalam audit trail
10. Alamat harus lengkap minimal sampai tingkat kelurahan/desa

---

## 4. Entity: Lokasi Tugas

### Tabel: `lokasi_tugas`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier lokasi tugas | Auto-generated UUID v4 |
| nama_lokasi | VARCHAR(255) | NOT NULL | Nama lokasi tugas | Minimal 3 karakter |
| kode_lokasi | VARCHAR(50) | UNIQUE, NOT NULL | Kode unik lokasi | Format: LOK-XXX-YYYY |
| jenis_lokasi | ENUM | NOT NULL | Jenis lokasi | Values: 'kantor', 'lapangan', 'komunitas', 'lembaga', 'lainnya' |
| alamat_lengkap | TEXT | NOT NULL | Alamat lengkap lokasi | - |
| kelurahan_desa | VARCHAR(100) | NOT NULL | Kelurahan/Desa | - |
| kecamatan | VARCHAR(100) | NOT NULL | Kecamatan | - |
| kabupaten_kota | VARCHAR(100) | NOT NULL | Kabupaten/Kota | - |
| provinsi | VARCHAR(100) | NOT NULL | Provinsi | - |
| kode_pos | VARCHAR(10) | NULLABLE | Kode pos | - |
| latitude | DECIMAL(10,8) | NOT NULL | Koordinat latitude | Range: -90 to 90 |
| longitude | DECIMAL(11,8) | NOT NULL | Koordinat longitude | Range: -180 to 180 |
| radius_geofence | INTEGER | NOT NULL, DEFAULT 100 | Radius geofence dalam meter | Minimal 10, maksimal 1000 meter |
| pic_nama | VARCHAR(255) | NULLABLE | Nama PIC lokasi | - |
| pic_telepon | VARCHAR(20) | NULLABLE | Nomor telepon PIC | Format: +62xxx atau 08xxx |
| pic_email | VARCHAR(255) | NULLABLE | Email PIC | Valid email format |
| fasilitas | TEXT | NULLABLE | Fasilitas yang tersedia | - |
| kapasitas | INTEGER | NULLABLE | Kapasitas lokasi | Minimal 1 |
| jam_operasional | JSON | NULLABLE | Jam operasional lokasi | Valid JSON object |
| foto_lokasi_url | JSON | NULLABLE | Array URL foto lokasi | Valid JSON array of URLs |
| status | ENUM | NOT NULL, DEFAULT 'aktif' | Status lokasi | Values: 'aktif', 'nonaktif', 'dalam_renovasi' |
| catatan | TEXT | NULLABLE | Catatan tambahan | - |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- UNIQUE INDEX: `kode_lokasi`
- INDEX: `jenis_lokasi`
- INDEX: `status`
- INDEX: `kabupaten_kota`
- INDEX: `kecamatan`
- SPATIAL INDEX: `(latitude, longitude)`

### Business Rules
1. Kode lokasi harus unik dengan format LOK-XXX-YYYY
2. Koordinat latitude dan longitude wajib diisi
3. Radius geofence minimal 10 meter, maksimal 1000 meter
4. Jam operasional dalam format JSON: {"senin": "08:00-16:00", ...}
5. Status 'dalam_renovasi' temporary, akan kembali 'aktif'
6. Validasi koordinat harus dalam wilayah Indonesia
7. Foto lokasi maksimal 10 file
8. Perubahan koordinat harus diverifikasi oleh supervisor

---

## 5. Entity: Pengajuan Cuti

### Tabel: `pengajuan_cuti`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier pengajuan cuti | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID user yang mengajukan cuti | - |
| jenis_cuti | ENUM | NOT NULL | Jenis cuti | Values: 'tahunan', 'sakit', 'melahirkan', 'penting', 'besar', 'bersama', 'diluar_tanggungan_negara' |
| tanggal_mulai | DATE | NOT NULL | Tanggal mulai cuti | Minimal H+2 dari tanggal pengajuan |
| tanggal_selesai | DATE | NOT NULL | Tanggal selesai cuti | Harus >= tanggal_mulai |
| jumlah_hari | INTEGER | NOT NULL | Jumlah hari cuti | Auto-calculated |
| alasan | TEXT | NOT NULL | Alasan pengajuan cuti | Minimal 10 karakter |
| alamat_selama_cuti | TEXT | NOT NULL | Alamat selama cuti | - |
| no_telepon_selama_cuti | VARCHAR(20) | NOT NULL | Nomor telepon selama cuti | Format: +62xxx atau 08xxx |
| dokumen_pendukung | JSON | NULLABLE | Array URL dokumen pendukung | Valid JSON array of URLs |
| status | ENUM | NOT NULL, DEFAULT 'pending' | Status pengajuan | Values: 'pending', 'approved', 'rejected', 'cancelled' |
| approved_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang menyetujui | Harus role supervisor/koordinator |
| approved_at | TIMESTAMP | NULLABLE | Waktu persetujuan | - |
| rejected_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang menolak | Harus role supervisor/koordinator |
| rejected_at | TIMESTAMP | NULLABLE | Waktu penolakan | - |
| alasan_penolakan | TEXT | NULLABLE | Alasan penolakan | Wajib jika status rejected |
| catatan | TEXT | NULLABLE | Catatan tambahan | - |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- INDEX: `user_id`
- INDEX: `jenis_cuti`
- INDEX: `status`
- INDEX: `tanggal_mulai`
- COMPOSITE INDEX: `(user_id, tanggal_mulai)`
- COMPOSITE INDEX: `(user_id, status)`

### Business Rules
1. Tanggal mulai minimal H+2 dari tanggal pengajuan (kecuali cuti sakit)
2. Tanggal selesai harus >= tanggal mulai
3. Jumlah hari dihitung otomatis (hari kerja, exclude weekend & libur nasional)
4. Cuti tahunan maksimal 12 hari per tahun
5. Cuti sakit memerlukan surat dokter jika > 2 hari
6. Cuti melahirkan: 3 bulan (90 hari)
7. Cuti besar: 3 bulan setelah bekerja 6 tahun berturut-turut
8. Status 'pending' dapat diubah oleh user (edit atau cancel)
9. Status 'approved' tidak dapat dibatalkan kecuali oleh admin
10. Status 'rejected' harus disertai alasan penolakan
11. User tidak bisa mengajukan cuti yang overlapping
12. Approval hanya dapat dilakukan oleh atasan langsung atau koordinator
13. Notifikasi email otomatis pada perubahan status

---

## 6. Entity: Evaluasi Kinerja

### Tabel: `evaluasi_kinerja`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier evaluasi | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID user yang dievaluasi | - |
| periode_bulan | INTEGER | NOT NULL | Bulan periode evaluasi | Range: 1-12 |
| periode_tahun | INTEGER | NOT NULL | Tahun periode evaluasi | >= 2020 |
| evaluator_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID evaluator | Harus role supervisor/koordinator |
| tanggal_evaluasi | DATE | NOT NULL | Tanggal evaluasi dilakukan | - |
| jumlah_aktivitas | INTEGER | NOT NULL, DEFAULT 0 | Jumlah aktivitas dalam periode | Auto-calculated |
| jumlah_subjek_dampingan | INTEGER | NOT NULL, DEFAULT 0 | Jumlah subjek dampingan aktif | Auto-calculated |
| target_aktivitas | INTEGER | NOT NULL | Target aktivitas per bulan | Minimal 15 |
| capaian_aktivitas | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Persentase capaian aktivitas | Auto-calculated: (jumlah/target)*100 |
| skor_kuantitas | DECIMAL(5,2) | NOT NULL | Skor aspek kuantitas | Range: 0-100 |
| skor_kualitas | DECIMAL(5,2) | NOT NULL | Skor aspek kualitas | Range: 0-100 |
| skor_ketepatan_waktu | DECIMAL(5,2) | NOT NULL | Skor ketepatan waktu | Range: 0-100 |
| skor_dokumentasi | DECIMAL(5,2) | NOT NULL | Skor kelengkapan dokumentasi | Range: 0-100 |
| skor_kolaborasi | DECIMAL(5,2) | NOT NULL | Skor kolaborasi tim | Range: 0-100 |
| skor_total | DECIMAL(5,2) | NOT NULL | Total skor evaluasi | Auto-calculated: avg of all scores |
| predikat | ENUM | NOT NULL | Predikat kinerja | Values: 'sangat_baik', 'baik', 'cukup', 'kurang' |
| kekuatan | TEXT | NOT NULL | Kekuatan yang dimiliki | Minimal 20 karakter |
| kelemahan | TEXT | NOT NULL | Kelemahan yang perlu diperbaiki | Minimal 20 karakter |
| rekomendasi | TEXT | NOT NULL | Rekomendasi pengembangan | Minimal 20 karakter |
| target_perbaikan | TEXT | NULLABLE | Target perbaikan periode berikutnya | - |
| catatan_evaluator | TEXT | NULLABLE | Catatan dari evaluator | - |
| catatan_user | TEXT | NULLABLE | Tanggapan dari user yang dievaluasi | - |
| status | ENUM | NOT NULL, DEFAULT 'draft' | Status evaluasi | Values: 'draft', 'final', 'acknowledged' |
| acknowledged_at | TIMESTAMP | NULLABLE | Waktu user acknowledge evaluasi | - |
| dokumen_pendukung | JSON | NULLABLE | Array URL dokumen pendukung | Valid JSON array of URLs |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- UNIQUE INDEX: `(user_id, periode_bulan, periode_tahun)`
- INDEX: `user_id`
- INDEX: `evaluator_id`
- INDEX: `periode_tahun`
- INDEX: `status`
- INDEX: `predikat`
- COMPOSITE INDEX: `(user_id, periode_tahun, periode_bulan)`

### Business Rules
1. Satu user hanya boleh memiliki satu evaluasi per periode
2. Evaluator harus berbeda dengan user yang dievaluasi
3. Semua skor dalam range 0-100
4. Skor total = rata-rata dari semua skor komponen
5. Predikat otomatis berdasarkan skor_total:
   - Sangat Baik: >= 85
   - Baik: 70-84
   - Cukup: 55-69
   - Kurang: < 55
6. Target aktivitas minimal 15 per bulan
7. Capaian aktivitas = (jumlah_aktivitas / target_aktivitas) * 100
8. Status 'draft' dapat diedit oleh evaluator
9. Status 'final' tidak dapat diedit, menunggu acknowledgment
10. Status 'acknowledged' berarti user sudah membaca dan menanggapi
11. User dapat memberikan catatan tanggapan
12. Evaluasi harus dilakukan dalam 7 hari setelah akhir periode

---

## 7. Entity: Sertifikat Pelatihan

### Tabel: `sertifikat_pelatihan`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier sertifikat | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID user pemilik sertifikat | - |
| nomor_sertifikat | VARCHAR(100) | UNIQUE, NOT NULL | Nomor sertifikat | Format: CERT-XXX-YYYY-NNNN |
| nama_pelatihan | VARCHAR(255) | NOT NULL | Nama pelatihan | Minimal 5 karakter |
| jenis_pelatihan | ENUM | NOT NULL | Jenis pelatihan | Values: 'teknis', 'manajerial', 'kepemimpinan', 'soft_skill', 'sertifikasi_profesi', 'lainnya' |
| kategori_pelatihan | VARCHAR(100) | NULLABLE | Kategori pelatihan | - |
| penyelenggara | VARCHAR(255) | NOT NULL | Nama penyelenggara pelatihan | - |
| tempat_pelaksanaan | VARCHAR(255) | NOT NULL | Tempat pelaksanaan pelatihan | - |
| tanggal_mulai | DATE | NOT NULL | Tanggal mulai pelatihan | - |
| tanggal_selesai | DATE | NOT NULL | Tanggal selesai pelatihan | Harus >= tanggal_mulai |
| durasi_jam | INTEGER | NOT NULL | Durasi pelatihan dalam jam | Minimal 1 jam |
| tanggal_terbit_sertifikat | DATE | NOT NULL | Tanggal terbit sertifikat | >= tanggal_selesai |
| masa_berlaku_mulai | DATE | NULLABLE | Tanggal mulai masa berlaku | - |
| masa_berlaku_selesai | DATE | NULLABLE | Tanggal akhir masa berlaku | Harus > masa_berlaku_mulai |
| is_sertifikasi | BOOLEAN | NOT NULL, DEFAULT FALSE | Apakah berupa sertifikasi profesi | - |
| lembaga_sertifikasi | VARCHAR(255) | NULLABLE | Nama lembaga sertifikasi | Wajib jika is_sertifikasi = TRUE |
| nomor_registrasi | VARCHAR(100) | NULLABLE | Nomor registrasi sertifikasi | - |
| tingkat_kompetensi | ENUM | NULLABLE | Tingkat kompetensi | Values: 'dasar', 'menengah', 'lanjut', 'ahli' |
| bidang_kompetensi | VARCHAR(255) | NOT NULL | Bidang kompetensi | - |
| materi_pelatihan | TEXT | NULLABLE | Ringkasan materi pelatihan | - |
| nilai_prestasi | DECIMAL(5,2) | NULLABLE | Nilai/prestasi yang diperoleh | Range: 0-100 |
| predikat | VARCHAR(50) | NULLABLE | Predikat kelulusan | - |
| instruktur_pengajar | TEXT | NULLABLE | Nama instruktur/pengajar | - |
| file_sertifikat_url | VARCHAR(500) | NOT NULL | URL file sertifikat | Valid URL, format PDF/Image |
| file_sertifikat_hash | VARCHAR(64) | NULLABLE | Hash SHA-256 file sertifikat | Untuk validasi integritas |
| dokumen_pendukung | JSON | NULLABLE | Array URL dokumen pendukung | Valid JSON array of URLs |
| status_verifikasi | ENUM | NOT NULL, DEFAULT 'pending' | Status verifikasi sertifikat | Values: 'pending', 'verified', 'rejected', 'expired' |
| verified_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang memverifikasi | Harus role admin/supervisor |
| verified_at | TIMESTAMP | NULLABLE | Waktu verifikasi | - |
| catatan_verifikasi | TEXT | NULLABLE | Catatan verifikasi | - |
| keterangan | TEXT | NULLABLE | Keterangan tambahan | - |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- UNIQUE INDEX: `nomor_sertifikat`
- INDEX: `user_id`
- INDEX: `jenis_pelatihan`
- INDEX: `status_verifikasi`
- INDEX: `tanggal_terbit_sertifikat`
- INDEX: `masa_berlaku_selesai`
- COMPOSITE INDEX: `(user_id, jenis_pelatihan)`
- COMPOSITE INDEX: `(user_id, status_verifikasi)`

### Business Rules
1. Nomor sertifikat harus unik dengan format CERT-XXX-YYYY-NNNN
2. Tanggal selesai >= tanggal mulai pelatihan
3. Tanggal terbit sertifikat >= tanggal selesai pelatihan
4. Durasi minimal 1 jam
5. Masa berlaku selesai harus > masa berlaku mulai (jika ada)
6. Jika is_sertifikasi = TRUE, lembaga_sertifikasi wajib diisi
7. File sertifikat wajib diupload (PDF atau Image)
8. Hash file digunakan untuk validasi integritas dokumen
9. Verifikasi dilakukan oleh admin atau supervisor
10. Status otomatis menjadi 'expired' jika melewati masa_berlaku_selesai
11. Sertifikat dengan status 'rejected' dapat direvisi dan diajukan ulang
12. File sertifikat maksimal 5MB
13. Notifikasi otomatis 30 hari sebelum masa berlaku habis

---

## 8. Entity: Audit Trail

### Tabel: `audit_trail`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier audit log | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NULLABLE | ID user yang melakukan aksi | NULL untuk system action |
| action_type | ENUM | NOT NULL | Tipe aksi | Values: 'create', 'read', 'update', 'delete', 'login', 'logout', 'approve', 'reject', 'verify', 'export', 'import' |
| table_name | VARCHAR(100) | NOT NULL | Nama tabel yang terpengaruh | - |
| record_id | UUID | NULLABLE | ID record yang terpengaruh | - |
| old_values | JSON | NULLABLE | Nilai sebelum perubahan | Valid JSON object |
| new_values | JSON | NULLABLE | Nilai setelah perubahan | Valid JSON object |
| changed_fields | JSON | NULLABLE | Array field yang berubah | Valid JSON array |
| description | TEXT | NULLABLE | Deskripsi aksi | - |
| ip_address | VARCHAR(45) | NULLABLE | IP address user | IPv4 atau IPv6 |
| user_agent | TEXT | NULLABLE | User agent browser/app | - |
| device_info | JSON | NULLABLE | Informasi device | Valid JSON object |
| location | VARCHAR(255) | NULLABLE | Lokasi geografis | - |
| request_method | VARCHAR(10) | NULLABLE | HTTP method | Values: GET, POST, PUT, PATCH, DELETE |
| request_url | VARCHAR(500) | NULLABLE | URL endpoint | - |
| response_status | INTEGER | NULLABLE | HTTP response status | - |
| response_time_ms | INTEGER | NULLABLE | Response time dalam milidetik | - |
| error_message | TEXT | NULLABLE | Error message jika ada | - |
| severity | ENUM | NOT NULL, DEFAULT 'info' | Tingkat severity | Values: 'debug', 'info', 'warning', 'error', 'critical' |
| status | VARCHAR(50) | NOT NULL, DEFAULT 'success' | Status aksi | Values: 'success', 'failed', 'partial' |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu aksi dilakukan | Auto-generated, immutable |

### Indexes
- PRIMARY KEY: `id`
- INDEX: `user_id`
- INDEX: `action_type`
- INDEX: `table_name`
- INDEX: `record_id`
- INDEX: `created_at`
- INDEX: `severity`
- COMPOSITE INDEX: `(table_name, record_id)`
- COMPOSITE INDEX: `(user_id, created_at)`
- COMPOSITE INDEX: `(action_type, created_at)`

### Business Rules
1. Audit trail bersifat immutable (tidak dapat diubah atau dihapus)
2. Semua aksi CRUD wajib dicatat
3. old_values dan new_values dalam format JSON
4. changed_fields berisi array nama field yang berubah
5. IP address dan user agent dicatat untuk keamanan
6. Severity level:
   - debug: logging development
   - info: aksi normal
   - warning: aksi yang perlu perhatian
   - error: aksi yang gagal
   - critical: aksi yang mempengaruhi sistem
7. Response time dicatat untuk monitoring performa
8. Error message dicatat untuk troubleshooting
9. Retention policy: data disimpan minimum 2 tahun
10. Akses audit trail terbatas untuk admin dan auditor
11. Export audit trail dalam format encrypted
12. Notifikasi otomatis untuk aksi critical

---

## 9. Entity: Kinerja Bulanan

### Tabel: `kinerja_bulanan`

| Field Name | Data Type | Constraints | Description | Business Rules |
|------------|-----------|-------------|-------------|----------------|
| id | UUID | PRIMARY KEY, NOT NULL | Unique identifier kinerja bulanan | Auto-generated UUID v4 |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ID user | - |
| periode_bulan | INTEGER | NOT NULL | Bulan periode | Range: 1-12 |
| periode_tahun | INTEGER | NOT NULL | Tahun periode | >= 2020 |
| jumlah_aktivitas | INTEGER | NOT NULL, DEFAULT 0 | Total aktivitas dalam bulan | Auto-calculated |
| jumlah_kunjungan | INTEGER | NOT NULL, DEFAULT 0 | Total kunjungan | Auto-calculated |
| jumlah_konseling | INTEGER | NOT NULL, DEFAULT 0 | Total konseling | Auto-calculated |
| jumlah_pelatihan | INTEGER | NOT NULL, DEFAULT 0 | Total pelatihan | Auto-calculated |
| jumlah_monitoring | INTEGER | NOT NULL, DEFAULT 0 | Total monitoring | Auto-calculated |
| jumlah_evaluasi | INTEGER | NOT NULL, DEFAULT 0 | Total evaluasi | Auto-calculated |
| jumlah_subjek_dampingan_aktif | INTEGER | NOT NULL, DEFAULT 0 | Jumlah subjek dampingan aktif | Auto-calculated |
| jumlah_subjek_dampingan_baru | INTEGER | NOT NULL, DEFAULT 0 | Jumlah subjek dampingan baru | Auto-calculated |
| jumlah_subjek_dampingan_lulus | INTEGER | NOT NULL, DEFAULT 0 | Jumlah subjek dampingan lulus | Auto-calculated |
| total_jam_kerja | DECIMAL(10,2) | NOT NULL, DEFAULT 0 | Total jam kerja | Auto-calculated dari aktivitas |
| rata_rata_jam_per_aktivitas | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Rata-rata jam per aktivitas | Auto-calculated |
| jumlah_hari_kerja_efektif | INTEGER | NOT NULL, DEFAULT 0 | Jumlah hari kerja efektif | Exclude weekend, libur, cuti |
| jumlah_hari_cuti | INTEGER | NOT NULL, DEFAULT 0 | Jumlah hari cuti | Auto-calculated |
| jumlah_hari_sakit | INTEGER | NOT NULL, DEFAULT 0 | Jumlah hari sakit | Auto-calculated |
| jumlah_keterlambatan | INTEGER | NOT NULL, DEFAULT 0 | Jumlah keterlambatan | - |
| persentase_kehadiran | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Persentase kehadiran | Auto-calculated |
| persentase_capaian_target | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Persentase capaian target | Auto-calculated |
| target_aktivitas | INTEGER | NOT NULL, DEFAULT 15 | Target aktivitas per bulan | Configurable |
| target_subjek_dampingan | INTEGER | NOT NULL, DEFAULT 10 | Target subjek dampingan | Configurable |
| skor_kinerja | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Skor kinerja keseluruhan | Auto-calculated |
| predikat_kinerja | ENUM | NOT NULL | Predikat kinerja | Values: 'sangat_baik', 'baik', 'cukup', 'kurang' |
| jumlah_dokumen_lengkap | INTEGER | NOT NULL, DEFAULT 0 | Jumlah aktivitas dengan dokumen lengkap | Auto-calculated |
| persentase_kelengkapan_dokumen | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Persentase kelengkapan dokumentasi | Auto-calculated |
| jumlah_aktivitas_terverifikasi | INTEGER | NOT NULL, DEFAULT 0 | Jumlah aktivitas terverifikasi | Auto-calculated |
| persentase_verifikasi | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Persentase aktivitas terverifikasi | Auto-calculated |
| rata_rata_waktu_verifikasi_hari | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Rata-rata waktu verifikasi (hari) | Auto-calculated |
| lokasi_tugas_dikunjungi | INTEGER | NOT NULL, DEFAULT 0 | Jumlah lokasi tugas berbeda yang dikunjungi | Auto-calculated |
| wilayah_cakupan | JSON | NULLABLE | Array wilayah yang dicakup | Valid JSON array |
| pencapaian | TEXT | NULLABLE | Pencapaian khusus | - |
| kendala | TEXT | NULLABLE | Kendala yang dihadapi | - |
| rencana_tindak_lanjut | TEXT | NULLABLE | Rencana bulan berikutnya | - |
| catatan | TEXT | NULLABLE | Catatan tambahan | - |
| status | ENUM | NOT NULL, DEFAULT 'draft' | Status laporan | Values: 'draft', 'submitted', 'approved', 'revised' |
| submitted_at | TIMESTAMP | NULLABLE | Waktu submit laporan | - |
| approved_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang approve | Harus role supervisor/koordinator |
| approved_at | TIMESTAMP | NULLABLE | Waktu approval | - |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pembuatan record | Auto-generated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | Waktu update terakhir | Auto-update |
| created_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang membuat record | - |
| updated_by | UUID | FOREIGN KEY (users.id), NULLABLE | User yang terakhir update | - |

### Indexes
- PRIMARY KEY: `id`
- UNIQUE INDEX: `(user_id, periode_bulan, periode_tahun)`
- INDEX: `user_id`
- INDEX: `periode_tahun`
- INDEX: `status`
- INDEX: `predikat_kinerja`
- COMPOSITE INDEX: `(periode_tahun, periode_bulan)`
- COMPOSITE INDEX: `(user_id, periode_tahun)`

### Business Rules
1. Satu user hanya boleh memiliki satu record per periode bulan
2. Data dihitung otomatis dari tabel aktivitas dan tabel terkait
3. Perhitungan dilakukan setiap akhir bulan (automated job)
4. Skor kinerja calculated dengan formula weighted:
   - Kuantitas aktivitas: 30%
   - Kualitas (verifikasi): 25%
   - Kelengkapan dokumentasi: 20%
   - Kehadiran: 15%
   - Capaian target: 10%
5. Predikat kinerja berdasarkan skor:
   - Sangat Baik: >= 85
   - Baik: 70-84
   - Cukup: 55-69
   - Kurang: < 55
6. Persentase kehadiran = (hari kerja efektif / total hari kerja) * 100
7. Persentase capaian = (jumlah aktivitas / target aktivitas) * 100
8. Status 'draft' dapat diedit oleh sistem
9. Status 'submitted' menunggu approval supervisor
10. Status 'approved' data final, tidak dapat diubah
11. Status 'revised' jika perlu perbaikan data
12. Notifikasi otomatis ke user dan supervisor setiap akhir bulan
13. Dashboard visualisasi trend kinerja 6 bulan terakhir

---

## 10. API Endpoints

### 10.1 Authentication & User Management

#### POST /api/auth/register
Register user baru
```json
Request Body:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "nama_lengkap": "John Doe",
  "nip": "199001012020011001",
  "role": "pendamping",
  "unit_kerja": "Dinas Sosial Jakarta",
  "no_telepon": "081234567890"
}

Response 201:
{
  "status": "success",
  "message": "User registered successfully",
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "nama_lengkap": "John Doe",
    "role": "pendamping"
  }
}
```

#### POST /api/auth/login
Login user
```json
Request Body:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response 200:
{
  "status": "success",
  "message": "Login successful",
  "data": {
    "token": "jwt_token",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "nama_lengkap": "John Doe",
      "role": "pendamping"
    }
  }
}
```

#### GET /api/users
Get all users (Admin only)

#### GET /api/users/:id
Get user by ID

#### PUT /api/users/:id
Update user

#### DELETE /api/users/:id
Delete user (soft delete via status)

---

### 10.2 Aktivitas Management

#### GET /api/aktivitas
Get all aktivitas with filters
```
Query Parameters:
- user_id (optional)
- subjek_dampingan_id (optional)
- tanggal_mulai (optional)
- tanggal_akhir (optional)
- jenis_aktivitas (optional)
- status_verifikasi (optional)
- page (default: 1)
- limit (default: 10)
```

#### POST /api/aktivitas
Create new aktivitas
```json
Request Body:
{
  "subjek_dampingan_id": "uuid",
  "lokasi_tugas_id": "uuid",
  "tanggal_aktivitas": "2025-12-21",
  "waktu_mulai": "09:00:00",
  "waktu_selesai": "12:00:00",
  "jenis_aktivitas": "kunjungan",
  "kategori_dampingan": "individu",
  "jumlah_peserta": 1,
  "deskripsi_aktivitas": "Kunjungan untuk assessment kondisi...",
  "tujuan_aktivitas": "Mengetahui kondisi terkini...",
  "latitude": -6.200000,
  "longitude": 106.816666
}
```

#### GET /api/aktivitas/:id
Get aktivitas by ID

#### PUT /api/aktivitas/:id
Update aktivitas

#### DELETE /api/aktivitas/:id
Delete aktivitas

#### POST /api/aktivitas/:id/verify
Verify aktivitas (Supervisor/Koordinator only)
```json
Request Body:
{
  "status_verifikasi": "verified",
  "catatan_verifikasi": "Aktivitas sudah sesuai"
}
```

#### POST /api/aktivitas/:id/upload-foto
Upload foto dokumentasi

#### GET /api/aktivitas/stats
Get aktivitas statistics

---

### 10.3 Subjek Dampingan Management

#### GET /api/subjek-dampingan
Get all subjek dampingan with filters

#### POST /api/subjek-dampingan
Create new subjek dampingan
```json
Request Body:
{
  "nama": "Jane Doe",
  "nik": "3174010101900001",
  "jenis_subjek": "individu",
  "kategori": "pmks",
  "tanggal_lahir": "1990-01-01",
  "jenis_kelamin": "P",
  "alamat_lengkap": "Jl. Example No. 123",
  "kelurahan_desa": "Kelurahan ABC",
  "kecamatan": "Kecamatan XYZ",
  "kabupaten_kota": "Jakarta Pusat",
  "provinsi": "DKI Jakarta",
  "no_telepon": "081234567890",
  "kondisi_ekonomi": "miskin",
  "tanggal_mulai_dampingan": "2025-01-01"
}
```

#### GET /api/subjek-dampingan/:id
Get subjek dampingan by ID

#### PUT /api/subjek-dampingan/:id
Update subjek dampingan

#### DELETE /api/subjek-dampingan/:id
Delete subjek dampingan (change status)

#### GET /api/subjek-dampingan/:id/aktivitas
Get aktivitas history of subjek dampingan

---

### 10.4 Lokasi Tugas Management

#### GET /api/lokasi-tugas
Get all lokasi tugas

#### POST /api/lokasi-tugas
Create new lokasi tugas
```json
Request Body:
{
  "nama_lokasi": "Kantor Kelurahan ABC",
  "kode_lokasi": "LOK-JKT-2025-001",
  "jenis_lokasi": "kantor",
  "alamat_lengkap": "Jl. Lokasi No. 1",
  "kelurahan_desa": "Kelurahan ABC",
  "kecamatan": "Kecamatan XYZ",
  "kabupaten_kota": "Jakarta Pusat",
  "provinsi": "DKI Jakarta",
  "latitude": -6.200000,
  "longitude": 106.816666,
  "radius_geofence": 100
}
```

#### GET /api/lokasi-tugas/:id
Get lokasi tugas by ID

#### PUT /api/lokasi-tugas/:id
Update lokasi tugas

#### DELETE /api/lokasi-tugas/:id
Delete lokasi tugas

#### GET /api/lokasi-tugas/nearby
Get nearby lokasi tugas based on coordinates

---

### 10.5 Pengajuan Cuti Management

#### GET /api/pengajuan-cuti
Get all pengajuan cuti

#### POST /api/pengajuan-cuti
Create new pengajuan cuti
```json
Request Body:
{
  "jenis_cuti": "tahunan",
  "tanggal_mulai": "2025-12-25",
  "tanggal_selesai": "2025-12-27",
  "alasan": "Liburan keluarga",
  "alamat_selama_cuti": "Jl. Cuti No. 1",
  "no_telepon_selama_cuti": "081234567890"
}
```

#### GET /api/pengajuan-cuti/:id
Get pengajuan cuti by ID

#### PUT /api/pengajuan-cuti/:id
Update pengajuan cuti

#### POST /api/pengajuan-cuti/:id/approve
Approve pengajuan cuti (Supervisor/Koordinator only)

#### POST /api/pengajuan-cuti/:id/reject
Reject pengajuan cuti (Supervisor/Koordinator only)
```json
Request Body:
{
  "alasan_penolakan": "Jadwal tidak memungkinkan"
}
```

#### POST /api/pengajuan-cuti/:id/cancel
Cancel pengajuan cuti

---

### 10.6 Evaluasi Kinerja Management

#### GET /api/evaluasi-kinerja
Get all evaluasi kinerja

#### POST /api/evaluasi-kinerja
Create new evaluasi kinerja
```json
Request Body:
{
  "user_id": "uuid",
  "periode_bulan": 12,
  "periode_tahun": 2025,
  "tanggal_evaluasi": "2025-12-31",
  "target_aktivitas": 20,
  "skor_kuantitas": 85.5,
  "skor_kualitas": 90.0,
  "skor_ketepatan_waktu": 88.0,
  "skor_dokumentasi": 92.0,
  "skor_kolaborasi": 87.5,
  "kekuatan": "Konsisten dalam dokumentasi...",
  "kelemahan": "Perlu peningkatan dalam...",
  "rekomendasi": "Mengikuti pelatihan..."
}
```

#### GET /api/evaluasi-kinerja/:id
Get evaluasi kinerja by ID

#### PUT /api/evaluasi-kinerja/:id
Update evaluasi kinerja

#### POST /api/evaluasi-kinerja/:id/acknowledge
User acknowledge evaluasi

#### GET /api/evaluasi-kinerja/user/:userId
Get evaluasi kinerja by user

---

### 10.7 Sertifikat Pelatihan Management

#### GET /api/sertifikat-pelatihan
Get all sertifikat pelatihan

#### POST /api/sertifikat-pelatihan
Create new sertifikat pelatihan
```json
Request Body (multipart/form-data):
{
  "nomor_sertifikat": "CERT-TRN-2025-0001",
  "nama_pelatihan": "Pelatihan Pendampingan Sosial",
  "jenis_pelatihan": "teknis",
  "penyelenggara": "Kementerian Sosial",
  "tempat_pelaksanaan": "Jakarta",
  "tanggal_mulai": "2025-11-01",
  "tanggal_selesai": "2025-11-05",
  "durasi_jam": 40,
  "tanggal_terbit_sertifikat": "2025-11-10",
  "bidang_kompetensi": "Pendampingan Sosial",
  "file_sertifikat": <file>
}
```

#### GET /api/sertifikat-pelatihan/:id
Get sertifikat pelatihan by ID

#### PUT /api/sertifikat-pelatihan/:id
Update sertifikat pelatihan

#### POST /api/sertifikat-pelatihan/:id/verify
Verify sertifikat (Admin/Supervisor only)

#### DELETE /api/sertifikat-pelatihan/:id
Delete sertifikat pelatihan

---

### 10.8 Audit Trail

#### GET /api/audit-trail
Get audit trail logs (Admin only)
```
Query Parameters:
- user_id (optional)
- action_type (optional)
- table_name (optional)
- tanggal_mulai (optional)
- tanggal_akhir (optional)
- severity (optional)
- page (default: 1)
- limit (default: 50)
```

#### GET /api/audit-trail/:id
Get audit trail by ID

#### POST /api/audit-trail/export
Export audit trail (Admin only)

---

### 10.9 Kinerja Bulanan

#### GET /api/kinerja-bulanan
Get all kinerja bulanan

#### GET /api/kinerja-bulanan/:id
Get kinerja bulanan by ID

#### GET /api/kinerja-bulanan/user/:userId
Get kinerja bulanan by user

#### POST /api/kinerja-bulanan/generate
Generate kinerja bulanan (System/Admin only)
```json
Request Body:
{
  "user_id": "uuid",
  "periode_bulan": 12,
  "periode_tahun": 2025
}
```

#### PUT /api/kinerja-bulanan/:id
Update kinerja bulanan

#### POST /api/kinerja-bulanan/:id/submit
Submit kinerja bulanan

#### POST /api/kinerja-bulanan/:id/approve
Approve kinerja bulanan (Supervisor/Koordinator only)

#### GET /api/kinerja-bulanan/stats/trend
Get trend statistics

---

### 10.10 Dashboard & Reports

#### GET /api/dashboard/overview
Get dashboard overview

#### GET /api/dashboard/stats
Get dashboard statistics

#### GET /api/reports/aktivitas
Generate aktivitas report

#### GET /api/reports/kinerja
Generate kinerja report

#### POST /api/reports/export
Export report to PDF/Excel

---

## General API Response Format

### Success Response
```json
{
  "status": "success",
  "message": "Operation successful",
  "data": {},
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  }
}
```

### Error Response
```json
{
  "status": "error",
  "message": "Error message",
  "errors": [
    {
      "field": "email",
      "message": "Email is required"
    }
  ]
}
```

## HTTP Status Codes
- 200: OK
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Unprocessable Entity
- 500: Internal Server Error

---

**Document Version:** 1.0.0  
**Last Updated:** 2025-12-21  
**Maintained by:** confire-code
