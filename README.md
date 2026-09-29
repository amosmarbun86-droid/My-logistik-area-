# Denah Gudang & Alur Operasional — Interaktif

Visualisasi interaktif denah gudang PT Nusantara Express Kilat: alur kerja operasional (inbound → sortir → bagging → hub → outbound → return), lengkap dengan animasi operator/forklift/truk, dan **status live** yang terhubung ke [Alarm Dashboard 24 Jam](https://alarm-telegram-24jam.onrender.com/).

🔗 **Live:** https://my-logistik-area.vercel.app

---

## Fitur

- **Denah interaktif** — klik area mana pun di gambar (Inbound, Penampungan, Sortir, Bagging, Hub, Barhal, Treaser, Security, Return, Kantor CCTV, Titik Kumpul) untuk melihat penjelasan dan animasi operator/forklift/truk di zona itu.
- **Timeline alur operasional (8 langkah)** — bisa dijalankan otomatis (▶ Mulai Alur) atau manual per langkah:
  1. Inbound (Dock 01–05 & 06–10) — kedua dock sama-sama area bongkar
  2. Penampungan
  3. Sortir Manual
  4. Bagging & Packing
  5. Hub (distribusi via forklift)
  6. Staging (di sisi Hub kiri/kanan)
  7. Outbound (muat truk di sisi Hub kiri/kanan)
  8. Return Processing — proses **terpisah/paralel**, bukan lanjutan outbound
- **Live Alarm Status** — panel yang menarik data real-time dari Alarm Dashboard 24 Jam (rute, jam, status: Freeload / Sedang Proses / Selesai / Sudah Sandar).
- **Label nama hub dinamis** — kotak HUB 01–20 di denah otomatis berganti menampilkan nama tujuan (bukan lagi "HUB 01", dst) selama rute itu berstatus Freeload/Sedang Proses. Satu mobil = satu kotak; kalau tujuannya lebih dari satu (mis. "Balige +2"), klik kotaknya untuk lihat daftar lengkap tujuan hub mobil tersebut.
- **Kontrol simulasi** — Play / Pause / Reset / Fullscreen / kecepatan animasi (0.5×–2×).

---

## Struktur File

```
├── index.html          # Seluruh aplikasi (HTML + CSS + JS dalam satu file)
├── denah-gudang-1.webp # Gambar denah gudang (harus ada di folder yang sama)
└── README.md
```

Semua logika ada di satu file `index.html` — tidak ada build step, tidak ada dependency. Tinggal buka di browser atau deploy sebagai static site (Vercel / GitHub Pages).

---

## Integrasi dengan Alarm Dashboard 24 Jam

Halaman ini **membaca (read-only)** data jadwal dari endpoint publik yang disediakan project [Alarm Dashboard 24 Jam](../alarm-telegram-24jam):

```
GET https://alarm-telegram-24jam.onrender.com/api/status-live
```

Endpoint ini sengaja dibuat terpisah dari sistem utama dashboard:
- **Tidak butuh password.**
- **Tidak bisa menulis/mengubah data apa pun** — cuma baca status hari ini (`route`, `slot`, `start`, `selesai`, `status`, `sudah_sandar`).
- **Tidak mengekspos `FIREBASE_SECRET`** ke browser — semua akses Firebase tetap terjadi di server Render, bukan di kode client yang repo-nya publik.

Kalau endpoint itu berubah alamat, update konstanta ini di `index.html`:

```js
const LIVE_STATUS_URL = 'https://alarm-telegram-24jam.onrender.com/api/status-live';
```

Data di-refresh otomatis tiap 15 detik (`LIVE_POLL_MS`).

### Cara kerja pemetaan rute → hub

Satu baris `route` di Alarm Dashboard berisi daftar hub tujuan satu mobil, dipisah tanda `-`, misalnya:

```
Siborong - Borong DC > Ulu Barumun Hub - Sosa Hub - Huta Raja Tinggi Hub
```

Fungsi `routeHubs()` mem-parsing ini jadi `["Ulu Barumun", "Sosa", "Huta Raja Tinggi"]`. Satu mobil ditampilkan sebagai **satu kotak** di denah (menempati satu nomor HUB), berlabel tujuan pertama + jumlah sisanya (mis. `Ulu Barumun +2`). Klik kotaknya untuk melihat semua tujuan.

Penempatan nomor kotak saat ini **otomatis** (kosong terdekat). Untuk mengunci hub tujuan tertentu ke nomor kotak tetap, isi:

```js
const HUB_SLOT_MAP = {
  'ULU BARUMUN': 3,
  'SOSA': 4,
  // ...
};
```

---

## Cara Deploy / Update

1. Edit `index.html` (atau ganti `denah-gudang-1.webp` kalau ada revisi denah — **nama file harus persis sama**, path relatif, case-sensitive).
2. Commit & push ke branch `main` di repo `My-logistik-area-`.
3. Vercel akan otomatis build ulang dan deploy (biasanya < 1 menit).

Tidak perlu `npm install` atau proses build apa pun — ini murni static HTML.

---

## Proyek Terkait

- **Alarm Dashboard 24 Jam** — sumber data live (Flask + Firebase Realtime Database), deployed di Render.
