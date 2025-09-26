# Panduan Final: Integrasi Popup Modular Preline UI ke 3DVista

Dokumen ini merangkum seluruh proses, kendala, dan solusi untuk mengintegrasikan sistem popup dinamis ke dalam tur virtual 3DVista menggunakan Tailwind CSS dan Preline UI.

---
## **1. Penyiapan Awal Proyek**

1.  **Buat Folder Proyek:** Buat folder utama untuk semua file proyek Anda.
    * Contoh: `D:\3dvista-projects\dashboard\`

2.  **Inisialisasi NPM:** Buka terminal di dalam folder tersebut dan jalankan `npm init -y`.

3.  **Buat Struktur Folder:** Buat dua sub-folder di dalam direktori utama:
    * `src/`: Untuk file sumber CSS.
    * `inject/`: Untuk semua file popup HTML dan "mesin" JavaScript (`main.js`).

---
## **2. Instalasi & Penanganan Error Umum**

Tahap ini mencakup instalasi semua *package* yang dibutuhkan dan solusi untuk error yang sering terjadi.

1.  **Install Dependensi:** Jalankan perintah ini untuk menginstall Tailwind, PostCSS, Autoprefixer, dan Preline.
    ```bash
    npm install -D tailwindcss postcss autoprefixer preline
    ```

2.  **Kendala & Solusi yang Sering Terjadi:**
    * **Kendala:** Menjalankan `npx tailwindcss init` sebelum instalasi akan menyebabkan error `npm error could not determine executable to run`.
        * **Solusi:** Pastikan selalu menjalankan `npm install` **sebelum** menjalankan perintah `npx` yang terkait.
    * **Kendala:** Error `Error: Cannot find module 'preline/plugin'`.
        * **Solusi:** Preline UI v2.0 ke atas tidak lagi menggunakan plugin Tailwind. Hapus `require('preline/plugin')` dari `tailwind.config.js` dan tambahkan path JS Preline ke array `content`.
    * **Kendala:** Styling popup tidak muncul.
        * **Solusi:** Pastikan path ke folder `inject` (`./inject/**/*.{html,js}`) terdaftar di `tailwind.config.js` dan gunakan file `.html` terpisah untuk setiap popup.
    * **Kendala:** Error `404 Not Found` saat popup dipanggil.
        * **Solusi:** Pastikan seluruh folder proyek berada di dalam folder root server lokal (misalnya `C:\xampp\htdocs\`) dan akses melalui URL `http://localhost/nama_folder_proyek/`. Periksa juga salah ketik pada nama file/folder.

---
## **3. Konfigurasi Proyek**

Setelah instalasi berhasil, konfigurasikan proyek Anda.

1.  **Inisialisasi Tailwind:**
    ```bash
    npx tailwindcss init -p
    ```

2.  **Konfigurasi `tailwind.config.js`:**
    ```javascript
    /** @type {import('tailwindcss').Config} */
    module.exports = {
      content: [
        './*.{html,js}',
        './inject/**/*.{html,js}',
        './node_modules/preline/dist/*.js',
      ],
      theme: {
        extend: {},
      },
      plugins: [],
    }
    ```

3.  **Konfigurasi `package.json`:** Tambahkan skrip `dev` dan `build`.
    ```json
    "scripts": {
      "dev": "tailwindcss -i ./src/input.css -o ./dist/output.css --watch",
      "build": "tailwindcss -i ./src/input.css -o ./dist/output.css"
    },
    ```

4.  **Buat `src/input.css`:**
    ```css
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    ```

---
## **4. Penyiapan "Mesin" Inject (`main.js`)**

Ini adalah file inti yang mengontrol semua logika popup. Buat file ini di `inject/main.js`.

