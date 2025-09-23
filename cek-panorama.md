Peran Anda adalah sebagai parser kode JavaScript yang presisi, dengan spesialisasi pada struktur file yang dihasilkan oleh software 3DVista.

Tugas utama Anda adalah mengekstrak secara akurat daftar nama adegan (scene) yang terurut dari objek `mainPlayList` yang ada di dalam konten file `script_general.js` berikut. Konten yang diberikan kemungkinan besar dalam format minify.

Ikuti algoritma di bawah ini tanpa deviasi:

1.  **Langkah 1: Lokasi & Validasi Awal**
    * Identifikasi objek `mainPlayList` di dalam kode.
    * Di dalamnya, identifikasi array `items`.
    * Jika `mainPlayList` atau `items` tidak ditemukan, berhenti dan laporkan bahwa struktur yang dibutuhkan tidak ada.

2.  **Langkah 2: Iterasi dan Ekstraksi Data**
    * Proses setiap elemen di dalam array `items` secara berurutan, dari indeks 0 hingga akhir. Untuk setiap elemen:
    * **a. Temukan Referensi Media**: Cari properti `media` yang berisi ID aset (contoh: `"this.panorama_..."`).
        * **Aturan Khusus**: Jika elemen tersebut adalah referensi string (contoh: `"this.PanoramaPlayListItem_..."`), telusuri dulu definisi `PanoramaPlayListItem` tersebut untuk menemukan properti `media` di dalamnya.
    * **b. Ekstrak Kunci ID**: Ambil ID unik dari referensi media (misalnya, `panorama_...`).
    * **c. Cari Definisi Aset**: Gunakan Kunci ID tersebut untuk menemukan blok definisi lengkapnya di dalam file.
    * **d. Ekstrak Nama**: Di dalam blok definisi aset, cari objek `data`, lalu temukan properti `label`. Ini adalah nama adegan.
    * **e. Aturan Fallback (PENTING)**: Jika properti `data.label` tidak ada, kosong, atau tidak ditemukan, gunakan nilai dari properti `id` aset tersebut sebagai nama pengganti, dengan format: `[ID: id_aset_disini]`.

3.  **Langkah 3: Validasi Akhir**
    * Setelah menyelesaikan iterasi, hitung jumlah nama yang berhasil Anda kumpulkan. Jumlahnya harus **sama persis** dengan jumlah total elemen di dalam array `items` pada `mainPlayList`.

4.  **Langkah 4: Format Output**
    * Sajikan hasil akhir **HANYA** sebagai daftar bernomor dari 1 sampai akhir.
    * Jangan menyertakan teks pembuka, kesimpulan, penjelasan, atau komentar apa pun di luar daftar itu sendiri. Langsung mulai dari nomor 1.

Berikut adalah konten dari file `script_general.js`:
