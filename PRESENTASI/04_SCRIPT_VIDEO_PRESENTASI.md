# Script Video Presentasi: Bagaimana Saya Membangun Sistem Manajemen Klinik Terintegrasi (ERM, ERP, & POS)

**Judul Video:** Saya Membangun Sistem Manajemen Klinik Terintegrasi (ERM, ERP, & POS)  
**Estimasi Durasi:** 3 - 4 Menit  
**Target Audiens:** Sesama Developer, Calon Klien Klinik, dan Pengikut di Media Sosial Pribadi  
**Gaya Penyampaian:** Kasual, Edukatif, dan Hands-On (William Jakfar Style - Format "How I Built X")  

---

## 🎬 Kerangka Struktur Script

| Waktu | Bagian | Fokus Utama |
| :--- | :--- | :--- |
| **00:00 - 00:25** | **1. Hook & Pembuktian** | Menunjukkan dashboard sistem yang sudah berjalan riil di operasional klien untuk tugas akhir. |
| **00:25 - 00:50** | **2. Tantangan & Ide** | Menjelaskan ribetnya manajemen klinik tradisional yang aplikasinya kepisah-pisah. |
| **00:50 - 01:25** | **3. Langkah 1: Sistem Medis (ERM)** | Membangun antrean real-time dan pengisian form pemeriksaan SOAP Dokter. |
| **01:25 - 02:00** | **4. Langkah 2: Logistik & Stok Obat** | Integrasi resep obat otomatis, kartu stok, dan peringatan kedaluwarsa (ED). |
| **02:00 - 02:30** | **5. Langkah 3: POS Kasir Terpadu** | Menggabungkan tagihan tindakan + resep obat, cetak nota, dan etiket aturan pakai. |
| **02:30 - 03:00** | **6. Langkah 4: Otak Keuangan (ERP)** | Membuat laporan laba rugi otomatis, perhitungan payroll staf, dan status aset. |
| **03:00 - 03:30** | **7. Tech Stack & Performa SPA** | Membedah framework Laravel 12 + Livewire 4 agar transisi layar super cepat. |
| **03:30 - 03:50** | **8. Kesimpulan & Call to Action** | Manfaat nyata bagi operasional klinik dan informasi kontak demo/inquiry. |

---

## 📝 Naskah Detail & Skenario Visual

### Bagian 1: Hook & Pembuktian (00:00 - 00:25)
* **Visual:**
  * Pembuka video memperlihatkan cuplikan cepat layar Dashboard Admin yang dipenuhi grafik keuangan interaktif (Chart.js) dan daftar pasien.
  * Transisi cepat memperlihatkan printer kasir mencetak nota fisik atau etiket obat.
* **Audio (Voiceover):**
  * *"Project yang kamu lihat di layar ini awalnya saya kembangkan untuk penelitian tugas akhir kuliah saya. Tapi menariknya, sekarang sistem manajemen klinik terintegrasi ini sudah resmi diimplementasikan langsung untuk operasional klien nyata. Jadi di video kali ini, saya mau share gimana alur kerja dan modul-modul penting yang ada di sistem ini."*
* **Catatan Teknis:**
  * Musik latar (*Background Music*): Musik santai, lo-fi beats, atau musik bernada tekno-minimalis.
  * Tampilkan teks besar di layar: **Tugas Akhir Berujung Klien Nyata**.

---

### Bagian 2: Tantangan & Ide (00:25 - 00:50)
* **Visual:**
  * Presenter (jika merekam wajah) berbicara ke kamera, atau visualisasi diagram alur klinik konvensional yang berantakan (rekam medis kertas bertumpuk, kertas resep hilang).
* **Audio (Voiceover):**
  * *"Kenapa sistem ini penting? Karena kalau kamu perhatiin, operasional klinik itu sering kali ribet karena aplikasinya terpisah-pisah. Aplikasi rekam medis beda, logistik beda, kasir beda, bahkan sampai pembukuan keuangannya pun beda. Karena databasenya terpisah dan tidak sinkron, akhirnya klinik butuh banyak karyawan cuma buat nge-run masing-masing aplikasi tersebut secara manual. Makanya, saya yang saat itu terlibat dalam penyusunan sistem operasional, punya inisiatif buat menyatukan ERM, ERP, dan POS kasir langsung dalam satu aplikasi web terpadu dengan database yang tersinkronisasi."*
* **Catatan Teknis:**
  * Gunakan transisi cepat bergaya *glitch* atau *slide* saat menyebutkan masalah.

---

