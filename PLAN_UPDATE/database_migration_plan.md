# Rencana Migrasi Database Lintas-Platform (MySQL ke PostgreSQL / Supabase)

Dokumen ini menjelaskan rencana langkah-demi-langkah untuk mengubah query mentah (raw query) yang spesifik untuk MySQL menjadi query standar SQL (ANSI-compliant) agar aplikasi dapat dihosting di Render dengan database PostgreSQL di Supabase tanpa memutus kecocokan dengan MySQL.

---

## 1. Analisis Masalah
Saat ini, aplikasi menggunakan fungsi internal MySQL berikut:
*   `DATE(expression)`: Digunakan untuk memotong timestamp menjadi tanggal (`YYYY-MM-DD`). Di PostgreSQL, fungsi ini tidak menerima tipe timestamp secara langsung.
*   `YEAR(expression)`: Digunakan untuk mengambil angka tahun. PostgreSQL tidak memiliki fungsi `YEAR()`.
*   `MONTH(expression)`: Digunakan untuk mengambil angka bulan. PostgreSQL tidak memiliki fungsi `MONTH()`.

---

## 2. Solusi Lintas-Database (Database-Agnostic)
Kita akan mengganti fungsi-fungsi tersebut dengan alternatif SQL standar yang didukung baik oleh MySQL maupun PostgreSQL:
1.  **Pengganti `DATE(col)`**: `CAST(col AS date)`
2.  **Pengganti `YEAR(col)`**: `EXTRACT(YEAR FROM col)`
3.  **Pengganti `MONTH(col)`**: `EXTRACT(MONTH FROM col)`

---

## 3. Daftar File yang Akan Diubah

### A. Model: `Visit.php`
*   **File**: `app/Models/Visit.php` (Line 57)
*   **Sebelumnya**:
    ```php
    $dailyVisitIds = self::whereRaw('DATE(COALESCE(scheduled_at, created_at)) = ?', [$actualDate])
    ```
*   **Sesudah**:
    ```php
    $dailyVisitIds = self::whereRaw('CAST(COALESCE(scheduled_at, created_at) AS date) = ?', [$actualDate])
    ```

---

### B. Livewire Component: `Queue.php`
*   **File**: `app/Livewire/Medical/Queue.php` (Line 47)
*   **Sebelumnya**:
    ```php
    ->whereRaw('DATE(COALESCE(scheduled_at, created_at)) <= ?', [now()->format('Y-m-d')])
    ```
*   **Sesudah**:
    ```php
    ->whereRaw('CAST(COALESCE(scheduled_at, created_at) AS date) <= ?', [now()->format('Y-m-d')])
    ```

---

### C. Livewire Component: `Sidebar.php`
*   **File**: `app/Livewire/Layouts/Sidebar.php` (Line 24, 33, 42, 51, 60)
*   **Sebelumnya**:
    ```php
    DATE(COALESCE(scheduled_at, created_at))
    ```
*   **Sesudah**:
    ```php
    CAST(COALESCE(scheduled_at, created_at) AS date)
    ```

---

### D. Livewire Component: `PenugasanShift.php`
*   **File**: `app/Livewire/Hrd/PenugasanShift.php` (Line 378)
*   **Sebelumnya**:
    ```php
    DATE(COALESCE(scheduled_at, visits.created_at))
    ```
*   **Sesudah**:
    ```php
    CAST(COALESCE(scheduled_at, visits.created_at) AS date)
    ```

---

### E. Livewire Component: `Pendaftaran.php`
*   **File**: `app/Livewire/FrontOffice/Pendaftaran.php` (Line 203)
*   **Sebelumnya**:
    ```php
    DATE(COALESCE(scheduled_at, created_at))
    ```
*   **Sesudah**:
    ```php
    CAST(COALESCE(scheduled_at, created_at) AS date)
    ```

---

### F. Livewire Component: `Pembayaran.php`
*   **File**: `app/Livewire/Cashier/Pembayaran.php` (Line 63)
*   **Sebelumnya**:
    ```php
    DATE(COALESCE(scheduled_at, created_at))
    ```
*   **Sesudah**:
    ```php
    CAST(COALESCE(scheduled_at, created_at) AS date)
    ```

---

### G. Livewire Component: `Dashboard.php`
*   **File**: `app/Livewire/Dashboard.php` (Line 44, 62, 68, 72, 119, 240, 257, 291, 320, 341)
    *   Ganti semua panggilan `DATE(...)` dengan `CAST(... AS date)`.
*   **File**: `app/Livewire/Dashboard.php` (Line 159, 170, 176, 219)
    *   Ganti semua panggilan `YEAR(...)` dengan `EXTRACT(YEAR FROM ...)`.
*   **File**: `app/Livewire/Dashboard.php` (Line 160, 171, 177)
    *   Ganti semua panggilan `MONTH(...)` dengan `EXTRACT(MONTH FROM ...)`.

---

## 4. Konfigurasi Supabase / PostgreSQL di `.env`
Saat melakukan deploy ke Render dengan database Supabase, perbarui konfigurasi file `.env` sebagai berikut:

```env
DB_CONNECTION=pgsql
DB_HOST=your-supabase-db-host.supabase.co
DB_PORT=5432
DB_DATABASE=postgres
DB_USERNAME=postgres
DB_PASSWORD=your-supabase-password
```

*(Catatan: Pastikan ekstensi `pdo_pgsql` aktif di server hosting Render Anda).*

---

## 5. Rencana Pengujian (Verification Plan)
1.  **Pengujian MySQL (Lokal)**: Jalankan aplikasi di environment Laragon dengan MySQL aktif. Pastikan dashboard statistik, daftar antrean, sidebar counter, dan nomor antrean pasien berjalan lancar tanpa error SQL.
2.  **Pengujian PostgreSQL**:
    *   Siapkan database PostgreSQL lokal atau uji coba di Supabase.
    *   Ubah `.env` menggunakan PostgreSQL.
    *   Jalankan migrasi: `php artisan migrate:fresh --seed`.
    *   Buka aplikasi dan verifikasi semua fungsi pembacaan data berjalan normal tanpa error database.
