# Hosting & Deployment Plan: Render + Supabase (PostgreSQL)

Dokumen ini berisi panduan langkah demi langkah untuk meng-hosting sistem informasi klinik (**clinic-management-system-erm-erp-pos**) secara online agar dapat diakses oleh publik sebagai demo.

---

## 1. Persiapan Database (Supabase)

Supabase menyediakan database PostgreSQL gratis yang sangat stabil.

1. Buka [Supabase](https://supabase.com/) dan masuk/buat akun baru.
2. Buat proyek baru (*New Project*):
   * **Project Name**: `clinic-erp-demo`
   * **Database Password**: `xkripsi13semester` (Buat password yang kuat dan catat password ini).
   * **Region**: Pilih `Singapore (ap-southeast-1)` agar latensi akses dari Indonesia minimal.
   * **Pricing**: Pilih **Free Tier**.
3. Tunggu hingga proses pembuatan database selesai (~2 menit).
4. Buka menu **Project Settings** (ikon gerigi di kiri bawah) -> **Database**.
5. Gulir ke bawah ke bagian **Connection Parameters**, lalu salin informasi berikut untuk diinput ke Render nanti:
   * **Host**: `aws-0-...pooler.supabase.com`
   * **Database name**: `postgres`
   * **Port**: `5432`
   * **User**: `postgres.xxxxxxxxxx`
   * **Password**: Password database yang Anda buat tadi.

---

## 2. Persiapan Kode Aplikasi (GitHub)

1. Pastikan seluruh perubahan kode terakhir Anda (terutama perbaikan migrasi dan query database-agnostic) sudah dipush ke GitHub.
2. Catat URL repositori GitHub Anda (misal: `https://github.com/username/clinic-management-system-erm-erp-pos`).

---

## 3. Deployment Aplikasi di Render

Render menyediakan layanan hosting web gratis yang sangat cocok untuk aplikasi PHP/Laravel.

1. Buka [Render](https://render.com/) dan masuk menggunakan akun GitHub Anda.
2. Klik tombol **New +** (kanan atas) -> pilih **Web Service**.
3. Pilih opsi **Connect a repository** dan hubungkan repositori proyek klinik ini dari akun GitHub Anda.
4. Di halaman konfigurasi Web Service, atur parameter berikut:
   * **Name**: `klinik-kencana-demo` (ini akan menjadi subdomain demo Anda, misal: `https://klinik-kencana-demo.onrender.com`)
   * **Region**: `Singapore (Southeast Asia)`
   * **Branch**: `main`
   * **Language**: `PHP`
5. Atur perintah build dan start berikut:
   * **Build Command**:
     ```bash
     composer install --no-dev --optimize-autoloader && npm install && npm run build
     ```
   * **Start Command**:
     ```bash
     vendor/bin/heroku-php-nginx public/
     ```
   * **Instance Type**: Pilih **Free**.

---

## 4. Konfigurasi Environment Variables (Render)

Sebelum menekan tombol Deploy, klik tab **Advanced** di bagian bawah konfigurasi Render untuk menambahkan **Environment Variables** (sama seperti isi `.env` lokal Anda):

| Key | Value | Keterangan |
| :--- | :--- | :--- |
| `APP_NAME` | `Klinik Kencana Medika` | Nama aplikasi |
| `APP_ENV` | `production` | Mode produksi |
| `APP_KEY` | `base64:/DY7LIZjCGapPw7ZtzJLYhNgx1xXCROKZZPKfZuZDZE=` | Salin dari `.env` lokal Anda |
| `APP_DEBUG` | `false` | Matikan debug log di production |
| `APP_URL` | `https://klinik-kencana-demo.onrender.com` | Sesuaikan dengan URL Render Anda |
| `DB_CONNECTION` | `pgsql` | Wajib postgresql |
| `DB_HOST` | *(Host Supabase Anda)* | Dari parameter Supabase |
| `DB_PORT` | `5432` | Port PostgreSQL |
| `DB_DATABASE` | `postgres` | Nama database |
| `DB_USERNAME` | *(Username Supabase)* | Mulai dengan `postgres.xxxx` |
| `DB_PASSWORD` | *(Password Supabase)* | Password database Anda |
| `SESSION_DRIVER` | `database` | *Penting*: Di Render Free Tier, filesystem bersifat *ephemeral* (sementara). Menyimpan sesi di database mencegah user tiba-tiba ter-logout otomatis saat server restart. |
| `CACHE_STORE` | `database` | Menyimpan cache di database |
| `LOG_CHANNEL` | `stderr` | Agar log error terkirim ke panel log Render |

Klik **Create Web Service**. Render akan memulai proses build pertama kali.

---

## 5. Menjalankan Migrasi & Seeder Pertama Kali di Render

Karena database Supabase masih kosong, Anda harus menjalankan perintah migrasi setelah Web Service Render selesai ter-deploy (*Active*).

### Cara A: Melalui Shell Render (Paling Praktis)
1. Buka dashboard proyek Anda di Render.
2. Di menu sebelah kiri, klik **Shell**.
3. Jalankan perintah migrasi dan seeding berikut:
   ```bash
   php artisan migrate --force
   php artisan db:seed --force
   ```

### Cara B: Otomatisasi Via Build Command
Anda juga bisa menambahkan perintah migrasi otomatis di akhir **Build Command** Render Anda:
```bash
composer install --no-dev --optimize-autoloader && npm install && npm run build && php artisan migrate --force && php artisan db:seed --force
```

Setelah langkah ini selesai, demo aplikasi klinik Anda sudah online dan dapat diakses oleh publik secara gratis!

---

## 6. Mencegah Render & Supabase Masuk Mode Tidur (UptimeRobot)

Layanan gratis Render akan menonaktifkan (*spin down*) aplikasi jika tidak ada aktivitas selama 15 menit, dan Supabase akan menangguhkan (*pause*) database setelah 7 hari tanpa kueri masuk.

Untuk menjaga keduanya tetap aktif 24/7 (tanpa jeda pemuatan lambat saat pertama kali dibuka oleh pengunjung):

1. Buka website [UptimeRobot](https://uptimerobot.com/) dan buat akun gratis.
2. Di dashboard UptimeRobot, klik tombol **Add New Monitor**.
3. Konfigurasikan monitor baru sebagai berikut:
   * **Monitor Type**: `HTTP(s)`
   * **Friendly Name**: `Klinik Kencana Demo`
   * **URL (or IP)**: Masukkan URL demo Render Anda (misalnya: `https://klinik-kencana-demo.onrender.com`)
   * **Monitoring Interval**: Atur ke **5 minutes** (setiap 5 menit).
4. Klik **Create Monitor**.
5. UptimeRobot akan secara otomatis melakukan ping/mengunjungi website demo Anda setiap 5 menit. Hal ini menjaga server Render tetap aktif/panas dan database Supabase tetap berjalan tanpa pernah di-*pause*.
