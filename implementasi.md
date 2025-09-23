# Panduan Implementasi Addon Panonesia Editor v1.1

Dokumen ini adalah panduan praktis untuk mengintegrasikan *framework addon* Panonesia Editor v1.1 ke dalam proyek tur virtual 3DVista Anda.

---

## Langkah 1: Siapkan Struktur Folder & File

Pertama, buka folder hasil *export* tur 3DVista Anda. Buat beberapa folder dan file baru agar strukturnya sesuai dengan sistem addon kita.

1.  Buat folder baru bernama `addon-kustom`.
2.  Buat folder baru lagi bernama `data-popups`.
3.  Di dalam folder `addon-kustom`, buat dua file kosong:
    * `addon-style.css`
    * `addon-loader.js`
4.  Di dalam folder `data-popups`, buat satu atau beberapa file konten, misalnya `konten.js`.

Struktur akhir folder Anda akan terlihat seperti ini:

- FOLDER-TUR-ANDA/
  |- index.htm
  |- script.js
  |- ... (file 3dvista lainnya) ...
  |
  |- addon-kustom/
  |  |- addon-loader.js         <-- (Akan kita isi di Langkah 2)
  |  |- addon-style.css         <-- (Akan kita isi di Langkah 2)
  |
  |- data-popups/
  |  |- konten.js               <-- (Akan kita isi di Langkah 3)
  |
  |- media/
     |- ... (semua aset gambar tur) ...
---

## Langkah 2: Isi File Inti Addon (`Engine`)

Salin dan tempel kode di bawah ini ke dalam file yang sesuai di folder `addon-kustom`.

### ### File: `addon-kustom/addon-style.css`

```css
/* addon-kustom/addon-style.css (Final v1.1) */
@import url('[https://fonts.googleapis.com/css2?family=Montserrat&family=Roboto&display=swap](https://fonts.googleapis.com/css2?family=Montserrat&family=Roboto&display=swap)');

:root {
    --popup-bg: #ffffff; --popup-text: #333333; --popup-title: #000000;
    --popup-btn-bg: #007bff; --popup-btn-text: #ffffff;
}
body.tema-gelap .popup-content {
    --popup-bg: #2c2c2c; --popup-text: #e0e0e0; --popup-title: #ffffff;
    --popup-btn-bg: #555555; --popup-btn-text: #ffffff;
}

.popup-overlay {
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    background-color: rgba(0, 0, 0, 0.7);
    z-index: 99999; display: flex; padding: 20px; box-sizing: border-box;
    -webkit-backdrop-filter: blur(4px); backdrop-filter: blur(4px);
    font-family: Arial, sans-serif;
    opacity: 0;
    transition: opacity 0.3s ease;
}
.popup-overlay.visible {
    opacity: 1;
}

.popup-content {
    background-color: var(--popup-bg); color: var(--popup-text);
    padding: 25px; border-radius: 12px;
    box-shadow: 0 5px 20px rgba(0,0,0,0.3);
    text-align: center; overflow-y: auto; max-height: 100%;
    position: relative;
    transform: scale(0.95);
    opacity: 0;
    transition: transform 0.3s ease, opacity 0.3s ease;
}
.popup-overlay.visible .popup-content {
    transform: scale(1);
    opacity: 1;
}

.close-x-btn {
    position: absolute; top: 10px; right: 15px;
    background: none; border: none; font-size: 28px;
    color: var(--popup-text); cursor: pointer; line-height: 1;
    opacity: 0.5; transition: opacity 0.2s ease;
}
.close-x-btn:hover { opacity: 1; }

/* Posisi */
.popup-overlay.posisi-tengah  { justify-content: center; align-items: center; }
.popup-overlay.posisi-atas    { justify-content: center; align-items: flex-start; }
.popup-overlay.posisi-bawah   { justify-content: center; align-items: flex-end; }
.popup-overlay.posisi-kiri    { justify-content: flex-start; align-items: center; }
.popup-overlay.posisi-kanan   { justify-content: flex-end; align-items: center; }

/* Ukuran */
.posisi-tengah .popup-content, .posisi-atas .popup-content, .posisi-bawah .popup-content { width: 650px; max-width: 90%; }
.posisi-kiri .popup-content, .posisi-kanan .popup-content { width: 400px; max-width: 90%; height: auto; max-height: 90%; }

/* Konten */
.popup-content h2 { margin-top: 0; color: var(--popup-title); }
.popup-content .deskripsi { color: var(--popup-text); line-height: 1.6; text-align: left; }
.popup-content img.info-image { width: 100%; height: auto; border-radius: 8px; margin-bottom: 15px; }

/* Galeri */
.gallery-container { position: relative; width: 100%; overflow: hidden; margin: 15px 0; border-radius: 8px; }
.gallery-images { display: flex; transition: transform 0.4s ease-in-out; }
.gallery-images img { width: 100%; flex-shrink: 0; }
.gallery-nav { position: absolute; top: 50%; transform: translateY(-50%); background-color: rgba(0, 0, 0, 0.5); color: white; border: none; padding: 10px; cursor: pointer; font-size: 20px; border-radius: 50%; width: 44px; height: 44px; display: flex; justify-content: center; align-items: center; }
.gallery-nav.prev { left: 15px; } .gallery-nav.next { right: 15px; }

/* Tombol Aksi */
.action-buttons-container { margin-top: 20px; display: flex; flex-direction: column; gap: 10px; }
.action-btn { background-color: var(--popup-btn-bg); color: var(--popup-btn-text); border: none; padding: 12px 20px; border-radius: 5px; cursor: pointer; font-weight: bold; text-align: center; width: 100%; transition: filter 0.2s ease; }
.action-btn:hover { filter: brightness(1.1); }
---

