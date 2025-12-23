# JariBicara: Aplikasi Penerjemah Bahasa Isyarat Real-time

JariBicara adalah sebuah prototipe aplikasi Android yang mampu menerjemahkan Bahasa Isyarat Indonesia (BISINDO) abjad statis menjadi teks secara *real-time* menggunakan kamera perangkat. Aplikasi ini dibangun dengan pendekatan modern, mengintegrasikan *On-Device Machine Learning* dengan UI deklaratif dan layanan *cloud*.

---

## Fitur Utama

- **Terjemahan Real-time**: Menggunakan kamera untuk mendeteksi dan menerjemahkan isyarat tangan secara langsung.
- **Logika Pengetikan Cerdas**: Dilengkapi dengan filter kepercayaan (95%) dan jeda waktu (1 detik) untuk menghasilkan teks yang lebih akurat dan natural, serta menghindari duplikasi.
- **Dukungan Kamera Ganda**: Pengguna dapat dengan mudah beralih antara kamera depan dan belakang.
- **Integrasi Cloud**: Hasil teks dapat diunggah ke Firebase Realtime Database untuk disimpan sebagai riwayat.
- **UI Modern**: Seluruh antarmuka aplikasi dibangun menggunakan Jetpack Compose untuk tampilan yang bersih dan responsif.
- **Deteksi Offline**: Proses deteksi dan klasifikasi berjalan sepenuhnya di perangkat (*on-device*), sehingga tidak memerlukan koneksi internet untuk fungsi utamanya.

---

## Arsitektur & Alur Kerja

Aplikasi ini memiliki arsitektur yang memisahkan antara logika deteksi, logika tampilan, dan layanan eksternal.

#### Diagram Alur Data
Diagram ini menggambarkan bagaimana data gambar dari kamera diolah hingga menjadi teks dan disimpan di *cloud*.

```mermaid
graph TD
    subgraph "Perangkat Android (Aplikasi JariBicara)"
        direction LR

        subgraph "Lapisan Tampilan (View Layer)"
            UI_Main[MainActivity (UI Utama)] --Mulai--> UI_Cam[CameraActivity (UI Kamera)];
        end

        subgraph "Lapisan Logika & Deteksi (Logic & Detection Layer)"
            UI_Cam --Menggunakan--> CamX[CameraX];
            CamX --Image Stream--> HLH[HandLandmarkerHelper];
            HLH --21 Titik Tangan--> SLC[SignLanguageClassifier];
            SLC --Hasil Klasifikasi--> UI_Cam;
        end
        
        UI_Cam --Tombol Upload--> FB_Logic[Logika Upload];

    end

    subgraph "Layanan Cloud"
        FB_DB[(Firebase Realtime DB)];
    end

    FB_Logic --Kirim Data (Teks & Timestamp)--> FB_DB;
    
    style FB_DB fill:#ffa,stroke:#f60,stroke-width:2px
    style UI_Main fill:#dcf,stroke:#609,stroke-width:2px
    style UI_Cam fill:#dcf,stroke:#609,stroke-width:2px
```

---

## Siklus Hidup (Lifecycle) & Manajemen State di Jetpack Compose

Pengelolaan siklus hidup dan *state* di `CameraActivity` sangat penting agar aplikasi efisien dan tidak boros sumber daya.

1.  **`remember`**:
    *   *State* atau data yang bisa berubah di UI, seperti `typedText`, `classificationResult`, dan `cameraSelector`, disimpan menggunakan `remember { mutableStateOf(...) }`. Ini memastikan data tidak hilang saat UI di-render ulang (recomposition).

2.  **`LaunchedEffect`**:
    *   Blok ini digunakan untuk menjalankan proses penyiapan kamera (mengikat `Preview` dan `ImageAnalysis` ke *lifecycle*).
    *   `LaunchedEffect` akan berjalan saat `CameraScreen` pertama kali muncul. Ia juga akan **berjalan ulang secara otomatis** jika *state* `cameraSelector` berubah (misalnya, saat pengguna menekan tombol ganti kamera), sehingga kamera akan me-restart dengan konfigurasi yang baru.

3.  **`DisposableEffect`**:
    *   Ini adalah bagian krusial untuk "membersihkan" sumber daya. Saat pengguna meninggalkan `CameraScreen` (misalnya menekan tombol kembali), blok `onDispose` di dalam `DisposableEffect` akan dieksekusi.
    *   **Potongan Kode Penting (`CameraActivity.kt`):**
        ```kotlin
        DisposableEffect(lifecycleOwner) {
            onDispose {
                cameraExecutor.shutdown() // Mematikan thread kamera
                handLandmarkerHelper.clearHandLandmarker() // Melepaskan model AI dari memori
            }
        }
        ```
    *   Ini sangat penting untuk mencegah kebocoran memori (*memory leak*) dan memastikan kamera dan model AI dilepaskan dengan benar.

---

## Struktur Proyek

```
.
├── app/
│   ├── src/main/
│   │   ├── java/com/example/signdecs/
│   │   │   ├── MainActivity.kt        # Layar utama/pembuka
│   │   │   ├── CameraActivity.kt      # Layar kamera, logika deteksi, dan UI
│   │   │   ├── HandLandmarkerHelper.kt # Kelas bantuan untuk MediaPipe Hand Landmarker
│   │   │   └── SignLanguageClassifier.kt # Kelas untuk menjalankan model .tflite
│   │   ├── assets/
│   │   │   ├── hand_landmarker.task
│   │   │   ├── revised_hand_sign_model.tflite
│   │   │   └── labels.txt
│   │   └── google-services.json       # File konfigurasi Firebase
│   └── build.gradle.kts             # Konfigurasi build dan dependensi
├── trainlab/
│   └── trainlab.ipynb               # Notebook untuk melatih model AI
└── README.md                        # File ini
```

---

## Teknologi yang Digunakan

- **Bahasa**: Kotlin
- **UI**: Jetpack Compose (Modern, Declarative UI)
- **Akses Kamera**: CameraX (Bagian dari Android Jetpack)
- **Deteksi Tangan**: Google MediaPipe (Hand Landmarker)
- **Klasifikasi Isyarat**: Google TensorFlow Lite
- **Database Cloud**: Google Firebase (Realtime Database)
- **Build System**: Gradle

---

## Cara Menjalankan Proyek

1.  *Clone* repositori ini ke komputer Anda.
2.  Buka proyek menggunakan versi terbaru Android Studio.
3.  **Penting**: Unduh file `google-services.json` Anda sendiri dari Firebase Console, dan letakkan di dalam direktori `app/`.
4.  Lakukan *Sync Project with Gradle Files* untuk mengunduh semua dependensi yang diperlukan.
5.  Build dan jalankan aplikasi pada perangkat Android fisik atau emulator.
