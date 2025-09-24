
# Alur Produk (Backend‑Aware) — v1.3 (Modular Flow Spec)
**Fokus file ini:** alur end‑to‑end & struktur data/export (tanpa detail UI).  
**Sinkron dengan `implementasi-dashboard.md` dan `kontrak.md`.**

---

## 1) Tujuan & Scope
- Bangun **CMS popup** + **popup‑menu** (popup‑only).
- **Scene = penamaan/pengelompokan** (tanpa import pano/thumbnail).
- Preview realtime (iframe + `postMessage`).
- Export **satu folder** untuk integrasi 3DVista.

---

## 2) Flow End‑to‑End
1) **Create Project**
   - Field: `name`, `slug` (auto).
   - Tanya: **Aktifkan Popup‑Menu global?** (Ya/Skip).
   - Buat storage:
     ```
     /storage/projects/{slug}/assets/
       ├─ gambar/
       └─ dokumen/
     ```
2) **Scenes (Naming Only)**
   - Tambah daftar scene: `key` (mis. `s01`, `lobby`) + `name`.
3) **Edit per Scene**
   - Tambah **Popups**: image/text/pdf/youtube/custom.
   - Upload asset: saat itu atau browse dari library.
4) **Popup‑Menu**
   - Di level Project (global) **atau** per‑scene, berisi **items** yang refer ke `popupId` atau aksi lain.
5) **Export**
   - Tulis **`index.html`**, **`aset-custom/css/style.css`**, **`aset-custom/js/data.js`**, **`aset-custom/js/script.js`**, salin semua **gambar/dokumen** yang dipakai.
   - Sertakan **Dokumen “Trigger JS”** (daftar ID + snippet 1 baris).

---

## 3) Skema Data (Top‑Level)
```json
{
  "project": {
    "title": "Hotel A",
    "menu": {
      "enabled": true,
      "position": "left|right|top|bottom",
      "theme": "light|dark",
      "items": [
        { "label": "About",   "icon": "aset-custom/gambar/user.svg",  "action": "show", "target": "pop-s01-txt-001" },
        { "label": "Gallery", "icon": "aset-custom/gambar/photo.svg", "action": "show", "target": "pop-s01-gal-002" },
        { "label": "PDF",     "icon": "aset-custom/gambar/pdf.svg",   "action": "url",  "target": "aset-custom/dokumen/menu.pdf" }
      ]
    }
  },
  "scenes": [
    {
      "id": "s01",
      "name": "Lobby",
      "popups": [
        {
          "id": "pop-s01-img-001",
          "type": "template",
          "templateKey": "popup-image",
          "props": {
            "title": "Poster",
            "img": "aset-custom/gambar/poster.webp",
            "position": "Center",
            "theme": "Light"
          }
        },
        {
          "id": "pop-s01-gal-002",
          "type": "template",
          "templateKey": "popup-gallery",
          "props": {
            "images": [
              { "src": "aset-custom/gambar/a.webp", "caption": "Lobby" }
            ],
            "backdrop": "blur",
            "controls": true,
            "captions": true
          }
        }
      ]
    }
  ]
}
```

---

## 4) Database & Storage (ringkas)
- **Tabel minimal**  
  - `projects { id, name, slug, config_json, size_bytes, deleted_at }`  
  - `scenes { id, project_id, key, name, order, deleted_at }`  
  - `popups { id, scene_id, id_key, type, mode, props_json, order, status, deleted_at }`  
  - `media` (Spatie) untuk file.  
- **Soft‑delete** + garbage collector (hapus **orphan** & trash lama).  
- **Where‑used** untuk media (aman saat hapus/rename).

---

## 5) Export — Struktur Folder
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
**CSP di `<head>` index.html:**  
`default-src 'self'; img-src 'self' data: https:; script-src 'self'; style-src 'self' 'unsafe-inline'; frame-src https://www.youtube.com https://player.vimeo.com;`

---

## 6) Runtime (Ekspor)
- `data.js` → `window.PANONESIA_CONFIG` (skema di atas).
- `script.js`:
  - Parse config.
  - Render **menu** (jika enabled).
  - Listener `postMessage` untuk **show/hide/switch/url/custom**.
  - **Lightbox gallery** vanilla: modal center, backdrop dim/blur, **Prev/Next/Close**, keyboard (← → Esc), caption.
  - Render popup **on‑demand** (bukan eager).

---

## 7) Kriteria Selesai (MVP)
- Preview patch < 300 ms; export = parity.
- Menu memicu popup/aksi.
- Gallery ringan & responsif.
- Dokumen **Trigger JS** lengkap.
