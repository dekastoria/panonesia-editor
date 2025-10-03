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
Domain inti aplikasi berpusat pada entitas berikut dengan struktur data yang disarankan:

Project: Representasi keseluruhan tur virtual.

id: string

name: string

panoramas: Array<Panorama>

Panorama: Entitas yang merepresentasikan sebuah scene di 3DVista.

id: number (e.g., 1, 2, 3)

name: string (e.g., "1. Ruang Tamu")

popups: Array<Popup>

Popup: Entitas anak dari Panorama.

id: string (e.g., "pop-1-text-001")

name: string

type: 'text' | 'img' | 'gallery' | 'embed' | 'custom'

htmlContent: string

Asset: Representasi file di Asset Library.

id: string

name: string (e.g., "sofa.jpg")

path: string (e.g., "assets/images/sofa.jpg")

blobUrl: string (URL sementara untuk pratinjau)

file: File (objek file asli untuk ekspor)

Export Artifacts: Hasil akhir proyek dalam format .zip, berisi:

inject/: Folder berisi semua file HTML pop-up.

dist/core.css: Satu file CSS yang telah dioptimalkan.

assets/: Folder berisi semua media yang digunakan.

registry.json: File manifest JSON.

loader.js: Skrip pemuat dinamis.

Behavior Layer
Alur kerja pengguna (User Flow) utama adalah sebagai berikut:

Project & Asset Management: Pengguna mengelola proyek, panorama, dan aset media melalui antarmuka utama.

Editing Workflow: Pengguna memilih sebuah pop-up dan masuk ke Editor View. Saat mengklik "Extract Content", sistem harus:

Membuat DOMParser untuk mem-parsing htmlContent dari pop-up yang aktif.

Deteksi Tema: Melakukan iterasi pada semua elemen dan classList untuk mencari kelas warna Tailwind yang cocok dengan tailwindColorMap. Prioritaskan kelas yang paling relevan (misalnya, bg- pada <button> untuk primary color) untuk mengisi state themeColors.

Buat Form: Melakukan iterasi kedua untuk membuat form fields. Beri data-editable-id unik pada setiap elemen yang diekstrak. Buat field untuk textContent, src (pada <img>), dan href (pada <a> yang tidak memiliki <a> anak).

Final Export:

Langkah A (Klien): Mengumpulkan htmlContent dari setiap pop-up di semua panorama.

Langkah B (Klien -> Server): Mengirim array dari semua htmlContent ke API Route /api/generate-css.

Langkah C (Server): API Route menggabungkan semua HTML menjadi satu string, menuliskannya ke file sementara, lalu menjalankan Tailwind CSS Compiler (child_process) yang menargetkan file sementara tersebut.

Langkah D (Server -> Klien): API Route membaca hasil core.css dan mengirimkannya kembali sebagai respons.

Langkah E (Klien): Aplikasi klien menggunakan JSZip untuk membuat struktur folder, menambahkan semua Export Artifacts, dan memicu unduhan menggunakan FileSaver.js.

Working Memory
Sistem harus mampu mengelola state aplikasi secara keseluruhan, termasuk struktur proyek, daftar aset, dan state dari pop-up yang sedang aktif diedit. Gunakan useState untuk state komponen lokal dan useContext + useReducer untuk state global yang kompleks (struktur proyek dan aset).

Assumptions
Aplikasi dijalankan di lingkungan Node.js (via Next.js).

Browser pengguna modern.

Tujuan integrasi adalah 3DVista yang mampu menjalankan JavaScript.

Pengguna memiliki pemahaman dasar HTML dan kelas Tailwind.

Gaps
loader.js Specification: Perlu didefinisikan fungsinya. Minimal harus memiliki fungsi global seperti window.panonesiaLoader.show(popupId) yang dapat menerima id pop-up, mencari URL-nya di registry.json, membuat iframe di dalam modal container, dan menampilkannya dengan transisi fade-in.

Asset Management: Fungsionalitas untuk menghapus atau mengganti nama aset di state assets perlu ditambahkan.

Project Persistence: Mekanisme penyimpanan data proyek perlu dipilih. Rekomendasi awal: Gunakan localStorage untuk menyimpan seluruh state proyek dalam format JSON, dengan tombol "Save Project" dan "Load Project".

Initial Tasks
Setup Proyek: Inisialisasi proyek Next.js dengan Tailwind CSS.

Bangun Layout Inti: Buat hirarki komponen React: <App>, <Header>, <DashboardView>, <EditorView>, <TemplatesView>. Terapkan tema visual "Shadcn".

Implementasi Editor View: Di dalam <EditorView>, buat komponen <PanoramaSidebar>, <MainEditor (berisi <PreviewIframe> dan <CodeEditor>), dan <LiveEditorSidebar>.

Buat API Route untuk Kompilasi CSS: Implementasikan endpoint /api/generate-css.

Implementasikan Fungsi Ekspor: Buat fungsi handleExport yang mengintegrasikan JSZip, FileSaver.js, dan memanggil API Route.

Standar
arsitektur: Next.js dengan App Router. Logika sisi server ditangani oleh API Routes.

ui_kit: Tailwind CSS, dengan bahasa desain terinspirasi dari Shadcn UI/Vercel.

bahasa_pemrograman: JavaScript (React/Next.js), dengan Node.js untuk sisi server.

data_fetching: State management client-side (useState, useContext). Fetch API untuk API Route.

validasi: Validasi input dasar di sisi klien (misalnya, memastikan nama panorama tidak kosong).

testing: Direkomendasikan Jest/Vitest (unit), Cypress/Playwright (E2E).

a11y: Menggunakan HTML semantik dan atribut ARIA.

seo: Tidak relevan.

git: GitHub Flow.

rilis: npm run build dan npm start.

Target
kinerja: Pengalaman editing real-time. Proses ekspor cepat. core.css yang dihasilkan harus sangat optimal.

keamanan: Sanitasi input HTML untuk
