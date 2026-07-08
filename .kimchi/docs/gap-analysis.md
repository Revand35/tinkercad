# Gap Analysis Report — `tinkercad-web`

**Tanggal audit**: 2026-07-07
**Auditor**: pi (model `minimax-m3`) via ferment `019f3b43-05c2-746b-9cf2-5294054174b2`
**Proyek**: `/mnt/c/Users/USER/Downloads/Tinkercad_Web_Presentation/tinkercad-web/`
**Lingkup**: Hanya observasi (read-only). Tidak ada kode yang diubah selama audit.

---

## 1. Ringkasan Eksekutif

Proyek adalah **presentasi web statis** (HTML tunggal + folder `img/`) berjudul *"Desain 3D dengan Tinkercad — Modul Pelatihan"* untuk siswa SMA/SMK. Bahasa Indonesia. Vanilla HTML5/CSS/JS tanpa build tool, tanpa package manager, tanpa dependency.

**Runtime health**: **HIJAU** — presentasi termuat dengan benar di HTTP server, semua aset gambar 200 OK, konsistensi ID DOM bersih, JavaScript balanced dan IIFE ditutup rapi.

**Celah utama yang ditemukan**: **44 item** yang dikategorikan ke dalam 7 kategori, dengan **10 item berprioritas P0** (perlu ditangani segera), **12 item P1** (penting), dan **22 item P2** (nice-to-have).

**Temuan paling kritis**:
1. Tidak ada dokumentasi (README, .gitignore) — onboarding contributor tertahan.
2. Tidak ada `meta description` / favicon — preview share dan tab browser kosong.
3. Bobot gambar 6.1 MB (PNG polos) — lama dimuat di kelas dengan koneksi terbatas.
4. Tidak ada deep-link per slide — sulit mengarahkan siswa ke bagian tertentu.
5. Kegagalan fullscreen *silent* (`requestFullscreen().catch(()=>{})`) — F tidak bekerja, user tidak tahu kenapa.

Rekomendasi: tangani keenam P0 sebelum dipakai di kelas; P1 bisa di-batch dalam sprint berikutnya.

---

## 2. Metode

| Langkah | Alat | Hasil |
|---|---|---|
| Daftar file | `ls`, `find` | 1 file `index.html` (724 baris, 36.4 KB), folder `img/` (11 PNG, total 6.1 MB), `.git/`, `.vscode/launch.json` |
| HTTP serve | `python3 -m http.server 8080` (PID 1125) | Listening di `127.0.0.1:8080` |
| Verifikasi route | `curl -I`, `curl -w` | `/` dan `/index.html` → 200, `text/html`, 37318 bytes |
| Verifikasi aset | Loop `curl` per path gambar | 10/10 gambar dirujuk → 200 OK |
| Konsistensi ID DOM | `diff` antara `getElementById()` calls dan `id="..."` declarations | 7/7 ID cocok |
| Keseimbangan JS | Ekstrak blok `<script>`, hitung `{`/`}`, `(`/`)`, `[`/`]` | Braces 20/20, parens 78/78, brackets 5/5 — balanced |
| Grep statis | `grep -nE` untuk pola tertentu | TODO/FIXME: 0; aria attrs: 3; console: 0 |

---

## 3. Status Verifikasi Runtime

### ✅ Yang sudah benar (tidak perlu diubah)

| Aspek | Bukti | File:Line |
|---|---|---|
| HTTP 200 untuk `/` dan `/index.html` | `curl -w '%{http_code}'` = `200` | runtime |
| Semua 10 gambar referensi 200 OK | Loop curl, semua berawalan `200` | runtime |
| Counter "01 / 16" cocok dengan jumlah slide | `grep -c '<section class="slide'` = `16`, counter di line 306 | `index.html:306`, `:319-...` |
| 7 ID JS cocok dengan deklarasi HTML | `diff` exit 0 | `index.html:639,649-659,669` ↔ `index.html:306-310,639-642` |
| JavaScript balanced | `{` 20 / `}` 20, `(` 78 / `)` 78, `[` 5 / `]` 5 | blok `<script>` (lines ~646-723) |
| IIFE ditutup rapi | `})();` di akhir script | `index.html:723` |
| `prefers-reduced-motion` dihormati | 2 media query ada | `index.html:123-125`, `:258` |
| Semua `<img>` punya `alt` text | 10 `<img>`, 10 `alt=`, semua bermakna | `index.html:388,419,426,463,470,493,494,513,520,552` |
| `lang="id"` disetel | sesuai konten | `index.html:2` |
| Preconnect Google Fonts | `rel="preconnect"` untuk googleapis & gstatic | `index.html:7-8` |
| Slide transition CSS solid | `.slide` + `.slide.active` dengan transition cubic-bezier | `index.html:108-128` |

