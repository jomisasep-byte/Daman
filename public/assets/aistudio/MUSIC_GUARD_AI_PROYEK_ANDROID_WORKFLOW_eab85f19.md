# MUSIC GUARD AI — Proyek Android dengan Workflow Lengkap

**Versi dokumen:** 1.0  
**Platform target:** Android APK  
**Desain:** Cyber / futuristik  
**Fitur:** Pemindaian video, pemindaian audio, riwayat, dan laporan  
**Status:** Dokumen spesifikasi dan panduan implementasi

---

## 1. Tujuan proyek

MUSIC GUARD AI adalah aplikasi Android untuk membantu kreator YouTube memeriksa musik yang terdapat di dalam video atau file audio sebelum konten dipublikasikan.

Aplikasi membantu:
- memilih file video atau audio dari perangkat;
- mengekstrak audio dari video;
- membagi audio menjadi beberapa segmen;
- mengenali kandidat lagu melalui layanan pengenalan musik resmi;
- mencatat judul lagu, artis, dan waktu kemunculan jika tersedia;
- mencocokkan hasil dengan catatan lisensi milik pengguna;
- memberi kategori risiko dan rekomendasi tindakan;
- menyimpan riwayat pemeriksaan;
- mengekspor laporan Markdown dan JSON.

**Batasan penting:** aplikasi ini bukan pemindai Content ID YouTube dan tidak dapat menjamin video bebas klaim atau blokir. Pengenalan lagu hanya mengidentifikasi kemungkinan lagu; status hak penggunaan harus diperiksa melalui lisensi dan YouTube Studio Checks.

Aplikasi tidak mengubah pitch, kecepatan, noise, atau karakter audio untuk menghindari sistem deteksi hak cipta.

## 2. Pilihan teknologi

| Bagian | Teknologi |
|---|---|
| Bahasa Android | Kotlin |
| Antarmuka | Jetpack Compose |
| Desain | Dark cyber, aksen neon cyan/ungu |
| Pemilihan media | Android Photo Picker / Storage Access Framework |
| Pemrosesan media | MediaExtractor / MediaCodec atau FFmpeg melalui integrasi yang sesuai |
| Penyimpanan riwayat | Room Database |
| Preferensi | DataStore |
| Pengenalan musik | API resmi seperti AudD atau ACRCloud |
| Laporan | Markdown dan JSON |
| Build | Gradle + Android Studio |

Gunakan versi Android Gradle Plugin, Kotlin, Compose, SDK, dan dependensi yang kompatibel satu sama lain. Kunci versi dependensi pada proyek agar build dapat direproduksi.

## 3. Workflow utama aplikasi

```text
[Dashboard MUSIC GUARD AI]
            |
            v
    [Pilih Video / Audio]
            |
            v
      [Validasi File]
            |
            v
    [Baca Durasi & Audio]
            |
            v
 [Ekstraksi Audio jika Video]
            |
            v
 [Segmentasi Audio + Timestamp]
            |
            v
 [Music Recognition API Resmi]
            |
            v
 [Normalisasi Hasil Pengenalan]
            |
            +--------------------------+
            |                          |
            v                          v
    [Catatan Lisensi]           [Kandidat Lagu]
            |                          |
            +------------+-------------+
                         |
                         v
               [Analisis Risiko]
                         |
                         v
                [Hasil Pemindaian]
                         |
              +----------+----------+
              |          |          |
              v          v          v
           [History] [Laporan MD] [Laporan JSON]
                         |
                         v
               [YouTube Studio Checks]
```

### Langkah 1 — Pilih file

Pengguna memilih:
- video: MP4, MOV, MKV, atau format lain yang didukung perangkat;
- audio: MP3, WAV, M4A, AAC, FLAC, atau format lain yang didukung.

Aplikasi harus membaca file melalui URI dari Android picker. Jangan menganggap URI selalu merupakan path filesystem.

### Langkah 2 — Validasi

Periksa:
- file dapat dibaca;
- ukuran file masuk batas yang ditetapkan aplikasi;
- media memiliki track audio;
- durasi media valid;
- format dapat diproses.

Jika tidak ada audio, tampilkan pesan yang jelas dan jangan lanjutkan pemindaian.

### Langkah 3 — Ekstraksi audio

Jika input berupa video, ekstrak track audio tanpa mengubah file video asli. Buat file sementara di cache aplikasi dan hapus setelah proses selesai.

Contoh perintah konseptual FFmpeg:

```bash
ffmpeg -i input.mp4 -vn -ac 1 -ar 44100 -c:a pcm_s16le extracted.wav
```