```javascript
// File: inject/main.js (Versi Final - Modular + Efek Fade)
document.addEventListener('DOMContentLoaded', function() {
  const injectHTML = '<div id="inject-container"></div>';
  document.body.insertAdjacentHTML('beforeend', injectHTML);

  const INJECT_CONTAINER = document.getElementById('inject-container');

  // === PUSAT KONFIGURASI POPUP (PANEL PENGATURAN ANDA) ===
  const popupRegistry = {
    'inject/pop-pano1-profile-001.html': {
      title: 'Profil Pengguna',
      styles: { width: '600px', height: 'auto', align: 'flex-end', justify: 'center', padding: '2rem', radius: '1rem' }
    },
    'inject/pop-pano1-tabs-002.html': {
      title: 'Fitur dengan Tabs',
      // Tidak ada 'styles' berarti akan menggunakan defaultStyles (besar & di tengah)
    }
  };

  const defaultStyles = {
    width: '1100px', height: '90vh', align: 'center', justify: 'center', padding: '2rem', radius: '1rem'
  };
  
  // === BAGIAN MESIN (ENGINE) - JANGAN DIUBAH DI BAWAH INI ===
  async function openInject(pathToFile) {
    if (!pathToFile) {
      console.error("Path ke file HTML popup tidak boleh kosong.");
      return;
    }
    const config = popupRegistry[pathToFile];
    const styles = config ? config.styles : defaultStyles;
    const title = config ? config.title : 'Popup';
    const injectCSS = `<style id="inject-style">#inject-container{position:fixed;inset:0;z-index:9999;display:flex;align-items:${styles.align};justify-content:${styles.justify};padding:${styles.padding};opacity:0;visibility:hidden;transition:visibility 0s .3s,opacity .3s ease}#inject-container.show{opacity:1;visibility:visible;transition:opacity .3s ease}#inject-overlay{position:absolute;inset:0;background-color:rgba(0,0,0,.6);backdrop-filter:blur(4px)}#inject-wrapper{position:relative;width:100%;height:${styles.height==='auto'?'auto':'100%'};max-width:${styles.width};max-height:${styles.height==='auto'?'90vh':styles.height};display:flex;transform:scale(.95);transition:transform .3s ease}#inject-container.show #inject-wrapper{transform:scale(1)}#inject-iframe{border:none;background-color:#fff;border-radius:${styles.radius};box-shadow:0 10px 30px rgba(0,0,0,.2);width:100%;height:100%;flex-grow:1}#inject-close-btn{position:absolute;top:-12px;right:-12px;z-index:10000;cursor:pointer;background-color:#374151;color:#fff;border:2px solid #fff;border-radius:9999px;width:32px;height:32px;font-size:16px;font-weight:700;display:flex;align-items:center;justify-content:center;transition:all .2s ease}#inject-close-btn:hover{transform:scale(1.1);background-color:#1f2937}</style>`;
    INJECT_CONTAINER.innerHTML = `${injectCSS}<div id="inject-overlay"></div><div id="inject-wrapper"><button id="inject-close-btn" onclick="closeInject()">X</button><iframe id="inject-iframe" title="${title}" src="${pathToFile}?v=${new Date().getTime()}"></iframe></div>`;
    setTimeout(() => { INJECT_CONTAINER.classList.add('show'); }, 20);
    document.getElementById('inject-overlay').addEventListener('click', closeInject);
  }
  function closeInject() {
    INJECT_CONTAINER.classList.remove('show');
  }
  window.openInject = openInject;
  window.closeInject = closeInject;
});
```

---
## **5. Penyiapan Modul Popup (Contoh Lengkap)**

1.  **Kaidah Penamaan:** `pop-<pano_number>-<type>-<nnn>.html`.

2.  **Contoh File Popup (`inject/pop-pano1-tabs-002.html`):**
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Features with Tabs</title>
      <link rel="stylesheet" href="../dist/output.css">
    </head>
    <body class="bg-transparent">
    
      <div class="max-w-[85rem] px-4 py-10 sm:px-6 lg:px-8 lg:py-14 mx-auto">
        <div class="relative p-6 md:p-16">
          <div class="relative z-10 lg:grid lg:grid-cols-12 lg:gap-16 lg:items-center">
            <div class="mb-10 lg:mb-0 lg:col-span-6 lg:col-start-8 lg:order-2">
              <h2 class="text-2xl text-gray-800 font-bold sm:text-3xl dark:text-neutral-200">
                Fully customizable rules to match your unique needs
              </h2>
              <nav class="grid gap-4 mt-5 md:mt-10" aria-label="Tabs" role="tablist" aria-orientation="vertical">
                <button type="button" class="hs-tab-active:bg-white hs-tab-active:shadow-md hs-tab-active:hover:border-transparent text-start hover:bg-gray-200 p-4 md:p-5 rounded-xl dark:hs-tab-active:bg-neutral-700 dark:hover:bg-neutral-700 active" id="tabs-with-card-item-1" data-hs-tab="#tabs-with-card-1" aria-controls="tabs-with-card-1" role="tab">
                  <span class="flex gap-x-6">
                    <svg class="shrink-0 mt-2 size-6 md:size-7 hs-tab-active:text-blue-600 text-gray-800 dark:hs-tab-active:text-blue-500 dark:text-neutral-200" xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 5.5A3.5 3.5 0 0 1 8.5 2H12v7H8.5A3.5 3.5 0 0 1 5 5.5z"/><path d="M12 2h3.5a3.5 3.5 0 1 1 0 7H12V2z"/><path d="M12 12.5a3.5 3.5 0 1 1 7 0 3.5 3.5 0 1 1-7 0z"/><path d="M5 19.5A3.5 3.5 0 0 1 8.5 16H12v3.5a3.5 3.5 0 1 1-7 0z"/><path d="M5 12.5A3.5 3.5 0 0 1 8.5 9H12v7H8.5A3.5 3.5 0 0 1 5 12.5z"/></svg>
                    <span class="grow">
                      <span class="block text-lg font-semibold hs-tab-active:text-blue-600 text-gray-800 dark:hs-tab-active:text-blue-500 dark:text-neutral-200">Advanced tools</span>
                      <span class="block mt-1 text-gray-800 dark:hs-tab-active:text-gray-200 dark:text-neutral-200">Use Preline thoroughly thought and automated libraries to manage your businesses.</span>
                    </span>
                  </span>
                </button>
                <button type="button" class="hs-tab-active:bg-white hs-tab-active:shadow-md hs-tab-active:hover:border-transparent text-start hover:bg-gray-200 p-4 md:p-5 rounded-xl dark:hs-tab-active:bg-neutral-700 dark:hover:bg-neutral-700" id="tabs-with-card-item-2" data-hs-tab="#tabs-with-card-2" aria-controls="tabs-with-card-2" role="tab">
                  <span class="flex gap-x-6">
                    <svg class="shrink-0 mt-2 size-6 md:size-7 hs-tab-active:text-blue-600 text-gray-800 dark:hs-tab-active:text-blue-500 dark:text-neutral-200" xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m12 14 4-4"/><path d="M3.34 19a10 10 0 1 1 17.32 0"/></svg>
                    <span class="grow">
                      <span class="block text-lg font-semibold hs-tab-active:text-blue-600 text-gray-800 dark:hs-tab-active:text-blue-500 dark:text-neutral-200">Smart dashboards</span>
                      <span class="block mt-1 text-gray-800 dark:hs-tab-active:text-gray-200 dark:text-neutral-200">Quickly Preline sample components, copy-paste codes, and start right off.</span>
                    </span>
                  </span>
                </button>
                <button type="button" class="hs-tab-active:bg-white hs-tab-active:shadow-md hs-tab-active:hover:border-transparent text-start hover:bg-gray-200 p-4 md:p-5 rounded-xl dark:hs-tab-active:bg-neutral-700 dark:hover:bg-neutral-700" id="tabs-with-card-item-3" data-hs-tab="#tabs-with-card-3" aria-controls="tabs-with-card-3" role="tab">
                  <span class="flex gap-x-6">
                    <svg class="shrink-0 mt-2 size-6 md:size-7 hs-tab-active:text-blue-600 text-gray-800 dark:hs-tab-active:text-blue-500 dark:text-neutral-200" xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275L12 3Z"/><path d="M5 3v4"/><path d="M19 17v4"/><path d="M3 5h4"/><path d="M17 19h4"/></svg>
                    <span class="grow">
                      <span class="block text-lg font-semibold hs-tab-active:text-blue-600 text-gray-800 dark:hs-tab-active:text-blue-500 dark:text-neutral-200">Powerful features</span>
                      <span class="block mt-1 text-gray-800 dark:hs-tab-active:text-gray-200 dark:text-neutral-200">Reduce time and effort on building modern look design with Preline only.</span>
                    </span>
                  </span>
                </button>
              </nav>
            </div>
            <div class="lg:col-span-6">
              <div class="relative">
                <div>
                  <div id="tabs-with-card-1" role="tabpanel" aria-labelledby="tabs-with-card-item-1">
                    <img class="shadow-xl shadow-gray-200 rounded-xl dark:shadow-gray-900/20" src="[https://images.unsplash.com/photo-1605629921711-2f6b00c6bbf4?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=560&h=720&q=80](https://images.unsplash.com/photo-1605629921711-2f6b00c6bbf4?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=560&h=720&q=80)" alt="Features Image">
                  </div>
                  <div id="tabs-with-card-2" class="hidden" role="tabpanel" aria-labelledby="tabs-with-card-item-2">
                    <img class="shadow-xl shadow-gray-200 rounded-xl dark:shadow-gray-900/20" src="[https://images.unsplash.com/photo-1665686306574-1ace09918530?ixlib=rb-4.0.3&ixid=MnwxMjA3fDF8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=560&h=720&q=80](https://images.unsplash.com/photo-1665686306574-1ace09918530?ixlib=rb-4.0.3&ixid=MnwxMjA3fDF8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=560&h=720&q=80)" alt="Features Image">
                  </div>
                  <div id="tabs-with-card-3" class="hidden" role="tabpanel" aria-labelledby="tabs-with-card-item-3">
                    <img class="shadow-xl shadow-gray-200 rounded-xl dark:shadow-gray-900/20" src="[https://images.unsplash.com/photo-1598929213452-52d72f63e307?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=560&h=720&q=80](https://images.unsplash.com/photo-1598929213452-52d72f63e307?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=560&h=720&q=80)" alt="Features Image">
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="absolute inset-0 grid grid-cols-12 size-full">
            <div class="col-span-full lg:col-span-7 lg:col-start-6 bg-gray-100 w-full h-5/6 rounded-xl sm:h-3/4 lg:h-full dark:bg-neutral-800"></div>
          </div>
        </div>
      </div>
      <script src="../node_modules/preline/dist/preline.js"></script>
    </body>
    </html>
    ```

---
## **6. Integrasi Final & Alur Kerja**

1.  **Edit `index.htm` dari 3DVista:** Tambahkan satu baris kode ini sebelum `</body>`.
    ```html
    <script src="inject/main.js"></script>
    ```

2.  **Trigger dari 3DVista:** Gunakan *action* "Run JavaScript" pada hotspot dengan perintah yang sesuai.
    ```javascript
    // Contoh memanggil popup baru
    openInject('inject/pop-pano1-tabs-002.html');
    ```

3.  **Alur Kerja:**
    * Jalankan `npm run dev` di terminal.
    * Jalankan server lokal (XAMPP).
    * Edit file di folder `inject/` atau `main.js`.
    * Refresh browser untuk melihat hasil.

4.  **Siap Tayang:**
    * Hentikan `npm run dev` (`Ctrl + C`).
    * Jalankan `npm run build` satu kali.
    * Upload seluruh folder proyek ke server hosting Anda.
