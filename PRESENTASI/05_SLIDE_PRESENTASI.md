---
marp: true
theme: default
paginate: true
backgroundColor: #F3F4F6
color: #111827
style: |
  section {
    font-family: 'Inter', sans-serif;
    padding: 40px 60px;
    font-size: 1.2em;
  }
  h1 {
    color: #0F766E;
    font-family: 'Outfit', sans-serif;
  }
  h2 {
    color: #D97706;
    border-bottom: 2px solid #E5E7EB;
    padding-bottom: 8px;
    font-family: 'Outfit', sans-serif;
  }
  footer {
    font-size: 0.5em;
    color: #9ca3af;
  }
  .highlight {
    color: #0F766E;
    font-weight: bold;
  }
  section.chapter {
    background-color: #111827;
    color: #F9FAFB;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
  section.chapter h1 {
    color: #F9FAFB;
    font-size: 3em;
    font-family: 'Playfair Display', Georgia, serif;
    margin-bottom: 10px;
  }
  section.chapter h2 {
    color: #2DD4BF;
    font-size: 1.6em;
    border-bottom: none;
    font-family: 'Outfit', sans-serif;
    margin-top: 0;
  }
---

Saya ingin kamu membuat slide untuk materi saya.

Ketentuan gambar ilustrasi: untuk menjaga konsistensi visual di seluruh slide, gunakan **Gaya Fotorealistik Sinematik (Realistic Cinematic Photo)** dengan panduan berikut:

*   **Pola Dasar Prompt (Master Template)**:
    > *`[Aktor/Subjek utama] [aktivitas/tindakan] inside a [lokasi/latar klinik], showing [detail teknologi/layar]. Realistic human features, detailed professional photography, cinematic lighting, shallow depth of field, 16:9 aspect ratio`*

*   **Warna Tema (Theme Colors)**:
    *   Primary: `#0F766E` (Teal)
    *   Primary Dark: `#2DD4BF` (Teal Light)
    *   Secondary: `#D97706` (Amber)
    *   Secondary Dark: `#FBBF24` (Amber Light)
    *   Background Light: `#F3F4F6` (Gray) / Surface Light: `#FFFFFF`
    *   Background Dark: `#111827` (Gray) / Surface Dark: `#1F2937`
    *   Text Light: `#111827` (Main), `#4B5563` (Body)
    *   Text Dark: `#F9FAFB` (Main), `#D1D5DB` (Body)


<!-- _class: chapter -->
<!-- _paginate: false -->

# SISTEM MANAJEMEN KLINIK
## ERM, ERP, & POS Terintegrasi

*Tugas Akhir & Implementasi Operasional Klien Nyata*

---

## Awal Mula & Latar Belakang Proyek

Bagaimana sistem terintegrasi ini lahir dan diimplementasikan:

*   **Riset Tugas Akhir**: Awalnya dikembangkan untuk kebutuhan penelitian tugas akhir kuliah saya.
*   **Kebutuhan Ekspansi Klinik**: Atasan di tempat saya bekerja ingin mengekspansi tempat praktik dokternya menjadi sebuah klinik baru.
*   **Solusi Sistem Terpadu**: Beliau meminta dibuatkan sistem operasional terpadu yang sekaligus bisa dipakai di praktik dokternya saat itu juga.
*   **Sudah Implementasi Nyata**: Sistem ini sudah resmi dan sukses diimplementasikan penuh untuk menunjang operasional klinik sehari-hari.

---

## Tantangan Klinik Tradisional

Sering kali operasional klinik tidak efisien karena:

*   **Aplikasi Terpisah-pisah**: Rekam medis, logistik obat, kasir, dan keuangan memiliki aplikasi/platform yang berbeda.
*   **Database Terpisah**: Data tidak sinkron, memicu selisih stok dan kesalahan rekap keuangan.
*   **Kebutuhan Banyak Karyawan**: Dibutuhkan banyak staf hanya untuk meng-input data yang sama berulang kali di masing-masing aplikasi secara manual.
*   **Rekam Medis Kertas**: Rawan hilang, rusak, dan menghambat antrean.

---

## Peta Integrasi & Arsitektur Sistem

```text
                  ┌───────── Pendaftaran & Antrean (FO)
                  ▼
┌────────────────────────────────────────────────────────┐
│             ERM (Electronic Medical Record)            │
│  [Resepsionis] ──> [Perawat] ──> [Dokter] ──> [Pasien] │
└──────────────────────────┬─────────────────────────────┘
                           │ Resep Otomatis & Tindakan
                           ▼
┌────────────────────────────────────────────────────────┐
│               ERP (Logistik & Keuangan)                │
│       [Apoteker (Stok Obat)] ──> [Kasir (POS)]         │
└──────────────────────────┬─────────────────────────────┘
                           │ Laporan Transaksi & Kehadiran
                           ▼
┌────────────────────────────────────────────────────────┐
│            ERP Back-Office & Payroll (Admin)           │
│   [Admin/Owner]: Laba/Rugi, Payroll Staf, Aset Klinik  │
└────────────────────────────────────────────────────────┘
```

---

## Komponen Sistem & Aktor Terlibat

Sistem menyatukan seluruh peran dalam satu database tersinkronisasi:

*   **ERM (Electronic Medical Record)**:
    *   *Aktor*: **Resepsionis** (Pendaftaran), **Perawat** (Tanda Vital/TTV), **Dokter** (Form SOAP & ICD-10).
*   **ERP (Enterprise Resource Planning)**:
    *   *Aktor*: **Apoteker** (Mutasi & Penyiapan Obat), **Admin/Owner** (Manajemen Aset & Keuangan).
*   **POS (Point of Sale)**:
    *   *Aktor*: **Kasir** (Konsolidasi Tagihan & Pembayaran).
*   **Admin/Superuser**: Memiliki akses penuh kelola data master staf, tarif, dan payroll.

---

## Disclaimer Keamanan & Privasi Data

Sebelum masuk ke demo fitur, perlu saya sampaikan:

*   **Identitas & Data Fiktif**: Seluruh nama klinik/perusahaan, identitas pasien, catatan medis, data stok obat, dan data keuangan telah disamarkan dengan data simulasi (fiktif).
*   **Kerahasiaan Pasien & Klien**: Modifikasi data ini ditujukan sepenuhnya demi menjaga keamanan rahasia dagang klien serta menjamin kerahasiaan riwayat medis pasien riil.
*   **Kepatuhan Regulasi**: Sejalan dengan kewajiban menjaga kerahasiaan data pasien sebagaimana diatur dalam **Permenkes Nomor 24 Tahun 2022 tentang Rekam Medis**.

---

<!-- _class: chapter -->

# BAB 01
## SISTEM MEDIS REAL-TIME (ERM)

---

## Modul Medis & ERM (Electronic Medical Record)

Fokus pada efisiensi layanan medis pasien:

*   **Live Queue Sync**: Antrean pasien sinkron otomatis ke layar dokter dan perawat begitu didaftarkan (tanpa reload).
*   **SOAP Standar Medis**: Formulir pengisian rekam medis terstruktur (*Subjective, Objective, Assessment, Plan*) dalam satu halaman.
*   **Cetak Riwayat Periodik**: Cetak atau ekspor rekam medis per rentang tanggal untuk kebutuhan rujukan.

---

<!-- _class: chapter -->

# BAB 02
## LOGISTIK & STOK OBAT

---

## Logistik & Inventaris Obat

Menjamin akurasi obat dan BHP medis:

*   **Notifikasi Meja Racik**: Resep dari dokter langsung terkirim secara otomatis ke bagian penyiapan obat.
*   **Kartu Stok Otomatis**: Pengurangan stok obat dan Bahan Habis Pakai (BHP) secara otomatis saat tindakan medis di-input.
*   **Peringatan Expired Date (ED)**: Alert sistem untuk obat yang mendekati masa kedaluwarsa demi keamanan pasien.

---

<!-- _class: chapter -->

# BAB 03
## POS KASIR TERPADU

---

## POS Kasir & Pembayaran Otomatis

Menghilangkan kesalahan input tagihan kasir:

*   **Tagihan Terintegrasi**: Jasa dokter, biaya tindakan, resep obat, dan BHP otomatis terkonsolidasi menjadi satu tagihan.
*   **Cetak Nota Sekali Klik**: Transaksi kasir langsung tercatat dan struk siap dicetak untuk pasien.
*   **Cetak Etiket Aturan Pakai**: Label aturan pakai obat bisa dicetak langsung baik dari modul Kasir maupun bagian penyiapan obat.

---

<!-- _class: chapter -->

# BAB 04
## OTAK KEUANGAN & ERP

---

## ERP & Laporan Keuangan Otomatis

Modul manajerial terpadu untuk pemilik klinik:

*   **Laporan Laba Rugi Real-Time**: Pendapatan bersih otomatis terhitung dari pemasukan kasir dikurangi HPP obat/BHP dan biaya operasional.
*   **Payroll Penggajian Terintegrasi**: Menghitung gaji pokok, bagi hasil jasa medis dokter, insentif, dan kehadiran staf.
*   **Pemeliharaan Aset**: Monitoring kelayakan kondisi fisik aset tetap klinik secara berkala.

---

<!-- _class: chapter -->

# TEKNOLOGI
## DI BALIK LAYAR

---

## Keunggulan Tech Stack & Arsitektur

*   **Laravel 12 & Livewire 4 SPA**: Arsitektur Hybrid SPA yang menyajikan performa secepat aplikasi desktop tanpa reload browser.
*   **Instalasi Single-Server**: Hanya perlu di-install di satu komputer server. Jauh lebih praktis daripada meng-install aplikasi desktop di semua komputer klien.
*   **Akses Multi-Device**: Dapat diakses dengan lancar dari laptop, tablet, hingga smartphone melalui browser.

---

<!-- _class: chapter -->
<!-- _paginate: false -->

# COBA DEMO INTERAKTIF
## huggingface.co/spaces/FathurRozzaq/clinic-management-system-demo

*Diskusi & Pertanyaan: Hubungi Kontak di Deskripsi Video*
