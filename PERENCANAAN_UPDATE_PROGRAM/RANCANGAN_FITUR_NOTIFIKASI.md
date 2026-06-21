# Rancangan Sistem Notifikasi Terintegrasi (Real-Time Notification System)

Dokumen ini berisi rancangan teknis dan rencana implementasi fitur notifikasi di dalam aplikasi *Clinic Management System*. Fitur ini dirancang khusus untuk berjalan optimal pada **server lokal (intranet/offline)** tanpa bergantung pada koneksi internet (seperti Pusher/WebSockets eksternal).

---

## 1. Analisis Kebutuhan Fitur Notifikasi

Ada 3 jenis notifikasi utama yang dibutuhkan untuk mengintegrasikan alur kerja klinik:

| Jenis Notifikasi | Sumber Pemicu (Trigger) | Penerima (Target Role) | Media Output |
| :--- | :--- | :--- | :--- |
| **Resep Obat Masuk** | Dokter selesai menyimpan resep pemeriksaan pasien (`VisitPrescription`). | Bagian Penyiapan Obat / Farmasi | Suara Bel (*Chime Audio*), Toast Popup, & Badge Angka |
| **Stok Obat/BHP Menipis** | Pengurangan stok di kasir/tindakan yang menyebabkan `current_stock` < `min_stock` di tabel `items`. | Manajer Logistik / Admin | Visual Warning di Dashboard & Menu Logistik |
| **Obat Mendekati Kedaluwarsa** | Pengecekan harian otomatis (cron/scheduler) pada `expired_date` di tabel `inventory_restocks` (misal: sisa ≤ 60 hari). | Manajer Logistik / Admin | Visual Warning di Dashboard & Alert Menu Logistik |

---

## 2. Arsitektur Teknis Sistem Notifikasi

Untuk menjaga sistem tetap ringan, mandiri (dapat berjalan offline), dan mudah dipelihara:

1.  **Penyimpanan Data (Backend):** Menggunakan fitur bawaan **Laravel Database Notifications** yang menyimpan notifikasi di tabel `notifications` dalam bentuk JSON.
2.  **Sinkronisasi Real-Time (Frontend):** Menggunakan **Livewire Polling (`wire:poll`)** pada komponen bilah navigasi (*navbar*) yang mengecek database setiap 15–30 detik.
3.  **Umpan Balik Audio (Audio Feedback):** Menggunakan **HTML5 Audio API** di sisi klien (Alpine.js) untuk memutar file audio pendek ketika terdeteksi resep obat baru masuk.

---

## 3. Skema Basis Data

### A. Tabel `notifications` (Standar Laravel)
Tabel ini akan dibuat menggunakan perintah bawaan Laravel:
```bash
php artisan notifications:table
php artisan migrate
```

Struktur tabel `notifications`:
*   `id` (UUID, Primary Key)
*   `type` (String) - Jenis kelas notifikasi (misal: `App\Notifications\PrescriptionReceived`)
*   `notifiable_type` & `notifiable_id` - Polymorphic relation ke model `User` yang menerima notifikasi.
*   `data` (JSON) - Menyimpan payload custom (seperti `visit_id`, `patient_name`, `item_name`, `message`).
*   `read_at` (Timestamp, Nullable) - Waktu notifikasi dibaca oleh pengguna.
*   `created_at` & `updated_at` (Timestamp)

---

## 4. Alur Kerja Kode Program & Contoh Implementasi

### A. Kelas Notifikasi (Laravel Notification)
Contoh kelas `PrescriptionReceived` yang disimpan di `app/Notifications/PrescriptionReceived.php`:

```php
namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;

class PrescriptionReceived extends Notification
{
    use Queueable;

    protected $visit;

    public function __construct($visit)
    {
        $this->visit = $visit;
    }

    public function via($notifiable)
    {
        return ['database'];
    }

    public function toArray($notifiable)
    {
        return [
            'visit_id' => $this->visit->id,
            'patient_name' => $this->visit->patient->name,
            'doctor_name' => $this->visit->creator->name,
            'message' => 'Resep baru untuk pasien ' . $this->visit->patient->name . ' telah dikirim.',
            'type' => 'prescription_received'
        ];
    }
}
```

### B. Memicu Notifikasi (Triggering)
Ketika dokter menyimpan resep medis di Livewire component:
```php
// app/Livewire/Medical/PatientPrescription.php
public function savePrescription()
{
    // ... Logic simpan resep ke VisitPrescription ...

    // Kirim notifikasi ke semua staf dengan role 'farmasi'
    $pharmacyStaffs = User::where('role', 'farmasi')->get();
    foreach ($pharmacyStaffs as $staff) {
        $staff->notify(new PrescriptionReceived($this->visit));
    }
}
```

