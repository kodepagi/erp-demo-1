# ERP — Prototipe Demo (Dokumentasi Teknis)

Prototipe ERP yang bisa diklik untuk perusahaan air minum dalam kemasan (AMDK) dengan galon guna ulang.
Tidak ada backend. Semua data contoh dibuat di browser, dan aplikasi berjalan tanpa internet.

- **Panduan presentasi:** [README-FLOW.md](README-FLOW.md). Isi yang sama tampil di aplikasi pada `#/panduan`.
- **Demo daring:** https://kodepagi.github.io/erp-demo-1/

---

## Stack

| Bagian | Pilihan |
|---|---|
| Kerangka | Vue 3.5 + TypeScript 5.9 (`<script setup>`) |
| State | Pinia (UI) + `reactive`/`computed` Vue untuk data bersama |
| Routing | Vue Router, *hash history* (bisa dibuka dari `file://` dan hosting statis tanpa konfigurasi) |
| Build | Vite 8; mode `demo` memakai `vite-plugin-singlefile` untuk menghasilkan satu file HTML |
| Tes | Vitest |
| Markdown | `marked` untuk merender `README-FLOW.md` di halaman `#/panduan` |
| Font | Instrument Sans dan IBM Plex Mono, *self-hosted* (woff2) di `src/assets` |

Tidak ada pemanggilan jaringan saat aplikasi berjalan.

---

## Menjalankan

Butuh Node.js 20 atau lebih baru.

```bash
npm install
```
```bash
npm run dev
```

Buka `http://localhost:5173`. Server memakai `--host`, jadi bisa juga dibuka dari HP di jaringan yang sama.

| Perintah | Fungsi |
|---|---|
| `npm run dev` | Server pengembangan dengan hot reload |
| `npm test` | Tes unit (Vitest) |
| `npm run build` | Typecheck (`vue-tsc`) + build biasa ke `dist/` |
| `npm run build:demo` | Typecheck + build satu file ke `dist-demo/` |
| `npm run cek-netral` | Memindai `dist-demo/` dari nama merek sebelum dibagikan |
| `npm run preview` | Menyajikan hasil `dist/` secara lokal |

---

## Build demo dan distribusi

```bash
npm run build:demo
```

Menghasilkan dua file identik di `dist-demo/` (±0,6 MB, semua CSS, JS, dan font tertanam):

- `index.html`: untuk diunggah ke hosting statis (GitHub Pages, Netlify, Cloudflare Pages).
- `erp-demo.html`: untuk disalin lewat flashdisk atau email, lalu dibuka dengan klik dua kali.

`scripts/rename-demo.cjs` membuat salinan `erp-demo.html` setelah build.

### Pindai sebelum dibagikan

Isi aplikasi sengaja netral, tanpa nama merek calon klien. Sebelum mengunggah, pastikan bundel bersih:

```bash
npm run cek-netral
```

Daftar kata yang dicari ada di `scripts/cek-netral.cjs`. Perintah ini gagal (kode keluar 1) bila ada temuan.

### Memperbarui GitHub Pages

Repo `erp-demo-1` hanya berisi hasil build. Unggah ulang `index.html` dan `erp-demo.html` dari
`dist-demo/` lewat **Add file → Upload files**. Tambahkan `?v=2` (atau angka lain) di akhir tautan
supaya browser tidak memakai versi lama.

---

## Struktur

```
src/
  main.ts, App.vue          shell aplikasi (sidebar, bilah atas, area, toast, mode presenter)
  router/index.ts           semua rute
  stores/ui.ts              state UI: mode Excel, panel, presenter, sidebar, toast
  styles/                   variabel warna, font, utilitas global
  components/               komponen umum (sidebar, topbar, panel integrasi, kartu metrik, …)
  components/direksi/       komponen Dashboard Manajemen (biaya bocor, KPI, peta, keputusan, drill-down, asumsi)
  views/                    halaman
  data/                     seluruh data dan logika hitung (lihat di bawah)
README-FLOW.md              panduan presentasi (juga dirender di #/panduan)
scripts/rename-demo.cjs     pasca-build demo (salinan erp-demo.html)
scripts/cek-netral.cjs      pemindai nama merek di dist-demo/
```

