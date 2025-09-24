[master-spec-v1.4.md](https://github.com/user-attachments/files/22516579/master-spec-v1.4.md)
# Panonesia Editor — **Master Spec v1.4** (Popup‑Only, HTML Preview)
> **Acuan Dasar:** README.md (v1.4) — diselaraskan dengan `implementasi-dashboard.md`, `alur-produk.md`, dan `kontrak.md` terbaru.  
> **Perubahan penting:** (1) **Format ID** distandardkan: `pop-<scene>-<type>-<nnn>`. (2) **Scenes = naming only** (tanpa import panorama/thumbnail).

---

## 0) Tujuan
Pengguna **non‑coder** dapat:
- Membuat **popup interaktif** dari **galeri template** (image/text/pdf/youtube/custom).
- Mengisi konten via **form otomatis** (schema‑driven).
- Melihat **live preview** yang aman (iframe sandbox, HTML‑only).
- **Mengekspor ZIP** (paritas dengan preview) → satu folder rapi siap dipakai.
- Menggunakan **Trigger JS** per‑popup di platform pihak ketiga (mis. 3DVista).

---

## 1) Ruang Lingkup v1.4
- **Popup‑only** (tanpa viewer 360; dapat ditambah modul di rilis selanjutnya).
- **Tidak** mencakup AI panel untuk v1.4 (opsional di fase berikutnya).
- **Focus:** editor, menu popup, preview realtime, export ZIP, dokumentasi trigger.

---

## 2) Stack Teknis
**Wajib**
- **Laravel** (backend) + **Filament** (admin CRUD Projects/Scenes/Popups)
- **Spatie Media Library** (upload & manajemen aset)
- **Iframe sandbox** untuk preview + **`window.postMessage`** (dua arah)
- **DOMPurify** (sanitasi HTML pada preview/export)

**Opsional (disarankan UX editor)**
- **React + Vite**, **Tailwind**, **shadcn/ui**
- **Monaco Editor** (tab **Code** untuk mode `custom`)

---

## 3) Model Data Minimum
- **projects**: `name`, `slug`, `config_json`, timestamps
- **scenes**: `project_id`, `key`, `name`, `order`  ← **(tanpa thumbnail/pano)**
- **popups**: `scene_id`, `id_key`, `type` (`image|text|pdf|youtube|custom`), `mode` (`template|custom`), `props_json`, `order`, `status`
- **media**: dari Spatie (penyimpanan file)

> **`config_json`** menyimpan seluruh state proyek (scenes + popups + menu) dan menjadi **sumber kebenaran** untuk preview serta export (`data.js`).

---

## 4) UI/UX Editor (selaras dengan `implementasi-dashboard.md`)
- **Dashboard → Projects**: daftar proyek (badge jumlah scenes & total popups).
- **Project Detail → Scenes**: daftar **scene (nama/kode saja)** + tombol **+ Add Scene**.
- **Scene Editor** (per scene):
  - **Kiri – Builder**: daftar popup (search, filter chips type/position/theme/status/errors; multi‑select; duplicate/move/hide/delete).
  - **Kanan Atas – Preview**: **iframe sandbox** + **Device switcher** (Mobile 375 / Tablet 768 / Desktop) + zoom.
  - **Kanan Bawah – Tabs**: **Properties** (auto‑form dari schema), **Assets** (picker), **Console** (log/error), **Code** (hanya jika `mode=custom`).
  - **+ Add Popup** → **Template Gallery** (filter Style/Position/Theme) → pilih template → **Form Properties** → **Preview**.
- **Popup‑Menu** (global di project atau per‑scene): builder items (About/Services/Gallery/Video/PDF/Share/Contact/Custom). Item bisa **refer ke popup** atau **aksi** (switch/url/trigger).

**Aksesibilitas**
- Alt text untuk gambar; **focus trap** di modal; **Esc** menutup; tombol **X** wajib.

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
Field umum (contoh):
- `position` (Top/Bottom/Left/Right/Center), `theme` (Light/Dark)
- `title` (text), `body` (short/paragraph/HTML)
- `img` (asset), `pdf` (asset), `youtubeId` (text)
- `action` (opsional): `switch-scene | url | pdf | youtube | trigger | none`

Mode popup:
- `template`: tampilkan **Properties** sederhana (form).
- `custom`: tab **Code** (Monaco) untuk **HTML/CSS/JS** langsung.

---

## 6) Preview Runtime (HTML‑only)
- **Iframe**: `sandbox="allow-scripts allow-modals"` (tanpa `allow-same-origin`) untuk isolasi.
- **Komunikasi**: `postMessage` dua arah (lihat **`kontrak.md`**):
  - Editor → Preview: `init`, `update`, `bulk`, `inspect`, `refresh`
  - Preview → Editor: `ready`, `select`, `log`, `diagnostics`
- **Patch update** per‑node berdasarkan `id` (hindari re‑render penuh).
- **Sanitasi** HTML custom dengan **DOMPurify**.
- **Device switcher** mengubah **lebar iframe** + zoom.

---

## 7) Aset & Folder Proyek
Saat **Create Project**, buat folder:
```
/storage/projects/{slug}/assets/
  ├─ gambar/
  └─ dokumen/
```
- **Upload Assets** (drag‑drop) → file mendapat URL publik: `/storage/projects/{slug}/assets/...`
- **Assets picker** memudahkan menyisipkan path ke field Properties/Code.

---

## 8) Penamaan ID Popup (distandardkan)
- **Format final (wajib):** `pop-<scene>-<type>-<nnn>`  
  - `<scene>`: kode scene (mis. `s01`, `lobby`) — kebab‑case, ≤ 20 char.  
  - `<type>`: `img|txt|pdf|yt|vid|gal|cus|menu`.  
  - `<nnn>`: nomor 3 digit (`001`, `002`, …).  
  - Contoh: `pop-s02-txt-002`, `pop-lobby-gal-003`, `pop-global-menu-main` (khusus menu boleh slug).

> **Alasan**: mudah diketik di 3DVista, konsisten, dan jelas memisahkan scene/type/urutan.

---

## 9) Trigger JS per‑Popup (Integrasi 3DVista)
**SHOW (one‑liner):**
```js
(function(){var M={proto:"panonesia/1",type:"trigger",action:"show",id:"pop-s01-img-001"};try{window.postMessage(M,"*");window.parent&&window.parent.postMessage(M,"*");new BroadcastChannel("panonesia").postMessage(M);}catch(e){}})();
```
**HIDE (one‑liner):**
```js
(function(){var M={proto:"panonesia/1",type:"trigger",action:"hide",id:"pop-s01-img-001"};try{window.postMessage(M,"*");window.parent&&window.parent.postMessage(M,"*");new BroadcastChannel("panonesia").postMessage(M);}catch(e){}})();
```
**Helper global (opsional, ditempel sekali):**
```js
window.POP={
  show:id=>{var M={proto:"panonesia/1",type:"trigger",action:"show",id};window.postMessage(M,"*");try{window.parent.postMessage(M,"*");new BroadcastChannel("panonesia").postMessage(M);}catch(e){}},
  hide:id=>{var M={proto:"panonesia/1",type:"trigger",action:"hide",id};window.postMessage(M,"*");try{window.parent.postMessage(M,"*");new BroadcastChannel("panonesia").postMessage(M);}catch(e){}}
};
// Contoh: POP.show("pop-s02-txt-002");
```

> Semua snippet & listener **harus identik** dengan `kontrak.md` (single source of truth).

---

## 10) Export ZIP (paritas dengan preview)
Struktur akhir:
```
Proyek-Tur-Virtual/
├─ index.html
└─ aset-custom/
   ├─ css/style.css
   ├─ js/data.js      # window.PANONESIA_CONFIG = {...}
   ├─ js/script.js    # runtime + listener + gallery
   ├─ gambar/...
   └─ dokumen/...
```
**CSP (di `<head>` index.html):**
```
default-src 'self';
img-src 'self' data: https:;
script-src 'self';
style-src 'self' 'unsafe-inline';
frame-src https://www.youtube.com https://player.vimeo.com;
```

> Sertakan dokumen **“Trigger JS”** (Markdown/HTML) berisi daftar semua **ID** + snippet per‑aksi.

---

## 11) Kriteria Selesai (MVP)
- Preview patch < 300 ms; two‑way select stabil.
- Popup‑Menu tampil & memicu popup/aksi.
- Gallery ringan (backdrop blur, prev/next/close, keyboard, caption).
- Export ZIP = parity dengan preview.
- Dokumen **Trigger JS** lengkap & valid.

---

## 12) Referensi Dokumen
- **kontrak.md** — *single source of truth* (ID, postMessage, struktur ZIP, skema menu, field minimal, snippet).
- **implementasi-dashboard.md** — UI/UX & interaksi editor.
- **alur-produk.md** — alur end‑to‑end & sisi backend (DB/storage/export).
