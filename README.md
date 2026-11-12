# Akuisisi Data Akomodasi di Kecamatan Kintamani

Tugas Praktik BIFP-243 Akuisisi dan Manajemen Data

- Nama: I Gusti Ngura Dwi Andika Putra
- NIM: 2501010016

## Tujuan

Mengambil data akomodasi (hotel, guest house, hostel, motel, apartment, dan chalet) di Kecamatan Kintamani, Kabupaten Bangli, Bali dari OpenStreetMap, menyimpan respons mentahnya, mengubahnya menjadi tabel CSV, memeriksa kualitasnya, lalu membandingkan jumlahnya dengan data BPS.

## Sumber data

| Sumber | Keterangan |
|---|---|
| OpenStreetMap lewat Overpass API | `https://overpass-api.de/api/interpreter`, lisensi ODbL |
| BPS Kabupaten Bangli | Dipakai sebagai pembanding. Tahun data 2023: 19 hotel dan 39 penginapan, total 58 akomodasi |

Saya tidak mengambil data dari situs pemesanan atau OTA (Booking.com, Agoda, Traveloka, tiket.com, Airbnb, Tripadvisor) maupun dari halaman Google Maps.

## Tanggal akses

Data OSM diakses pada 12 November 2026. Nama berkas mentah memuat waktu pengambilan, dan rinciannya dicatat di `docs/provenance.json`.

## Metode akuisisi

1. Query Overpass memakai area administratif bernama Kintamani dengan `admin_level=6`, lalu mengambil objek (`node`, `way`, `relation`) dengan tag `tourism` bernilai `hotel`, `guest_house`, `hostel`, `motel`, `apartment`, atau `chalet`. Query memakai `out center tags` supaya bangunan mendapat titik tengah.
2. Query dikirim dengan Python (`requests`) memakai timeout 120 detik dan retry dengan jeda yang bertambah dua kali lipat saat server membalas error atau timeout.
3. Respons mentah disimpan apa adanya di `data/raw/` dalam format JSON.
4. JSON diratakan menjadi tabel dengan `pandas` dan disimpan sebagai `data/interim/akomodasi_kintamani.csv`.
5. Saya memeriksa duplikat, nama kosong, kolom kosong, dan rentang koordinat.
6. Jumlah akomodasi OSM dibandingkan dengan angka BPS, lalu 10 data acak (`random_state=42`) dicek secara manual.

## Hasil singkat

Total ada **32 akomodasi**.

| Jenis | Jumlah |
|---|---|
| hotel | 16 |
| guest_house | 9 |
| hostel | 4 |
| chalet | 2 |
| motel | 1 |

Pemeriksaan kualitas:

- Duplikat berdasarkan `osm_type` dan `osm_id`: 0
- Koordinat kosong: 0 (latitude -8,2838 sampai -8,2189; longitude 115,2546 sampai 115,4129)
- Tanpa nama: 1
- Kolom kosong: `desa` 32 dari 32, `bintang` 31, `jumlah_kamar` 30, `website` 30

Perbandingan dengan BPS: 32 ÷ 58 × 100% = 55,2%. Angka ini hanya rasio kasar jumlah OSM terhadap jumlah BPS, bukan ukuran kelengkapan yang pasti. Tahun datanya berbeda (OSM 2026, BPS 2023) dan BPS mengelompokkan akomodasi menjadi hotel dan penginapan, sedangkan OSM memakai tag `tourism`.

Verifikasi manual 10 sampel: 7 sesuai, 1 jenis berbeda, 1 perlu pemeriksaan objek (nama objek menunjukkan titik pintu masuk hostel), dan 1 tidak ditemukan. Detailnya ada di `docs/verifikasi_manual_10_sampel.csv`.

## Batasan dan etika

- Nomor telepon, email, dan akun media sosial tidak diambil dan tidak ada di dataset.
- Berkas di `data/raw/` tidak saya ubah setelah tersimpan.
- Query ke Overpass sempat dijalankan beberapa kali selama pengujian dan percobaan ulang, termasuk satu kali server membalas status 502. Akibatnya ada beberapa berkas mentah di `data/raw/`. Semuanya saya biarkan dan tidak dihapus.
- OSM diisi oleh relawan, jadi datanya tidak lengkap dan tidak boleh dianggap sensus akomodasi. Kolom `desa` kosong di semua baris, sehingga rincian per desa tidak bisa dibuat.

## Atribusi

Data © OpenStreetMap contributors, dilisensikan di bawah Open Database License (ODbL). https://www.openstreetmap.org/copyright

## Cara menjalankan ulang

1. Buka `notebooks/akuisisi_akomodasi.ipynb` di Google Colab atau Jupyter.
2. Pasang pustaka yang dipakai: `pip install requests pandas matplotlib folium`.
3. Jalankan semua sel dari atas. Sel akuisisi mengirim query ke Overpass, jadi jalankan satu kali saja. Kalau berkas mentah sudah ada di `data/raw/`, lewati sel pengambilan dan lanjutkan dari sel perataan agar server tidak terbebani.

## Struktur repository

```
data/raw/        respons mentah Overpass (JSON)
data/interim/    akomodasi_kintamani.csv
notebooks/       akuisisi_akomodasi.ipynb
docs/            data_dictionary.csv, data_ethics_checklist.csv, perbandingan_bps.csv,
                 data_bps_kintamani.csv, verifikasi_manual_10_sampel.csv, provenance.json
reports/         laporan akomodasi Kintamani (PDF)
README.md
```