### 🟡 Yang bisa ditingkatkan

| Aspek | Catatan |
|---|---|
| Touch swipe | `dx > 50` threshold reasonable, tapi tidak ada haptic/visual feedback |
| Fullscreen | Berhasil saat diizinkan, tapi error di-swallow silently |
| `.vscode/launch.json` | Port 8080 hardcode, tidak menjelaskan opsi serve lain |

---

## 4. Daftar Celah

### 4.1 Dokumentasi

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| D1 | **Tidak ada `README.md`** — kontributor/presenter tidak tahu cara menjalankan, struktur, lisensi | **P0** | `ls` tidak menemukan `README*` |
| D2 | **Tidak ada `.gitignore`** — `.vscode/`, file temporary, dsb. akan ikut commit | **P0** | `ls -la .gitignore` → ENOENT |
| D3 | Tidak ada komentar header di `index.html` menjelaskan tujuan, versi, kontak | P2 | `index.html:1` langsung `<!DOCTYPE>` |

### 4.2 Build / Deploy

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| B1 | **Tidak ada `meta name="description"`** — preview share kosong | **P0** | `grep description index.html` → 0 matches |
| B2 | **Tidak ada favicon** — tab browser blank | **P0** | `grep favicon index.html` → 0 matches |
| B3 | **Tidak ada OG / Twitter card meta** — preview share tidak menarik | P1 | `grep -E 'og:\|twitter:'` → 0 matches |
| B4 | **Tidak ada fallback font** — jika Google Fonts diblokir/timeout, teks dalam fallback default sans | **P0** | `index.html:9` — `font-display: swap` membantu tapi tidak ada stack fallback eksplisit di CSS |
| B5 | **Tidak ada service worker** — presentasi gagal total jika offline | P1 | `grep serviceWorker` → 0 matches |
| B6 | `.vscode/launch.json` port 8080 hardcode; tidak menjelaskan `python3 -m http.server` | P2 | `.vscode/launch.json:11` |
| B7 | Tidak ada script `npm run` / `make serve` — berulang-ulang ketik command manual | P2 | tidak ada `package.json`/`Makefile` |
| B8 | Repo git **belum ada commit sama sekali** | P2 | `git log` → "fatal: your current branch 'master' does not have any commits yet" |

### 4.3 Aksesibilitas

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| A1 | **Tidak ada `aria-live`** di area slide — screen reader tidak tahu konten berubah | **P0** | `grep aria-` → hanya 3 (semua pada tombol nav) |
| A2 | **Tidak ada focus management** saat slide berpindah — pengguna keyboard kehilangan konteks | **P0** | `grep '\.focus('` → 0 matches; `grep tabindex` → 0 matches |
| A3 | Tidak ada skip-link "Lewati ke konten" | P1 | tidak ada `<a class="skip">` |
| A4 | Tombol ruler-tick tanpa `aria-label` yang menjelaskan "Loncat ke slide N" | P1 | `index.html:660-666` — `<button>` dibuat tanpa aria |
| A5 | `<html lang="id">` ✓ tapi tidak ada `<html dir="ltr">` eksplisit | P2 | `index.html:2` |
| A6 | Touch swipe tidak ada fallback untuk pengguna switch device | P2 | `index.html:712-720` — hanya touchstart/touchend |
| A7 | Tidak ada preferensi tema terang/gelap (`prefers-color-scheme`) | P2 | tidak ada media query |

### 4.4 Fitur Presentasi

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| F1 | **Tidak ada deep-link via URL hash** — tidak bisa share link ke slide tertentu | **P0** | `grep window.location.hash` → 0 matches |
| F2 | **Tidak ada presenter mode / speaker notes** | P1 | tidak ada overlay/mode terpisah |
| F3 | Tidak ada slide overview / thumbnail grid (Esc untuk overview) | P1 | tidak ada `<div class="overview">` |
| F4 | Tidak ada progress dots (hanya ruler ticks) | P2 | `index.html:307-310` — counter saja |
| F5 | Tidak ada pause-on-hover untuk auto-play (saat ini tidak auto-play, tapi bisa jadi fitur) | P2 | tidak ada autoplay |
| F6 | Tidak ada toggle "mode gelap/terang" dalam app | P2 | tidak ada switch |
| F7 | **Tidak ada print stylesheet** — pencetakan handout menghasilkan layout rusak | P1 | `grep '@media print'` → 0 matches |
| F8 | Tidak ada `Ctrl+P` shortcut untuk buka dialog print langsung | P2 | keyboard handler tidak tangani `p`/`P` |

