# SML Warehouse — Mockup & Demo

Prototipe sistem **Warehouse Internal** untuk PT Sarana Mitra Luas Tbk (SML), perusahaan sewa forklift.
Sistem ini mencakup tiga jenis "gudang" yang dimiliki SML: **gudang suku cadang**, **yard unit forklift**, dan **baterai**.

> Status: **mockup konsep untuk diskusi internal**. Semua data adalah contoh (dummy). Warna dan logo masih placeholder sampai ada panduan brand resmi.

## Cara membuka

- **Lokal:** unduh repo, lalu buka `index.html` di browser (Chrome/Edge). Tidak perlu instalasi.
- **Online:** aktifkan GitHub Pages (Settings → Pages → Branch `main` / root), atau impor repo ini ke Vercel sebagai situs statis.

## Isi demo

| Menu | Yang bisa dicoba |
|---|---|
| Dashboard | Nilai stok, SKU di bawah minimum, utilisasi armada, jatuh tempo PM, mutasi stok terakhir |
| Stok Part | Filter & cari, kartu stok per part, lokasi rak, lot/kedaluwarsa, label QR |
| Penerimaan | PO tiba → karantina QC → QC lulus & putaway |
| Pengeluaran ke WO | Pick part untuk work order (saran rak FEFO), biaya part per WO |
| Transfer Cabang | Buat, kirim, terima transfer (status IN-TRANSIT) |
| Stock Opname | Hitung fisik, selisih & nilai, ajukan penyesuaian |
| Unit Yard | Status setiap forklift, hour meter, progres PM, riwayat, ubah status |
| Baterai | SoH, siklus, lithium vs lead-acid |
| Aplikasi Scan (HP) | Simulasi aplikasi lapangan: scan part → ambil untuk WO, scan unit → inspeksi |

Pilihan cabang di kanan atas mengubah data yang ditampilkan.

## Mengganti warna brand

Buka `index.html`, lalu ubah token di bagian paling atas `<style>`:

```css
--brand:#0E2C52;   /* warna utama perusahaan (sidebar, header) */
--accent:#F5B400;  /* warna tombol aksi */
```

Ubah juga nilai yang sama di dua blok mode gelap tepat di bawahnya.

## Dokumen

- [`docs/RANCANGAN.md`](docs/RANCANGAN.md): rancangan sistem (modul, model data, alur proses, role, tahapan pembangunan).
