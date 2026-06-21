# Panduan Penyiapan Aset Pendukung Video (Slide, Gambar, & Audio)

Dokumen ini berisi panduan teknis dan referensi bagi Anda untuk membuat, mencari, dan menyiapkan aset-aset visual serta audio sebelum masuk ke tahap perekaman (*production*) dan pengeditan video (*post-production*).

---

## 1. Panduan Pembuatan & Ekspor Slide (Chapter Cards)
Slide digunakan sebagai penanda transisi bab (*Chapter Cards*). Kita menggunakan **Marp CLI** untuk mengubah dokumen Markdown slide menjadi file gambar resolusi tinggi secara otomatis.

### Langkah Kerja:
1.  **Gunakan File Sumber:** File slide utama berada di [`05_SLIDE_PRESENTASI.md`](file:///e:/ProgramFiles/laragon/www/clinic-management-system-erm-erp-pos/PRESENTASI/05_SLIDE_PRESENTASI.md).
2.  **Ekspor Menjadi Gambar (PNG):**
    Jalankan perintah berikut di terminal komputer Anda (pastikan Node.js terinstal):
    ```bash
    # Mengonversi slide menjadi kumpulan gambar PNG di dalam folder 'slides'
    npx @marp-team/marp-cli 05_SLIDE_PRESENTASI.md --images png -o slides/
    ```
    Setiap halaman slide akan diekspor menjadi file `slides.001.png`, `slides.002.png`, dst. dengan rasio aspek **16:9 (Widescreen)** yang pas dengan resolusi video YouTube/LinkedIn.

---

## 2. Panduan Pencarian & Pembuatan Gambar (B-Roll & Background)

### A. Rekomendasi Situs Stok Foto Gratis (Bebas Hak Cipta)
Gunakan situs berikut untuk mencari gambar pendukung (B-roll) yang menggambarkan operasional klinik tradisional yang berantakan atau bertumpuk:
*   **Unsplash** ([unsplash.com](https://unsplash.com))
*   **Pexels** ([pexels.com](https://pexels.com))
*   **Pixabay** ([pixabay.com](https://pixabay.com))

### B. Kata Kunci Pencarian (*Search Keywords*):
*   *Untuk Bagian Masalah:* `"paper medical records folder"`, `"messy desk office papers"`, `"clinic patient queue"`, `"handwritten doctor prescription"`.
*   *Untuk Bagian Farmasi/Kasir:* `"pharmacy medicine bottles shelf"`, `"cashier print receipt"`, `"doctor writing prescription"`.

### C. Panduan Prompt Pembuatan Gambar Custom (AI Image Generation)
Jika ingin membuat gambar ilustrasi latar belakang atau *placeholder* menggunakan AI, gunakan pola prompt berikut:
> *"Minimalist dark tech background, deep dark blue and slate gray colors, clean abstract geometric lines, subtle neon cyan accents, 16:9 aspect ratio, cinematic lighting, professional studio presentation look --no realistic people"*

---

## 3. Panduan Pencarian Audio (BGM & Sound Effects)

### A. Rekomendasi Platform Musik & SFX Gratis (Bebas Hak Cipta)
*   **YouTube Audio Library** (Akses via YouTube Studio - Bagian Koleksi Audio).
*   **Pixabay Audio** ([pixabay.com/music](https://pixabay.com/music) & [pixabay.com/sound-effects](https://pixabay.com/sound-effects)).
*   **Freesound** ([freesound.org](https://freesound.org)) - Sangat bagus untuk mencari efek suara (*SFX*) spesifik.

### B. Pencarian Musik Latar (*Background Music / BGM*):
Cari jenis musik instrumental tanpa vokal yang mendukung konsentrasi dan bernada cerdas/modern:
*   *Kata Kunci:* `"tech ambient"`, `"minimalist corporate technology"`, `"chill lo-fi beats coding"`, `"business progress instrumental"`.
*   *Aturan Editing:* Pastikan volume musik berada di kisaran **-20dB hingga -25dB** saat voiceover berbicara.

### C. Pencarian Efek Suara (*Sound Effects / SFX*):
Gunakan kata kunci berikut di Pixabay Audio atau Freesound untuk transisi Chapter Card:
*   *Dentingan Piano (Chapter Card):* `"single piano note echo"`, `"cinematic piano hit"`.
*   *Ketukan Mekanis (Transisi cepat):* `"typewriter key click"`, `"mechanical keyboard switch click"`.
*   *Efek Transisi Visual:* `"deep sub-bass drop"`, `"clean camera shutter click"`, `"digital notification chime"`.
