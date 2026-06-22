# Hosting & Deployment Plan: Render + Supabase (PostgreSQL)

Dokumen ini berisi panduan langkah demi langkah yang sangat detail untuk meng-hosting sistem informasi klinik (**clinic-management-system-erm-erp-pos**) secara online agar dapat diakses oleh publik sebagai demo.

---

## 1. Persiapan Database (Supabase)

Supabase menyediakan database PostgreSQL gratis yang sangat stabil.

### Langkah Detail Pembuatan:
1. Buka website **[Supabase](https://supabase.com/)** di browser Anda.
2. Klik tombol **Sign in** di kanan atas, lalu pilih **Continue with GitHub** untuk masuk menggunakan akun GitHub Anda.
3. Di halaman dashboard utama Supabase, klik tombol **New Project** (tombol hijau).
4. Pilih organisasi Anda (biasanya nama akun GitHub Anda).
5. Isi formulir pembuatan proyek baru:
   * **Project Name**: Isi dengan `clinic-erp-demo`.
   * **Database Password**: `xkripsi13semester` Ketik password yang kuat. **PENTING**: Catat password ini sekarang karena Supabase tidak akan menampilkannya lagi demi alasan keamanan!
   * **Region**: Pilih **Singapore (ap-southeast-1)** agar koneksi database ke web hosting di Asia berlatensi sangat rendah (cepat).
   * **Pricing**: Pilih **Free** (Paket gratis).
6. Klik tombol **Create new project** dan tunggu proses alokasi database selesai (~2 sampai 3 menit).

### Cara Menemukan Parameter Koneksi (Host, User, Port):
Setelah proyek Supabase aktif (status bar hijau berubah menjadi *Active*):
1. Klik tombol **Connect** (berwarna hijau/kontras) yang terletak di bagian kanan atas navbar dashboard proyek Supabase Anda.
2. Pada panel/modal menu **Connect to project** yang muncul, pilih tab **Direct Connection** (atau pilih tab **URI** untuk melihat connection string lengkap).
3. Dari parameter koneksi yang ditampilkan, salin nilai-nilai berikut untuk dimasukkan ke konfigurasi Render nanti:
   * **Host**: Salin alamat host Anda (formatnya menyerupai: `aws-0-ap-southeast-1.pooler.supabase.com`).
   * **Database name**: Secara default adalah `postgres`.
   * **Port**: Secara default adalah `5432`.
   * **User**: Salin username database Anda (formatnya menyerupai: `postgres.xxxxxxxxxxxxxxxxxxxx`).
   * **Password**: Gunakan password yang Anda buat secara manual pada langkah sebelumnya (jika lupa, klik tombol **Reset database password** pada halaman settings database untuk membuat password baru).

---

## 2. Persiapan Kode Aplikasi (GitHub)

1. Buka website **[GitHub](https://github.com/)** dan masuk ke akun Anda.
2. Klik foto profil Anda di kanan atas -> pilih **Your repositories**.
3. Klik pada repositori proyek klinik Anda: **`clinic-management-system-erm-erp-pos`**.
4. Salin URL repositori tersebut dari address bar browser Anda (contoh format: `https://github.com/username/clinic-management-system-erm-erp-pos`).
5. Pastikan seluruh perubahan kode terakhir (terutama perbaikan migrasi dan query PostgreSQL) sudah dipush ke branch `main`.

---

## 3. Deployment Aplikasi di Render

Karena Render tidak menyediakan opsi bahasa PHP secara langsung di dashboard, kita menggunakan **Docker** yang merupakan standar industri terbaik untuk menjalankan Laravel di Render. File `Dockerfile` konfigurasi Nginx + PHP 8.2 sudah disediakan di root proyek utama Anda.

### Langkah Detail Deployment:
1. Buka website **[Render](https://render.com/)** di browser Anda.
2. Klik tombol **Sign in** di kanan atas, lalu pilih **GitHub** agar akun Render langsung terhubung dengan repositori GitHub Anda.
3. Di halaman dashboard utama Render, klik tombol biru **New +** di sebelah kanan atas, lalu pilih **Web Service**.
4. Di bagian **Connect a repository**:
   * Jika repositori Anda belum muncul di daftar, klik tombol **Configure GitHub App** di bawahnya untuk memberi izin Render membaca repositori Anda.
   * Setelah repositori `clinic-management-system-erm-erp-pos` muncul, klik tombol **Connect** di sebelahnya.
5. Pada formulir konfigurasi Web Service, isi parameter berikut:
   * **Name**: Isi dengan `klinik-kencana-demo` (nama ini akan menentukan subdomain URL web Anda, misalnya: `https://klinik-kencana-demo.onrender.com`).
   * **Region**: Pilih **Singapore (Southeast Asia)** agar dekat dengan lokasi database Supabase Anda.
   * **Branch**: Pilih `main`.
   * **Language**: Pilih **Docker**.
   * *(Catatan: Kolom Build Command dan Start Command tidak perlu diisi karena otomatis dijalankan berdasarkan konfigurasi di `Dockerfile`)*.
   * **Instance Type**: Pilih **Free** (di bagian paling bawah).

---

## 4. Konfigurasi Environment Variables (Render)

Sebelum mengklik tombol deploy, Anda harus memasukkan konfigurasi environment (sama seperti file `.env` lokal Anda).

1. Pada halaman konfigurasi Render yang sama, gulir ke bawah dan klik tab **Advanced** (atau jika sudah terlanjur terdeploy, klik menu **Environment** di sidebar kiri dashboard proyek Render Anda).
2. Anda dapat memasukkannya satu per satu dengan mengeklik **Add Environment Variable**, atau cara yang **jauh lebih cepat**: klik tombol **Raw Editor** / **Secret File** lalu tempel (*copas*) seluruh baris konfigurasi di bawah ini sekaligus:

```env
APP_NAME="Klinik Kencana Medika"
APP_ENV=production
APP_KEY=base64:/DY7LIZjCGapPw7ZtzJLYhNgx1xXCROKZZPKfZuZDZE=
APP_DEBUG=false
APP_URL=https://klinik-kencana-demo.onrender.com
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
> **Catatan Sebelum Copas:**
> * Ubah nilai `APP_KEY` dengan isi key dari `.env` lokal Anda jika berbeda.
> * Ubah `APP_URL` dengan URL Render Anda sendiri.
> * Sesuaikan `DB_HOST`, `DB_USERNAME`, dan `DB_PASSWORD` dengan data asli dari database Supabase Anda.

3. Setelah semua variabel dimasukkan, klik tombol biru **Create Web Service** (atau **Save Changes** jika Anda mengeditnya lewat tab Environment).
4. Render akan memulai proses build secara otomatis. Proses ini biasanya memakan waktu 3 sampai 5 menit.

---

## 5. Menjalankan Migrasi & Seeder Pertama Kali di Render

Karena database baru Anda di Supabase masih kosong (belum ada tabel sama sekali), Anda harus membuat tabel dan mengisinya dengan user default agar aplikasi bisa digunakan.

### Cara A: Melalui Shell Terintegrasi Render (Paling Praktis)
1. Tunggu hingga status Web Service di Render berubah menjadi hijau (**Live**).
2. Lihat menu navigasi di sebelah kiri dashboard Render Anda, lalu klik menu **Shell**.
3. Tunggu hingga terminal interaktif termuat di browser Anda.
4. Jalankan perintah migrasi tabel berikut lalu tekan **Enter**:
   ```bash
   php artisan migrate --force
   ```
5. Setelah migrasi sukses, jalankan perintah seeding untuk membuat user demo default (misal: admin, kasir, dokter) dengan menekan **Enter**:
   ```bash
   php artisan db:seed --force
   ```

### Cara B: Otomatisasi Lewat Pre-deploy Command
Karena kita menggunakan Docker, Render menyediakan opsi **Pre-deploy Command** yang otomatis berjalan di kontainer baru sebelum aplikasi dipublikasikan:
1. Masuk ke dashboard Render Anda -> pilih Web Service Anda -> klik menu **Settings** di sidebar kiri.
2. Cari bagian **Pre-deploy Command**, lalu klik **Edit** dan masukkan perintah berikut:
   ```bash
   php artisan migrate --force
   ```
3. Klik **Save Changes**. Dengan ini, database akan otomatis dimigrasi setiap kali Render melakukan rebuild aplikasi. (Catatan: Untuk seeding pertama kali tetap direkomendasikan melalui **Shell** seperti Cara A).

---

## 6. Mencegah Render & Supabase Masuk Mode Tidur (UptimeRobot)

Untuk layanan gratis, Render akan menonaktifkan (*spin down*) aplikasi jika tidak ada aktivitas selama 15 menit, dan Supabase akan menangguhkan (*pause*) database setelah 7 hari tanpa kueri masuk.

Untuk menjaga keduanya tetap aktif 24/7 (tanpa jeda pemuatan lambat saat pertama kali dibuka oleh pengunjung):

### Langkah Detail UptimeRobot:
1. Buka website **[UptimeRobot](https://uptimerobot.com/)** di browser Anda.
2. Klik tombol hijau **Register for FREE** di kanan atas dan buat akun baru.
3. Setelah masuk ke dashboard UptimeRobot, klik tombol hijau **+ Add New Monitor** di sebelah kiri atas.
4. Isi konfigurasi monitor baru Anda:
   * **Monitor Type**: Pilih **HTTP(s)**.
   * **Friendly Name**: Ketik nama bebas, contoh: `Demo Klinik Kencana`.
   * **URL (or IP)**: Masukkan URL demo aplikasi Render Anda (contoh: `https://klinik-kencana-demo.onrender.com`).
   * **Monitoring Interval**: Atur ke **5 minutes** (setiap 5 menit sekali).
5. Klik tombol **Create Monitor** di bagian bawah.
6. UptimeRobot akan mengirim ping otomatis ke website Anda setiap 5 menit. Kunjungan berkala ini mencegah server Render tertidur dan menjaga database Supabase tetap aktif tanpa pernah di-*pause*.