### 4.5 Performa

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| PE1 | **Total bobot gambar 6.1 MB** (PNG polos) — perlu waktu lama di kelas dengan Wi-Fi terbatas | **P0** | `du -sh img/` = 6.1M; `ls -laSh img/` top = 967K |
| PE2 | **Tidak ada `loading="lazy"`** pada `<img>` di slide yang belum aktif | P1 | `grep loading=` → 0 matches |
| PE3 | **Tidak ada `width`/`height` attribute** pada `<img>` — risiko CLS (Cumulative Layout Shift) | P1 | `grep -E '<img[^>]*width'` → 0 matches |
| PE4 | Tidak ada `decoding="async"` pada `<img>` | P2 | `grep decoding=` → 0 matches |
| PE5 | Tidak ada responsive `srcset` — gambar sama di mobile dan proyektor | P2 | `grep srcset=` → 0 matches |
| PE6 | Tidak ada preconnect ke CDN untuk self-host nanti | P2 | `index.html:7-8` hanya Google Fonts |
| PE7 | Google Fonts `display=swap` ada tapi `display=optional` akan lebih baik untuk presentasi (no FOUT) | P2 | `index.html:9` |

### 4.6 Robustness

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| R1 | **`requestFullscreen().catch(()=>{})`** — kegagalan di-swallow, user tidak tahu F tidak bekerja | **P0** | `index.html:715` |
| R2 | Tidak ada fallback jika `requestFullscreen` API tidak tersedia (browser lama) | P1 | tidak ada cek `'requestFullscreen' in element` |
| R3 | Tidak ada error handler untuk `<img>` gagal load | P1 | tidak ada `onerror` pada `<img>` |
| R4 | Tidak ada deteksi jika JS dimatikan — halaman statis tetap tampil tapi navigasi macet | P1 | tidak ada `<noscript>` fallback |
| R5 | Touch swipe threshold 50px — mungkin terlalu sensitif di layar kecil | P2 | `index.html:718` |
| R6 | Key handler override `Space` tanpa cek apakah user di input field (meski tidak ada input field, defensive) | P2 | `index.html:706` |

### 4.7 Maintainability

| # | Celah | Prioritas | Bukti |
|---|---|---|---|
| M1 | **3 simbol SVG tidak terpakai** (`i-box`, `i-palette`, `i-layer`) — dead code | P2 | `grep -c 'i-box'` = 1 (cuma di `<symbol>`); sama untuk `i-palette`, `i-layer` |
| M2 | **1 gambar orphan** (`Screenshot_2026-07-06_194720.png`, 41.2 KB) di disk, tidak dirujuk HTML | P2 | `comm -23` output: `Screenshot_2026-07-06_194720.png` |
| M3 | Single-file 724-baris HTML — sulit maintain saat slide bertambah | P2 | struktur keseluruhan inline |
| M4 | Tidak ada `data-slide` attributes untuk identifikasi slide yang stabil | P2 | `<section class="slide">` tanpa pengenal |
| M5 | CSS keyframes & kelas tidak dikelompokkan per section | P2 | file inline |
| M6 | Tidak ada file terpisah untuk CSS/JS jika proyek akan di-extend | P2 | semua inline |

---

## 5. Ringkasan Prioritas

### 🔴 P0 — Harus segera (10 item)
- **D1**: Buat `README.md` (cara jalankan, struktur, target audiens)
- **D2**: Buat `.gitignore` (minimal: `.vscode/`, `*.log`, `.DS_Store`)
- **B1**: Tambah `meta name="description"`
- **B2**: Tambah favicon
- **B4**: Tambah fallback font stack di CSS
- **A1**: Tambah `aria-live="polite"` di `<main class="stage">`
- **A2**: Tambah focus management saat `goTo(i)`
- **F1**: Sync `window.location.hash` di `update()` dan baca di awal
- **PE1**: Konversi gambar ke WebP/JPEG + kompres (target ≤ 1.5 MB total)
- **R1**: Beri feedback user saat fullscreen gagal (toast/log)

