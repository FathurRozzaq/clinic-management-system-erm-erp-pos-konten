# Rancangan Trigger, Penerima, dan Mekanisme Notifikasi

Dokumen ini mendetailkan setiap pemicu (*trigger*), siapa penerima yang ditargetkan, isi payload notifikasi, serta bagaimana notifikasi dikirimkan dan dibatasi secara spesifik (misalnya, hanya untuk dokter/perawat tertentu yang ditugaskan).

---

## 1. Matriks Notifikasi Berdasarkan Transisi Status

Berikut adalah rancangan detail seluruh jenis notifikasi transaksi di klinik:

### 1. Pasien Siap TTV (`PatientReadyForTtv`)
*   **Pemicu (Trigger):** Resepsionis menyimpan pendaftaran pasien baru atau check-in booking sehingga status kunjungan menjadi `siap ttv`.
*   **Penerima (Recipient):** 
    *   Jika `nurse_id` kunjungan diisi spesifik: Hanya perawat dengan ID tersebut.
    *   Jika `nurse_id` kosong: Seluruh user dengan role `perawat` (bersifat antrean terbuka).
*   **Payload JSON:**
    ```json
    {
      "visit_id": 125,
      "patient_name": "Budi Santoso",
      "no_rm": "RM-00104",
      "queue_number": "A-012",
      "message": "Pasien Budi Santoso siap untuk pemeriksaan tanda vital (TTV)."
    }
    ```
*   **Saluran Output:** Toast kuning di dashboard perawat, pemutaran suara chime bip lembut, penambahan angka badge merah di bel navigasi.

### 2. Pasien Siap Periksa Dokter (`PatientReadyForExamination`)
*   **Pemicu (Trigger):** Perawat selesai menginput vital signs (TTV) dan mengklik "Submit SOAP" yang mengarahkan status ke `siap periksa`.
*   **Penerima (Recipient):** Hanya Dokter (`doctor_id`) yang dipilih pada kunjungan pasien tersebut. Dokter lain tidak menerima notifikasi ini.
*   **Payload JSON:**
    ```json
    {
      "visit_id": 125,
      "patient_name": "Budi Santoso",
      "no_rm": "RM-00104",
      "queue_number": "A-012",
      "nurse_name": "Evelyn Putri",
      "message": "Pemeriksaan awal (TTV) selesai oleh Perawat Evelyn. Pasien siap diperiksa."
    }
    ```
*   **Saluran Output:** Toast biru di layar dokter yang dituju, suara chime notifikasi, badge navbar bertambah.

### 3. Resep Obat Masuk ke Farmasi (`PrescriptionReceived`)
*   **Pemicu (Trigger):** Dokter menyelesaikan SOAP, meresepkan obat, dan mengklik "Submit SOAP" dengan mengubah status ke `farmasi`.
*   **Penerima (Recipient):** Seluruh user dengan role `apoteker` (kolektif/grup).
*   **Payload JSON:**
    ```json
    {
      "visit_id": 125,
      "patient_name": "Budi Santoso",
      "no_rm": "RM-00104",
      "queue_number": "A-012",
      "doctor_name": "dr. Dian Pratama",
      "message": "Resep obat baru dikirim oleh dr. Dian Pratama untuk disiapkan."
    }
    ```
*   **Saluran Output:** Berdering kuat (bel peracikan) di layar Meja Racik, Toast hijau, popup daftar antrean langsung ter-refresh.

### 4. Obat Selesai & Siap Bayar (`PatientReadyForPayment`)
*   **Pemicu (Trigger):** 
    1.  Apoteker menekan tombol "Serahkan Obat / Handover" di Meja Racik yang mengubah status ke `kasir`.
    2.  Atau Dokter mengirim pasien langsung ke kasir karena pemeriksaan selesai tanpa obat (status langsung berubah ke `kasir`).
*   **Penerima (Recipient):** Seluruh user dengan role `kasir` (kolektif/grup).
*   **Payload JSON:**
    ```json
    {
      "visit_id": 125,
      "patient_name": "Budi Santoso",
      "no_rm": "RM-00104",
      "queue_number": "A-012",
      "source": "farmasi",
      "message": "Resep obat selesai dikemas. Silakan lakukan penyelesaian pembayaran."
    }
    ```
