# HERTS / HERTZ — Website Dashboard Pemantauan Jantung & Paru (EIT + ECG)

Antarmuka web untuk sistem pemantauan jantung dan paru berbasis **Electrical Impedance Tomography (EIT)** dengan tambahan **ECG**, dikembangkan sebagai bagian dari capstone Teknik Biomedis (UGM). Website memiliki dua portal: **pasien** dan **dokter**.

> **Status: prototipe frontend.** Semua data adalah **data dummy fiktif**, belum ada autentikasi, dan belum terhubung ke backend/database. Tahap berikutnya: integrasi **Supabase**.
>
> **Catatan penamaan:** merek yang tampil di UI, `<title>`, dan komentar kode adalah **HERTS** ("High-speed Electrical Reconstruction Tomography System"). Nama project/capstone ini disebut **HERTZ** di dokumentasi lain. Penamaan di kode sengaja tidak diubah.

## Teknologi

| Lapisan | Yang dipakai |
|---|---|
| Markup | HTML5 statis (14 halaman) |
| Gaya | CSS biasa (3 file) dengan design token, tanpa preprocessor |
| Logika | JavaScript vanilla (ES5-style, tanpa modul/bundler), 3 file eksternal + skrip inline per halaman |
| Visualisasi | `<canvas>` 2D (citra EIT ilustratif) dan SVG yang dibangun lewat JS (strip ECG) di `doctor-session-detail.html`; sisi pasien memakai gambar statis (`eit-circle.png`, `eit-recorded.png`, `ecg-strip.png`) |
| Font | IBM Plex Sans via Google Fonts |
| Build / package manager | **Tidak ada** (tidak ada `package.json`, framework, atau langkah build) |

## Struktur folder

```
.
├── index.html                      Beranda / landing page
├── information.html                Information & Guide
├── login.html                      Pemilihan peran (pasien / dokter)
│
├── patient-signin.html             Sign in pasien
├── patient-monitoring.html         Monitoring pasien
├── patient-history.html            Riwayat sesi pasien
├── patient-session-detail.html     Detail sesi (?session=SES-001)
├── patient-profile.html            Profil pasien
│
├── doctor-signin.html              Sign in dokter
├── doctor-overview.html            Overview dokter
├── doctor-patients.html            Daftar pasien
├── doctor-patient-detail.html      Detail pasien (?patient=P001)
├── doctor-session-detail.html      Detail sesi (?session=SES-018), canvas EIT + ECG
├── doctor-profile.html             Profil dokter
│
└── assets/
    ├── patient-portal.css          Sistem visual utama (dipakai portal pasien DAN dokter)
    ├── doctor-portal.css           Tambahan gaya portal dokter
    ├── signin.css                  Gaya halaman sign in
    ├── sessions.js                 Data dummy sesi pasien (HERTS_SESSIONS)
    ├── doctor.js                   Data dummy dokter/pasien/sesi + shell header/footer portal dokter
    ├── signin.js                   Validasi form sign in + navigasi prototipe
    └── *.png / *.jpg               Logo, ikon, ilustrasi, gambar EIT/ECG
```

Semua tautan antarhalaman dan referensi aset memakai **path relatif**, dengan semua halaman HTML berada di root. Karena itu folder bisa dipindah atau dibuka di mana saja tanpa konfigurasi.

## Alur halaman

- `index.html` → `information.html` / `login.html`
- `login.html` → `patient-signin.html` atau `doctor-signin.html`
- Sign in pasien → `patient-monitoring.html` ⇄ `patient-history.html` → `patient-session-detail.html?session=…`; `patient-profile.html` dari header
- Sign in dokter → `doctor-overview.html` ⇄ `doctor-patients.html` → `doctor-patient-detail.html?patient=…` → `doctor-session-detail.html?session=…`; `doctor-profile.html` dari header (di-render lewat `assets/doctor.js`)

## Menjalankan secara lokal

Website ini statis, jadi tidak ada instalasi dan tidak ada build.

**Cara cepat:** buka `index.html` langsung di browser (double-click).

**Disarankan:** jalankan server statis lokal dari folder ini, agar perilakunya mendekati hosting sungguhan:

```bash
# Python 3
python3 -m http.server 8000
# lalu buka http://localhost:8000
```

atau `npx serve .` (butuh Node.js) atau ekstensi *Live Server* di VS Code.

### Koneksi internet

Satu-satunya dependensi eksternal adalah **Google Fonts (IBM Plex Sans)**, dimuat lewat `fonts.googleapis.com` dan `fonts.gstatic.com`. Tanpa internet, website tetap berfungsi penuh; hanya font yang jatuh ke font sistem (`system-ui, Segoe UI, Roboto, sans-serif`), sehingga tipografi sedikit berbeda. Tidak ada library JavaScript/CSS dari CDN.

## Data & kredensial

- **Tidak ada** password, API key, token, atau file konfigurasi sensitif di repository ini.
- Data dummy ada di dua tempat, dan keduanya diberi komentar *fictional sample data*:
  - `assets/sessions.js` → `HERTS_SESSIONS` (sesi pasien)
  - `assets/doctor.js` → `HD_DOCTOR`, `HD_PATIENTS`, `HD_SESSIONS`, `HD_TODAY`
- Nama "Budi Artanto" pada header portal pasien ditulis langsung di HTML masing-masing halaman pasien.
- Saat integrasi Supabase: jangan menaruh `service_role` key di kode frontend. Hanya `anon` key + Row Level Security yang aman berada di sisi klien. `.gitignore` sudah mengecualikan `.env*` (kecuali `.env.example`).

## Titik integrasi backend yang sudah ditandai di kode

| Lokasi | Catatan |
|---|---|
| `assets/signin.js` | Saat ini **tanpa autentikasi**: email valid + password apa pun diterima, lalu redirect ke `data-success-url`. Ada `TODO (backend integration)`. |
| `assets/doctor.js` | `TODO (backend)`: ganti `HD_PATIENTS` / `HD_SESSIONS` dengan pemanggilan API yang dibatasi pada dokter yang login. |
| `assets/sessions.js` | Format baris: `[id, tanggal, jam mulai, durasi (menit), rekaman tersedia, heart rate]`. |
| `assets/doctor.js` | Format `HD_SESSIONS`: `[id sesi, id pasien, tanggal, jam mulai, durasi, status: active\|completed\|stopped]`. |

Halaman detail membaca parameter lewat query string (`?session=`, `?patient=`).

## Masalah yang diketahui / belum diselesaikan

1. **Tautan "Create Account" rusak (disengaja sebagai placeholder):** `patient-signin.html` → `patient-register.html` dan `doctor-signin.html` → `doctor-register.html`. Kedua halaman belum ada (sudah ditandai di komentar HTML).
2. **Email kontak placeholder:** `XXXXXXX@gmail.com` di footer semua halaman (juga di template footer `assets/doctor.js`) perlu diganti email nyata. Email `*@example.com` pada profil adalah data dummy.
3. **Tidak ada guard autentikasi:** semua halaman dapat dibuka langsung lewat URL tanpa sign in; portal dokter dan pasien belum dipisahkan oleh sesi.
4. **`assets/eit-frame.png` tidak dirujuk** oleh HTML, CSS, maupun JS mana pun. File dipertahankan karena belum dipastikan tidak terpakai.
5. **Belum diuji di browser.** Pemeriksaan yang sudah dilakukan: seluruh referensi lokal diverifikasi, sintaks semua JS (eksternal dan inline) divalidasi, dan semua file terlayani lewat server lokal. Tampilan, canvas, dan interaksi perlu dicek manual di browser.
