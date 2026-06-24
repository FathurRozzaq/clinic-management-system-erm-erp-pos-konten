# Panduan Hosting & Deployment: Hugging Face Spaces (Docker + SQLite)

Dokumen ini berisi panduan langkah demi langkah yang telah direvisi untuk meng-hosting sistem informasi klinik (**clinic-management-system-erm-erp-pos**) secara online secara gratis menggunakan **Hugging Face Spaces** dengan arsitektur berbasis **Docker** dan database lokal **SQLite**.

Dengan setup ini, Anda **tidak memerlukan database eksternal (seperti Supabase)**, sehingga proses deployment menjadi jauh lebih sederhana, cepat, dan 100% gratis tanpa memerlukan kartu kredit.

---

## Keuntungan Menggunakan Hugging Face Spaces + Docker + SQLite
1. **Spesifikasi Mesin Besar**: Dijalankan di atas container dengan spesifikasi 2 vCPU dan 16 GB RAM (CPU Basic - Free).
2. **Tanpa Database Eksternal**: SQLite berjalan langsung di dalam container sebagai berkas lokal (`database.sqlite`), menyederhanakan konfigurasi.
3. **Pre-Seeded Database**: Proses migrasi database dan pengisian data demo (seeding) dilakukan langsung pada saat proses build image Docker. Aplikasi siap digunakan secara instan begitu kontainer aktif.
4. **Keamanan Cookie & HTTPS**: Konfigurasi server telah disesuaikan agar berjalan di bawah HTTPS Hugging Face dengan cookie samesite/secure yang kompatibel dengan Livewire.

---

## 1. File Konfigurasi Dockerfile

Untuk menjalankan aplikasi Laravel di Hugging Face Spaces, kita menggunakan `Dockerfile` yang diletakkan di root direktori proyek. Konfigurasi `Dockerfile` yang digunakan adalah sebagai berikut:

```dockerfile
FROM webdevops/php-nginx:8.4-alpine

# Install NodeJS dan NPM untuk compile assets Vite
RUN apk add --no-cache nodejs npm

# Set working directory ke /app (standar webdevops)
WORKDIR /app

# Copy seluruh file aplikasi ke dalam container
COPY . /app

# Hindari error/warning kepemilikan git di dalam container
RUN git config --global --add safe.directory /app

# Konfigurasi environment default webdevops
ENV WEB_DOCUMENT_ROOT=/app/public
ENV PHP_DATE_TIMEZONE=Asia/Jakarta
ENV PHP_MEMORY_LIMIT=512M

# Ubah port Nginx default dari 80 ke 7860 (keharusan Hugging Face)
RUN sed -i 's/listen 80;/listen 7860;/g' /opt/docker/etc/nginx/vhost.conf \
    && sed -i 's/listen 80 default_server;/listen 7860 default_server;/g' /opt/docker/etc/nginx/vhost.conf

# Install dependensi PHP (Composer)
RUN composer install --no-dev --optimize-autoloader

# Buat file .env default jika tidak ada (untuk deployment)
RUN echo "APP_NAME=ClinicManagementSystem" > .env \
    && echo "APP_ENV=local" >> .env \
    && echo "APP_KEY=" >> .env \
    && echo "APP_DEBUG=true" >> .env \
    && echo "LOG_CHANNEL=stderr" >> .env \
    && echo "DB_CONNECTION=sqlite" >> .env \
    && echo "DB_DATABASE=/app/database/database.sqlite" >> .env \
    && echo "SESSION_DRIVER=file" >> .env \
    && echo "QUEUE_CONNECTION=sync" >> .env \
    && echo "CACHE_STORE=file" >> .env \
    && echo "FORCE_HTTPS=true" >> .env \
    && echo "SESSION_SECURE_COOKIE=true" >> .env \
    && echo "SESSION_SAME_SITE=none" >> .env \
    && echo "SESSION_PARTITIONED_COOKIE=true" >> .env

# Buat file database SQLite kosong
RUN mkdir -p database && touch database/database.sqlite

# Generate APP_KEY
RUN php artisan key:generate

# Jalankan migrasi dan seeder ke database SQLite
RUN php artisan migrate --force --seed

# Install dependensi JS dan compile assets menggunakan Vite
RUN npm install && npm run build

# Atur permission seluruh folder agar bisa ditulis oleh web server (running as application)
RUN chown -R application:application /app

# Port standar yang dibuka (Hugging Face)
EXPOSE 7860
```

---

## 2. Pembuatan Space di Hugging Face

