# Jejak BBM Trayek 03 — WebGIS Rute & Risiko Macet Bus AKAP Cilegon–Jakarta

WebGIS interaktif untuk merekonstruksi rute dan konsumsi bahan bakar bus AKAP trayek Cilegon–Jakarta, diperluas menjadi estimasi pola operasi 7 hari lengkap dengan analisis titik rawan macet dan perkiraan BBM yang terbuang akibatnya.

**Tugas:** Week 5 — CAAS GIS Lab (perpanjangan tugas Week 2–4: routing & WebGIS)

## Demo

- **Live WebGIS:** _(isi link Vercel setelah deploy)_
- **Repo GitHub:** _(isi link repo)_

## Latar Belakang & Parameter Kendaraan

| Parameter | Nilai | Alasan |
|---|---|---|
| Kendaraan | Bus AKAP (kode K-03) | Objek studi bibit dataset Week 2–4 |
| Trayek | Cilegon – Jakarta | ± 126 km per trip, melewati koridor Cilegon–Serang–Tangerang–Jakarta |
| Jenis BBM | Biosolar | BBM umum untuk bus AKAP kelas ekonomi/bisnis di Indonesia |
| Harga BBM | Rp6.800/L | Mengikuti harga acuan Biosolar subsidi saat data dibuat |
| Efisiensi acuan pabrikan | 3,5 km/L | Nilai umum konsumsi bus besar bermuatan penuh di jalan campuran (tol + arteri) |
| Interval rekam GPS | 30 detik | Cukup rapat untuk menangkap perlambatan/berhenti tanpa data berlebihan |

## Alur Preprocessing Data

1. **`gps_mentah.csv`** — data GPS mentah 2 trip (siang & malam), 1 hari observasi, kolom: `trip_id, waktu, latitude, longitude, kecepatan_kmh, jarak_km`.
2. Titik-titik GPS per trip disambung menjadi **`rute.geojson`** (LineString), dengan properti hasil agregasi per trip: jarak total, kecepatan rata-rata/maksimum, durasi, konsumsi BBM (liter jalan, liter idle, liter total), biaya, dan estimasi BBM boros.
3. Titik awal dan akhir tiap trip diekstrak ke **`titik_ujung.geojson`** (Point) untuk marker keberangkatan/tiba.
4. **`ringkasan.json`** merangkum statistik gabungan seluruh trip (total km, total liter, total biaya, dll).
5. **`titik_macet.geojson`** (baru — Week 5): karena data GPS mentah hanya mencatat dua kondisi (bergerak normal atau berhenti total 0 km/jam, tanpa fase "merayap pelan"), kemacetan **tidak bisa dideteksi langsung dari data**. Titik rawan macet dibangun sebagai estimasi berbasis pengetahuan umum area yang dilalui rute (pasar, simpang padat, kawasan industri, area kota padat), masing-masing diberi level risiko dan estimasi tambahan waktu berhenti.
6. **`simulasi_7hari.json`** (baru — Week 5): proyeksi 14 trip (7 hari × 2 keberangkatan) dari pola 2 trip asli, dengan variasi jarak wajar (±2–3%) dan faktor risiko macet yang lebih tinggi pada hari kerja dibanding akhir pekan.

## Metode Perhitungan Risiko Macet

- Rate konsumsi BBM saat idle diturunkan langsung dari data asli: `liter_idle / menit_idle` pada tiap trip (≈ 0,0367 L/menit, konsisten pada trip siang maupun malam).
- Untuk setiap titik rawan macet, estimasi BBM terbuang = `rate idle (L/menit) × estimasi tambahan menit berhenti/merayap di titik tersebut`.
- Total risiko per lintasan (jika kendaraan mengalami macet di seluruh titik) dijumlahkan dan ditampilkan di panel serta popup peta.
- Pada simulasi 7 hari, faktor macet per hari (0–1) dikalikan ke total risiko dasar untuk mensimulasikan hari-hari yang lebih/lebih tidak macet.

## Pertanyaan yang Dijawab (dengan angka, terlihat di peta)

1. **Berapa total BBM & biaya untuk 2 trip dalam 1 hari?** → Panel "Ringkasan Operasi", ~79,4 L / Rp540.217 (lihat popup tiap rute untuk rincian per trip).
2. **Di mana saja titik rawan macet di sepanjang jalur, dan berapa potensi BBM yang terbuang?** → Layer titik + segmen rute berwarna oranye/merah, dengan popup berisi level risiko, estimasi menit tambahan, dan estimasi liter/rupiah terbuang per titik.
3. **Berapa proyeksi total BBM dan biaya dalam 1 minggu operasi, termasuk kerugian akibat macet?** → Panel "Simulasi 1 Minggu" dan tabel harian, dengan kolom khusus "BBM Macet" per hari.

## Fitur WebGIS

- Basemap OpenStreetMap dengan tema gelap kustom (filter CSS).
- Layer rute per trip (siang/malam) dengan filter tampil/sembunyi.
- Popup detail di setiap rute, titik ujung, dan titik rawan macet.
- Segmen rute berwarna beda di sekitar titik rawan macet (radius ±2,5 km).
- Panel ringkasan: statistik harian, statistik risiko macet, dan simulasi 7 hari.
- Toggle tampil/sembunyi layer titik & segmen macet.
- Legenda dan catatan keterbatasan data.

## Keterbatasan Data

- Data GPS bersifat simulasi/rekonstruksi, bukan rekaman kendaraan sungguhan.
- Titik rawan macet adalah **estimasi**, bukan deteksi otomatis dari data GPS (lihat penjelasan di atas).
- Rute mengikuti jalur titik GPS asli tanpa snapping ke jaringan jalan resmi, sehingga bisa terjadi deviasi kecil dari jalan sebenarnya.
- Simulasi 7 hari adalah proyeksi statistik dari 1 hari observasi, bukan data historis 7 hari yang sesungguhnya.

## Struktur Repo

```
├── index.html              # Aplikasi WebGIS (HTML/CSS/JS + Leaflet)
├── data/
│   ├── rute.geojson         # Rute LineString per trip + statistik BBM
│   ├── titik_ujung.geojson  # Titik keberangkatan & tiba
│   ├── ringkasan.json       # Ringkasan statistik gabungan
│   ├── titik_macet.geojson  # Titik rawan macet + estimasi BBM risiko
│   ├── simulasi_7hari.json  # Proyeksi operasi 7 hari
│   └── gps_mentah_sumber.csv # Data GPS mentah (sumber sebelum diproses)
└── README.md
```

## Cara Menjalankan Lokal

Karena `index.html` memuat data lewat `fetch()`, perlu dijalankan lewat server lokal (tidak bisa dibuka langsung sebagai file):

```bash
python3 -m http.server 8000
# buka http://localhost:8000
```

## Deploy

Repo ini adalah situs statis (HTML + JSON/GeoJSON), bisa langsung di-deploy ke Vercel/Netlify/GitHub Pages tanpa build step.

## Teknologi

- [Leaflet.js](https://leafletjs.com/) 1.9.4 — library peta interaktif
- OpenStreetMap — basemap
- Vanilla JavaScript — logika aplikasi (tanpa framework)
-