Perintah ini hanya contoh pemrosesan media. Integrasi FFmpeg ke APK harus memakai paket Android yang lisensi dan kompatibilitas ABI-nya telah diperiksa. Alternatifnya, gunakan Android MediaExtractor dan MediaCodec.

### Langkah 4 — Segmentasi

Bagi audio menjadi potongan yang sesuai dengan batas layanan pengenalan musik. Simpan:
- nomor segmen;
- waktu mulai;
- waktu selesai;
- lokasi file sementara;
- status pemrosesan.

Jangan menetapkan durasi segmen secara permanen sebelum memeriksa dokumentasi dan batas API penyedia yang dipilih.

### Langkah 5 — Pengenalan lagu

Kirim segmen audio ke API resmi menggunakan HTTPS. Simpan hasil seperti:
- judul;
- artis;
- album jika tersedia;
- tautan metadata jika disediakan;
- confidence atau skor kecocokan jika disediakan;
- timestamp segmen;
- status permintaan API.

API pengenalan musik bukan database lisensi dan bukan database Content ID YouTube.

### Langkah 6 — Pemeriksaan lisensi

Pengguna dapat memasukkan informasi lisensi musik:
- judul dan artis;
- sumber musik;
- jenis lisensi;
- platform yang diizinkan;
- izin monetisasi;
- wilayah;
- masa berlaku;
- kewajiban atribusi;
- lokasi bukti pembelian atau dokumen izin.

Aplikasi tidak boleh menandai musik aman hanya karena nama lagu ditemukan dalam daftar. Cakupan lisensi harus sesuai dengan penggunaan sebenarnya.

### Langkah 7 — Analisis risiko

Gunakan empat kategori:

| Kategori | Makna | Tindakan |
|---|---|---|
| MERAH | Lagu teridentifikasi, bukti izin belum ada atau tidak sesuai | Jangan publikasikan sebelum hak penggunaan jelas |
| KUNING | Lagu teridentifikasi, lisensi perlu diperiksa | Verifikasi platform, monetisasi, wilayah, dan masa berlaku |
| HIJAU | Bukti izin yang relevan tersedia dan telah diperiksa | Tetap lakukan YouTube Checks |
| ABU-ABU | Lagu tidak teridentifikasi atau API gagal | Bukan bukti bebas hak cipta; periksa sumber musik secara manual |

**Jangan menyamakan “tidak ditemukan” dengan “bebas copyright”.**

### Langkah 8 — Riwayat

Simpan data lokal:
- nama file;
- tanggal pemeriksaan;
- durasi;
- jumlah segmen;
- kandidat lagu;
- status risiko;
- lokasi laporan.

Sediakan opsi hapus riwayat. Jangan menghapus file asli pengguna.

### Langkah 9 — Laporan

Ekspor:
- Markdown (`.md`) untuk laporan yang mudah dibaca;
- JSON (`.json`) untuk pemrosesan lanjutan.

Laporan harus memuat waktu pemeriksaan, file, kandidat lagu, timestamp, status lisensi, risiko, alasan, dan rekomendasi.

## 4. Desain antarmuka cyber

### Dashboard

Elemen:
- logo MUSIC GUARD AI;
- indikator status sistem;
- tombol Scan Video;
- tombol Scan Audio;
- kartu Total Scans;
- kartu Risk Alerts;
- daftar pemindaian terbaru;
- navigasi Dashboard, History, Reports, Settings.

### Halaman pemindaian

Elemen:
- nama file dan ukuran;
- durasi;
- waveform atau progress visual;
- tahap proses yang sedang berjalan;
- daftar segmen;
- tombol Cancel;
- status API.

### Halaman hasil

Elemen:
- ringkasan status risiko;
- daftar kandidat lagu;
- timestamp kemunculan;
- informasi lisensi;
- alasan status;
- rekomendasi tindakan;
- tombol Export MD;
- tombol Export JSON.

### Gaya visual

- latar belakang hitam atau biru gelap;
- aksen cyan dan ungu;
- kartu dengan garis tipis;
- animasi ringan saat pemindaian;
- waveform yang bergerak selama proses;
- hindari animasi berlebihan yang memperlambat perangkat.

## 5. Struktur proyek Android

