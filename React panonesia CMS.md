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


Context Engineering: Panonesia CMS
Dokumen ini mendefinisikan konteks teknis dan arsitektural untuk pembangunan aplikasi Panonesia CMS, sebuah CMS khusus untuk pop-up interaktif pada tur virtual 3DVista.

Peran: Senior Software Architect

Tone: Ringkas, Solutif

Tingkat Teknis: Sedang

System Layer
Sistem yang akan dibangun adalah sebuah Single-Page Application (SPA) yang berjalan di lingkungan desktop modern. Tujuannya adalah sebagai "studio desain" offline untuk mengelola seluruh aset (HTML, CSS, gambar) dari pop-up virtual tour. Output akhirnya adalah sebuah paket proyek .zip yang mandiri, teroptimasi, dan siap untuk diunggah ke server hosting dan diintegrasikan dengan 3DVista.

Domain Layer
Domain inti aplikasi berpusat pada tiga entitas utama:

Project: Representasi keseluruhan tur virtual, yang terdiri dari:

Panorama: Entitas utama yang merepresentasikan sebuah scene di 3DVista. Memiliki ID unik (1, 2, ...) dan nama deskriptif ("Ruang Tamu").

Pop-up: Entitas anak dari Panorama. Memiliki ID terstruktur (pop-<panoId>-<type>-<nnn>), nama deskriptif, tipe (text, img, embed, custom), dan konten HTML.

Asset Library: Sebuah repositori terpusat untuk semua media (gambar, PDF, dll.) yang diunggah oleh pengguna. Setiap aset memiliki path unik yang akan digunakan di dalam HTML pop-up.

Export Artifacts: Hasil akhir yang akan digenerasi oleh sistem, terdiri dari:

inject/: Folder berisi semua file HTML pop-up.

dist/core.css: Satu file CSS yang telah dioptimalkan oleh Tailwind JIT Compiler.

assets/: Folder berisi semua media yang digunakan.

registry.json: File manifest JSON yang memetakan panorama ke pop-up miliknya.

loader.js: Skrip pemuat dinamis untuk dieksekusi oleh 3DVista.

Behavior Layer
Alur kerja pengguna (User Flow) utama adalah sebagai berikut:

Project Setup: Pengguna membuat struktur proyek dengan menambahkan beberapa Panorama di sidebar.

Asset Management: Pengguna beralih ke tab "Assets" untuk mengunggah semua gambar dan PDF yang dibutuhkan.

Pop-up Creation: Pengguna memilih sebuah Panorama, mengklik "Add New Pop-up", dan memilih tipe (text, img, embed, custom).

Editing Workflow:

Sistem membuka layar Editor untuk pop-up yang baru dibuat.

Pengguna menempelkan kode HTML dasar (jika custom) atau memodifikasi template yang ada.

Pengguna mengklik "Extract Content". Sistem memindai HTML dan secara otomatis:

Mengisi panel "Color Theme" dengan warna-warna yang terdeteksi dari kelas Tailwind.

Membuat form interaktif untuk setiap teks, gambar (src), dan link (href/action).

Pengguna mengedit konten melalui form. Untuk gambar/PDF, pengguna mengklik "Browse" untuk memilih dari Asset Library. Untuk teks, pengguna mengklik ikon pensil untuk membuka toolbar styling (bold, font, warna, dll.).

Semua perubahan di form akan diperbarui secara real-time di pratinjau.

Final Export: Setelah semua pop-up selesai, pengguna mengklik "Export Final Project". Sistem memicu API Route untuk mengkompilasi core.css, lalu membungkus semua Export Artifacts ke dalam satu file .zip dan menyajikannya untuk diunduh.

Working Memory
Pada setiap sesi, sistem harus mampu mengelola state aplikasi secara keseluruhan, yang mencakup:

Struktur proyek saat ini (daftar panorama dan pop-up di dalamnya beserta konten HTML-nya).

Daftar aset yang telah diunggah di Media Library.

State dari pop-up yang sedang aktif diedit (konten, warna, gaya).

Assumptions
Aplikasi akan dijalankan di lingkungan Node.js (via Next.js). Pengguna memahami cara menginstal dan menjalankan proyek Next.js secara lokal.

Browser yang digunakan pengguna adalah browser modern (Chrome, Firefox, Edge) yang mendukung fitur-fitur seperti DOMParser, Fetch API, dan child_process di sisi server.

Tujuan akhir integrasi adalah 3DVista, yang mampu mengeksekusi JavaScript dari hotspot.

Pengguna memiliki pemahaman dasar tentang HTML dan kelas-kelas Tailwind CSS.

Gaps
Detail loader.js: Spesifikasi rinci untuk loader.js belum didefinisikan. Perlu riset tentang cara terbaik berinteraksi dengan API 3DVista untuk menampilkan pop-up dengan transisi.

Manajemen Aset Lanjutan: Fungsionalitas untuk menghapus atau mengganti nama aset di Media Library belum dijabarkan.

Validasi & Error Handling: Belum ada spesifikasi untuk validasi input (misalnya, URL yang valid) atau penanganan error (misalnya, jika kompilasi CSS gagal).

Initial Tasks
Setup Proyek: Inisialisasi proyek Next.js dengan Tailwind CSS, PostCSS, dan Autoprefixer.

Bangun Layout Inti: Buat komponen layout utama: Header, Sidebar (dengan fungsionalitas tab), dan area Konten Utama.

Implementasi Manajemen Panorama & Pop-up: Buat state management untuk menambah, memilih, dan menampilkan daftar panorama serta pop-up.

Integrasikan Editor (Beta 2): Masukkan logika editor komponen (yang sebelumnya ada di index.html) sebagai "Editor View" yang dinamis.

Buat API Route untuk Kompilasi CSS: Implementasikan endpoint /api/generate-css yang menerima HTML dan menjalankan Tailwind CLI menggunakan child_process.

Implementasikan Fungsi Ekspor: Buat fungsi handleExport yang mengintegrasikan JSZip dan memanggil API Route untuk menghasilkan paket .zip akhir.

Standar
Arsitektur: Next.js dengan App Router. Logika sisi server ditangani oleh API Routes.

UI Kit: Tailwind CSS.

Bahasa Pemrograman: JavaScript (React/Next.js).

Data Fetching: State management client-side (React Hooks: useState, useContext). Fetch API untuk komunikasi dengan API Route internal.

Validasi: Validasi input dasar di sisi klien.

Testing: Direkomendasikan Jest/Vitest untuk unit testing dan Cypress/Playwright untuk E2E testing.

A11y (Aksesibilitas): Menggunakan HTML semantik dan atribut ARIA jika diperlukan.

SEO: Tidak relevan untuk aplikasi ini.

Git: Menggunakan Gitflow (feature branches, develop, main) atau GitHub Flow.

Rilis: npm run build untuk membuat build produksi, yang kemudian dijalankan dengan npm start.

Target
Kinerja: Pengalaman editing harus real-time tanpa lag. Proses "Export Final Project" untuk proyek besar (90+ pop-up) harus selesai dalam beberapa detik. File core.css yang dihasilkan harus sangat optimal.

Keamanan: Meskipun dijalankan secara lokal, semua input HTML harus disanitasi sebelum ditampilkan di iframe untuk mencegah XSS.

Skalabilitas: Arsitektur harus mampu menangani ratusan panorama dan pop-up tanpa degradasi performa pada aplikasi editor.