### 🟡 P1 — Penting (12 item)
- B3 (OG tags), B5 (service worker), A3 (skip-link), A4 (aria-label ruler ticks), F2 (presenter mode), F3 (slide overview), F7 (print stylesheet), PE2 (`loading="lazy"`), PE3 (`width`/`height` img), R2 (fullscreen API cek), R3 (`<img>` onerror), R4 (noscript fallback)

### 🟢 P2 — Nice-to-have (22 item)
- D3 (komentar header), B6 (port hardcode), B7 (script serve), B8 (initial commit), A5 (`dir="ltr"`), A6 (switch device), A7 (`prefers-color-scheme`), F4 (progress dots), F5 (pause-on-hover), F6 (toggle tema), F8 (`Ctrl+P` shortcut), PE4 (`decoding="async"`), PE5 (`srcset`), PE6 (preconnect CDN), PE7 (`display=optional`), R5 (swipe threshold), R6 (key handler defensive), M1 (3 unused SVG), M2 (1 orphan img), M3 (single-file), M4 (`data-slide`), M5 (CSS grouping), M6 (pisah file)

---

## 6. Rekomendasi Langkah Lanjutan

1. **Iterasi 1 (P0 saja)**:
   - Buat README minimal + `.gitignore`
   - Tambah favicon (1 baris link)
   - Tambah `meta description`
   - Tambah `aria-live` + focus mgmt
   - Sync hash URL
   - Konversi gambar ke WebP (gunakan `cwebp` atau `imagemin`)
   - Toast/error untuk fullscreen failure

2. **Iterasi 2 (P1)**:
   - Print stylesheet
   - `loading="lazy"` + `width`/`height` pada img
   - OG tags untuk share preview
   - Skip-link + aria-label untuk ruler ticks

3. **Iterasi 3 (P2)**:
   - Bersihkan dead code (3 SVG symbols, 1 orphan image)
   - Pisahkan CSS/JS ke file terpisah
   - Tambah toggle tema
   - Tambah presenter mode

4. **Pertimbangan arsitektur**: jika slide akan bertambah (>30), pertimbangkan generator statis (Eleventy/Hugo) atau pisahkan slide ke file JSON + render dengan template.

---

## 7. Lampiran: Dump Verifikasi

### 7.1 HTTP responses
```
GET /              → 200 text/html 37318 bytes (6.4ms)
GET /index.html    → 200 text/html 37318 bytes (7.2ms)
HEAD /             → HTTP/1.0 200 OK, Server: SimpleHTTP/0.6 Python/3.12.3
HEAD /index.html   → HTTP/1.0 200 OK, Content-Length: 37318
```

### 7.2 Asset status
```
200  size=357650   img/Screenshot_2026-07-06_194611.png
200  size=948881   img/Screenshot_2026-07-06_194737.png
200  size=904234   img/Screenshot_2026-07-06_194749.png
200  size=887394   img/Screenshot_2026-07-06_194824.png
200  size=989195   img/Screenshot_2026-07-06_194848.png
200  size=640136   img/Screenshot_2026-07-06_194902.png
200  size=31442    img/Screenshot_2026-07-06_194920.png
200  size=495736   img/Screenshot_2026-07-06_194937.png
200  size=543083   img/Screenshot_2026-07-06_194958.png
200  size=513700   img/Screenshot_2026-07-07_085821.png
                                          ─────────────
                                  Total = 6,094,449 bytes ≈ 6.1 MB
```

### 7.3 DOM ID consistency
```
JS getElementById('curNum'), ('fsBtn'), ('nextBtn'), ('prevBtn'),
   ('rulerFill'), ('rulerPointer'), ('rulerTrack')
HTML id="curNum"      ✓ line 306
HTML id="fsBtn"       ✓ line 309
HTML id="nextBtn"     ✓ line 310
HTML id="prevBtn"     ✓ line 308
HTML id="rulerFill"   ✓ line 640
HTML id="rulerPointer"✓ line 641
HTML id="rulerTrack"  ✓ line 639
→ diff: identical
```

### 7.4 JS balance
```
Open braces: 20    Close braces: 20
Open parens: 78    Close parens: 78
Open brackets: 5   Close brackets: 5
IIFE closes with: })();   (line 723)
```

### 7.5 Server lifecycle
- Started: `nohup python3 -u -m http.server 8080 --bind 127.0.0.1` → PID 1125
- Log: `Serving HTTP on 127.0.0.1 port 8080 ...`
- Stopped: akan dimatikan di langkah 7 ferment

---

*Akhir laporan.*