### C. Komponen Real-Time Receiver (Livewire Component)
Komponen `NotificationBell` yang diletakkan di Navbar (`app/Livewire/Common/NotificationBell.php`):

```php
namespace App\Livewire\Common;

use Livewire\Component;
use Auth;

class NotificationBell extends Component
{
    public $unreadCount = 0;
    public $notifications = [];

    public function getListeners()
    {
        return [
            'echo:notifications,NotificationSent' => 'fetchNotifications', // Opsional jika menggunakan WebSockets nanti
        ];
    }

    public function mount()
    {
        $this->fetchNotifications();
    }

    // Dipanggil secara otomatis oleh Livewire Polling
    public function fetchNotifications()
    {
        if (Auth::check()) {
            $user = Auth::user();
            $this->notifications = $user->unreadNotifications()->take(5)->get()->toArray();
            
            $newCount = $user->unreadNotifications()->count();
            // Jika ada notifikasi baru bertambah, trigger event suara di browser
            if ($newCount > $this->unreadCount) {
                $this->dispatch('play-chime');
            }
            $this->unreadCount = $newCount;
        }
    }

    public function markAsRead($id)
    {
        $notification = Auth::user()->notifications()->find($id);
        if ($notification) {
            $notification->markAsRead();
        }
        $this->fetchNotifications();
    }

    public function render()
    {
        return view('livewire.common.notification-bell');
    }
}
```

### D. Tampilan HTML & Integrasi Audio (Blade + Alpine.js)
Letakkan baris berikut pada file template komponen `notification-bell.blade.php`:

```html
<div x-data="{ open: false }" 
     wire:poll.15s="fetchNotifications" 
     class="relative">
     
    <!-- Bell Icon & Badge -->
    <button @click="open = !open" class="relative p-2 text-gray-600 hover:text-gray-900 focus:outline-none">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9"></path>
        </svg>
        
        <!-- Red Badge Counter -->
        @if($unreadCount > 0)
            <span class="absolute top-0 right-0 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-white transform translate-x-1/2 -translate-y-1/2 bg-red-600 rounded-full">
                {{ $unreadCount }}
            </span>
        @endif
    </button>

    <!-- Dropdown List Notifikasi -->
    <div x-show="open" @click.away="open = false" class="absolute right-0 mt-2 w-80 bg-white rounded-md shadow-lg overflow-hidden z-50 border border-gray-200">
        <div class="py-2">
            @forelse($notifications as $n)
                <div class="px-4 py-3 hover:bg-gray-50 border-b border-gray-100 flex justify-between items-center">
                    <div>
                        <p class="text-sm text-gray-800">{{ $n['data']['message'] }}</p>
                        <span class="text-xs text-gray-400">{{ \Carbon\Carbon::parse($n['created_at'])->diffForHumans() }}</span>
                    </div>
                    <button wire:click="markAsRead('{{ $n['id'] }}')" class="text-xs text-blue-500 hover:underline">
                        Tandai Baca
                    </button>
                </div>
            @empty
                <div class="px-4 py-6 text-center text-gray-500 text-sm">
                    Tidak ada notifikasi baru.
                </div>
            @endforelse
        </div>
    </div>

    <!-- Audio Player (Triggered by Livewire Event) -->
    <audio id="notif-chime" src="{{ asset('audio/notification_chime.mp3') }}" preload="auto"></audio>

    <script>
        document.addEventListener('livewire:initialized', () => {
            @this.on('play-chime', (event) => {
                const audio = document.getElementById('notif-chime');
                if (audio) {
                    audio.play().catch(error => {
                        console.log('Autoplay dicegah oleh browser, membutuhkan interaksi pengguna terlebih dahulu.');
                    });
                }
            });
        });
    </script>
</div>
```

---

## 5. Rencana Tahapan Pengujian Fitur
1.  **Pengujian Database:** Memastikan data resep masuk ter-record dengan benar di database ketika disubmit dokter.
2.  **Pengujian Kecepatan Polling:** Memastikan interval 15 detik pada Livewire poll tidak membebani database local server SQLite/MySQL.
3.  **Pengujian Audio:** Memastikan suara berdering (*play*) saat tab browser bagian penyiapan obat aktif di latar belakang (background tab).
