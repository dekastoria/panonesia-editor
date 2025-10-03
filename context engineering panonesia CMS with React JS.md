Context Engineering: Panonesia CMS
Dokumen ini mendefinisikan konteks teknis dan arsitektural untuk pembangunan aplikasi Panonesia CMS, sebuah CMS khusus untuk pop-up interaktif pada tur virtual 3DVista, dengan pembaruan visual dan arsitektur terbaru.

peran: "Senior Software Architect"

tone: "ringkas, solutif"

tingkat_teknis: sedang

System Layer
Sistem yang akan dibangun adalah sebuah Single-Page Application (SPA) yang berjalan di lingkungan desktop modern. Tujuannya adalah sebagai "studio desain" offline untuk mengelola seluruh aset dari pop-up virtual tour.

Sistem ini menggunakan pendekatan hybrid untuk CSS:

Saat Live Editing: Pratinjau (iframe) memuat Tailwind via CDN untuk kecepatan dan pengalaman real-time yang instan.

Saat Final Export: Sebuah file core.css yang teroptimasi penuh akan dikompilasi di sisi server dan disertakan dalam paket ekspor untuk performa produksi maksimal.

Output akhirnya adalah sebuah paket proyek .zip yang mandiri, teroptimasi, dan siap untuk diunggah ke server hosting dan diintegrasikan dengan 3DVista. Antarmuka pengguna (UI) akan mengadopsi tema terang (light mode) yang bersih, modern, dan minimalis, terinspirasi dari estetika Shadcn UI/Vercel.

Domain Layer
Domain inti aplikasi berpusat pada entitas berikut:

Project: Representasi keseluruhan tur virtual, yang dapat dipilih dari sebuah Dashboard Overview. Terdiri dari:

Panorama: Entitas yang merepresentasikan sebuah scene di 3DVista (ID unik dan nama deskriptif).

Pop-up: Entitas anak dari Panorama (ID terstruktur, nama, tipe, konten HTML).

Asset Library: Repositori terpusat untuk semua media (gambar, PDF, dll.).

Template Library: Koleksi komponen HTML siap pakai dengan fungsi "Copy Code".

Export Artifacts: Hasil akhir proyek dalam format .zip, berisi:

inject/: Folder berisi semua file HTML pop-up.

dist/core.css: Satu file CSS yang telah dioptimalkan.

assets/: Folder berisi semua media yang digunakan.

registry.json: File manifest JSON.

loader.js: Skrip pemuat dinamis.

Behavior Layer
Alur kerja pengguna (User Flow) utama adalah sebagai berikut:

Project & Asset Management: Pengguna mengelola proyek, panorama, dan aset media melalui antarmuka utama.

Editing Workflow: Pengguna memilih sebuah pop-up dan masuk ke Editor View (tata letak 3 kolom) untuk mengedit konten, gaya, dan tema warna secara real-time (menggunakan pratinjau dengan CDN).

Final Export:

Langkah A & B (Klien): Saat pengguna mengklik "Export Final Project", aplikasi (React) mengumpulkan semua kode HTML dari semua pop-up dalam proyek dan mengirimkannya ke API Route.

Langkah C (Server): Di server, API Route menjalankan Tailwind CSS Compiler (child_process). Compiler ini memindai semua HTML yang diterima, menghasilkan satu core.css yang hanya berisi gaya untuk kelas-kelas yang digunakan (proses Purging/Tree-Shaking).

Langkah D (Server -> Klien): API Route mengirimkan kembali core.css yang sudah jadi dan super optimal ke aplikasi klien.

Langkah E (Klien): Aplikasi klien menerima core.css, lalu menggunakan JSZip untuk membungkus semua Export Artifacts (HTMLs, core.css, registry.json, loader.js, gambar) ke dalam satu file .zip dan menyajikannya untuk diunduh menggunakan FileSaver.js.

Working Memory
Sistem harus mampu mengelola state aplikasi secara keseluruhan, termasuk struktur proyek, daftar aset, dan state dari pop-up yang sedang aktif diedit.

Assumptions
Aplikasi dijalankan di lingkungan Node.js (via Next.js).

Browser pengguna modern.

Tujuan integrasi adalah 3DVista yang mampu menjalankan JavaScript.

Pengguna memiliki pemahaman dasar HTML dan kelas Tailwind.

Gaps
Spesifikasi rinci untuk loader.js (efek transisi).

Fungsionalitas manajemen aset (hapus, ganti nama).

Mekanisme penyimpanan data proyek (misalnya, localStorage).

Initial Tasks
Setup Proyek: Inisialisasi proyek Next.js dengan Tailwind CSS, PostCSS, dan Autoprefixer.

Bangun Layout Inti: Buat komponen layout utama (Header, 3 Tampilan Utama, dll.) dengan tema visual "Shadcn".

Implementasi Editor View: Integrasikan mesin editor fungsional dari Beta 2 ke dalam tata letak 3 kolom.

Buat API Route untuk Kompilasi CSS: Implementasikan endpoint /api/generate-css yang menerima HTML dan menjalankan Tailwind CLI menggunakan child_process.

Implementasikan Fungsi Ekspor: Buat fungsi handleExport yang mengintegrasikan JSZip, FileSaver.js, dan memanggil API Route.

Standar
arsitektur: Next.js dengan App Router. Logika sisi server ditangani oleh API Routes.

ui_kit: Tailwind CSS, dengan bahasa desain terinspirasi dari Shadcn UI/Vercel.

bahasa_pemrograman: JavaScript (React/Next.js), dengan Node.js untuk sisi server (API Routes).

data_fetching: State management client-side (React Hooks). Fetch API untuk API Route.

validasi: Validasi input dasar di sisi klien.

testing: Direkomendasikan Jest/Vitest (unit), Cypress/Playwright (E2E).

a11y: Menggunakan HTML semantik.

seo: Tidak relevan.

git: GitHub Flow.

rilis: npm run build dan npm start.

Target
kinerja: Pengalaman editing real-time. Proses ekspor cepat. core.css yang dihasilkan harus sangat optimal.

keamanan: Sanitasi input HTML untuk mencegah XSS.

skalabilitas: M
