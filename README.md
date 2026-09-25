# toko_e — Database Toko Furnitur

Database MySQL untuk sistem penjualan furnitur, mencakup manajemen barang, pelanggan, transaksi, pengadaan, pengiriman, dan membership.

---

## Struktur Database

Database: `toko_e`

### Tabel

| Tabel | Deskripsi |
|---|---|
| `barang` | Katalog produk furnitur |
| `supplier` | Data pemasok barang |
| `kasir` | Data karyawan kasir |
| `membership` | Tier keanggotaan pelanggan |
| `pelanggan` | Data pelanggan beserta membership dan jarak pengiriman |
| `pengiriman` | Jasa dan tarif pengiriman |
| `payment` | Metode pembayaran per tier membership |
| `transaksi` | Header transaksi (tanggal & waktu) |
| `detail_transaksi` | Baris detail tiap transaksi |
| `detail_pengadaan` | Baris detail tiap pengadaan barang |
| `pengadaan` | Header pengadaan dari supplier |

### Relasi Utama

- `pelanggan` → `membership` (FK `id_membership`)
- `payment` → `membership` (FK `id_membership`)
- `pengadaan` → `supplier` (FK `id_supplier`)
- `pengadaan` → `detail_pengadaan` (FK `id_pengadaan`)
- `detail_pengadaan` → `barang` (FK `id_barang`)
- `detail_transaksi` → `transaksi`, `kasir`, `pelanggan`, `membership`, `barang`, `pengiriman`, `payment`

---

## ERD

![ERD toko_e](erd.png)

---

## Trigger

### `kelola_stok_after_transaksi`
Dijalankan **AFTER INSERT** pada `detail_transaksi`.
- Mengurangi `stok_barang` sejumlah `jumlah_barang` yang dibeli.
- Jika stok turun di bawah 20, otomatis di-*refill* ke 25.

### `tambah_stok_after_pengadaan`
Dijalankan **AFTER INSERT** pada `detail_pengadaan`.
- Menambah `stok_barang` sejumlah barang yang masuk dari pengadaan.

---

## Stored Procedure

### `generate_nota(transaksi_id)`
Menghasilkan nota lengkap satu transaksi, berisi:
- Info kasir, pelanggan, membership
- Daftar barang yang dibeli
- Subtotal barang, diskon (aktif jika member & subtotal ≥ Rp10.000.000), bunga kredit (jika *Credit*), ongkir, membership fee
- Grand total akhir

```sql
CALL generate_nota('TRX-001');
```

### `barang_terlaris(top_n)`
Menampilkan `top_n` barang dengan total unit terjual terbanyak.

```sql
CALL barang_terlaris(3);
```

### `barang_per_bulan(bulan_ke)`
Menampilkan penjualan per barang dikelompokkan per bulan. Isi `0` untuk semua bulan.

```sql
CALL barang_per_bulan(0);  -- semua bulan
CALL barang_per_bulan(4);  -- bulan April
```

---

## Cara Pakai

```sql
-- 1. Buat dan aktifkan database
CREATE DATABASE IF NOT EXISTS toko_e;
USE toko_e;

-- 2. Jalankan SQL_Syntax.sql secara penuh
-- (sudah mencakup DDL, trigger, procedure, dan data dummy)
```

---

## Data Dummy

| Entitas | Jumlah |
|---|---|
| Membership tier | 4 (Non-Member, Bronze, Silver, Gold) |
| Barang | 10 |
| Supplier | 5 |
| Kasir | 5 |
| Pelanggan | 10 |
| Pengiriman | 5 |
| Payment | 7 |
| Pengadaan | 10 |
| Transaksi | 5 |
| Detail transaksi | 7 baris |

---

## Requirement

- MySQL 8.0+
- Tidak ada dependensi eksternal