# Rencana Pembuatan Seeder Data Dummy Fiktif (Demo & Video Production)

Dokumen ini berisi rencana teknis untuk merombak seluruh sistem *seeding* database pada aplikasi klinik (**clinic-management-system-erm-erp-pos**). Tujuannya adalah mengganti data yang sensitif (seperti rekam medis pasien asli, nominal keuangan riil, data aset, gaji karyawan, dan data vendor) menjadi **100% data fiktif (dummy)** yang aman untuk kebutuhan pembuatan video demo dan publikasi di internet (seperti Hugging Face Spaces).

---

## 1. Prinsip Sensor dan Anonymisasi Data

Seluruh data yang dapat diasosiasikan dengan klinik asli atau pasien nyata wajib diubah dengan ketentuan sebagai berikut:

| Kategori Data | Status Saat Ini | Rencana Perubahan (100% Fiktif) |
| :--- | :--- | :--- |
| **Identitas Pasien** | Nama, NIK, alamat, No. HP asli/mirip | Menggunakan library **Faker (locale: id_ID)** untuk generate nama acak Indonesia, alamat fiktif, nomor HP acak (`0812xxxxxxxx`), dan NIK buatan. |
| **Rekam Medis (SOAP)** | Catatan subjektif, objektif, diagnosa asli | Menyusun template catatan medis standar untuk penyakit umum (Flu, Hipertensi, Maag/Dispepsia, Sakit Gigi) menggunakan ICD-10 generik. |
| **Resep Obat** | Obat riil, racikan, dosis pasien asli | Menggunakan daftar obat generik populer (Paracetamol, Amoxicillin, Vitamin C) dengan aturan pakai simulasi. |
| **Logistik & Supplier** | Data pembelian riil, nama PBF asli | Menggunakan nama supplier fiktif seperti `PBF Kimia Farma (Simulasi)`, `PBF Medika Sejahtera`, dengan transaksi restock acak. |
| **Keuangan (Cash Ledger)** | Catatan kas masuk/keluar operasional riil | Nominal kas disesuaikan dengan nilai standar demonstrasi. Kategori kas masuk/keluar disederhanakan. |
| **Beban & Aset** | Daftar inventaris dan pengeluaran kantor riil | Membatasi data aset hanya pada inventaris umum (Meja Dokter, Komputer, Stetoskop) dan beban umum (Listrik, Air, ATK) dengan angka fiktif. |
| **Gaji & Karyawan** | Nama staf asli dan gaji riil | Menggunakan nama karyawan fiktif (misal: "dr. Andi Wijaya", "Suster Budi") dengan nominal gaji simulasi. |

---

## 2. Strategi Teknis Menggunakan Model Factories & Faker

Sebagian besar file seeder saat ini berukuran sangat besar (beberapa mencapai ratusan kilobyte) karena menyimpan array data statis hasil ekspor database riil. Kita akan mengganti pendekatan ini menggunakan **Laravel Model Factories** dan **Faker**.

### Keuntungan Pendekatan Factory:
1. **Keamanan 100%**: Tidak ada data riil yang tidak sengaja tertinggal karena data digenerate secara acak oleh program saat seeding dijalankan.
2. **Ukuran File Sangat Kecil**: Mengurangi ukuran file seeder dari total megabytes menjadi hanya beberapa kilobyte kode PHP bersih.
3. **Fleksibilitas Skala**: Jumlah data demo (misal: jumlah pasien, rekam medis, transaksi) dapat disesuaikan hanya dengan mengubah angka parameter perulangan (looping).

---

## 3. Rincian Konversi File Seeder

Berikut adalah rencana penanganan untuk masing-masing dari 33 berkas seeder:

### A. Kelompok Master Data Independen (Tetap Statis tapi Bersih/Fiktif)
* **`UnitSeeder.php`**: Berisi satuan obat standar (Pcs, Tablet, Botol, Ampul). *Aman.*
* **`ServiceCategorySeeder.php`**: Kategori layanan umum (Pemeriksaan Dokter, Tindakan Medis, Laboratorium, Administrasi). *Aman.*
* **`AccountSeeder.php`**: Daftar akun kas (Kas Kecil, Kas Besar, Bank Mandiri Klinik - Simulasi). *Aman.*
* **`ShiftSeeder.php`**: Jam kerja kerja operasional (Pagi, Siang, Malam). *Aman.*
* **`SalaryComponentSeeder.php`**: Komponen gaji (Gaji Pokok, Tunjangan Jabatan, Uang Makan). *Aman.*
* **`Icd10Seeder.php`**: Menggunakan file CSV ICD-10 generik yang berisi kode penyakit umum secara internasional. *Aman.*

