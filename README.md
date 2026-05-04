# RVSTORE — Service Center POS

Aplikasi Point of Sale (POS) mobile-first untuk usaha service gadget/handphone. Dibangun dengan HTML/CSS/JS vanilla + Supabase sebagai backend dan dapat diinstall sebagai PWA di Android/iOS.

---

## Fitur Utama

- ✅ Tambah & kelola order service HP
- ✅ Status order: Antrian → Proses → Selesai → Diambil
- ✅ Cetak resi thermal (58mm / 80mm)
- ✅ Laporan rekap harian/mingguan/bulanan/tahunan
- ✅ Cetak rekap A4 dengan tanda tangan
- ✅ Pencarian & filter order
- ✅ PWA — bisa diinstall di HP seperti app native
- ✅ Offline-capable (data tersimpan di Supabase cloud)

---

## Setup Supabase

### 1. Buat Project Supabase

1. Buka [https://supabase.com](https://supabase.com) dan buat akun (gratis)
2. Klik **New Project**
3. Isi nama project (misal: `rvstore`), password database, dan pilih region terdekat (Singapore)
4. Tunggu project selesai dibuat (~2 menit)

### 2. Buat Tabel di Supabase

1. Di dashboard Supabase, klik **SQL Editor** (ikon terminal di sidebar kiri)
2. Klik **New Query**
3. Copy-paste SQL berikut dan klik **Run**:

```sql
-- Buat tabel orders
CREATE TABLE orders (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  receipt_number text UNIQUE NOT NULL,
  customer_name text NOT NULL,
  customer_phone text,
  phone_type text NOT NULL,
  damage_types text[] NOT NULL DEFAULT '{}',
  damage_description text,
  modal_cost numeric NOT NULL DEFAULT 0,
  service_cost numeric NOT NULL DEFAULT 0,
  profit numeric GENERATED ALWAYS AS (service_cost - modal_cost) STORED,
  status text NOT NULL DEFAULT 'Antrian' CHECK (status IN ('Antrian','Proses','Selesai','Diambil')),
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- Index untuk performa
CREATE INDEX orders_status_idx ON orders(status);
CREATE INDEX orders_created_at_idx ON orders(created_at);

-- Aktifkan Row Level Security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Policy untuk akses anon (tanpa login)
CREATE POLICY "Allow anon select" ON orders FOR SELECT TO anon USING (true);
CREATE POLICY "Allow anon insert" ON orders FOR INSERT TO anon WITH CHECK (true);
CREATE POLICY "Allow anon update" ON orders FOR UPDATE TO anon USING (true);
CREATE POLICY "Allow anon delete" ON orders FOR DELETE TO anon USING (true);

-- Trigger auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = now(); RETURN NEW; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER orders_updated_at
  BEFORE UPDATE ON orders
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### 3. Ambil URL dan Anon Key

1. Di sidebar Supabase, klik **Settings** → **API**
2. Salin:
   - **Project URL** (contoh: `https://abcdefgh.supabase.co`)
   - **anon public** key (string panjang dimulai dengan `eyJ...`)

### 4. Masukkan ke Aplikasi

Setelah deploy, buka aplikasi → tab **Pengaturan** → isi:
- **Supabase URL**: paste Project URL
- **Supabase Anon Key**: paste anon key
- Klik **Simpan Pengaturan**

> Atau edit langsung di `index.html` pada baris:
> ```js
> let SUPABASE_URL = '...' // ganti dengan Project URL kamu
> let SUPABASE_ANON_KEY = '...' // ganti dengan anon key kamu
> ```

---

## Deploy ke GitHub Pages

### Langkah 1: Buat Repository GitHub

1. Buka [https://github.com](https://github.com) dan login
2. Klik **+** → **New repository**
3. Nama repo: `rvstore` (atau terserah)
4. Centang **Public**
5. Klik **Create repository**

### Langkah 2: Upload File

**Cara A — Upload via Web (paling mudah):**
1. Di halaman repo, klik **Add file** → **Upload files**
2. Upload ketiga file: `index.html`, `manifest.json`, `sw.js`
3. Klik **Commit changes**

**Cara B — Via Git:**
```bash
git init
git add .
git commit -m "Initial commit RVSTORE POS"
git remote add origin https://github.com/USERNAME/rvstore.git
git push -u origin main
```

### Langkah 3: Aktifkan GitHub Pages

1. Di repo, klik **Settings** tab
2. Scroll ke **Pages** (di sidebar kiri)
3. Source: pilih **Deploy from a branch**
4. Branch: pilih **main**, folder: **/ (root)**
5. Klik **Save**
6. Tunggu ~1–2 menit, URL akan muncul seperti: `https://USERNAME.github.io/rvstore/`

---

## Install PWA di HP

### Android (Chrome):

1. Buka URL aplikasi di Chrome Android
2. Tunggu beberapa detik, akan muncul banner **"Tambahkan ke layar utama"**
3. Jika tidak muncul: ketuk menu **⋮** → **Tambahkan ke layar utama** / **Install app**
4. Konfirmasi → aplikasi terinstall seperti app native

### iOS (Safari):

1. Buka URL di Safari iPhone/iPad
2. Ketuk tombol **Share** (kotak dengan panah ke atas)
3. Scroll dan pilih **Add to Home Screen**
4. Ubah nama jika mau → ketuk **Add**
5. Ikon RVSTORE akan muncul di home screen

---

## Struktur File

```
rvstore/
├── index.html      ← Aplikasi utama (HTML + CSS + JS semua dalam satu file)
├── manifest.json   ← Konfigurasi PWA (nama, ikon, tema)
├── sw.js           ← Service Worker (offline cache)
└── README.md       ← Panduan ini
```

> **Catatan:** Untuk ikon PWA yang sebenarnya, tambahkan file `icon-192.png` dan `icon-512.png` di folder yang sama. Tanpa ikon, PWA tetap berjalan normal namun menggunakan ikon default browser.

---

## Penggunaan Aplikasi

### Tambah Order Baru
1. Tap tab **Tambah** (ikon +)
2. Isi data pelanggan, type HP, pilih jenis kerusakan
3. Masukkan modal (harga spare part) dan biaya service
4. Tap **Simpan Order** — nomor resi otomatis dibuat

### Ubah Status Order
1. Tap **Riwayat** → pilih order
2. Di halaman Detail, gunakan tombol status untuk mengubah: Antrian → Proses → Selesai → Diambil

### Cetak Resi
1. Buka Detail Order → tap **Cetak Resi**
2. Dialog print browser akan muncul — pilih printer thermal atau simpan sebagai PDF

### Laporan
1. Tap tab **Laporan**
2. Pilih periode (Harian/Mingguan/Bulanan/Tahunan)
3. Gunakan tombol ◀ ▶ untuk berpindah periode
4. Tap **Cetak Rekap A4** untuk mencetak laporan formal

---

## Keamanan

Aplikasi ini menggunakan **Supabase anon key** yang memperbolehkan akses baca-tulis tanpa autentikasi. Ini sesuai untuk penggunaan internal toko dengan satu perangkat/staf.

Jika perlu multi-user dengan login, tambahkan autentikasi Supabase Auth dan update RLS policy sesuai kebutuhan.

---

## Troubleshooting

| Masalah | Solusi |
|---------|--------|
| "Supabase belum dikonfigurasi" | Buka Pengaturan → isi URL dan Key |
| Data tidak muncul | Cek koneksi internet, pastikan tabel sudah dibuat |
| Tidak bisa install PWA | Pastikan diakses via HTTPS (GitHub Pages sudah HTTPS) |
| Print resi tidak sesuai | Sesuaikan ukuran kertas di Settings → 58mm atau 80mm |
| Error saat simpan order | Cek SQL table sudah dijalankan dan RLS policy aktif |

---

## Lisensi

MIT License — bebas digunakan dan dimodifikasi untuk keperluan bisnis.
