# Smart Sheet AI

Lembar kerja (spreadsheet) berbahasa Indonesia dengan **Rumus Satu Klik**: pilih data, tekan tombol, lihat pratinjau hasil, lalu terapkan. Tanpa perlu menghafal atau mengetik rumus. Berjalan sepenuhnya di peramban, menyimpan data di perangkat, dan tetap bisa dipakai tanpa internet (PWA).

Rancangan acuan: *SMART SHEET AI v1.0* (26 Oktober 2026).

## Fitur utama

- Buku kerja dengan banyak sheet, grid virtual (mulus pada puluhan ribu baris), Undo/Redo, salin/tempel, isi otomatis, bekukan baris, filter, urut, validasi data, format bersyarat.
- **Rumus Satu Klik** (±55 operasi dalam 9 kelompok): Dasar, Statistik, Persentase, Logika, Pencarian, Kondisional, Teks, Tanggal, Keuangan, serta alat Data (urut, duplikat, sel kosong, pembersihan, rekap). Setiap operasi menampilkan pratinjau dan rumus hidup; satu kali Terapkan = satu langkah Undo.
- Mesin rumus sendiri dengan alias fungsi Indonesia (JUMLAH, JIKA, CARIV, …) dan penanganan sel kosong/teks yang aman.
- Impor dan ekspor **XLSX** serta **CSV** (pratinjau, deteksi tipe kolom, laporan masalah); ekspor mempertahankan rumus, format angka/tanggal, lebar kolom, bekuan, filter.
- **Dashboard**: kartu KPI dan grafik (batang, garis, lingkaran, area, sebar) dengan filter tanggal/kategori; simpan sebagai PNG atau cetak.
- **Dokumen**: kuitansi (terbilang otomatis), faktur, laporan tabel, surat keterangan, daftar hadir; A4/F4, nomor dokumen berurutan, cetak atau simpan PDF dari peramban.
- 15 **templat** (penjualan, buku kas, stok, nilai siswa, anggaran RT/desa, dan lain-lain).
- **Asisten** perintah bahasa Indonesia ("jumlahkan kolom Total", "rekap Total per bulan", …). Berjalan lokal berbasis aturan; selalu menampilkan rencana dan menunggu persetujuan.
- **Backup/pulihkan** JSON (opsional dienkripsi AES-256-GCM, PBKDF2 250.000 iterasi), snapshot otomatis, deteksi konflik bila dibuka di dua tab.
- Mode terang/gelap, navigasi papan ketik, label ARIA, tampilan ponsel.

## Menjalankan

Tidak ada tahap build.

```bash
npm run serve        # http://localhost:8080
npm test             # uji unit mesin rumus, XLSX, wizard, templat, backup
npm run test:ui      # uji penerimaan di Chromium (butuh paket playwright)
npm run perf         # uji performa 1.000 / 10.000 / 100.000 baris
```

Buka lewat `http://localhost`, bukan `file://`, agar service worker dan IndexedDB aktif.

## Publikasi ke GitHub Pages

1. Dorong repositori ini ke GitHub (cabang `main`).
2. Di **Settings → Pages**, pilih **Source: GitHub Actions**.
3. Workflow `.github/workflows/deploy.yml` menjalankan `npm test`, lalu menerbitkan folder `web/`.
4. Semua alamat berkas bersifat relatif, sehingga berfungsi di `https://<akun>.github.io/<repo>/`.

Saat versi baru terbit, pengguna melihat pemberitahuan "Versi baru tersedia"; halaman dimuat ulang setelah mereka menyetujui, dan perubahan disimpan lebih dulu.

## Struktur

```
web/
  index.html, manifest.webmanifest, sw.js, icons/
  css/app.css
  vendor/xlsx.mini.min.js      SheetJS (Apache-2.0), hanya untuk membaca XLSX
  js/
    util.js engine.js model.js      mesin: alamat, rumus, data, Undo
    wizards.js wizardui.js          Rumus Satu Klik
    xlsxio.js backup.js store.js    berkas, backup, IndexedDB
    grid.js sheetview.js tools.js   tampilan lembar
    charts.js dashboard.js docs.js docsui.js templates.js assistant.js assistantui.js
    persist.js state.js ui.js home.js settings.js importui.js app.js
tests/   uji unit (node:test), uji UI dan performa (Playwright)
docs/    peta jalan dan batasan
```

Mesin rumus (`engine.js`, `model.js`) tidak bergantung pada DOM sehingga dapat diuji terpisah.

## Privasi dan keamanan

- Data buku kerja hanya disimpan di IndexedDB peramban; tidak ada server dan tidak ada pelacakan.
- Repositori **tidak memuat kunci API atau rahasia apa pun**. Asisten bawaan tidak mengirim data keluar.
- Dokumen hasil cetak ditampilkan dalam iframe ber-sandbox; teks dari sel selalu di-escape.
- Backup tanpa passphrase berisi data apa adanya; simpan di tempat yang aman. Passphrase yang hilang tidak dapat dipulihkan.
- Data peramban bisa hilang bila pengguna membersihkan data situs atau penyimpanan penuh. Unduh backup secara berkala. Aplikasi meminta penyimpanan persisten dan memberi peringatan bila penyimpanan gagal.

## Hasil pengujian (lingkungan pengembangan, Chromium)

- `npm test`: 37 uji unit lulus (mesin rumus, XLSX, wizard, templat, backup).
- `npm run test:ui`: F-01 sampai F-06 dan F-08 sampai F-10 lulus otomatis; F-07 diuji sebagian (lihat bawah).
- F-07: berkas hasil ekspor dibuka ulang dengan openpyxl dan LibreOffice; rumus, format tanggal/Rupiah, dan nilai terhitung sesuai. Belum diuji di Microsoft Excel.
- Performa (`npm run perf`, 6 kolom, waktu dalam milidetik):

| Baris | Terapkan Kali | Urutkan | Simpan | Muat | 20× render gulir |
|---|---|---|---|---|---|
| 1.000 | 5 | 30 | 27 | 6 | 100 |
| 10.000 | 21 | 185 | 121 | 87 | 83 |
| 100.000 | 467 | 2.242 | 2.119 | 800 | 100 |

Pada 100.000 baris operasi "Jumlah" kolom memakan ±1,9 detik dan berkas JSON ±15 MB; masih bisa dipakai, tetapi ini titik yang perlu dioptimalkan. Belum diuji: Firefox/Safari, perangkat ponsel sungguhan, pembaca layar, dan penyimpanan penuh.

## Lisensi

MIT. Pustaka SheetJS Community Edition dilisensikan Apache-2.0 (lihat `web/vendor/xlsx.LICENSE.txt`).
