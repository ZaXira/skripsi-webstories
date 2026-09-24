# Penilaian Kerusakan Lahan Pertanian — Web Stories

Halaman scrollytelling interaktif untuk menyajikan hasil skripsi **"Penilaian Kerusakan Lahan Pertanian Terdampak Bencana Menggunakan Citra Satelit Resolusi Sangat Tinggi Berbasis Deep Learning dan Machine Learning: Studi Kasus Tsunami Palu 2018"**.

Penulis: **Rafliansyah Tondau** · D-IV Komputasi Statistik · Politeknik Statistika STIS · 2026

Live artefak web (pipeline): https://huggingface.co/spaces/ZaXira/penilaian-kerusakan-lahan

---

## Ringkasan Penelitian

| | |
|---|---|
| **Studi kasus** | Tsunami Palu 28 September 2018 (M 7,5) |
| **Data** | 18 tile citra VHR pre & post-disaster, 3.271 poligon lahan pertanian |
| **Tujuan 1** | Segmentasi instance lahan — Mask R-CNN vs YOLOv8-seg → **Mask R-CNN terpilih** (Instance F1 = 0,6589) |
| **Tujuan 2** | Klasifikasi kerusakan — Random Forest vs LightGBM → **RF terpilih** (Kappa = 0,5556) |
| **Tujuan 3** | Pipeline end-to-end integrasi T1 + T2 → **xView2 Score = 0,3103** |
| **Artefak web** | Gradio + HF Spaces ZeroGPU, validasi regresi 100% identik pada jalur GeoTIFF |

---

## Struktur Folder

```
thesis-stories/
├── index.html              ← Halaman web stories (satu file, self-contained)
├── README.md               ← File ini
├── CLAUDE.md               ← Konteks untuk Claude Code (baca ini dulu)
├── .gitignore              ← Mengecualikan aset sumber berukuran besar
├── img/                    ← Aset web (sudah dioptimasi, ~1 MB total)
│   ├── overlay_163.webp / .jpg   ← 3-panel viz Tujuan 3 per tile
│   ├── overlay_169.webp / .jpg
│   ├── overlay_184.webp / .jpg
│   └── overlay_195.webp / .jpg
├── Tujuan 3/               ← Sumber mentah dari Kaggle — TIDAK di-commit, TIDAK di-deploy
│   ├── tujuan3_bundle.zip
│   ├── tujuan3_overlay/          ← PNG asli 3574x1339 (~3,5 MB/file)
│   └── output/, __results___files/
└── .github/
    └── workflows/
        └── deploy.yml          ← Deploy otomatis ke GitHub Pages
```

> **Catatan aset:** Gambar overlay disajikan lewat `<picture>` — browser modern mengambil `.webp`, sisanya jatuh ke `.jpg`. Versi web diperkecil ke lebar 1800 px (dari 3574 px), sehingga total `img/` turun dari ~13,6 MB menjadi ~1 MB. PNG resolusi penuh tetap tersimpan di `Tujuan 3/tujuan3_overlay/`.
>
> Folder `Tujuan 3/` diabaikan oleh `.gitignore` **dan** dikecualikan dari artefak GitHub Pages lewat langkah *Stage site* di `deploy.yml`, supaya repo dan halaman tetap ringan.

---

## Cara Menjalankan Lokal

Tidak perlu build tool — buka saja file HTML langsung:

```bash
# Opsi 1: langsung di browser
open index.html

# Opsi 2: live server via Python (menghindari CORS untuk aset lokal)
python3 -m http.server 8080
# lalu buka http://localhost:8080

# Opsi 3: VS Code Live Server extension
# Klik kanan index.html → "Open with Live Server"
```

---

## Deploy ke GitHub Pages

### Otomatis (via GitHub Actions)

Buat repo di GitHub, push semua file, lalu aktifkan Pages dari **Settings → Pages → Source: GitHub Actions**.

File `.github/workflows/deploy.yml` sudah disiapkan — setiap push ke `main` akan mendeploy otomatis.

> Langkah *Stage site* menyalin isi repo ke `_site/` dengan mengecualikan `Tujuan 3/`, `*.zip`, `CLAUDE.md`, dan `.github/` — jadi hanya `index.html`, `img/`, dan `README.md` yang ter-deploy.

### Manual

```bash
git init
git add .
git commit -m "init: thesis web stories"
git remote add origin https://github.com/<USERNAME>/thesis-stories.git
git push -u origin main
```

---

## Menambah Aset Visual dari Kaggle

Overlay Tujuan 3 sudah terpasang di bagian **Tujuan 3** (`#s6`) sebagai galeri 4 tile yang bisa diklik untuk diperbesar.

Untuk menambah atau memperbarui tile: salin PNG sumber ke `Tujuan 3/tujuan3_overlay/`, jalankan skrip berikut, lalu tambahkan satu blok `<figure class="ov-fig">` di dalam `.overlay-grid` pada `#s6`.

```python
from PIL import Image

for n in (163, 169, 184, 195):
    im = Image.open(f'Tujuan 3/tujuan3_overlay/tujuan3_3panel_00000{n}.png').convert('RGBA')
    bg = Image.new('RGB', im.size, (255, 255, 255))
    bg.paste(im, mask=im.split()[-1])
    w = 1800
    bg = bg.resize((w, round(bg.height * w / bg.width)), Image.LANCZOS)
    bg.save(f'img/overlay_{n}.webp', 'WEBP', quality=86, method=6)
    bg.save(f'img/overlay_{n}.jpg', 'JPEG', quality=82, optimize=True)
```

Confusion matrix (`cm_rf_test.png`, `cm_t3_test.png`) dan screenshot artefak HF belum dipasang — tambahkan ke `img/` bila diperlukan.

---

## Dependensi Eksternal

| Library | CDN | Versi |
|---|---|---|
| Chart.js | cdnjs.cloudflare.com | 4.4.1 (dikunci) |
| DM Serif Display | Google Fonts | — |
| IBM Plex Sans | Google Fonts | — |
| IBM Plex Mono | Google Fonts | — |

Tidak ada build step, tidak ada `package.json`, tidak ada framework. Semua logika ada di `index.html`.

---

## Angka Penelitian — JANGAN DIUBAH

Semua angka hasil penelitian di bawah ini **final dan terverifikasi**. Jangan modifikasi tanpa konfirmasi eksplisit dari penulis.

### Tujuan 1 (Test Set, 4 tile)
| Metrik | YOLOv8-seg | Mask R-CNN |
|---|---:|---:|
| Instance Precision | 0,6092 | **0,8131** |
| Instance Recall | 0,5943 | 0,5813 |
| Instance F1 | 0,5973 | **0,6589** |
| Instance IoU | 0,7492 | **0,8421** |
| Pixel F1 | 0,6963 | **0,7178** |

### Tujuan 2 (Test Set, 604 poligon)
| Metrik | Random Forest | LightGBM |
|---|---:|---:|
| Accuracy | **0,6639** | 0,6507 |
| F1-Macro | **0,4903** | 0,4849 |
| Kappa (quadratic) | **0,5556** | 0,4566 |

### Tujuan 3 (4 tile, gabungan)
| Metrik | Nilai |
|---|---:|
| F1 Lokalisasi | 0,7062 |
| F1 Damage (harmonic) | 0,1406 |
| **Skor Gabungan xView2** | **0,3103** |
| mIoU (4 kelas) | 0,4005 |

---

## Kontak

Rafliansyah Tondau · [Hugging Face](https://huggingface.co/ZaXira)