### Bagian 3: Langkah 1: Sistem Medis / ERM (00:50 - 01:25)
* **Visual:**
  * **Chapter Card (1.5 detik):** Slide hitam gelap minimalis dengan teks Serif elegan *"BAB 01: SISTEM MEDIS REAL-TIME (ERM)"* beranimasi zoom-in lambat. Diiringi bunyi dentingan piano tunggal (*single piano note echo*).
  * Transisi potong cepat ke rekaman layar modul pendaftaran pasien baru.
  * Menunjukkan layar registrasi pasien baru di modul pendaftaran.
  * Pindah layar ke layar Dokter secara real-time. Tunjukkan bagaimana antrean bertambah tanpa perlu reload halaman.
  * Tampilkan proses pengetikan di form SOAP (Subjective, Objective, Assessment, Plan) dan penulisan resep obat.
* **Audio (Voiceover):**
  * *"Langkah pertama, saya bangun modul Electronic Medical Record atau ERM. Di sini, begitu staf pendaftaran memasukkan data pasien baru, daftar antreannya bakal langsung ter-update secara real-time di layar perawat maupun dokter tanpa perlu reload halaman browser. Dokter juga bisa dengan mudah mencatat rekam medis dengan format standar SOAP dan menginput resep obat langsung di halaman yang sama."*
* **Catatan Teknis:**
  * Lakukan *zoom-in* pada baris pengisian SOAP untuk menekankan kejelasan fiturnya.
  * Teks di layar: **Step 1: Real-Time ERM & SOAP**.

---

### Bagian 4: Langkah 2: Logistik & Stok Obat (01:25 - 02:00)
* **Visual:**
  * **Chapter Card (1.5 detik):** Slide hitam gelap minimalis dengan teks Serif elegan *"BAB 02: LOGISTIK & STOK OBAT"* beranimasi zoom-in lambat. Diiringi bunyi dentingan piano tunggal (*single piano note echo*).
  * Transisi potong cepat ke rekaman layar modul Farmasi. Memperlihatkan antrean resep masuk.
  * Mengarahkan kursor ke menu Kartu Stok obat yang berkurang otomatis dan visualisasi peringatan obat kedaluwarsa dengan warna merah/kuning.
* **Audio (Voiceover):**
  * *"Langkah kedua adalah menghubungkan resep dokter ke modul logistik. Begitu dokter selesai nginput resep, bagian penyiapan obat akan langsung menerima notifikasi di meja racik secara instan. Hebatnya lagi, sistem logistik ini bakal otomatis memotong Kartu Stok obat dan melacak bahan medis habis pakai yang terpakai. Saya juga nambahin alert otomatis untuk ngedeteksi obat yang mendekati masa expired biar aman."*
* **Catatan Teknis:**
  * Efek suara notifikasi bip yang halus saat resep obat masuk ke bagian penyiapan obat.
  * Teks di layar: **Step 2: Smart Inventory & Expired Date Alert**.

---

### Bagian 5: Langkah 3: POS Kasir Terpadu (02:00 - 02:30)
* **Visual:**
  * **Chapter Card (1.5 detik):** Slide hitam gelap minimalis dengan teks Serif elegan *"BAB 03: POS KASIR TERPADU"* beranimasi zoom-in lambat. Diiringi bunyi dentingan piano tunggal (*single piano note echo*).
  * Transisi potong cepat ke rekaman layar modul Kasir. Menampilkan daftar tagihan pasien yang sudah terisi otomatis dengan biaya jasa dokter, tindakan, obat, dan BHP.
  * Mengklik tombol bayar dan menampilkan struk cetakan nota yang rapi.
* **Audio (Voiceover):**
  * *"Langkah ketiga, kita buat kasirnya. Untuk meminimalkan kesalahan hitung, modul POS kasir di aplikasi ini otomatis menggabungkan seluruh biaya tindakan dokter, obat dari bagian penyiapan, sampai bahan habis pakai menjadi satu tagihan utuh. Kasir tinggal klik satu kali saja untuk mencetak nota pembayaran sekaligus label etiket aturan pakai obatnya secara praktis."*
* **Catatan Teknis:**
  * Gunakan efek suara ketikan keyboard kasir.
  * Teks di layar: **Step 3: Integrated Cashier POS & Label Printing**.

---

