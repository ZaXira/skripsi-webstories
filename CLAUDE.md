# CLAUDE.md — Konteks Claude Code untuk Proyek Thesis Web Stories

Baca file ini sebelum menyentuh kode apapun di proyek ini.

---

## Apa proyek ini

Halaman web **satu file** (`index.html`) yang menyajikan hasil skripsi S1 tentang penilaian kerusakan lahan pertanian dari citra satelit (Tsunami Palu 2018). Tidak ada framework, tidak ada build step. Semua ada di `index.html` — CSS, JS, markup.

**Baca README.md** untuk ringkasan penelitian dan struktur folder lengkap.

---

## ATURAN PALING PENTING

### ❌ JANGAN ubah angka hasil penelitian
Semua angka metrik di `index.html` (F1, Precision, Recall, IoU, Kappa, xView2, mIoU) adalah **final dan terverifikasi dari eksperimen Kaggle**. Jangan koreksi, bulatkan berbeda, atau "perbaiki" angka ini kecuali diminta eksplisit dengan angka penggantinya.

Angka yang dimaksud ada di:
- Data array Chart.js (`data: [0.6092, 0.5943, ...]`)
- Isi tabel HTML (`<td>0,6589</td>`)
- Teks kartu skor (`<div class="scv">0,3103</div>`)

### ❌ JANGAN ubah warna kelas kerusakan
Tiga warna ini adalah bagian dari standar visualisasi penelitian — sama persis dengan visualisasi di artefak web dan Bab IV skripsi:
- `#27ae60` → Tidak Terdampak (hijau)
- `#e67e22` → Terdampak Sedang (oranye)  
- `#c0392b` → Terdampak Berat (merah)

### ❌ JANGAN ganti library CDN tanpa konfirmasi
Chart.js **4.4.1** dari cdnjs dikunci karena diuji dengan versi ini. Jangan upgrade atau ganti host.

---

## Struktur `index.html`

File berisi section berurutan — jangan ubah urutan atau id-nya:

| ID | Konten |
|---|---|
| `#s0` | Hero (canvas animasi, judul) |
| `#s1` | Konteks masalah (stat tiles, class chips) |
| `#s2` | Dataset (data-grid, chart distribusi kelas) |
| `#s3` | Pipeline metodologi (flow diagram, 6 feat cards) |
| `#s4` | Tujuan 1 — segmentasi (chart bar, tabel metrik) |
| `#s5` | Tujuan 2 — klasifikasi (chart bar, per-class table, feat importance) |
| `#s6` | Tujuan 3 — end-to-end (score cards, tile table, oracle callout) |
| `#s7` | Artefak web (4 finding cards, validation table, HF link) |
| `#s8` | Kesimpulan (conc cards, footer) |

### CSS token system
Semua warna didefinisikan sebagai CSS custom properties di `:root`. Dark mode ditangani via `@media (prefers-color-scheme: dark)` + `[data-theme="dark"]`. **Jangan hardcode warna** di luar token — selalu pakai `var(--token)`.

### Komponen JS
Semua JS ada dalam satu IIFE di akhir file:
1. `THEME` — toggle dark/light, simpan ke localStorage
2. `HERO CANVAS` — animasi parsel berwarna + scan line
3. `PROGRESS NAV` — dots navigasi, IntersectionObserver per section
4. `SCROLL REVEAL` — fade-in via IntersectionObserver `.rv` → `.in`
5. `FEATURE IMPORTANCE BARS` — animasi bar saat section masuk viewport
6. `CHARTS` — tiga Chart.js: donut distribusi, bar T1, bar T2

---

## Task yang umum diminta

### Tambah gambar overlay Tujuan 3
Gambar overlay hasil penelitian (3-panel per tile) ada di `tujuan3_bundle.zip`. Setelah di-extract ke `img/`:

1. Tambahkan grid gambar setelah `.tile-wrap` di section `#s6`
2. Gunakan struktur:
   ```html
   <div class="overlay-grid">
     <figure class="ov-fig">
       <img src="img/overlay_163.png" alt="Overlay tile 00000163" loading="lazy">
       <figcaption>Tile 00000163 — n_gt = 113</figcaption>
     </figure>
     <!-- dst -->
   </div>
   ```
3. CSS `.overlay-grid`: `display:grid; grid-template-columns: repeat(auto-fit, minmax(380px,1fr)); gap: 20px;`
4. Gambar pakai `max-width: 100%; border-radius: var(--r);`

### Tambah section baru
- Tambahkan `<section id="sN">` dengan class `.inner` di dalamnya
- Tambahkan id ke array `sids` di JS navigation
- Tambahkan `<button>` ke `.prog-nav` (otomatis kalau pakai fungsi di JS)
- Pastikan semua elemen pakai class `.rv` untuk reveal animation

### Update teks tanpa ubah angka
Aman untuk diubah: teks deskriptif, caption, keterangan metodologi. Jangan ubah angka, warna kelas, atau nama model (`Mask R-CNN`, `YOLOv8-seg`, `Random Forest`, `LightGBM`).

### Deploy ke GitHub Pages
```bash
git add .
git commit -m "update: <deskripsi singkat>"
git push origin main
# GitHub Actions akan deploy otomatis ke Pages
```

---

## Path file penting

```
index.html          ← SATU-SATUNYA file yang perlu diedit untuk konten web
README.md           ← Dokumentasi publik
CLAUDE.md           ← File ini
img/                ← Aset visual (opsional, tambahkan saat tersedia)
.github/workflows/deploy.yml  ← GitHub Actions deploy
```

---

## Yang TIDAK ada di proyek ini (dan tidak perlu ditambahkan)

- `package.json` / `node_modules` — tidak perlu, tidak ada build step
- File `.tif`, `.gpkg`, `.pth`, `.joblib` — terlalu besar, tetap di Kaggle/HF
- Framework (React, Vue, Svelte) — overkill untuk satu halaman statis
- CSS preprocessor (Sass, Less) — CSS custom properties sudah cukup
- Test suite — bukan proyek software, bukan dependency production