*   **Saluran Output:** Toast ungu di layar kasir, dering cash register lembut, daftar antrean Pembayaran ter-refresh secara real-time.

### 5. Resep Dikembalikan oleh Farmasi (`PrescriptionReturned`)
*   **Pemicu (Trigger):** Apoteker mengklik tombol "Kembalikan ke Dokter" di Meja Racik karena stok obat tidak cukup atau butuh konfirmasi resep (status kembali menjadi `periksa`).
*   **Penerima (Recipient):** Hanya Dokter (`doctor_id`) yang merawat pasien tersebut.
*   **Payload JSON:**
    ```json
    {
      "visit_id": 125,
      "patient_name": "Budi Santoso",
      "no_rm": "RM-00104",
      "pharmacist_name": "Apoteker Susi",
      "reason": "Obat Amoxicillin 500mg habis di depo farmasi.",
      "message": "PERINGATAN: Resep pasien Budi Santoso dikembalikan oleh Farmasi. Alasan: Obat habis."
    }
    ```
*   **Saluran Output:** Toast merah/orange di layar dokter yang merawat, suara alert bip berulang, rekam medis SOAP terbuka kembali untuk revisi.

### 6. Pembayaran Selesai (`VisitCompleted`)
*   **Pemicu (Trigger):** Kasir mengklik tombol "Proses Pembayaran" dan transaksi disimpan dengan status `selesai`.
*   **Penerima (Recipient):** Dokter (`doctor_id`) dan Perawat (`nurse_id`) yang menangani kunjungan tersebut.
*   **Payload JSON:**
    ```json
    {
      "visit_id": 125,
      "patient_name": "Budi Santoso",
      "invoice_number": "INV-260619-0125",
      "message": "Pasien Budi Santoso telah menyelesaikan pembayaran dan diperbolehkan pulang."
    }
    ```
*   **Saluran Output:** Status antrean di panel command center dokter/perawat otomatis berpindah ke kelompok selesai tanpa mengganggu pekerjaan aktif.

---

## 2. Mekanisme Pengiriman dan Pembatasan Penerima

Untuk memastikan notifikasi hanya diterima oleh pengguna yang relevan:

### A. Pengiriman Bertarget (Spesifik User)
Ketika memicu notifikasi yang bersifat personal (seperti ke dokter atau perawat tertentu), kode program akan menyasar ID pengguna tersebut secara langsung menggunakan metode bawaan Laravel:

```php
// Memicu notifikasi dari Perawat ke Dokter spesifik yang menangani
if ($visit->doctor_id) {
    $doctor = User::find($visit->doctor_id);
    if ($doctor) {
        $doctor->notify(new PatientReadyForExamination($visit));
    }
}
```

### B. Pengiriman Kolektif (Grup Role)
Ketika memicu notifikasi yang bersifat umum ke unit (seperti Farmasi atau Kasir), sistem akan mencari semua staf aktif dengan peran (*role*) tersebut:

```php
// Memicu notifikasi ke seluruh staf Kasir
$cashiers = User::where('role', 'kasir')->where('status', 'aktif')->get();
foreach ($cashiers as $cashier) {
    $cashier->notify(new PatientReadyForPayment($visit, 'farmasi'));
}
```

---

## 3. Penanganan Sisi Klien (Real-Time Pulling)

Karena sistem beroperasi secara offline pada server lokal, browser setiap staf akan melakukan pemeriksaan berkala (long polling) menggunakan Livewire:

1.  Setiap browser memanggil metode `fetchNotifications()` setiap 15 detik menggunakan `wire:poll.15s`.
2.  Sistem membaca database untuk mengambil daftar notifikasi baru milik user yang sedang login (`auth()->user()->unreadNotifications`).
3.  Jika jumlah notifikasi baru bertambah dari sebelumnya, Livewire menembakkan event browser (`play-chime`).
4.  JavaScript/Alpine.js menangkap event tersebut dan memutar audio sesuai jenis notifikasi yang masuk (misal: bel lonceng untuk farmasi, alarm bip untuk resep yang dikembalikan).
