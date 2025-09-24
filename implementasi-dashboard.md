
# Implementasi Dashboard (Popup‑Only) — v1.3 (Modular UI/UX Spec)
**Fokus file ini:** rancangan **UI/UX editor** & interaksi (tanpa detail backend).  
**Sinkron dengan `alur-produk.md` dan `kontrak.md`.**

---

## 1) Prinsip & Cakupan
- Editor **popup‑only** dengan **preview realtime** (iframe sandbox + `postMessage`).
- **Scene = penamaan saja** (unit pengelompokan), **tanpa** upload panorama/thumbnail.
- Output akhir diekspor oleh backend sebagai paket ZIP (lihat alur di `alur-produk.md`).

---

## 2) Layout Editor
**Dua kolom (resizable, tersimpan di localStorage):**
- **Kiri — Builder**
  - Project bar: nama, status “Saved hh:mm”.
  - **Scenes list**: daftar scene (kode + nama), tombol **+ Add Scene** (nama & kode).
  - **Popups list** (per scene): search + filter chips (type, status, errors).
  - Aksi cepat popup: **Duplicate**, **Move to Scene**, **Hide/Show**, **Delete**.
  - **+ Add Popup** → buka **Template Gallery** (filter: Style/Position/Theme/Type).
- **Kanan — Split vertikal**
  - **Atas: Live Preview** (`<iframe sandbox="allow-scripts allow-modals">`), toolbar: Device (375/768/fluid), Zoom, Refresh.
  - **Bawah: Tabs**
    - **Properties** (auto‑form dari schema)
    - **Assets** (picker: insert path dari library proyek)
    - **Console** (bridge `log/warn/error` dari preview)
    - **Code** (muncul **hanya** jika mode `custom`).

> **Two‑way select:** klik node di preview → fokus item di Builder + buka Properties.

---

## 3) Alur Menambahkan Popup
1) Klik **+ Add Popup** → **Template Gallery**.
2) Pilih **template** (mis. image/text/pdf/youtube/custom).
3) Preview langsung terpasang di kanan; **Properties** muncul otomatis dari schema.
4) Isi field: **title**, **body** (short/paragraph/HTML), **media** (Upload baru / Browse), **style** (position, theme, size, rounded, shadow), **close behavior** (X/overlay/ESC).
5) **Autosave** (debounce 150–300 ms).

**Mode Custom:** tab **Code** (Monaco) → HTML/CSS/JS, disuntik ke preview dalam sandbox.

---

## 4) Popup‑Menu (UI)
- Lokasi editor: di level **Project** (global) **atau** per‑scene (opsional).
- **Builder (kiri):**
  - Header: **Position** (`left/right/top/bottom`) + **Theme** (`light/dark`).
  - **Items list**: icon + label + target (popup/URL/trigger) + status; drag‑reorder, enable/hide, duplicate, delete.
  - **+ Add Item** → pilih **Action**:
    - `Show Popup` → pilih **Existing** atau **Create new** (mini‑wizard).
    - `Switch Scene` → pilih kode scene.
    - `Open URL` (termasuk PDF).
    - `Custom Trigger` (string bebas).
- **Tabs (kanan bawah):**
  - **Properties (menu):** position, theme, size, spacing, rounded, shadow, **backdrop blur strength**.
  - **Items:** array editor (label, icon, action, target, tooltip).
  - **Assets:** picker ikon/gambar.
  - **Code (optional):** CSS override menu.

**Mini‑wizard “Create new”:**
- **Image Gallery:** Upload/Pick multi‑image; opsi backdrop `dim/blur` (sandblast), rounded, shadow; kontrol **Prev/Next/Close**, **keyboard (← → Esc)**; captions (filename/EXIF/custom).
- **Video Gallery (YouTube):** IDs/URLs, autoplay, mute, controls → modal + backdrop.
- **PDF Viewer:** 1/multi PDF → tab baru atau modal `<iframe>`; tombol Download opsional.
- **About/Services/Contact:** title, paragraph (rich), cover image (opsional), button action.
- **Custom:** HTML/CSS/JS sandbox.

> Wizard selalu menghasilkan **popup baru** & otomatis **link** item menu ke `popupId`.

---

## 5) Interaksi Preview
- Editor → Preview: `init`, `update {path,value}`, `bulk {patches:[...]}`, `inspect`, `refresh`.
- Preview → Editor: `ready`, `select`, `log`, `diagnostics`.
- Patch per‑node (berbasis `id`) untuk kinerja, **hindari rerender penuh**.
- Sanitasi HTML custom memakai DOMPurify (di preview).

---

## 6) Diagnostics (UI)
- Badge merah pada Builder bila ada error.
- Klik error → fokus field terkait di Properties.
- Minimal rules: **asset 404**, **ID ganda**, **tipe file tidak didukung**, **file terlalu besar**, **YouTube ID invalid**.

---

## 7) Naming & Snippet (UI Feedback)
- Setiap popup diberi **ID** format **`pop-<scene>-<type>-<nnn>`** (lihat `kontrak.md`).
- Tersedia tombol **Copy JS Trigger** (SHOW/HIDE) untuk 3DVista (lihat snippet di `kontrak.md`).

---

## 8) Kriteria UX
- Respons patch < 300 ms, device switcher mulus.
- Two‑way select stabil.
- Autosave + Undo/Redo.
- Template Gallery & Popup‑Menu nyaman untuk non‑coder.
