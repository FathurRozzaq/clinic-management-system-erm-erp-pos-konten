# Rancangan Dukungan Multi-Role & Praktik Dokter Mandiri (Klinik Kecil)

Dokumen ini berisi analisis dan rancangan solusi untuk mengakomodasi operasional klinik kecil atau praktik dokter mandiri. Pada skenario ini, staf sering kali merangkap tugas (misalnya, seorang `admin` atau `resepsionis` juga bertindak sebagai `kasir` dan pembantu `farmasi` pada hari yang sama).

---

## 1. Masalah Desain Notifikasi Kaku (Single-Role Bias)

Jika sistem notifikasi dirancang dengan asumsi satu role hanya dikerjakan oleh satu akun khusus (misal: notifikasi kasir hanya dikirim ke user dengan `role = 'kasir'`), maka akan timbul kendala berikut pada praktik mandiri:
1.  **Notifikasi Tidak Sampai:** Admin yang sedang menjaga meja kasir tidak akan mendapat notifikasi suara/visual saat obat selesai disiapkan, karena sistem hanya mengirimkannya ke role `kasir`.
2.  **Banjir Notifikasi Mandiri (*Self-Notification Loop*):** Jika satu orang mendaftarkan pasien, lalu dia sendiri yang menerima notifikasinya di layar yang sama, ini akan menimbulkan gangguan suara berdering yang tidak perlu.

---

## 2. Solusi A: Penerima Berbasis Kemampuan Akses (Permission-Based Routing)

Alih-alih memfilter penerima menggunakan kolom `role` secara kaku di database, sistem notifikasi akan dikirimkan berdasarkan **izin akses modul** (*capability/permissions*). 

Kita mendefinisikan grup penerima notifikasi menggunakan logika otorisasi yang fleksibel:

### Tabel Pemetaan Hak Terima Notifikasi

| Jenis Notifikasi | Izin Modul yang Dibutuhkan | Penerima Riil (Roles) | Keterangan |
| :--- | :--- | :--- | :--- |
| **`PatientReadyForTtv`** | Mengakses Antrean Perawat | `perawat`, `admin` | Jika perawat tidak ada, admin dapat membantu TTV. |
| **`PatientReadyForExamination`** | Mengakses SOAP Dokter | `dokter` (spesifik `doctor_id`) | Bersifat klinis personal, tidak didelegasikan ke admin umum. |
| **`PrescriptionReceived`** | Mengakses Meja Racik Farmasi | `apoteker`, `admin` | Admin dapat membantu menyiapkan obat jadi di klinik kecil. |
| **`PatientReadyForPayment`** | Mengakses Kasir POS | `kasir`, `resepsionis`, `admin` | Meja depan merangkap kasir dan pendaftaran. |

### Contoh Implementasi Query Penerima di Laravel:
```php
// Pemicu Notifikasi Pembayaran (Kasir)
$receivers = User::where('status', 'aktif')
    ->whereIn('role', ['kasir', 'resepsionis', 'admin'])
    ->get();

foreach ($receivers as $user) {
    // Mencegah mengirim notifikasi ke diri sendiri yang memicu transisi status
    if ($user->id !== Auth::id()) {
        $user->notify(new PatientReadyForPayment($visit));
    }
}
```

---

## 3. Solusi B: Mode Praktik Mandiri (Small Practice Mode Toggle)

Untuk memberikan fleksibilitas penuh antara klinik besar dan praktik dokter mandiri, kita menambahkan pengaturan global di konfigurasi sistem (`config/clinic.php` atau tabel pengaturan database):

```php
return [
    // Jika TRUE, notifikasi non-medis dilebur ke satu meja depan terpadu
    'small_practice_mode' => env('CLINIC_SMALL_PRACTICE_MODE', true),
];
```

### Konsekuensi Logika Sistem jika `small_practice_mode` Aktif:

1.  **Peleburan Saluran Notifikasi:**
    Semua notifikasi untuk pendaftaran (`PatientReadyForTtv`), farmasi (`PrescriptionReceived`), dan pembayaran (`PatientReadyForPayment`) akan masuk ke bel navigasi yang sama pada akun `admin` dan `resepsionis`.
2.  **Penyederhanaan Tampilan (Single Dashboard Command):**
    Satu komputer di meja depan dapat membuka tab kasir dan pendaftaran sekaligus tanpa perlu bolak-balik mengganti akun, dan semua notifikasi masuk secara sinkron.
3.  **Dukungan Multi-Tab Audio:**
    Jika admin membuka tab pendaftaran dan tab kasir di browser yang sama, mekanisme Alpine.js akan memastikan suara notifikasi hanya berdering satu kali saja menggunakan koordinasi `localStorage` atau pengecekan fokus tab.

---

## 4. Solusi C: Eliminasi Notifikasi Mandiri (Self-Notification Filter)

Sistem harus cerdas untuk mendeteksi siapa yang melakukan aksi. Jika pengguna A (misalnya akun `admin`) sedang membuka halaman Meja Racik Farmasi dan mengklik "Serahkan Obat", lalu status berubah ke `kasir`:
*   Sistem **tidak akan** membunyikan suara chime kasir di komputer pengguna A, karena pengguna A adalah orang yang memicu perubahan tersebut.
*   Namun, jika ada staf kasir lain (pengguna B) di komputer lain, bel di komputer pengguna B tetap akan berdering.

### Implementasi Filter di Livewire Component (`NotificationBell.php`):
```php
public function fetchNotifications()
{
    if (Auth::check()) {
        $user = Auth::user();
        
        // Ambil notifikasi belum dibaca
        $query = $user->unreadNotifications();
        
        // Filter: Jangan tampilkan/bunyikan jika dipicu oleh diri sendiri
        $this->notifications = $query->take(5)->get()
            ->filter(fn($n) => ($n->data['actor_id'] ?? null) !== $user->id)
            ->toArray();
            
        $newCount = count($this->notifications);
        if ($newCount > $this->unreadCount) {
            $this->dispatch('play-chime');
        }
        $this->unreadCount = $newCount;
    }
}
```
Untuk mendukung filter ini, setiap pembuatan notifikasi wajib menyertakan `'actor_id' => Auth::id()` di dalam payload data notifikasinya.
