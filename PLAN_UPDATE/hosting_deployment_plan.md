# Hosting & Deployment Plan: Serv00 (100% Gratis Tanpa Kartu Kredit)

Dokumen ini berisi panduan langkah demi langkah yang sangat detail untuk meng-hosting sistem informasi klinik (**clinic-management-system-erm-erp-pos**) secara online secara gratis menggunakan **Serv00** (layanan hosting gratis dengan fitur SSH, PHP, dan database lengkap).

---

## 1. Pendaftaran Akun Serv00

1. Buka website **[Serv00 Registration](https://www.serv00.com/webhost/register)**.
2. Isi formulir pendaftaran:
   * **First name** & **Last name**: Nama Anda.
   * **E-mail**: Masukkan email aktif Anda.
   * **Username**: Username pilihan Anda (ini juga akan menjadi subdomain Anda, misal: `klinik.serv00.net` jika Anda memilih domain bawaan).
3. Klik **Create Account**.
4. Buka email masuk dari Serv00 (judul: *New account created*). **PENTING**: Catat data berikut dari email tersebut:
   * **Web panel address**: Alamat login dashboard Anda (misal: `https://panel.serv00.com/`).
   * **SSH/FTP server**: Alamat server Anda (misal: `s15.serv00.com`).
   * **IP address**: Alamat IP server.
   * **Password**: Password akun Anda (berlaku untuk login panel, SSH, dan FTP).

---

## 2. Membuat Database PostgreSQL di Serv00

Serv00 menyediakan database PostgreSQL gratis yang berjalan 24/7 tanpa sistem tidur.

### Langkah Detail:
1. Login ke **Web panel** Serv00 Anda (menggunakan URL, username, dan password dari email).
2. Di sidebar kiri, klik menu **Databases** -> pilih **PostgreSQL**.
3. Klik tombol **Add database**.
4. Konfigurasikan database baru:
   * **Database name**: Isi dengan nama database Anda (misal: `clinicdb`). Nama akhir database akan digabung dengan username Anda (format: `username_clinicdb`).
   * **Database user**: Pilih **Create new user**.
   * **Username**: Isi username baru (misal: `clinicuser`). Format akhirnya: `username_clinicuser`.
   * **Password**: Masukkan password yang kuat untuk database ini.
5. Klik **Add**.
6. Simpan informasi koneksi database Anda:
   * **DB_HOST**: `localhost` (karena database berjalan di server yang sama dengan aplikasi).
   * **DB_PORT**: `5432`
   * **DB_DATABASE**: `username_clinicdb`
   * **DB_USERNAME**: `username_clinicuser`
   * **DB_PASSWORD**: Password database yang Anda buat tadi.

---

## 3. Konfigurasi Domain & PHP Version

Secara default, Serv00 membuat satu website otomatis menggunakan nama domain `username.serv00.net`. Kita harus mengonfigurasi PHP agar kompatibel dengan Laravel (minimal PHP 8.1/8.2).

1. Pada Web panel Serv00, klik menu **WWW websites** di sidebar kiri.
2. Cari nama domain Anda (misal: `username.serv00.net`), lalu klik tombol **Manage** di sebelah kanannya.
3. Ubah pengaturan berikut:
   * **PHP version**: Pilih **PHP 8.2** (atau versi lebih baru yang didukung).
4. Klik **Save**.

---

## 4. Deployment Aplikasi via SSH

Laravel memerlukan akses command line (CLI) untuk mengunduh dependensi (Composer) dan memigrasi database. Serv00 memberikan akses SSH penuh secara gratis.

### Langkah Detail:
1. Buka Terminal (Linux/macOS) atau CMD/PowerShell (Windows) di komputer lokal Anda.
2. Masuk ke server Serv00 Anda menggunakan perintah SSH:
   ```bash
   ssh username@server_anda.serv00.com
   ```
   *(Ganti `username` dengan username Serv00 Anda, dan `server_anda` dengan alamat server dari email, contoh: `ssh fathur@s15.serv00.com`)*.
3. Masukkan password akun Serv00 Anda jika diminta, lalu tekan **Enter**.
4. Masuk ke direktori domain Anda:
   ```bash
   cd domains/username.serv00.net
   ```
5. Hapus folder `public_html` bawaan Serv00 karena kita akan menggantinya dengan folder `public` milik Laravel:
   ```bash
   rm -rf public_html
   ```
6. Clone repositori kode Laravel Anda dari GitHub ke direktori ini:
   ```bash
   git clone https://github.com/FathurRozzaq/clinic-management-system-erm-erp-pos.git temp_src
   ```
7. Pindahkan seluruh file dari hasil clone ke folder utama, lalu hapus folder sementara tersebut:
   ```bash
   mv temp_src/* ./
   mv temp_src/.* ./
   rm -rf temp_src
   ```
8. Buat **symbolic link** (jalan pintas) bernama `public_html` yang mengarah ke folder `public` Laravel agar server web membacanya dengan benar:
   ```bash
   ln -s public public_html
   ```

---

## 5. Menginstal Dependensi & Konfigurasi `.env`

Masih di dalam terminal SSH Serv00 pada direktori `domains/username.serv00.net`:

1. Buat file `.env` produksi dengan menyalin dari `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Edit file `.env` menggunakan editor teks terminal `nano`:
   ```bash
   nano .env
   ```
3. Sesuaikan baris-baris berikut di editor `nano`:
   ```env
   APP_NAME="Klinik Kencana Medika"
   APP_ENV=production
   APP_DEBUG=false
   APP_URL=https://username.serv00.net

   DB_CONNECTION=pgsql
   DB_HOST=localhost
   DB_PORT=5432
   DB_DATABASE=username_clinicdb
   DB_USERNAME=username_clinicuser
   DB_PASSWORD=password_database_anda
   ```
   *(Tekan **Ctrl + O** lalu **Enter** untuk menyimpan, kemudian tekan **Ctrl + X** untuk keluar dari editor nano).*
4. Buat kunci aplikasi Laravel Anda:
   ```bash
   php artisan key:generate
   ```
5. Instal dependensi PHP (Composer) tanpa menyertakan library development untuk menghemat ruang dan RAM:
   ```bash
   composer install --no-dev --optimize-autoloader
   ```
6. Jalankan perintah migrasi tabel dan seeding data default:
   ```bash
   php artisan migrate --force
   php artisan db:seed --force
   ```

---

## 6. Kompilasi Aset JS & CSS (Vite)

Karena aplikasi menggunakan Vite untuk mengelola file Livewire/Tailwind, aset JS/CSS harus di-build terlebih dahulu.

### Cara A: Build Langsung di Serv00 (Praktis)
Jalankan perintah berikut di terminal SSH Serv00 Anda:
```bash
npm install
npm run build
```

### Cara B: Build Lokal (Jika RAM Server Serv00 Terbatas)
Jika kompilasi di server gagal karena limitasi memori (*Out of Memory*):
1. Buka terminal lokal di komputer Anda.
2. Masuk ke proyek lokal klinik Anda, lalu jalankan:
   ```bash
   npm run build
   ```
3. Folder `public/build` akan terisi file kompilasi terbaru.
4. Anda bisa mengunggah folder `public/build` lokal tersebut langsung ke folder `domains/username.serv00.net/public/build` di Serv00 menggunakan aplikasi FTP seperti **FileZilla** (menggunakan detail host SSH, username, dan password Serv00 Anda).

---

Aplikasi klinik Anda kini sudah online sepenuhnya, tidak akan pernah mati atau tidur, dan dapat diakses gratis oleh orang asing melalui link domain bawaan Serv00 Anda (`https://username.serv00.net`)!
