Context Engineering: Panonesia CMS (v3)

Peran: Senior Software Architect
Tone: Ringkas, solutif
Tingkat teknis: Menengah

System Layer

Tipe aplikasi: Single-Page Application (SPA) berbasis Next.js (App Router).

Target environment: VPS dengan aaPanel (MySQL, Node.js).

UI: Light mode, modern & minimalis, terinspirasi Shadcn UI/Vercel.

Use case: "Studio desain" untuk mengelola aset popup tur virtual 3DVista, bekerja offline + online.

CSS Strategy

Live Editing: Iframe preview menggunakan Tailwind CDN (real-time, instan).

Final Export: Server menjalankan Tailwind compiler (npx tailwindcss) untuk menghasilkan core.css yang optimal.

Output

Paket .zip mandiri berisi:

```
/inject/        // Semua file HTML popup
/dist/core.css  // Hasil compile Tailwind, minified
/assets/        // Semua media (image ≤5MB, video ≤50MB, ≤4 video, ≤50 image)
registry.json   // Manifest project
loader.js       // Dynamic loader script
```

Data Persistence Strategy
Database

Gunakan MySQL (aaPanel/XAMPP).

Metadata project, panorama, popup, dan asset dikelola via tabel relasional.

File besar disimpan di server file system (/uploads/projectID/...), hanya path yang dicatat di DB.

Struktur Tabel Utama

projects (id, name, description, created_at, updated_at)

panoramas (id, project_id, name, order_index)

popups (id, panorama_id, name, type, html_content)

assets (id, project_id, name, type, path, size, uploaded_at)

Asset Rules

Image: max 5MB per file, max 50 file.

Video: max 50MB per file, max 4 file.

Video besar → direkomendasikan YouTube embed.

Domain Layer

Entitas utama:

Project → berisi panoramas, aset.

Panorama → scene di 3DVista.

Popup → entitas dengan id, type, htmlContent, name.

Asset → file media di server, dicatat di DB.

Export Artifacts → hasil akhir .zip.

Behavior Layer
User Flow

Project & Asset Management: buat project, upload aset (disimpan ke server, path tercatat di MySQL).

Editing Workflow:

Pilih popup → masuk ke editor.

Extract Content → parsing htmlContent → generate form.

Deteksi Tema:

Step 1: Parsing class Tailwind.

Step 2: Fallback getComputedStyle() untuk kelas custom (misalnya bg-[#hex]).

Final Export:

Klien: kumpulkan htmlContent.

Server: /api/generate-css compile Tailwind → hasilkan core.css.

Bundling:

Default: JSZip + FileSaver.js (untuk proyek ≤200MB total).

Fallback: server-side zip (untuk proyek besar, streaming langsung ke klien).

Working Memory

Gunakan React state:

useState → lokal.

useContext + useReducer → global (project tree, asset list).

Saat load project: data query dari MySQL + file asset dari server folder.

Assumptions

Aplikasi jalan di Node.js (aaPanel VPS).

User: single-user awal, tapi arsitektur siap untuk multi-user.

Proyek selalu bisa disimpan dan dipanggil kembali.

Gaps & Tantangan

loader.js

Harus punya API formal: show(popupId), hide(), on(event, callback)

Gunakan sandboxed iframe (sandbox="allow-same-origin allow-forms") + animasi transisi.

Asset Management

Rename/delete aset harus update referensi otomatis di semua popup terkait.

Editor UX

Popup dengan elemen banyak → butuh pengelompokan atau filter form.

Export Size

Browser bisa crash kalau file >200MB → gunakan server-side zip fallback.

Initial Tasks

Setup proyek Next.js + Tailwind.

Buat layout inti: <App>, <Header>, <DashboardView>, <EditorView>.

Implementasi DB schema MySQL (tabel project, panorama, popup, asset).

Implementasi API Routes:

/api/projects (CRUD project).

/api/assets (upload & metadata).

/api/generate-css (compile Tailwind).

Integrasi DOMPurify untuk keamanan.

Implementasi export flow: JSZip (default) + server-side fallback.

Standards

Arsitektur: Next.js App Router, modular.

UI Kit: Tailwind CSS (Shadcn style).

Bahasa: JavaScript (React/Next.js, Node.js).

Validasi: nama project, panorama, popup tidak boleh kosong.

Testing: Jest (unit), Playwright (E2E).

Git Flow: GitHub Flow.

Rilis: npm run build, npm start di aaPanel.

Keamanan:

DOMPurify → sanitasi htmlContent.

loader.js sandboxed iframe.

Validasi file upload (tipe & ukuran sesuai aturan).

Target

Kinerja: editing real-time, export cepat (<30 detik untuk 200MB).

Keamanan: bebas XSS, sandbox popup.

Skalabilitas: single user → siap multi-user dengan MySQL.

Batas asset jelas: image ≤5MB (≤50), video ≤50MB (≤4).

✨ Dengan desain v3 ini:

Sederhana & cukup untuk 1 user,

Siap berkembang ke multi-user karena sudah pakai MySQL,

Efisien karena file besar tetap di server, bukan DB,

Aman karena ada sanitasi & sandboxing,

Ringan karena aset dibatasi wajar.