1. Buka website **[Hugging Face](https://huggingface.co/)**.
2. Klik **Sign Up** di kanan atas dan buat akun baru Anda (100% gratis, tanpa kartu kredit).
3. Setelah masuk, klik foto profil Anda di kanan atas -> pilih **New Space**.
4. Isi formulir pembuatan Space:
   * **Space Name**: Isi dengan nama demo Anda (misalnya: `clinic-management-demo`).
   * **License**: Pilih `mit` (atau bebas).
   * **SDK / Space template**: Pilih **Docker**.
   * **Docker template**: Pilih **Blank** (Penting! Jangan pilih template lain).
   * **Space hardware**: Pilih **CPU Basic (Free - 2 vCPU - 16 GB RAM - 50GB storage)**.
   * **Visibility**: Pilih **Public** (wajib untuk gratis).
5. Klik **Create Space** di bagian paling bawah.

---

## 3. Konfigurasi Git LFS (Penting untuk Berkas Biner)

Hugging Face Spaces membatasi pengunggahan berkas biner besar secara langsung melalui Git tradisional. Untuk menghindari penolakan push, pastikan berkas gambar (`.png`, `.jpg`) dan dokumen (`.pdf`) dilacak menggunakan Git LFS.

Sebelum melakukan push, pastikan file `.gitattributes` di root direktori proyek Anda berisi aturan berikut:
```text
*.png filter=lfs diff=lfs merge=lfs -text
*.pdf filter=lfs diff=lfs merge=lfs -text
```

Jika belum, jalankan perintah pelacakan ini pada terminal lokal Anda:
```bash
git lfs track "*.png"
git lfs track "*.pdf"
git add .gitattributes
```

---

## 4. Cara Mendeploy Kode Aplikasi ke Hugging Face

### A. Mendapatkan Access Token Hugging Face
1. Klik foto profil Anda di kanan atas website Hugging Face -> pilih **Settings**.
2. Di sidebar kiri, klik **Access Tokens**.
3. Klik tombol **New token**.
4. Isi nama token (misal: `deploy-clinic`), ubah **Token type** menjadi **Write**, lalu klik **Generate a token**.
5. Salin token yang dihasilkan (token ini akan digunakan sebagai password saat melakukan push Git).

### B. Menghubungkan Git Lokal & Push Kode
1. Buka terminal atau Git Bash di komputer Anda, lalu masuk ke folder repositori proyek:
   ```bash
   cd e:\ProgramFiles\laragon\www\clinic-management-system-demo
   ```
2. Daftarkan alamat repositori Space Hugging Face Anda sebagai remote Git (ganti `USERNAME` dan `SPACE_NAME` sesuai akun Anda):
   ```bash
   git remote add hf https://huggingface.co/spaces/USERNAME/SPACE_NAME
   ```
3. Lakukan commit dan push berkas ke Hugging Face:
   ```bash
   git add .
   git commit -m "Deploy Laravel Clinic App via Docker and SQLite"
   git push hf main
   ```
4. Ketika Git meminta **Username**, masukkan username Hugging Face Anda.
5. Ketika meminta **Password**, tempelkan (*paste*) **Access Token** yang telah Anda salin sebelumnya.

---

## 5. Pemantauan Build & Akses Aplikasi

1. Setelah proses push selesai, buka halaman Space Anda di browser.
2. Anda akan melihat status Space berubah menjadi **Building** selagi Hugging Face mengunduh base image, mengompilasi aset Vite, menjalankan migrasi database, dan mengisi data seeder.
3. Setelah selesai, status akan berubah menjadi **Running**.
4. Klik tab **App** untuk melihat aplikasi berjalan.
5. **Mendapatkan URL Langsung (Tanpa Iframe)**:
   * Klik ikon titik tiga di kanan atas Space -> pilih **Embed this Space**.
   * Salin link dari kolom **Direct URL** (formatnya: `https://username-space-name.hf.space`). URL ini dapat dibagikan dan diakses langsung tanpa frame Hugging Face.

---

## 6. Menjaga Server Tetap Aktif (Uptime Monitoring)

Secara default, Hugging Face Space versi gratis akan memasuki mode tidur (*sleep*) jika tidak menerima lalu lintas pengunjung selama beberapa jam. Untuk menjaga agar demo aplikasi tetap aktif 24/7:

1. Buat akun gratis di **[UptimeRobot](https://uptimerobot.com/)**.
2. Tambahkan monitor baru dengan tipe **HTTP(S)**.
3. Masukkan **Direct URL** Space Anda (dari langkah 5).
4. Atur interval pemantauan ke **5 menit** sekali.
5. UptimeRobot akan mengirimkan request HTTP secara berkala yang akan mencegah kontainer Hugging Face memasuki mode tidur.