### Bagian 6: Langkah 4: Otak Keuangan / ERP (02:30 - 03:00)
* **Visual:**
  * **Chapter Card (1.5 detik):** Slide hitam gelap minimalis dengan teks Serif elegan *"BAB 04: OTAK KEUANGAN & ERP"* beranimasi zoom-in lambat. Diiringi bunyi dentingan piano tunggal (*single piano note echo*).
  * Transisi potong cepat ke rekaman layar modul Keuangan. Menampilkan halaman Laporan Keuangan Laba Rugi dengan visual grafik interaktif (Chart.js) yang berubah-ubah sesuai tanggal.
  * Beralih ke halaman Payroll Staf dan halaman inventarisasi kondisi kelayakan Aset.
* **Audio (Voiceover):**
  * *"Langkah keempat, ini adalah otak dari seluruh sistem, yaitu modul ERP. Di sini, semua transaksi penjualan dari kasir tadi langsung diolah secara otomatis menjadi Laporan Laba Rugi bulanan—lengkap dengan perhitungan modal obat, biaya operasional klinik, hingga payroll penggajian staf berdasarkan kehadiran dan bagi hasil jasa medis."*
* **Catatan Teknis:**
  * Lakukan gerakan kamera *pan-up* dinamis pada grafik Chart.js laba rugi.
  * Teks di layar: **Step 4: Business Intelligence & Payroll Automation**.

---

### Bagian 7: Tech Stack & Performa SPA (03:00 - 03:30)
* **Visual:**
  * **Chapter Card (1.5 detik):** Slide hitam gelap minimalis dengan teks Serif elegan *"TEKNOLOGI DI BALIK LAYAR"* beranimasi zoom-in lambat. Diiringi bunyi dentingan piano tunggal (*single piano note echo*).
  * Transisi potong cepat ke rekaman layar cuplikan struktur folder project di VS Code, serta logo teknologi: Laravel 12, Livewire 4, dan Tailwind CSS 4.
  * Memperlihatkan betapa cepatnya perpindahan tab menu aplikasi tanpa ada loading screen browser (khas SPA).
* **Audio (Voiceover):**
  * *"Nah, kamu mungkin penasaran kenapa performa perpindahan halamannya kenceng banget tanpa ada loading browser berulang-ulang. Di balik layar, saya menggunakan Laravel 12 dikombinasikan dengan Livewire 4 dan Alpine.js. Arsitektur Hybrid SPA ini bikin aplikasi web terasa sangat ringan dan responsif layaknya aplikasi desktop. Dan enaknya lagi, instalasi aplikasi ini hanya perlu dilakukan di satu komputer server saja. Ini jauh lebih praktis dibanding aplikasi desktop tradisional yang mengharuskan kita meng-install aplikasinya satu per satu di setiap komputer yang mau mengakses. Plus, karena berbasis web, aplikasi ini bisa diakses dengan lancar dari device apa saja—mulai dari laptop, tablet, sampai smartphone kamu."*
* **Catatan Teknis:**
  * Teks di layar: **The Tech Stack: Laravel 12 + Livewire 4 SPA**.

---

### Bagian 8: Kesimpulan & Call to Action (03:30 - 03:50)
* **Visual:**
  * Layar dashboard utama dengan teks ucapan terima kasih dan kontak email atau akun media sosial.
  * Teks atau link untuk request demo privat yang muncul secara estetik.
* **Audio (Voiceover):**
  * *"Pembangunan project ini telah membuktikan kalau integrasi sistem yang matang bisa memangkas waktu kerja, mencegah kebocoran stok, dan membuat pencatatan keuangan jadi jauh lebih rapi. Kalau kamu pemilik klinik, dokter yang mau buka praktik sendiri, pengelola rumah sakit, atau teman-teman yang tertarik untuk belajar atau berdiskusi soal sistem ERM dan ERP kesehatan seperti ini, langsung saja hubungi saya lewat kontak yang ada di deskripsi di bawah. Terima kasih!"*
* **Catatan Teknis:**
  * Musik latar perlahan memudar (*fade out*).
  * Teks di layar: **Private Demo & Inquiry - Contact in Description**.

---

## 💡 Tips Perekaman Video Tambahan

1. **Persiapan Data Demonstrasi:** Jalankan seeder (`php artisan migrate --seed`) agar data rekam medis, antrean, inventaris obat, dan laporan keuangan terisi dengan data sampel yang terlihat realistis selama perekaman.
2. **Kualitas Audio:** Rekam suara di ruangan yang tenang menggunakan mikrofon eksternal, lalu beri sedikit filter kompresi agar terdengar hangat dan profesional khas narasi YouTube.
3. **Perekaman Layar:** Gunakan OBS Studio pada resolusi 1080p 60fps dengan mode layar penuh (F11) agar detail antarmuka aplikasi terlihat tajam dan tidak pecah saat ditonton di smartphone.
