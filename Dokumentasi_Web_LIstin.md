Tentu, dengan senang hati. Ini adalah dokumentasi lengkap untuk proyek aplikasi web "List In", yang dirancang seolah-olah sebuah buku panduan untuk teman-teman Anda. Dokumentasi ini mencakup semua aspek yang Anda minta: apa, mengapa, bagaimana, rincian fitur, arsitektur, hingga estimasi waktu pembuatan.

***

# Buku Dokumentasi Aplikasi "List In"

 <!-- Ganti dengan URL logo jika ada -->

**Versi:** 1.0
**Tanggal:** 24 Mei 2024
**Disusun oleh:** Asisten AI

---

## Daftar Isi

1.  [**Bab 1: Pendahuluan - Apa itu List In?**](#bab-1-pendahuluan---apa-itu-list-in)
    *   1.1. Visi & Tujuan
    *   1.2. Untuk Siapa Aplikasi Ini?
    *   1.3. Ringkasan Fitur Utama

2.  [**Bab 2: Arsitektur & Teknologi - Bagaimana Cara Membuatnya?**](#bab-2-arsitektur--teknologi---bagaimana-cara-membuatnya)
    *   2.1. Arsitektur Aplikasi
    *   2.2. Tumpukan Teknologi (Technology Stack)
    *   2.3. Struktur Direktori Proyek

3.  [**Bab 3: Desain Database - Di Mana Data Disimpan?**](#bab-3-desain-database---di-mana-data-disimpan)
    *   3.1. Skema Relasi Database
    *   3.2. Rincian Tabel (`users`, `tasks`, `password_resets`)

4.  [**Bab 4: Rincian Fitur - Kok Bisa Jalan Begini?**](#bab-4-rincian-fitur---kok-bisa-jalan-begini)
    *   4.1. Manajemen Pengguna (Autentikasi)
    *   4.2. Manajemen Tugas (CRUD)
    *   4.3. Dasbor & Laporan Produktivitas
    *   4.4. Notifikasi & Pengalaman Pengguna (UX)
    *   4.5. Fitur Bintang: Bot Manager (AI)

5.  [**Bab 5: Proses & Waktu Pembuatan - Berapa Lama?**](#bab-5-proses--waktu-pembuatan---berapa-lama)
    *   5.1. Tahapan Pengembangan
    *   5.2. Estimasi Waktu

6.  [**Bab 6: Kesimpulan & Arah Pengembangan Selanjutnya**](#bab-6-kesimpulan--arah-pengembangan-selanjutnya)

---

## Bab 1: Pendahuluan - Apa itu List In?

### 1.1. Visi & Tujuan

**List In** adalah aplikasi web manajemen tugas (To-Do List) yang dirancang untuk membantu individu mengatur pekerjaan, proyek, dan tugas harian mereka dengan cara yang sederhana, visual, dan efisien.

**Tujuannya** adalah menyediakan platform yang tidak hanya fungsional untuk mencatat tugas, tetapi juga memberikan wawasan tentang produktivitas pengguna melalui laporan dan visualisasi data. Filosofi utamanya adalah **kesederhanaan yang kuat**: antarmuka yang bersih dan intuitif, namun didukung oleh fitur-fitur canggih di baliknya.

### 1.2. Untuk Siapa Aplikasi Ini?

Aplikasi ini ditujukan untuk:
*   **Pelajar & Mahasiswa:** Mengelola tugas kuliah, jadwal belajar, dan deadline.
*   **Pekerja Lepas (Freelancer):** Melacak proyek dari berbagai klien.
*   **Profesional:** Mengatur tugas-tugas kantor dan target pribadi.
*   **Siapa pun** yang ingin lebih terorganisir dan produktif dalam kehidupan sehari-hari.

### 1.3. Ringkasan Fitur Utama

*   **Manajemen Tugas (CRUD):** Membuat, membaca, memperbarui, dan menghapus tugas dengan mudah.
*   **Autentikasi Aman:** Sistem registrasi, login, dan reset password yang aman.
*   **Dasbor Interaktif:** Visualisasi data tugas secara *real-time* (status, performa).
*   **Laporan Produktivitas:** Analisis mendalam tentang tugas yang selesai dengan kalender interaktif dan unduhan laporan PDF.
*   **Prioritas & Status:** Kategorikan tugas berdasarkan urgensi (Tinggi, Sedang, Rendah) dan progres (Belum Mulai, Dikerjakan, Selesai).
*   **Pencarian & Filter:** Temukan tugas dengan cepat berdasarkan kata kunci, status, prioritas, atau tanggal.
*   **Tema Gelap/Terang:** Pilihan tema untuk kenyamanan visual pengguna.
*   **Bot Manager (AI):** Asisten cerdas untuk mengelola tugas (termasuk perintah batch) melalui percakapan bahasa alami.

---

## Bab 2: Arsitektur & Teknologi - Bagaimana Cara Membuatnya?

### 2.1. Arsitektur Aplikasi

List In dibangun menggunakan arsitektur **Client-Server** monolitik yang umum. Artinya, kode untuk tampilan (frontend) dan logika (backend) berada dalam satu proyek yang sama.

1.  **Client (Frontend):** Ini adalah apa yang pengguna lihat di browser mereka (HTML, CSS, JavaScript). Ketika pengguna mengklik tombol, browser mengirim permintaan ke server.
2.  **Server (Backend):** Ini adalah "otak" aplikasi (ditulis dalam PHP). Server menerima permintaan dari browser, memproses logika (misalnya, menyimpan tugas ke database), dan mengirimkan kembali halaman HTML yang sudah jadi atau data (dalam format JSON) ke browser.
3.  **Database:** Ini adalah "lemari penyimpanan" data (MySQL). Semua informasi seperti data pengguna, tugas, dll., disimpan di sini.

### 2.2. Tumpukan Teknologi (Technology Stack)

Ini adalah daftar "bahan-bahan" yang kita gunakan untuk membangun List In.

| Kategori | Teknologi | Mengapa Dipilih? |
| :--- | :--- | :--- |
| **Backend** | **PHP 8+** | Bahasa yang sangat umum untuk web, mudah dipelajari, didukung oleh hampir semua layanan hosting, dan memiliki banyak library. Cukup kuat untuk skala proyek ini. |
| **Database** | **MySQL/MariaDB** | Sistem database relasional yang andal, gratis, dan cepat. Sempurna untuk data terstruktur seperti pengguna dan tugas yang memiliki hubungan jelas (satu pengguna memiliki banyak tugas). |
| **Frontend** | **HTML5 & CSS3** | Standar fundamental untuk membangun struktur dan gaya halaman web. |
| | **JavaScript (Vanilla)** | Digunakan untuk interaktivitas di sisi klien tanpa perlu framework berat. Contohnya untuk membuka/menutup popup, validasi form, dan menangani AJAX. |
| **Library** | **Chart.js** | Library JavaScript untuk membuat diagram (grafik batang, garis) yang indah dan interaktif di dasbor dan laporan. |
| | **Flatpickr** | Library JavaScript untuk membuat *date picker* (pemilih tanggal) yang ringan dan modern. |
| | **TCPDF** | Library PHP untuk membuat file PDF secara dinamis untuk fitur "Unduh Laporan". |
| | **PHPMailer** | Library PHP untuk mengirim email, digunakan pada fitur "Lupa Password". |
| **API Eksternal** | **Google Gemini API** | Digunakan untuk "otak" dari fitur Bot Manager. Gemini menerima teks percakapan dan memahaminya, lalu memberikan saran dalam format JSON. |
| **Lingkungan** | **LARAGON** | Paket perangkat lunak yang memudahkan menjalankan server Apache, PHP, dan MySQL di komputer lokal untuk pengembangan. |

### 2.3. Struktur Direktori Proyek

Memahami struktur file membantu menavigasi kode dengan mudah.

```
/list-in-project/
├── css/
│   ├── style.css         # Styling utama aplikasi
│   └── auth.css          # Styling khusus halaman login/register
├── images/
│   └── ...               # Aset gambar (logo, placeholder, dll.)
├── includes/
│   ├── db.php            # Skrip koneksi database
│   ├── header.php        # Bagian atas setiap halaman (termasuk menu, notif)
│   ├── sidebar.php       # Menu navigasi samping
│   ├── footer.php        # Bagian bawah setiap halaman (termasuk skrip JS)
│   └── task_helper.php   # Fungsi untuk merender kartu tugas
├── uploads/
│   └── profile_pictures/ # Folder untuk menyimpan foto profil pengguna
├── vendor/               # Library dari Composer (PHPMailer, TCPDF, dll.)
│
├── dashboard.php         # Halaman dasbor utama
├── manajemen_tugas.php   # Halaman untuk mengelola tugas aktif
├── tambah_tugas.php      # Form untuk menambah tugas
├── riwayat.php           # Halaman untuk melihat riwayat tugas
├── laporan.php           # Halaman laporan produktivitas
├── bot_manager.php       # Halaman antarmuka chatbot AI
├── gemini_handler.php    # Backend (server) untuk chatbot AI
├── profil.php            # Halaman profil pengguna
├── login.php             # Halaman login
├── register.php          # Halaman registrasi
├── logout.php            # Skrip untuk proses logout
├── config.php            # File konfigurasi sentral (DB, API Key, dll.)
└── ...                   # File-file pendukung lainnya
```

---

## Bab 3: Desain Database - Di Mana Data Disimpan?

### 3.1. Skema Relasi Database

Database adalah fondasi aplikasi. Kita memiliki tiga tabel utama yang saling berhubungan.

*   Seorang `user` dapat memiliki **banyak** `tasks`.
*   Relasi ini disebut **One-to-Many**. Ini dihubungkan oleh `user_id` di tabel `tasks` yang merujuk ke `id` di tabel `users`.

### 3.2. Rincian Tabel

#### 1. Tabel `users`
Menyimpan semua informasi tentang pengguna.
```sql
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) NOT NULL, -- Dibuat NOT NULL karena login Google dihapus
  `profile_image` varchar(255) DEFAULT 'images/placeholder-profile.png',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `email_verified_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`)
);
```
*   `password`: Kolom ini **tidak** menyimpan password asli, melainkan hasil *hash* yang dibuat oleh fungsi `password_hash()` di PHP. Ini adalah praktik keamanan standar.

#### 2. Tabel `tasks`
Tabel inti yang menyimpan semua tugas dari semua pengguna.
```sql
CREATE TABLE `tasks` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` text,
  `priority` enum('Low','Medium','High') NOT NULL DEFAULT 'Medium',
  `status` enum('Not Started','In Progress','Completed') NOT NULL DEFAULT 'Not Started',
  `due_date` date DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `last_overdue_notif_sent` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  CONSTRAINT `tasks_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
);
```
*   `user_id`: Kunci asing (foreign key) yang menghubungkan tugas ini ke pemiliknya di tabel `users`.
*   `ON DELETE CASCADE`: Jika seorang pengguna dihapus, semua tugas miliknya akan otomatis terhapus juga.

#### 3. Tabel `password_resets`
Tabel sementara untuk fitur "Lupa Password".
```sql
CREATE TABLE `password_resets` (
  `email` varchar(100) NOT NULL,
  `token` varchar(255) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`email`)
);
```
*   `token`: Kode acak dan unik yang dikirim ke email pengguna. Token ini memiliki masa berlaku (misalnya, 1 jam) untuk keamanan.

---

## Bab 4: Rincian Fitur - Kok Bisa Jalan Begini?

### 4.1. Manajemen Pengguna (Autentikasi)

*   **Registrasi & Login:**
    *   **Bagaimana?** `register.php` menerima data, PHP melakukan validasi, lalu `password_hash()` mengenkripsi password sebelum disimpan ke DB. `login.php` memverifikasi email dan password (menggunakan `password_verify()`) lalu membuat *session* (`$_SESSION['user_id'] = ...`) untuk menandai bahwa pengguna sudah masuk.
    *   **Mengapa?** Hashing password sangat penting untuk keamanan. Jika database bocor, password pengguna tidak akan terekspos dalam bentuk teks biasa. Session adalah cara standar untuk menjaga status login pengguna saat berpindah halaman.

*   **Reset Password:**
    *   **Bagaimana?** Pengguna memasukkan email di `forgot_password.php`. PHP membuat token acak, menyimpannya di tabel `password_resets`, lalu menggunakan **PHPMailer** untuk mengirim link reset ke email pengguna. Saat link diklik, `reset_password.php` memverifikasi token dan masa berlakunya sebelum mengizinkan pengguna membuat password baru.
    *   **Mengapa?** Proses berbasis token ini memastikan hanya orang yang memiliki akses ke email tersebut yang dapat mereset password.

### 4.2. Manajemen Tugas (CRUD)

CRUD adalah singkatan dari Create, Read, Update, Delete.

*   **Create (Membuat):**
    *   **Bagaimana?** Form di `tambah_tugas.php` mengirim data via POST. PHP memvalidasi data dan menjalankan query SQL `INSERT INTO tasks ...`.
    *   **Kok bisa?** Query `INSERT` adalah perintah standar SQL untuk menambahkan baris baru ke tabel database.

*   **Read (Membaca):**
    *   **Bagaimana?** Halaman seperti `manajemen_tugas.php` menjalankan query `SELECT * FROM tasks WHERE user_id = ? ...`. PHP mengambil hasilnya, lalu fungsi `render_task_card()` di `task_helper.php` mengubah data tersebut menjadi kartu HTML yang rapi.
    *   **Mengapa?** `WHERE user_id = ?` sangat penting agar pengguna hanya bisa melihat tugas miliknya sendiri, bukan milik orang lain.

*   **Update (Memperbarui):**
    *   **Bagaimana?** `edit_tugas.php` mengambil data tugas yang ada untuk mengisi form. Setelah disimpan, PHP menjalankan query `UPDATE tasks SET ... WHERE id = ?`. Di halaman manajemen, ada juga update status cepat via **AJAX**: JavaScript mengirim permintaan ke `ajax_update_task_status.php` tanpa perlu me-refresh halaman.
    *   **Mengapa?** AJAX (Asynchronous JavaScript and XML) memberikan pengalaman pengguna yang lebih baik karena perubahan bisa terjadi secara instan.

*   **Delete (Menghapus):**
    *   **Bagaimana?** Tombol hapus mengirimkan `action=delete` dan `task_id` ke server. PHP kemudian menjalankan query `DELETE FROM tasks WHERE id = ?`.
    *   **Kok bisa?** JavaScript memberikan prompt `confirm()` untuk memastikan pengguna tidak sengaja menghapus tugas.

### 4.3. Dasbor & Laporan Produktivitas

*   **Bagaimana?**
    1.  **Statistik Dasbor:** PHP menggunakan query SQL agregat seperti `SELECT status, COUNT(*) FROM tasks ... GROUP BY status` untuk menghitung jumlah tugas berdasarkan statusnya.
    2.  **Grafik/Diagram:** Data hasil query di atas dikirim ke JavaScript. Library **Chart.js** kemudian mengambil data ini dan "menggambar" diagram batang atau garis yang interaktif di halaman.
    3.  **Laporan PDF:** Pengguna memilih filter di `laporan.php`. Data filter dan gambar grafik (di-capture sebagai Base64) dikirim ke `generate_report.php`. Di sana, library **TCPDF** mengambil data ini dan menyusunnya menjadi dokumen PDF yang profesional, yang kemudian diunduh oleh pengguna.
*   **Mengapa?** Visualisasi data membantu pengguna memahami pola produktivitas mereka dengan cepat, yang merupakan nilai tambah utama dari aplikasi ini.

### 4.4. Notifikasi & Pengalaman Pengguna (UX)

*   **Bagaimana?**
    *   **Sistem Notifikasi:** Saat sebuah aksi (misal, menambah tugas) berhasil, sebuah pesan disimpan di `$_SESSION['notification_messages']`. File `header.php` selalu memeriksa session ini dan menampilkannya sebagai popup. Notifikasi ini berbasis session, artinya akan hilang setelah ditampilkan atau saat sesi berakhir.
    *   **Tema Gelap/Terang:** JavaScript mendeteksi pilihan pengguna dan menyimpannya di `localStorage` browser. Ini membuat pilihan tema tetap tersimpan bahkan setelah browser ditutup. JavaScript akan menambahkan/menghapus kelas `dark-theme-active` pada tag `<html>`, dan CSS akan menangani perubahan warnanya.
*   **Mengapa?** Fitur-fitur ini, meskipun kecil, secara signifikan meningkatkan kenyamanan dan pengalaman pengguna.

### 4.5. Fitur Bintang: Bot Manager (AI)

Ini adalah fitur paling kompleks.

*   **Bagaimana Cara Kerjanya?**
    1.  **Input Pengguna:** Pengguna mengetik perintah (misal: "buatkan 5 tugas web dari 3 sampai 7 deadline 21 juni 2025") di `bot_manager.php`.
    2.  **Kirim ke Server:** JavaScript mengirim teks ini ke `gemini_handler.php`.
    3.  **Rekayasa Prompt (Prompt Engineering):** Di `gemini_handler.php`, PHP tidak langsung mengirim teks pengguna ke AI. Ia membangun sebuah "prompt" yang lebih lengkap:
        *   **Instruksi Sistem:** Berisi perintah-perintah ketat tentang bagaimana AI harus bersikap, format output yang diinginkan (JSON), dan contoh-contoh.
        *   **Konteks:** PHP mengambil daftar tugas aktif pengguna dari database dan menyertakannya dalam prompt.
        *   **Histori Percakapan:** Seluruh riwayat obrolan sebelumnya.
    4.  **Panggil API Gemini:** PHP mengirim prompt lengkap ini ke Google Gemini API.
    5.  **Respons AI:** Gemini menganalisis prompt dan merespons dengan teks bahasa alami dan, yang terpenting, sebuah **blok JSON** yang berisi "saran" aksi (`[{"intent": "create_suggestion", "title": "web 3", ...}, ...]`).
    6.  **Kirim Kembali ke Klien:** PHP mengirim kembali teks dan array JSON ini ke JavaScript.
    7.  **Render Kartu Konfirmasi:** JavaScript menerima array JSON. Untuk **setiap objek** di dalam array, ia membuat satu kartu konfirmasi HTML yang interaktif. Ini adalah kunci untuk menangani permintaan batch.
    8.  **Aksi Pengguna:** Pengguna mengklik "Lanjutkan" pada salah satu kartu.
    9.  **Eksekusi Final:** JavaScript mengirim permintaan **hanya untuk satu tugas itu** kembali ke `gemini_handler.php` dengan `action=confirm_crud_action`.
    10. **Eksekusi Database:** PHP menerima permintaan konfirmasi yang spesifik ini dan akhirnya menjalankan query SQL (`INSERT`, `UPDATE`, atau `DELETE`) pada database.

*   **Mengapa Bisa?**
    *   Kunci utamanya adalah **pemisahan tugas**. AI (Gemini) hanya bertugas **memahami bahasa manusia dan mengubahnya menjadi data terstruktur (JSON)**. Ia tidak pernah menyentuh database secara langsung.
    *   Logika bisnis dan eksekusi ke database tetap sepenuhnya dikontrol oleh **server PHP kita**. Ini menjaga keamanan dan kendali penuh atas aplikasi.
    *   JavaScript bertindak sebagai perantara yang cerdas, mengubah data JSON dari server menjadi antarmuka yang bisa digunakan oleh pengguna (kartu konfirmasi).

---

## Bab 5: Proses & Waktu Pembuatan - Berapa Lama?

Proyek dengan skala seperti ini dapat dipecah menjadi beberapa fase. Estimasi ini untuk satu developer yang fokus.

### 5.1. Tahapan Pengembangan

1.  **Fase 1: Perencanaan & Desain (2-3 hari)**
    *   Mendefinisikan semua fitur yang diinginkan.
    *   Merancang skema database (tabel, kolom, relasi).
    *   Membuat *wireframe* atau sketsa kasar tata letak halaman.

2.  **Fase 2: Fondasi & Backend Inti (4-5 hari)**
    *   Menyiapkan struktur proyek dan file `config.php`.
    *   Membuat sistem autentikasi pengguna (registrasi, login, logout, profil).
    *   Membuat fungsionalitas dasar CRUD untuk tugas.

3.  **Fase 3: Pengembangan Frontend & UX (4-5 hari)**
    *   Menerapkan desain ke dalam HTML dan CSS (`style.css`).
    *   Memastikan desain responsif untuk tampilan mobile.
    *   Menambahkan interaktivitas JavaScript (popup, kalender Flatpickr, tema gelap).

4.  **Fase 4: Fitur Lanjutan (5-6 hari)**
    *   Membangun halaman dasbor dengan kalkulasi statistik.
    *   Mengintegrasikan Chart.js untuk membuat diagram.
    *   Membangun halaman laporan dengan kalender interaktif (AJAX).
    *   Mengintegrasikan TCPDF untuk membuat laporan PDF.

5.  **Fase 5: Integrasi AI Chatbot (4-5 hari)**
    *   Membuat halaman `bot_manager.php`.
    *   Membangun backend `gemini_handler.php`.
    *   Melakukan *prompt engineering* (ini bagian paling trial-and-error).
    *   Membuat logika JavaScript untuk render kartu konfirmasi dan menangani aksi.

6.  **Fase 6: Pengujian & Finalisasi (3-4 hari)**
    *   Menguji semua fitur secara menyeluruh.
    *   Memperbaiki bug.
    *   Memastikan konsistensi di berbagai browser.

### 5.2. Estimasi Waktu

*   **Total Waktu Pengembangan Aktif:** Sekitar **22 - 28 hari kerja**, atau kira-kira **4 hingga 6 minggu**.
*   Waktu ini bisa lebih cepat jika beberapa developer bekerja secara paralel atau lebih lama jika dilakukan secara paruh waktu.

---

## Bab 6: Kesimpulan & Arah Pengembangan Selanjutnya

List In adalah bukti nyata bagaimana teknologi web standar (PHP, MySQL, JavaScript) dapat digabungkan dengan API modern (Google Gemini) untuk menciptakan aplikasi yang fungsional, intuitif, dan cerdas.

**Pengembangan Selanjutnya yang Mungkin:**
*   **Kolaborasi Tim:** Mengizinkan pengguna berbagi tugas atau proyek dengan pengguna lain.
*   **Kategori Tugas:** Menambahkan kategori pada tugas (misal: "Pekerjaan", "Pribadi") untuk filtering yang lebih baik.
*   **Notifikasi Push:** Mengirim notifikasi deadline langsung ke browser atau perangkat pengguna.
*   **Aplikasi Mobile:** Membuat versi mobile native (Android/iOS) untuk akses yang lebih mudah.
*   **Analisis AI Lebih Dalam:** Meminta Bot Manager untuk menganalisis kebiasaan dan memberikan saran produktivitas ("Sepertinya Anda sering menunda tugas dengan prioritas tinggi. Coba pecah menjadi tugas-tugas kecil?").
