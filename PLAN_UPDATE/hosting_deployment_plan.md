# Hosting & Deployment Plan: Zeabur (100% Gratis Tanpa Kartu Kredit)

Dokumen ini berisi panduan langkah demi langkah yang sangat detail untuk meng-hosting sistem informasi klinik (**clinic-management-system-erm-erp-pos**) secara online secara gratis menggunakan **Zeabur** (platform deployment cloud modern yang mendukung Git-based Docker, gratis, dan tidak memerlukan verifikasi kartu kredit).

---

## 1. Persiapan Database (Supabase)

Supabase menyediakan database PostgreSQL gratis yang sangat stabil dan pendaftarannya tidak memerlukan kartu kredit.

### Langkah Detail Pembuatan:
1. Buka website **[Supabase](https://supabase.com/)** di browser Anda.
2. Klik tombol **Sign in** di kanan atas, lalu pilih **Continue with GitHub** untuk masuk menggunakan akun GitHub Anda.
3. Di halaman dashboard utama Supabase, klik tombol **New Project** (tombol hijau).
4. Pilih organisasi Anda (biasanya nama akun GitHub Anda).
5. Isi formulir pembuatan proyek baru:
   * **Project Name**: Isi dengan `clinic-erp-demo`.
   * **Database Password**: `xkripsi13semester` (Catat password ini baik-baik).
   * **Region**: Pilih **Singapore (ap-southeast-1)** agar koneksi database ke web hosting di Asia berlatensi sangat rendah (cepat).
   * **Pricing**: Pilih **Free** (Paket gratis).
6. Klik tombol **Create new project** dan tunggu proses alokasi database selesai (~2 sampai 3 menit).

### Cara Menemukan Parameter Koneksi:
Setelah proyek Supabase aktif:
1. Klik tombol **Connect** (berwarna hijau/kontras) yang terletak di bagian kanan atas navbar dashboard proyek Supabase Anda.
2. Pada panel/modal menu **Connect to project** yang muncul, pilih tab **Direct Connection**.
3. Salin parameter koneksi berikut untuk dimasukkan ke konfigurasi Zeabur nanti:
   * **Host**: Alamat host Anda (formatnya menyerupai: `aws-0-ap-southeast-1.pooler.supabase.com`).
   * **Database name**: Secara default adalah `postgres`.
   * **Port**: Secara default adalah `5432`.
   * **User**: Username database Anda (formatnya menyerupai: `postgres.xxxxxxxxxxxxxxxxxxxx`).

---

## 2. Deployment Aplikasi di Zeabur (Git-based Docker)

Zeabur akan mendeteksi file `Dockerfile` yang telah kita siapkan di root proyek secara otomatis dan mem-build-nya secara gratis.

### Langkah Detail Deployment:
1. Buka website **[Zeabur](https://zeabur.com/)** di browser Anda.
2. Klik tombol **Sign in / Login** di kanan atas, lalu masuk menggunakan akun **GitHub** Anda (proses ini instan dan tidak memerlukan input kartu kredit).
3. Di halaman dashboard utama Zeabur, klik tombol **Create Project** (atau **New Project**).
4. Pilih lokasi server terdekat: **Singapore (AWS / GCP)** untuk mendapatkan kecepatan maksimal di Indonesia.
5. Setelah ruang proyek berhasil dibuat, klik tombol **Deploy Service** -> pilih **GitHub**.
6. Anda mungkin perlu memberikan otorisasi akses ke repositori Anda. Klik **Configure** dan berikan izin akses ke repositori **`clinic-management-system-erm-erp-pos`**.
7. Setelah repositori Anda muncul di daftar, klik repositori tersebut untuk memulai deployment.
8. Zeabur akan membaca file `Dockerfile` di root proyek Anda dan memulai proses build secara otomatis.

---

## 3. Konfigurasi Environment Variables (Zeabur)

Sebelum proses build selesai, Anda perlu memasukkan konfigurasi environment (sama seperti file `.env` lokal Anda).

1. Klik pada service yang sedang di-deploy di dashboard Zeabur Anda.
2. Masuk ke tab **Variables** di bagian atas menu service.
3. Klik tombol **Bulk Edit** (atau ikon teks) agar Anda bisa menempelkan (*copas*) seluruh baris konfigurasi di bawah ini sekaligus:

```env
APP_NAME="Klinik Kencana Medika"
APP_ENV=production
APP_KEY=base64:/DY7LIZjCGapPw7ZtzJLYhNgx1xXCROKZZPKfZuZDZE=
APP_DEBUG=false
APP_URL=https://klinik-kencana-demo.zeabur.app
DB_CONNECTION=pgsql
DB_HOST=aws-0-ap-southeast-1.pooler.supabase.com
DB_PORT=5432
DB_DATABASE=postgres
DB_USERNAME=postgres.xxxxxxxxxxxxxxxxxxxx
DB_PASSWORD=xkripsi13semester
SESSION_DRIVER=database
CACHE_STORE=database
LOG_CHANNEL=stderr
```

> [!IMPORTANT]
> **Catatan Sebelum Menyimpan:**
> * Sesuaikan `APP_KEY` dengan isi dari file `.env` lokal Anda.
> * Sesuaikan `DB_HOST` dan `DB_USERNAME` dengan parameter koneksi dari Supabase Anda.
> * Klik **Save** setelah selesai. Zeabur akan otomatis melakukan *re-deploy* dengan konfigurasi baru.

---

## 4. Konfigurasi Domain Publik (Zeabur Domain)

Agar aplikasi dapat diakses oleh publik, Anda harus membuat subdomain gratis dari Zeabur.

1. Di dashboard service proyek Zeabur Anda, klik tab **Networking**.
2. Di bagian **Public Endpoints**, klik tombol **Generate Domain** (atau **Custom Domain** jika Anda punya domain sendiri).
3. Isi dengan nama subdomain yang Anda inginkan, misalnya `klinik-kencana-demo` (hasil akhirnya akan menjadi `klinik-kencana-demo.zeabur.app`).
4. Klik **Confirm**. Domain Anda sekarang aktif dan langsung memiliki SSL/HTTPS gratis.

---

## 5. Menjalankan Migrasi & Seeder Pertama Kali di Zeabur

Karena database Supabase masih kosong, kita harus memigrasi tabel dan melakukan seeding default.

1. Di dashboard service proyek Zeabur Anda, klik tab **Console** (atau **Terminal**).
2. Klik tombol **Connect** / **Open Terminal** untuk masuk ke terminal interaktif kontainer Docker aplikasi Anda.
3. Jalankan perintah migrasi database berikut:
   ```bash
   php artisan migrate --force
   ```
4. Setelah migrasi sukses, jalankan perintah seeding untuk membuat user demo bawaan:
   ```bash
   php artisan db:seed --force
   ```

---

## 6. Uptime Monitor (UptimeRobot)

Untuk paket gratis Zeabur, aplikasi akan masuk ke mode tidur (*auto-sleep*) jika tidak ada pengunjung dalam waktu tertentu demi menghemat sumber daya. 

Agar aplikasi web Anda selalu aktif/hangat (tidak ada delay loading awal) dan Supabase tidak di-pause otomatis oleh sistem:

1. Buka website **[UptimeRobot](https://uptimerobot.com/)** dan buat akun gratis.
2. Klik **+ Add New Monitor**.
3. Konfigurasikan monitor baru sebagai berikut:
   * **Monitor Type**: `HTTP(s)`
   * **Friendly Name**: `Demo Klinik Zeabur`
   * **URL (or IP)**: Masukkan URL domain Zeabur Anda (misalnya: `https://klinik-kencana-demo.zeabur.app`)
   * **Monitoring Interval**: Atur ke **5 minutes** (setiap 5 menit).
4. Klik **Create Monitor**.
5. UptimeRobot akan mem-ping website Anda setiap 5 menit untuk memastikan server Zeabur tetap aktif terus-menerus.