### B. Kelompok Master Data dengan Factory & Faker (Dibuat Fiktif Acak)
* **`UserSeeder.php`**: Membuat akun login simulasi untuk Dokter, Perawat, Kasir, Apoteker, dan Administrator dengan password standar (`password`) untuk keperluan pengetesan login oleh publik.
* **`PatientSeeder.php`**: Dikonversi menggunakan model factory dengan Faker `id_ID` untuk membuat ~50 pasien acak lengkap dengan NIK, alamat, dan tanggal lahir acak.
* **`PartnerSeeder.php`**: Mengisi ~5 nama supplier logistik buatan (misal: "PBF Medika Utama", "PBF Sentosa Jaya") dengan nomor kontak acak.
* **`AssetSeeder.php`**: Mengisi inventaris fiktif kantor (AC, Kursi, Timbangan Digital) dengan taksiran harga fiktif.

### C. Kelompok Master Data Terkait (Kombinasi Statis & Acak)
* **`ServiceSeeder.php`**: Daftar tindakan medis (Konsultasi Dokter, Pasang Infus, Nebulizer, Cek Gula Darah) beserta tarif nominal fiktif.
* **`ItemSeeder.php`**: Daftar obat-obatan generik dan alat kesehatan habis pakai (Spuit, Handscoon, Masker) dengan harga beli dan harga jual fiktif yang konsisten.
* **`UserScheduleSeeder.php`**: Jadwal praktik dokter mingguan yang digenerate secara otomatis menggunakan relasi user dokter yang ada.
* **`WorkShiftSeeder.php` & `UserSalaryComponentSeeder.php`**: Mengaitkan shift kerja dan tunjangan gaji pada user fiktif secara dinamis.

### D. Kelompok Transaksi & Rekam Medis (Dibuat Dinamis Berelasi)
* **`VisitSeeder.php`**: Membuat data kunjungan pasien secara acak dalam rentang waktu 3 bulan terakhir. Keluhan, catatan SOAP, dan diagnosa dibuat menggunakan variasi acak dari template penyakit umum.
* **`VisitServiceSeeder.php` & `VisitPrescriptionSeeder.php`**: Menambahkan tindakan medis dan resep obat generik secara acak ke setiap kunjungan yang sesuai.
* **`VisitConsumableSeeder.php`**: Mengurangi stok barang habis pakai (BHP) secara otomatis berdasarkan tindakan medis yang dilakukan.
* **`PaymentSeeder.php` & `PaymentDetailSeeder.php`**: Menghitung total tagihan kunjungan secara otomatis, mencatat pembayaran (Tunai/Transfer), kembalian, dan status lunas secara logis.
* **`CashLedgerSeeder.php`**: Mencatat mutasi kas masuk berdasarkan pembayaran pasien yang lunas, dan kas keluar berdasarkan transaksi operasional fiktif.
* **`InventoryRestockSeeder.php`, `InventoryReturnSeeder.php`, `InventoryWastageSeeder.php`**: Mensimulasikan alur logistik masuk dan keluar obat ke gudang dengan supplier fiktif.

### E. Kelompok Transaksi Penggajian (Dibuat Dinamis)
* **`PayrollSeeder.php` & `PayrollDetailSeeder.php`**: Membuat slip gaji bulanan simulasi untuk karyawan fiktif berdasarkan komponen gaji yang berlaku.
* **`PayrollFeeSeeder.php` & `PayrollAdjustmentSeeder.php`**: Mencatat bonus insentif tindakan dokter/perawat fiktif serta potongan absensi.

---

## 4. Langkah Implementasi & Eksekusi

1. **Membuat Model Factories**: 
   Membuat file factory baru untuk model-model utama yang belum memilikinya di folder `database/factories/` (seperti `PatientFactory`, `VisitFactory`, `ItemFactory`, dll).
2. **Menyederhanakan Logika Seeder**:
   Mengganti isi fungsi `run()` pada masing-masing seeder dengan perulangan factory, misalnya:
   ```php
   // Contoh pada PatientSeeder.php
   Patient::factory()->count(50)->create();
   ```
3. **Pembersihan Database & Re-seeding**:
   Jalankan perintah berikut di lingkungan lokal untuk memastikan seeder baru berjalan tanpa hambatan dan relasi integritas database terjaga:
   ```bash
   php artisan migrate:fresh --seed
   ```
4. **Verifikasi**:
   Lakukan pengecekan pada tabel `patients`, `visits`, `payments`, dan `cash_ledgers` di database lokal untuk memastikan seluruh nama, rekam medis, dan nominal keuangan telah berubah total menjadi data simulasi/fiktif.
