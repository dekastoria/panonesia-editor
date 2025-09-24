# Panonesia Editor — Panduan Install (VS Code & aaPanel) 
**Edisi Zero‑Coding • Popup‑Only**  
_Ikuti langkah persis seperti di bawah. Tidak perlu paham pemrograman._

---

## A. Apa yang Akan Kamu Dapat
- **Dashboard Admin (Filament)** untuk membuat **Project**, **Scenes (nama saja)**, **Popups**, **Popup‑Menu**, upload **asset** (gambar/PDF), **preview realtime**, dan **Export ZIP**.
- **Output akhir**: satu folder rapi `Proyek-Tur-Virtual/` berisi `index.html + aset-custom/...` siap dipakai di 3DVista (via “Execute JavaScript” trigger atau langsung di-embed).

---

## B. Perangkat yang Perlu Diinstall (sekali saja)
1. **VS Code**  
2. **Git for Windows** (agar bisa clone project dan jalankan perintah)
3. **Node.js LTS** (cek: `node -v`, `npm -v`)
4. **Composer** (cek: `composer -V`)
5. **PHP 8.2+**  
   - **Windows termudah:** pakai **XAMPP** (aktifkan Apache & MySQL).  
     > Kamu **tidak wajib** pakai MySQL; kita sediakan mode **SQLite** (tanpa database server) untuk pemula.

> **Tips:** Buka **VS Code** → `Terminal` bawaan VS Code untuk semua perintah.

---

## C. Instalasi Lokal (Paling Mudah)
> Pilih **Mode 1 (SQLite)** jika ingin super cepat tanpa database server.  
> Pilih **Mode 2 (MySQL/XAMPP)** kalau kamu ingin sekaligus belajar database.

### 1) Ambil Kode Project
```bash
# buka folder kerja kamu dulu
# contoh: D:\proyekku  (pakai Explorer → klik kanan → Open in Terminal)
git clone https://github.com/dekastoria/panonesia-editor.git
cd panonesia-editor
```

### 2) Siapkan Environment
```bash
# salin file contoh konfigurasi
copy .env.example .env   # (Windows)
# atau: cp .env.example .env  # (macOS/Linux)
```

### 3) Pilih Salah Satu Mode Database
#### Mode 1 — **SQLite (tanpa server, paling simple)**
Edit file `.env`:
```
DB_CONNECTION=sqlite
DB_DATABASE=./database/database.sqlite
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USERNAME=
DB_PASSWORD=
```
Buat file databasenya:
```bash
mkdir database
type NUL > database\database.sqlite
```

#### Mode 2 — **MySQL (XAMPP)**
- Buka `http://localhost/phpmyadmin` → **Create Database**: `panonesia_db`
- Edit `.env`:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=panonesia_db
DB_USERNAME=root
DB_PASSWORD=    # (kosong jika default XAMPP)
```

### 4) Install Dependensi
```bash
composer install
npm install
php artisan key:generate
php artisan migrate
php artisan storage:link
```

### 5) Buat Akun Admin (Filament)
```bash
php artisan make:filament-user
# isi: name, email, password
```

### 6) Jalankan Aplikasi
```bash
# terminal 1
php artisan serve
# buka http://127.0.0.1:8000/admin → login

# terminal 2
npm run dev
# (untuk build produksi nanti gunakan: npm run build)
```

> **Siap!** Kamu sudah bisa membuat **Project → Scenes (nama saja) → Popups → Popup‑Menu**, upload asset, preview realtime, dan **Export ZIP**.

---

## D. Alur Pemakaian Singkat (Zero‑Coding)
1. **Login** → `/admin` (Filament).  
2. **Create Project** → jawab “Aktifkan Popup‑Menu global?” (Ya/Skip).  
3. **Scenes** → tambah daftar **kode scene** saja (contoh: `s01`, `s02`, `lobby`).  
4. **Popups** (per scene) → pilih template (image/text/pdf/youtube/custom) → isi form → **preview**.  
5. **Popup‑Menu** (project level atau per scene) → tambah **items** (About, Gallery, Video, PDF, Share, Contact, Custom).  
6. **Assets** → upload atau browse dari library proyek.  
7. **Export** → dapat satu folder `Proyek-Tur-Virtual/` + **Dokumen Trigger JS** (ID & snippet 1‑baris untuk 3DVista).

---

## E. Struktur Output (Hasil Export)
```
Proyek-Tur-Virtual/
├─ index.html
└─ aset-custom/
   ├─ css/style.css
   ├─ js/data.js      # window.PANONESIA_CONFIG = {...}
   ├─ js/script.js    # runtime + listener + gallery ringan
   ├─ gambar/...
   └─ dokumen/...
```
**CSP** (di `<head>` `index.html`):  
`default-src 'self'; img-src 'self' data: https:; script-src 'self'; style-src 'self' 'unsafe-inline'; frame-src https://www.youtube.com https://player.vimeo.com;`