```text
music-guard-ai/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── gradle/
│   └── libs.versions.toml
├── app/
│   ├── build.gradle.kts
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   ├── java/com/musicguard/ai/
│       │   │   ├── MainActivity.kt
│       │   │   ├── MusicGuardApp.kt
│       │   │   ├── navigation/
│       │   │   │   └── AppNavigation.kt
│       │   │   ├── ui/
│       │   │   │   ├── theme/
│       │   │   │   ├── dashboard/
│       │   │   │   ├── scanner/
│       │   │   │   ├── result/
│       │   │   │   ├── history/
│       │   │   │   └── settings/
│       │   │   ├── data/
│       │   │   │   ├── local/
│       │   │   │   ├── model/
│       │   │   │   └── repository/
│       │   │   ├── media/
│       │   │   │   ├── MediaValidator.kt
│       │   │   │   ├── AudioExtractor.kt
│       │   │   │   └── AudioSegmenter.kt
│       │   │   ├── recognition/
│       │   │   │   ├── MusicRecognitionApi.kt
│       │   │   │   └── RecognitionRepository.kt
│       │   │   ├── risk/
│       │   │   │   └── RiskAnalyzer.kt
│       │   │   └── report/
│       │   │       ├── MarkdownExporter.kt
│       │   │       └── JsonExporter.kt
│       │   └── res/
│       └── test/
├── README.md
└── PROJECT_WORKFLOW.md
```

## 6. Model data utama

Contoh model konseptual Kotlin:

```kotlin
data class ScanRecord(
    val id: String,
    val fileName: String,
    val mediaType: String,
    val durationMs: Long,
    val scannedAt: Long,
    val status: String
)

data class MusicMatch(
    val title: String?,
    val artist: String?,
    val album: String?,
    val startMs: Long,
    val endMs: Long,
    val confidence: Double?
)

data class LicenseRecord(
    val title: String,
    val artist: String?,
    val source: String,
    val allowedPlatforms: List<String>,
    val monetizationAllowed: Boolean?,
    val territory: String?,
    val validUntil: String?,
    val proofUri: String?
)

data class ScanReport(
    val scanId: String,
    val matches: List<MusicMatch>,
    val riskStatus: String,
    val reasons: List<String>,
    val recommendations: List<String>
)
```

Model di atas merupakan rancangan awal. Validasi, migrasi database, dan serialisasi perlu ditambahkan pada implementasi.

## 7. Keamanan API key

Jangan menaruh API secret permanen di source code APK. Nilai yang ditanam di APK dapat diekstrak.

Pilihan yang disarankan:
1. Gunakan backend milik pengembang untuk menyimpan secret.
2. APK mengirim sampel audio ke backend melalui HTTPS.
3. Backend memanggil layanan pengenalan musik.
4. Backend mengembalikan metadata hasil.
5. Terapkan autentikasi, rate limit, batas ukuran, dan kebijakan retensi file.

Untuk prototipe pribadi, pengguna dapat memasukkan API token sendiri, tetapi tetap pahami bahwa penyimpanan token di perangkat bukan perlindungan sempurna. Jangan membagikan APK yang berisi secret.

## 8. Izin dan privasi Android

Gunakan Android Photo Picker atau Storage Access Framework agar pengguna memilih file secara eksplisit. Hindari meminta akses seluruh penyimpanan jika tidak diperlukan.

Jika file dikirim ke layanan pihak ketiga:
- tampilkan pemberitahuan sebelum upload;
- jelaskan jenis data yang dikirim;
- sediakan pilihan batal;
- gunakan HTTPS;
- hapus file sementara setelah selesai;
- jelaskan kebijakan retensi penyedia API.

## 9. Persiapan build APK

### Perangkat pengembangan

Disarankan:
- komputer Windows, macOS, atau Linux;
- Android Studio versi stabil;
- Android SDK yang sesuai dengan konfigurasi proyek;
- JDK yang disyaratkan versi Android Gradle Plugin;
- koneksi internet untuk mengunduh dependensi Gradle.

### Langkah build

1. Ekstrak folder proyek.
2. Buka Android Studio.
3. Pilih **Open** dan arahkan ke folder `music-guard-ai`.
4. Tunggu Gradle Sync selesai.
5. Perbaiki jika ada pesan SDK atau JDK yang belum tersedia.
6. Jalankan unit test.
7. Jalankan aplikasi pada emulator atau perangkat Android.
8. Pilih **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
9. Setelah build sukses, ambil APK debug dari:

```text
app/build/outputs/apk/debug/app-debug.apk
```

Untuk distribusi, buat signed release APK melalui **Build → Generate Signed Bundle / APK**. Simpan keystore dan password dengan aman; jangan memasukkannya ke repositori publik.

### Build melalui terminal

Jika Gradle wrapper tersedia:

```bash
./gradlew assembleDebug
```

Pada Windows:

```bat
gradlew.bat assembleDebug
```

APK debug biasanya berada di:

```text
app/build/outputs/apk/debug/app-debug.apk
```

## 10. Catatan khusus build di Android/Termux

