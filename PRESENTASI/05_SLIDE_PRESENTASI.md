---
marp: true
theme: default
paginate: true
backgroundColor: #f9fafb
color: #1f2937
style: |
  section {
    font-family: 'Inter', sans-serif;
    padding: 40px 60px;
    font-size: 1.2em;
  }
  h1 {
    color: #1e3a8a;
    font-family: 'Outfit', sans-serif;
  }
  h2 {
    color: #2563eb;
    border-bottom: 2px solid #e5e7eb;
    padding-bottom: 8px;
    font-family: 'Outfit', sans-serif;
  }
  footer {
    font-size: 0.5em;
    color: #9ca3af;
  }
  .highlight {
    color: #2563eb;
    font-weight: bold;
  }
  section.chapter {
    background-color: #0B0F19;
    color: #f9fafb;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
  section.chapter h1 {
    color: #f9fafb;
    font-size: 3em;
    font-family: 'Playfair Display', Georgia, serif;
    margin-bottom: 10px;
  }
  section.chapter h2 {
    color: #38bdf8;
    font-size: 1.6em;
    border-bottom: none;
    font-family: 'Outfit', sans-serif;
    margin-top: 0;
  }
---

<!-- _class: chapter -->
<!-- _paginate: false -->

# SISTEM MANAJEMEN KLINIK
## ERM, ERP, & POS Terintegrasi

*Tugas Akhir & Implementasi Operasional Klien Nyata*

---

## Tantangan Klinik Tradisional

Sering kali operasional klinik tidak efisien karena:

*   **Aplikasi Terpisah-pisah**: Rekam medis, logistik obat, kasir, dan keuangan memiliki aplikasi/platform yang berbeda.
*   **Database Terpisah**: Data tidak sinkron, memicu selisih stok dan kesalahan rekap keuangan.
*   **Kebutuhan Banyak Karyawan**: Dibutuhkan banyak staf hanya untuk meng-input data yang sama berulang kali di masing-masing aplikasi secara manual.
*   **Rekam Medis Kertas**: Rawan hilang, rusak, dan menghambat antrean.

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

# DISKUSI & DEMO PRIVAT
## Hubungi Kontak di Deskripsi Video