---

## F. Deploy ke **aaPanel** (VPS) — Cara Termudah
> Asumsi: kamu sudah punya domain aktif di aaPanel.

1) **Buat Site**  
- aaPanel → **Website** → **Add Site**  
- Domain: `editor.domainku.com` (contoh)  
- PHP: **8.2** (atau lebih)  
- Buat **Database** (opsional kalau pakai MySQL)

2) **Upload Kode**  
- Opsi A (Paling mudah): **Upload ZIP** project ke `/www/wwwroot/editor.domainku.com`, lalu Extract.  
- Opsi B: **Git** (aaPanel Git manager) → clone repository.  

3) **Set Document Root** ke **`/public`**  
- Website → **Site Config** → **Document Root** → pilih folder **public**.

4) **Install Dependensi di Server**  
aaPanel → **Terminal/SSH**:
```bash
cd /www/wwwroot/editor.domainku.com
composer install --no-dev --optimize-autoloader
cp .env.example .env
php artisan key:generate
php artisan storage:link
# Pilih DB: SQLite (paling mudah) atau MySQL (isi kredensial .env)
php artisan migrate --force
```
**Mode tanpa Node di server (disarankan):**  
- Di laptop: `npm run build` → akan membuat folder `public/build`.  
- Upload isi `public/build` ke server (timpah jika ada).  
> (Kalau server kamu sudah ada Node, kamu boleh `npm ci && npm run build` langsung di VPS.)

5) **Permission**  
```bash
chown -R www:www /www/wwwroot/editor.domainku.com
chmod -R 775 storage bootstrap/cache
```

6) **Konfigurasi `.env` untuk Produksi**  
```
APP_ENV=production
APP_DEBUG=false
APP_URL=https://editor.domainku.com
# Pilih salah satu:
# SQLite
DB_CONNECTION=sqlite
DB_DATABASE=/www/wwwroot/editor.domainku.com/database/database.sqlite
# MySQL contoh
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=panonesia_db
# DB_USERNAME=namauser
# DB_PASSWORD=xxxx
```
Buat file SQLite (jika pakai SQLite):
```bash
mkdir -p database
touch database/database.sqlite
php artisan migrate --force
```

7) **Buat User Admin Filament**  
```bash
php artisan make:filament-user
```

8) **Akses**  
- **Admin**: `https://editor.domainku.com/admin`  
- Login pakai user yang dibuat tadi.  

> **Selesai!** Semua fitur dashboard, preview, dan export sudah jalan di VPS.

---

## G. Trigger untuk 3DVista (Contoh Cepat)
**Format ID**: `pop-<scene>-<type>-<nnn>` (mis. `pop-s01-img-001`).  
**One‑liner SHOW:**
```js
(function(){var M={proto:"panonesia/1",type:"trigger",action:"show",id:"pop-s01-img-001"};try{window.postMessage(M,"*");window.parent&&window.parent.postMessage(M,"*");new BroadcastChannel("panonesia").postMessage(M);}catch(e){}})();
```
**One‑liner HIDE:**
```js
(function(){var M={proto:"panonesia/1",type:"trigger",action:"hide",id:"pop-s01-img-001"};try{window.postMessage(M,"*");window.parent&&window.parent.postMessage(M,"*");new BroadcastChannel("panonesia").postMessage(M);}catch(e){}})();
```

> Dokumen **Trigger JS** lengkap akan ikut di file export.

---

## H. Troubleshooting Singkat
- **Halaman kosong / error 500 (VPS)** → cek `.env`, `APP_KEY` (jalankan `php artisan key:generate`), `APP_DEBUG=true` sementara lalu cek **Logs** aaPanel.  
- **Tidak bisa upload file** → jalankan `php artisan storage:link`, pastikan permission `storage` & `bootstrap/cache`.  
- **Login /admin gagal** → ulangi `php artisan make:filament-user`.  
- **Preview tidak muncul** di lokal → pastikan `npm run dev` aktif di terminal.  
- **Migrasi gagal (MySQL)** → cek kredensial `.env` & buat database-nya di phpMyAdmin.

---

## I. Checklist Selesai
- [ ] Bisa login `/admin`  
- [ ] Buat Project + (opsional) Popup‑Menu global  
- [ ] Tambah Scenes (nama/kode saja)  
- [ ] Tambah Popups + Preview jalan  
- [ ] Upload/Browse assets berfungsi  
- [ ] Export ZIP → struktur folder rapi & **Dokumen Trigger JS** ikut  
- [ ] (VPS) Site hidup di domain kamu, `/public` jadi document root

---

**Catatan**: Panduan ini sudah selaras dengan dokumen **implementasi-dashboard.md** dan **alur-produk.md** (popup‑only, schema‑driven, postMessage realtime, export 1 folder).  
Kalau butuh, kamu bisa minta “versi PDF” dari panduan ini untuk dibagikan ke tim.