Membangun aplikasi Android native langsung di Termux lebih rumit karena memerlukan Android SDK, build tools, JDK, dan konfigurasi Gradle yang cocok.

Jalur paling stabil adalah:
- menulis dan mengelola source di perangkat Android jika diperlukan;
- membangun APK dengan Android Studio di komputer; atau
- menggunakan CI seperti GitHub Actions setelah workflow build disiapkan.

Jangan menganggap folder source otomatis menjadi APK. APK baru tersedia setelah proses build berhasil.

## 11. Pengujian

Uji skenario berikut sebelum rilis:

- video tanpa track audio;
- audio pendek dan panjang;
- format media yang tidak didukung;
- file rusak;
- koneksi internet terputus;
- API key salah;
- API rate limit;
- API timeout;
- lagu dikenali;
- lagu tidak dikenali;
- lisensi ada dan cakupannya sesuai;
- lisensi ada tetapi tidak mencakup monetisasi;
- ekspor laporan;
- pemulihan riwayat setelah aplikasi ditutup;
- pembatalan pemindaian;
- penghapusan file sementara.

Pastikan kegagalan API menghasilkan status ABU-ABU, bukan HIJAU.

## 12. Checklist rilis

- [ ] Dashboard cyber tampil baik.
- [ ] Pemilihan video berfungsi.
- [ ] Pemilihan audio berfungsi.
- [ ] Validasi media berjalan.
- [ ] Ekstraksi audio berjalan.
- [ ] Segmentasi dan timestamp benar.
- [ ] Integrasi API resmi berjalan.
- [ ] API key tidak ditanam sebagai secret di APK publik.
- [ ] Analisis risiko menampilkan alasan.
- [ ] Riwayat tersimpan secara lokal.
- [ ] Ekspor Markdown berfungsi.
- [ ] Ekspor JSON berfungsi.
- [ ] File sementara dibersihkan.
- [ ] Privasi upload dijelaskan.
- [ ] Pengujian perangkat selesai.
- [ ] Signed release APK berhasil dibuat.

## 13. Pemeriksaan akhir di YouTube

Setelah pra-pemeriksaan aplikasi:

1. Upload video ke YouTube Studio sebagai Private atau Unlisted jika sesuai kebutuhan.
2. Tunggu proses Checks selesai.
3. Periksa detail copyright jika ada klaim.
4. Periksa bagian video yang cocok, pemilik klaim, serta wilayah dan kebijakan yang berlaku.
5. Jika tidak memiliki izin, ganti musik atau dapatkan izin yang sesuai.
6. Jika klaim keliru dan hak memang dimiliki, siapkan bukti sebelum mengajukan sengketa.
7. Jangan menganggap hasil aplikasi eksternal sebagai jaminan lolos Content ID.

## 14. Referensi

- YouTube — Cara kerja Content ID: https://support.google.com/youtube/answer/2797370
- YouTube — Memahami klaim hak cipta: https://support.google.com/youtube/answer/6013276
- YouTube — Musik yang aman digunakan: https://support.google.com/youtube/answer/15577610
- Android Developers — Photo Picker: https://developer.android.com/training/data-storage/shared/photopicker
- Android Developers — MediaExtractor: https://developer.android.com/reference/android/media/MediaExtractor
- AudD API: https://audd.io/
- ACRCloud: https://www.acrcloud.com/

---

## 15. Rencana pengerjaan

**Tahap A — Fondasi aplikasi**
- buat proyek Kotlin;
- siapkan tema cyber;
- buat navigasi dan dashboard;
- siapkan pemilih video/audio.

**Tahap B — Mesin media**
- validasi file;
- baca durasi;
- ekstrak audio;
- segmentasi dan timestamp;
- kelola file sementara.

**Tahap C — Pengenalan musik**
- pilih penyedia API;
- implementasikan autentikasi;
- kirim segmen;
- parsing respons;
- tangani timeout dan rate limit.

**Tahap D — Risiko dan lisensi**
- buat database lisensi;
- cocokkan kandidat lagu;
- buat kategori risiko;
- tampilkan alasan dan rekomendasi.

**Tahap E — Riwayat dan laporan**
- simpan scan;
- tampilkan history;
- ekspor Markdown;
- ekspor JSON.

**Tahap F — Pengujian dan APK**
- uji berbagai media;
- uji API gagal;
- uji privasi dan penghapusan cache;
- build debug;
- buat signed release.

**Hasil akhir yang ditargetkan:** aplikasi Android cyber bernama MUSIC GUARD AI yang membantu kreator mengenali musik dalam video/audio, meninjau bukti lisensi, menyimpan riwayat, dan membuat laporan sebelum memublikasikan konten YouTube.
