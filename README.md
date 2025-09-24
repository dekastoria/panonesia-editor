
# Panonesia Editor — **CMS v1.4** (Popup‑Only, Simplified)

> **Ringkasan:** v1.3 menetapkan ruang lingkup **popup‑only** dengan preview **HTML/CSS/JS** di `<iframe sandbox>`. Dokumen ini menjadi rujukan final untuk implementasi dan menyelaraskan **stack**, **model data**, **UI/UX**, **runtime**, dan **export** dengan `panduan.md` serta `implementasi-dashboard.md` (versi popup‑only).

---

## 1) Tujuan Produk
Pengguna non‑coder dapat:
- Membuat **popup interaktif** dari **galeri template**.
- Mengisi konten melalui **form otomatis** (schema‑driven).
- Melihat **live preview** yang aman (iframe sandbox).
- **Mengekspor ZIP** siap pakai (paritas dengan preview).
- Menggunakan **trigger JS** per‑popup di platform pihak ketiga (mis. 3DVista).

---

## 2) Stack Teknis
**Wajib**
- **Laravel** (backend) + **Filament** (admin CRUD Projects/Scenes/Popups)
- **Spatie Media Library** (upload & manajemen aset)
- **Iframe sandbox** untuk preview + **`window.postMessage`** (dua arah)
- **DOMPurify** (sanitasi HTML pada preview/export)

**Opsional (disarankan untuk UX editor)**
- **React + Vite**, **Tailwind**, **shadcn/ui**
- **Monaco Editor** (tab **Code** untuk mode `custom`)

**Catatan ruang lingkup**
- Tidak ada engine **viewer 360** dan tidak ada **Panel AI** pada v1.3. (Dapat ditambahkan sebagai modul di rilis berikutnya.)

---

## 3) Model Data Minimum
- **projects**: `name`, `slug`, `config_json`, timestamps  
- **scenes**: `project_id`, `key`, `name`, `thumb`, `order`  
- **popups**: `scene_id`, `id_key`, `type` (`image|text|pdf|youtube|custom`), `mode` (`template|custom`), `props_json`, `order`, `status`  
- **media**: dari Spatie (penyimpanan file)

> **config_json** menyimpan seluruh state proyek (scenes + popups) dan menjadi sumber kebenaran untuk preview serta export (`data.js`).

---

## 4) UI/UX Editor (selaras dengan `implementasi-dashboard.md`)
- **Dashboard → Projects**: daftar proyek (badge jumlah scenes & total popups).  
- **Project Detail → Scenes**: daftar panorama/scene (thumbnail + counter) + tombol **+ Add Scene**.  
- **Scene Editor** (per scene):
  - **Kiri – Builder**: daftar popup (search, filter chips type/position/theme/status/errors; multi‑select; duplicate/move/hide/delete).
  - **Kanan Atas – Preview**: **iframe sandbox** + **Device switcher** (Mobile 375 / Tablet 768 / Desktop) + zoom.
  - **Kanan Bawah – Tabs**: **Properties** (auto‑form dari schema), **Assets** (picker), **Console** (log/error), **Code** (hanya jika `mode=custom`).
  - **+ Add Popup** → **Template Gallery** (filter Style/Position/Theme) → pilih template → **Form Properties** → **Preview**.

**Aksesibilitas**
- Alt text untuk gambar; **focus trap** di modal; bisa **tutup via Esc**; tombol **X** selalu tersedia.

---

## 5) Template → Schema → Auto‑Form
Struktur paket template:
```
templates/<templateKey>/
  meta.json
  schema.json      # definisi form & validasi (membangun Properties otomatis)
  template.html
  template.css
  template.js      # hook opsional (init/destroy)
  assets/...
```
**Field umum** (contoh):
- `position` (Top/Bottom/Left/Right/Center), `theme` (Light/Dark)
- `title` (text), `body` (short/paragraph/HTML)
- `img` (asset), `pdf` (asset), `youtubeId` (text)
- `action` (opsional): `switch-scene | url | pdf | youtube | trigger | none`

**Mode popup**
- `template`: tampilkan **Properties** sederhana (form).
- `custom`: tampilkan tab **Code** (Monaco) untuk **HTML/CSS/JS** langsung.

---

## 6) Preview Runtime (HTML‑only)
- **Iframe**: `sandbox="allow-scripts allow-modals"` (tanpa `allow-same-origin`) untuk isolasi penuh.
- **Komunikasi**: `postMessage` dua arah dengan protokol ringkas:  
  - Editor → Preview: `init`, `update`, `bulk`, `inspect`, `refresh`  
  - Preview → Editor: `ready`, `select`, `log`, `diagnostics`
- **Patch update** per‑node berdasarkan `id` (hindari re‑render penuh).  
- **Sanitasi** konten HTML user menggunakan **DOMPurify**.  
- **Device switcher** mengubah **lebar iframe** + zoom.

---

## 7) Aset & Folder Proyek
Saat **Create Project**, buat folder:
```
/storage/projects/{slug}/assets/
  ├─ gambar/
  └─ dokumen/
```
- **Upload Assets** (drag‑drop) → file mendapatkan URL publik: `/storage/projects/{slug}/assets/...`  
- **Assets picker** pada Properties/Code memudahkan memilih dan menyisipkan path.

---

## 8) Trigger JS per‑Popup (Integrasi 3DVista)
- **ID unik**: `<scene>--<type>--<slug>` (kebab‑case), contoh: `p1--image--text03`.  
- **Snippet** otomatis (1 baris) untuk `show` / `hide` (tombol **Copy JS Trigger** pada item).  
- **Runtime listener** disertakan sekali di `script.js` hasil export untuk menangani `show/hide` berdasarkan ID.

---

## 9) Export ZIP (paritas dengan preview)
Struktur akhir:
```
Proyek-Tur-Virtual/
├─ index.html
└─ aset-custom/
   ├─ css/style.css
   ├─ js/data.js        # window.PANONESIA_CONFIG = {...}
   ├─ js/script.js      # runtime + listener trigger
   ├─ gambar/...
   └─ dokumen/...
```
- **Path relatif** (bisa offline).  
- **CSP minimal** di `<head>` `index.html`:
  ```html
  <meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    img-src 'self' data: https:;
    script-src 'self';
    style-src 'self' 'unsafe-inline';
    frame-src https://www.youtube.com https://player.vimeo.com;
  ">
  ```

---

## 10) Acceptance (MVP v1.3)
- [ ] CRUD Projects/Scenes/Popups (Filament) + Upload Assets (Spatie)  
- [ ] Editor: Builder + Preview (iframe) + Tabs (Properties/Assets/Console/Code)  
- [ ] Template Gallery → Form (schema) → Live Preview  
- [ ] Autosave + Diagnostics dasar (asset 404, ID ganda, teks kosong)  
- [ ] Export ZIP sesuai struktur • Paritas preview/export  
- [ ] Trigger JS per‑popup tersedia (Copy)  

---

## 11) Roadmap (modul opsional setelah v1.3)
- Viewer 360 (Pannellum/PSV) sebagai **modul**.  
- Panel AI Assist sebagai **modul**.  
- Preset styling global, theming lanjutan, grid/snap.  
- Versioning/rollback yang lebih nyaman (diff visual).
