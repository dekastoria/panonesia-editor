# Panonesia Editor v1 🚀

**Sebuah aplikasi web *standalone* untuk membuat dan mengelola komponen UI (pop-up) kustom yang modern dan responsif untuk tur virtual 3DVista.**

![Contoh Tampilan Dashboard](https://i.ibb.co/L5Yw5W5/panonesia-editor-mockup.png)
*(Gambar di atas adalah mockup konseptual)*

---

## Latar Belakang & Masalah yang Diselesaikan

3DVista adalah *software* yang sangat powerful untuk membuat tur virtual. Namun, *Skin Editor* bawaannya memiliki beberapa keterbatasan:
1.  **Kesulitan Desain Responsif**: Membuat pop-up atau menu yang terlihat bagus di desktop dan mobile secara bersamaan sangat menantang dan memakan waktu.
2.  **Manajemen Skala Besar**: Untuk tur yang kompleks (misalnya museum) yang membutuhkan 50-90+ titik informasi, mengelola semua pop-up menjadi tidak efisien.
3.  **Kustomisasi Terbatas**: Kustomisasi tampilan (animasi, layout kompleks, styling modern) sulit dicapai tanpa intervensi manual pasca-*export*.

**Panonesia Editor v1** lahir untuk menyelesaikan masalah ini. Aplikasi ini adalah *tool* eksternal yang tidak menyentuh aplikasi 3DVista secara langsung, melainkan bekerja pada **hasil *export***-nya, memberikan kebebasan kustomisasi tanpa batas.

---

## Arsitektur & Alur Kerja

Panonesia Editor bekerja dengan memisahkan secara tegas antara **Aplikasi Editor**, **Paket Addon** yang dihasilkan, dan **Tur 3DVista** sebagai target implementasi.

**Alur Kerja Utama:**
1.  **Autentikasi**: Pengguna login ke *dashboard* menggunakan akun Google.
2.  **Manajemen Proyek**: Pengguna dapat membuat proyek baru atau mengelola proyek yang sudah tersimpan di databasenya.
3.  **Analisis & Desain**: Saat membuka proyek, pengguna meng-upload `script_general.js`. Aplikasi (via Gemini) menganalisisnya dan pengguna mulai mendesain pop-up di antarmuka tiga panel (Kontrol, Kanvas, Chat AI). Semua pekerjaan disimpan secara otomatis.
4.  **Generate Kode**: Aplikasi menghasilkan sebuah paket `.zip` yang berisi *framework addon* yang super ringan dan dioptimalkan.
5.  **Implementasi**: Pengguna mengekstrak paket addon ke dalam folder tur 3DVista, menautkannya di `index.htm`, dan memanggil pop-up dari *hotspot* dengan perintah JavaScript yang sederhana.

---

## Teknologi yang Digunakan (Tech Stack)

| Komponen | Teknologi | Keterangan |
| :--- | :--- | :--- |
| **Framework Aplikasi** | **Next.js (React)** | Pilihan ideal untuk aplikasi *full-stack* dengan kebutuhan *server-side rendering* dan API *routes*. |
| **Styling & UI Dashboard**| **Tailwind CSS** + **Shadcn/ui** | Mempercepat pembangunan UI *dashboard* yang modern, minimalis, dan sepenuhnya dapat dikustomisasi. |
| **Autentikasi** | **NextAuth.js** | Pustaka standar untuk mengimplementasikan login sosial (termasuk Google) dengan mudah dan aman. |
| **Database** | **PostgreSQL** (via **Prisma ORM**) | Pilihan yang sangat kuat dan skalabel. Prisma mempermudah interaksi antara kode dan database. |
| **Layanan AI** | **Google Gemini API** | Bertindak sebagai "asisten cerdas" di backend untuk analisis dan generasi konten/kode. |
| **Deployment** | **VPS (aaPanel)** | Fleksibel untuk men-deploy aplikasi Node.js dan manajemen database. |

---

## Implementasi Teknis

### Koneksi Database di aaPanel
1.  **Buat Database**: Gunakan fitur "Database" di aaPanel untuk membuat database PostgreSQL atau MySQL.
2.  **Simpan Kredensial**: Simpan informasi koneksi (Host, User, Password, Nama DB) sebagai **Environment Variables** di pengaturan "Node project" di aaPanel. **Jangan pernah** menulis kredensial ini langsung di dalam kode.
3.  **Koneksi dari Kode**: Di dalam backend Next.js, gunakan **Prisma** untuk mendefinisikan skema data (Pengguna, Proyek) dan terhubung ke database menggunakan *environment variables* tersebut.

### Koneksi dengan Gemini API
Koneksi ke Gemini API dilakukan secara eksklusif di sisi **backend** (Next.js API Routes) untuk keamanan. *API Key* disimpan sebagai *environment variable* di server dan tidak pernah diekspos ke *frontend*. Backend bertindak sebagai *proxy* yang aman antara pengguna dan layanan AI.

---

## Roadmap & Visi Masa Depan
- **v1 (Saat Ini)**: Fokus pada fungsionalitas inti *pop-up builder* dengan sistem login, manajemen proyek dasar, dan generasi paket addon yang modular.
- **v2**: Fitur kolaborasi tim, di mana satu proyek bisa diakses oleh beberapa pengguna.
- **v3**: Integrasi dengan layanan *cloud storage* (seperti Google Drive) untuk manajemen aset media langsung dari *dashboard*.
