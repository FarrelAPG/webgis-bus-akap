# Jejak BBM Trayek 03 — Bus AKAP Cilegon–Jakarta

WebGIS untuk memvisualisasikan rute dan konsumsi bahan bakar dua keberangkatan bus AKAP (K-03) pada trayek Cilegon–Jakarta dalam satu hari operasi. Dibuat untuk Tugas Week 5 — CAAS GIS.

**Live demo:** _(isi link Vercel setelah deploy)_

## Konsep

Bus AKAP menempuh rute tetap jarak jauh dengan konsumsi Biosolar yang signifikan. Proyek ini membandingkan dua keberangkatan pada hari yang sama (siang vs malam) untuk melihat pengaruh waktu keberangkatan terhadap efisiensi bahan bakar, durasi tempuh, dan waktu idle.

## Sumber Data

Semua data berada di `/data` dan digunakan apa adanya dari materi tugas (data simulasi, bukan rekaman kendaraan sungguhan):

| File | Isi |
|---|---|
| `gps_mentah.csv` | Log GPS mentah per 30 detik (trip_id, waktu, koordinat, kecepatan, jarak antar-titik) — 2 trip, 892 baris |
| `rute.geojson` | Rute (LineString) tiap trip, sudah dilengkapi metrik agregat per-trip (jarak, liter, biaya, idle) |
| `titik_ujung.geojson` | Titik keberangkatan & tiba tiap trip |
| `ringkasan.json` | Ringkasan total operasi 1 hari (2 trip) |

## Preprocessing

`gps_mentah.csv` dianalisis untuk mendeteksi rentang waktu idle (kecepatan = 0 km/jam berturut-turut), digunakan untuk menyusun catatan idle per trip yang ditampilkan di panel detail. Rute dan titik digambar langsung dari `rute.geojson` dan `titik_ujung.geojson` tanpa proses snapping ke jaringan jalan, karena data GPS sudah berupa jejak padat (interval 30 detik) yang mengikuti jalur tol asli.

## Parameter Kendaraan

- **Jenis kendaraan**: Bus AKAP (Antar Kota Antar Provinsi)
- **Bahan bakar**: Biosolar, Rp6.800/liter
- **Efisiensi acuan**: 3,5 km/liter — dipilih karena bus besar jarak jauh dengan muatan penuh dan konsumsi BBM diesel jauh lebih boros dibanding kendaraan kecil; digunakan sebagai baseline untuk menghitung liter "boros" (selisih dari efisiensi aktual terhadap acuan)
- Kecepatan dan konsumsi aktual bervariasi antar-trip karena kondisi lalu lintas berbeda (siang vs malam)

## Pertanyaan yang Dijawab

1. **Berapa total jarak dan BBM yang terpakai dalam satu hari operasi?**  
   253,5 km, 79,4 liter Biosolar, Rp540.217 — terlihat di panel Ringkasan Operasi.
2. **Trip mana yang lebih efisien, siang atau malam?**  
   Trip 2 (malam, berangkat 20:06) lebih efisien — 3,31 km/liter dibanding Trip 1 (siang) 3,08 km/liter — terlihat di panel detail masing-masing trip dan popup rute di peta.
3. **Berapa banyak waktu dan BBM terbuang saat kendaraan berhenti (idle)?**  
   Trip 1 idle 4 kali (termasuk jeda 5 & 10 menit), Trip 2 idle 2 kali (termasuk jeda 30 menit) — dicatat di bagian "Catatan idle" tiap trip, dihitung dari `gps_mentah.csv`.

## Keterbatasan Data

- Data bersifat simulasi, bukan rekaman kendaraan sungguhan (sesuai catatan di `ringkasan.json`).
- Titik berhenti (kecepatan 0) diasumsikan idle/istirahat — data GPS tidak membedakan ini dari kemacetan.
- Catatan kemacetan per hari bersifat kualitatif/perkiraan umum, bukan hasil pengukuran atau deteksi dari data GPS.
- Rute mengikuti jejak GPS asli, bukan hasil snapping ke jaringan jalan resmi, sehingga bisa terdapat deviasi kecil dari jalan sebenarnya.
- Hanya mencakup 1 hari operasi (2 trip) — belum cukup untuk generalisasi pola jangka panjang.

## Catatan Kemacetan per Hari

Karena data GPS hanya mencakup 1 hari observasi, ditambahkan catatan kualitatif (bukan hitungan liter/rupiah) tentang potensi macet untuk pola hari yang berbeda — Senin–Jumat siang, Senin–Jumat malam, Sabtu, dan Minggu — berdasarkan karakteristik umum area yang dilewati rute (pasar, kawasan industri, area kota padat). Ditampilkan sebagai teks di panel, terpisah dari data GPS aktual.

## Fitur

- Peta interaktif (Leaflet + basemap OpenStreetMap yang diberi filter gelap) dengan layer rute per-trip berwarna berbeda
- Popup pada rute dan titik awal/akhir berisi metrik lengkap (jarak, kecepatan, konsumsi, biaya)
- Filter tampil/sembunyikan trip (siang/malam)
- Panel ringkasan total dan detail per-trip, termasuk estimasi BBM boros dan biaya boros
- Catatan kualitatif potensi macet per pola hari
- Legenda dan catatan keterbatasan data langsung di panel

## Cara Menjalankan Lokal

Karena project memuat data lewat `fetch()`, perlu dijalankan lewat local server (bukan buka file langsung):

```bash
python3 -m http.server 8000
# buka http://localhost:8000
