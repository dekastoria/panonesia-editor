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

Proyek-Tur-Virtual/
├── index.html
└── aset-custom/
    ├── css/
    │   └── style.css
    ├── js/
    │   └── script.js
    ├── gambar/
    │   ├── latar-360.jpg
    │   └── logo.png
    └── dokumen/
        └── brosur.pdf

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
```

### File: addon-kustom/addon-loader.js
```
// addon-kustom/addon-loader.js (Final v1.1)

// Objek global untuk menampung semua data pop-up dari file modular
window.semuaKontenPopup = {};
let currentSlideIndex = 0;
let totalSlides = 0;

function inisialisasiAddon() {
    if (window.addonSudahAktif) return;
    const head = document.head || document.getElementsByTagName('head')[0];
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.type = 'text/css';
    link.href = 'addon-kustom/addon-style.css';
    head.appendChild(link);
    window.addonSudahAktif = true;
    console.log('✅ Panonesia Editor v1.1 Addon berhasil diaktifkan!');
}

function tampilkanPopup(idKonten) {
    const data = window.semuaKontenPopup[idKonten];
    if (!data) { console.error(`Konten untuk ID '${idKonten}' tidak ditemukan!`); return; }
    
    const popupLama = document.getElementById('popup-kustom-wrapper');
    if (popupLama) popupLama.remove();

    let htmlPopup = '';
    const tipe = data.tipe || 'info'; // Default ke 'info' jika tipe tidak ada
    switch (tipe) {
        case 'galeri': htmlPopup = buatPopupGaleri(data); break;
        default: htmlPopup = buatPopupInfo(data); break; // Semua variasi info
    }
    
    document.body.insertAdjacentHTML('beforeend', htmlPopup);
    const wrapper = document.getElementById('popup-kustom-wrapper');
    setTimeout(() => { wrapper.classList.add('visible'); }, 10);
    if (window.tour) window.tour.pause();
}

function tutupPopup(event) {
    const popup = document.getElementById('popup-kustom-wrapper');
    if (popup) {
        popup.classList.remove('visible');
        setTimeout(() => { popup.remove(); }, 300);
    }
    if (window.tour) window.tour.resume();
}

function buatTombolAksi(actions) {
    if (!actions || actions.length === 0) return '';
    let buttonsHTML = '<div class="action-buttons-container">';
    actions.forEach(action => {
        buttonsHTML += `<button class="action-btn" onclick="${action.perintah}">${action.label}</button>`;
    });
    buttonsHTML += '</div>';
    return buttonsHTML;
}

function buatPopupInfo(data) {
    const posisiClass = data.posisi || 'tengah';
    const style = data.style || {};
    const inlineStyle = `
        --popup-bg: ${style.warnaLatar || ''}; --popup-text: ${style.warnaTeks || ''};
        --popup-title: ${style.warnaTeks || ''}; font-family: ${style.font || ''};
    `;
    return `
        <div class="popup-overlay posisi-${posisiClass}" id="popup-kustom-wrapper" onclick="tutupPopup(event)">
            <div class="popup-content info-window" style="${inlineStyle}" onclick="event.stopPropagation()">
                <button class="close-x-btn" onclick="tutupPopup(event)">&times;</button>
                ${data.gambar ? `<img src="${data.gambar}" alt="${data.judul || ''}" class="info-image">` : ''}
                <div class="konten-teks">
                    ${data.judul ? `<h2>${data.judul}</h2>` : ''}
                    ${data.deskripsi ? `<div class="deskripsi">${data.deskripsi}</div>` : ''}
                </div>
                ${buatTombolAksi(data.actions)}
            </div>
        </div>
    `;
}