### Rute

| Rute | Halaman |
|---|---|
| `/manajemen` | Dashboard Manajemen (halaman awal) |
| `/inventory/galon` | Galon tracker; menerima `?cabang=`, `?status=`, `?siklus=` |
| `/inventory/galon/:serial` | Detail dan riwayat satu galon |
| `/inventory/compliance` | Kepatuhan BPA 2031 dan simulasi penggantian |
| `/sales/antar-rumah` | Pelanggan langganan, jadwal hari ini, konfirmasi serah terima |
| `/sales/rute` | Rute driver dan rit |
| `/sales/pelanggan/:id` | Detail pelanggan, agen, atau outlet |
| `/modul/:slug` | 11 modul pendukung (finance, purchase, logistics, …) |
| `/langkah-berikutnya` | Fase pengembangan dan kriteria sukses pilot |
| `/panduan` | Panduan presentasi (tidak ada di sidebar) |

---

## Lapisan data (`src/data`)

Semua angka berasal dari satu sumber, sehingga setiap layar selalu konsisten.

| File | Isi |
|---|---|
| `constants.ts` | Nilai bawaan: galon beredar, susut, harga, deposit, skala outlet/armada/titik antar, sebaran siklus dan umur |
| `asumsi.ts` | Store reaktif `A` yang bisa diubah lewat panel Asumsi, beserta semua turunan (susut, piutang, kirim ulang). `skalakan()` membagi total ke beberapa butir dengan jumlah tetap persis |
| `area.ts` | 5 area (plant/hub), pilihan area global, porsi stok per area |
| `cabang.ts` | 15 cabang: bobot stok, indeks susut, ketepatan kirim, posisi di peta |
| `saringan.ts` | Penyaring area bersama untuk semua halaman |
| `seed.ts` | Data contoh: 24 pelanggan/agen/outlet, 13 rute, sampel galon berseri, pengiriman |
| `transaksi.ts` | Transaksi hidup "Konfirmasi serah terima" dan jurnal antarmodul |
| `cockpit.ts` | Biaya bocor, KPI, ringkasan pagi |
| `keputusan.ts` | 7 keputusan; `setujui()` benar-benar mengubah state |
| `telusur.ts` | Drill-down semua kartu Dashboard Manajemen (nasional → area → cabang → contoh → modul) |
| `bpa.ts` | Proyeksi kesiapan BPA 2031 |
| `modul.ts` | Isi 11 modul pendukung, mengikuti area |
| `dampak.ts` | Isi panel "Jejak integrasi" dan versi "Cara Excel" |
| `presenter.ts` | 7 langkah mode presenter dan catatannya |

Aturan yang dijaga tes:

- Sebaran (siklus, umur, susut per cabang, posisi galon) selalu berjumlah persis sama dengan totalnya,
  juga setelah asumsi diubah.
- Angka seluruh area berjumlah sama dengan angka nasional.
- Di drill-down, angka anak selalu berjumlah sama dengan angka induknya.
- Menyetujui keputusan menurunkan bocor aktif sebesar estimasi hematnya, dan hemat itu dibekukan.

Memuat ulang halaman mengembalikan semua state ke kondisi awal. Tidak ada `localStorage`.

---

## Mengubah angka dan tampilan

- **Angka bawaan:** `src/data/constants.ts` dan `BAWAAN` di `src/data/asumsi.ts`.
- **Asumsi hemat keputusan:** `src/data/keputusan.ts` (misalnya `PERSEN_HEMAT`, `PERSEN_HEMAT_AUDIT`).
- **Warna dan font:** variabel di `src/styles/main.css`.
- **Langkah presentasi:** teks catatan di `src/data/presenter.ts`; panduan lengkap di `README-FLOW.md`.

Jalankan `npm test` setelah mengubah angka. Beberapa tes memeriksa nilai bawaan secara eksplisit.

---

## Batasan

- Prototipe untuk demo, bukan produk produksi: tidak ada login, basis data, maupun API.
- Data pelanggan, driver, nomor seri, dan sebagian nama cabang adalah ilustrasi.
- Angka skala adalah estimasi dan asumsi, bukan data internal perusahaan mana pun.
