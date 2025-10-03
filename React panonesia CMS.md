Spesifikasi Proyek: Panonesia CMS
1. Visi & Tujuan Utama
Panonesia CMS adalah sebuah aplikasi web modern yang berfungsi sebagai Content Management System (CMS) khusus untuk membuat dan mengelola pop-up interaktif dalam proyek tur virtual 360 (dibuat dengan 3DVista).

Tujuan utamanya adalah memberikan alur kerja yang sangat efisien, mulai dari mendesain pop-up, mengelola aset, hingga mengekspor sebuah proyek akhir yang terstruktur, ringan, dan siap untuk diintegrasikan ke dalam 3DVista. Aplikasi ini dirancang sebagai "studio desain" offline yang menghasilkan aset-aset siap pakai untuk lingkungan online.

2. Arsitektur & Alur Kerja Utama
Aplikasi ini akan memiliki antarmuka dashboard untuk mengelola keseluruhan proyek tur virtual.

A. Manajemen Proyek (Struktur)
Panorama: Pengguna dapat membuat dan menamai beberapa "Panorama" (misalnya, 1. Ruang Tamu, 2. Dapur). Setiap panorama akan memiliki ID numerik unik (1, 2, dst.) yang akan digunakan untuk penamaan file.

Pop-up: Di dalam setiap panorama, pengguna dapat membuat beberapa pop-up.

Penamaan Otomatis: Setiap pop-up akan secara otomatis diberi ID unik dengan format pop-<id_panorama>-<tipe>-<nomor_urut> (contoh: pop-1-text-001).

Tipe Pop-up: Saat membuat pop-up baru, pengguna akan memilih dari tipe yang telah disederhanakan:

Text: Untuk konten berbasis tulisan.

Img: Untuk menampilkan satu gambar.

Embed: Tipe serbaguna untuk menyematkan PDF, video, audio, atau halaman web lain.

Custom: Untuk kebebasan penuh menempelkan kode HTML dari awal.

B. Manajemen Aset (Media Library)
Aplikasi akan memiliki tab "Assets" terpisah di mana pengguna dapat meng-upload semua file media (gambar, PDF, dll.) yang akan digunakan di dalam pop-up.

Aset-aset ini akan dapat diakses dari dalam editor melalui tombol "Browse".

3. Fitur Rinci: Layar Editor
Layar "Edit" adalah inti dari aplikasi ini, tempat sebuah pop-up individual didesain dan dikustomisasi.

A. Fungsi Inti: Extract Content
Tombol "Extract Content": Ini adalah fitur utama di layar editor.

Cara Kerja: Saat ditekan, aplikasi akan memindai kode HTML yang ada di editor kode dan secara otomatis melakukan hal berikut:

Mendeteksi & Membuat Form: Menemukan semua konten yang bisa diedit (teks, src gambar, href link, action form) dan secara otomatis membuatkan form isian untuk masing-masing di panel editor kanan.

Mendeteksi Tema Warna: Secara cerdas memindai kelas-kelas warna Tailwind CSS (misalnya bg-yellow-400, text-gray-900) dan secara otomatis mengisi panel "Color Theme" dengan warna-warna yang terdeteksi dari komponen tersebut.

B. Fitur Editing di Panel Kanan
Color Theme:

Panel ini akan menampilkan warna-warna kunci yang terdeteksi (Primary, Secondary, Text, Background).

Pengguna dapat mengubah warna-warna ini menggunakan color picker, dan perubahan akan diterapkan secara real-time ke seluruh pratinjau.

Live Form (Konten):

Teks: Setiap isian teks akan memiliki ikon pensil (✏️) untuk membuka pop-up Toolbar Gaya Teks.

Toolbar Gaya Teks: Berisi kontrol untuk:

Bold & Italic (dengan indikator aktif/non-aktif).

Perataan Teks (kiri, tengah, kanan).

Pemilihan Font dari daftar Google Fonts.

Warna Teks Individual.

Gambar & Embed: Isian untuk URL gambar atau embed akan memiliki tombol "Browse..." untuk membuka pop-up Media Library dan memilih aset secara langsung.

Link & Form Action: Isian untuk href dan action akan diberi label yang jelas sesuai konteksnya.

4. Fungsi Ekspor Proyek
Ini adalah hasil akhir dari aplikasi.

Tombol "Export Final Project (ZIP)": Tombol ini akan memicu proses finalisasi.

Proses di Latar Belakang (menggunakan API Route di Next.js):

Mengumpulkan semua file HTML dari semua pop-up di semua panorama.

Menjalankan Tailwind CSS Compiler untuk memindai semua HTML tersebut dan menghasilkan satu file core.css yang sangat ringan dan optimal.

Membuat file registry.json secara otomatis, yang berisi "peta" dari semua panorama dan pop-up.

Menyertakan file loader.js yang akan digunakan di 3DVista untuk memuat pop-up.

Hasil Akhir: Sebuah file .zip tunggal yang berisi struktur folder yang rapi:

/inject/ (berisi semua file pop-....html)

/dist/ (berisi core.css)

/assets/ (berisi semua file gambar/media yang diunggah)

registry.json

loader.js

5. Tumpukan Teknologi (Tech Stack)
Framework Utama: Next.js (dengan React) untuk membangun antarmuka dan menyediakan kemampuan backend (API Routes).

Styling: Tailwind CSS untuk seluruh antarmuka aplikasi.

Kompilasi CSS (di Server): Tailwind CSS CLI, PostCSS, Autoprefixer, dijalankan melalui Node.js child_process di dalam API Route Next.js.

Library Frontend:

JSZip: Untuk membuat file .zip secara dinamis.

FileSaver.js: Untuk memicu unduhan file .zip.