function buatPopupGaleri(data) {
    const posisiClass = data.posisi || 'tengah';
    totalSlides = data.images.length; currentSlideIndex = 0;
    let imagesHTML = data.images.map(src => `<img src="${src}">`).join('');
    return `
        <div class="popup-overlay posisi-${posisiClass}" id="popup-kustom-wrapper" onclick="tutupPopup(event)">
            <div class="popup-content galeri-window" onclick="event.stopPropagation()">
                <button class="close-x-btn" onclick="tutupPopup(event)">&times;</button>
                ${data.judul ? `<h2>${data.judul}</h2>` : ''}
                <div class="gallery-container">
                    <div class="gallery-images" id="gallery-slider">${imagesHTML}</div>
                    ${totalSlides > 1 ? `<button class="gallery-nav prev" onclick="ubahSlide(-1)">&#10094;</button><button class="gallery-nav next" onclick="ubahSlide(1)">&#10095;</button>` : ''}
                </div>
                ${data.deskripsi ? `<div class="deskripsi">${data.deskripsi}</div>` : ''}
                ${buatTombolAksi(data.actions)}
            </div>
        </div>
    `;
}

function ubahSlide(direction) {
    currentSlideIndex = (currentSlideIndex + direction + totalSlides) % totalSlides;
    document.getElementById('gallery-slider').style.transform = `translateX(-${currentSlideIndex * 100}%)`;
}
```

Langkah 3: Isi File Konten
Ini adalah "pusat kendali" Anda. Buat file .js di dalam folder data-popups/ dan isi dengan definisi pop-up Anda.

### File: data-popups/konten.js (Contoh)

```
// data-popups/konten.js
// Ganti nama file ini sesuai kebutuhan (misal: lobby.js, pameran.js, dll.)

Object.assign(window.semuaKontenPopup, {

    // CONTOH 1: Pop-up judul saja, di posisi atas
    "infoJudul": {
        tipe: "info",
        posisi: "atas",
        judul: "Selamat Datang di Tur Virtual Kami"
    },

    // CONTOH 2: Pop-up deskripsi saja, di posisi bawah
    "infoNavigasi": {
        tipe: "info",
        posisi: "bawah",
        deskripsi: "<p>Gunakan hotspot navigasi untuk menjelajahi setiap ruangan. Klik pada objek untuk mendapatkan informasi detail.</p>"
    },

    // CONTOH 3: Pop-up info lengkap dengan gambar dan tombol aksi
    "infoObjekPenting": {
        tipe: "info",
        posisi: "kanan",
        judul: "Arca Ganesha",
        gambar: "media/gambar1.jpg", // Ganti dengan path gambar Anda
        deskripsi: "Arca ini berasal dari abad ke-8 dan merupakan salah satu koleksi tertua kami.",
        actions: [
            {
                label: "Pindah ke Ruang Arca",
                perintah: "window.tour.setMainMediaByName('Ruang Arca'); tutupPopup(event);" // Ganti 'Ruang Arca' dengan nama panorama tujuan Anda
            }
        ]
    },

    // CONTOH 4: Pop-up galeri dengan kustomisasi style
    "galeriRuangan": {
        tipe: "galeri",
        posisi: "tengah",
        judul: "Galeri Ruang Utama",
        images: ["media/gambar1.jpg", "media/gambar2.jpg"], // Ganti dengan path gambar Anda
        style: {
            warnaLatar: "#1a2530",
            warnaTeks: "#e0e0e0",
            font: "'Montserrat', sans-serif"
        }
    }
});
```

Langkah 4 & 5: Hubungkan ke 3DVista
### Tautkan Skrip di index.htm
Tambahkan baris-baris berikut di file index.htm Anda, tepat sebelum tag </body> penutup.
```
<script src="addon-kustom/addon-loader.js"></script>

    <script src="data-popups/konten.js"></script>
    ```

### ### Atur Aksi di Dalam 3DVista

1.  **Aksi `onTourStart`** (Di level `Player` -> `Actions`):
    ```javascript
    inisialisasiAddon();
    ```
2.  **Aksi `On Click`** (Di `Hotspot` mana pun):
    ```javascript
    tampilkanPopup('infoObjekPenting');
    ```

Selesai. Dengan format ini, Anda hanya perlu menyalin dan menempelkan kode ke file yang benar.
```

