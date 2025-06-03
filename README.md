Tentu, berdasarkan struktur `db_listin.sql` yang telah diperbarui dan kebutuhan fitur baru, berikut adalah query SQL yang bisa Anda jalankan untuk memperbarui database Anda yang sudah ada.

**Penting:**
*   **BACKUP DATABASE ANDA TERLEBIH DAHULU!** Sebelum menjalankan query `ALTER TABLE` atau `CREATE TABLE`, selalu buat cadangan database Anda untuk menghindari kehilangan data jika terjadi kesalahan.
*   Jalankan query ini melalui phpMyAdmin, command line MySQL, atau tool database lainnya.
*   Jika Anda sudah memiliki beberapa kolom atau tabel ini dari percobaan sebelumnya, query `ALTER TABLE ADD COLUMN IF NOT EXISTS` (untuk MySQL 8.0.1+) atau `CREATE TABLE IF NOT EXISTS` akan mencegah error. Untuk versi MySQL yang lebih lama, Anda mungkin perlu mengecek manual atau menghapus kolom/tabel tersebut jika sudah ada dengan struktur yang salah.

---

**Query SQL untuk Memperbarui Database `db_listin`:**

```sql
-- Menggunakan database db_listin
USE db_listin;

-- 1. Ubah tabel 'users'
-- Tambah kolom 'google_id' untuk Google OAuth
ALTER TABLE `users`
ADD COLUMN `google_id` VARCHAR(255) DEFAULT NULL AFTER `id`,
ADD UNIQUE INDEX `google_id_unique` (`google_id`);

-- Ubah kolom 'password' agar bisa NULL (untuk pengguna Google yang tidak punya password lokal)
ALTER TABLE `users`
MODIFY COLUMN `password` VARCHAR(255) DEFAULT NULL;

-- Tambah kolom 'email_verified_at' (opsional, tapi bagus untuk masa depan)
ALTER TABLE `users`
ADD COLUMN `email_verified_at` TIMESTAMP NULL DEFAULT NULL AFTER `created_at`;

-- Set default character set dan collation untuk tabel users jika belum
ALTER TABLE `users` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 2. Ubah tabel 'tasks' (jika diperlukan untuk character set)
-- Set default character set dan collation untuk tabel tasks jika belum
ALTER TABLE `tasks` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 3. Buat tabel baru 'password_resets' untuk fitur Lupa Kata Sandi
CREATE TABLE IF NOT EXISTS `password_resets` (
  `email` VARCHAR(100) NOT NULL,
  `token` VARCHAR(255) NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`email`),
  INDEX `token_idx` (`token`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Pesan bahwa pembaruan selesai (opsional, hanya untuk output di beberapa tool)
SELECT 'Pembaruan skema database selesai.' AS status;
```

---

**Penjelasan Query:**

1.  **`USE db_listin;`**: Memastikan query dijalankan pada database yang benar.
2.  **Perubahan pada tabel `users`**:
    *   `ALTER TABLE users ADD COLUMN google_id VARCHAR(255) DEFAULT NULL AFTER id;`: Menambahkan kolom `google_id` setelah kolom `id`. Kolom ini akan menyimpan ID unik pengguna dari Google.
    *   `ALTER TABLE users ADD UNIQUE INDEX google_id_unique (google_id);`: Membuat index unik pada `google_id` untuk memastikan tidak ada duplikasi.
    *   `ALTER TABLE users MODIFY COLUMN password VARCHAR(255) DEFAULT NULL;`: Mengubah kolom `password` sehingga bisa bernilai `NULL`. Ini penting karena pengguna yang login via Google mungkin tidak memiliki password lokal di aplikasi Anda.
    *   `ALTER TABLE users ADD COLUMN email_verified_at TIMESTAMP NULL DEFAULT NULL AFTER created_at;`: Menambahkan kolom `email_verified_at`. Ini bisa Anda gunakan di masa depan jika ingin mengimplementasikan sistem verifikasi email manual.
    *   `ALTER TABLE users CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`: Mengubah character set dan collation tabel `users` menjadi `utf8mb4` untuk dukungan karakter yang lebih luas (termasuk emoji).

3.  **Perubahan pada tabel `tasks`**:
    *   `ALTER TABLE tasks CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`: Sama seperti tabel `users`, ini untuk konsistensi dan dukungan karakter.

4.  **Pembuatan tabel `password_resets`**:
    *   `CREATE TABLE IF NOT EXISTS password_resets (...)`: Membuat tabel baru bernama `password_resets`. `IF NOT EXISTS` mencegah error jika tabel sudah ada.
        *   `email VARCHAR(100) NOT NULL`: Menyimpan email pengguna yang meminta reset. Ini adalah `PRIMARY KEY`.
        *   `token VARCHAR(255) NOT NULL`: Menyimpan token reset yang unik.
        *   `created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP`: Menyimpan waktu kapan token dibuat, untuk validasi kedaluwarsa.
        *   `INDEX token_idx (token)`: Membuat index pada kolom `token` untuk pencarian yang lebih cepat.
    *   `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;`: Menentukan engine tabel dan character set.

---

**Cara Menjalankan Query:**

*   **phpMyAdmin:**
    1.  Pilih database `db_listin` Anda dari daftar di sebelah kiri.
    2.  Klik tab "SQL".
    3.  Salin dan tempel semua query di atas ke dalam kotak teks.
    4.  Klik tombol "Go" atau "Kirim".
*   **MySQL Command Line:**
    1.  Buka terminal atau command prompt.
    2.  Login ke MySQL: `mysql -u root -p` (ganti `root` jika username Anda berbeda, lalu masukkan password).
    3.  Pilih database: `USE db_listin;`
    4.  Salin dan tempel query satu per satu atau sekaligus.
    5.  Tekan Enter setelah setiap query atau setelah semua query.

Setelah menjalankan query ini, struktur database Anda akan diperbarui untuk mendukung fitur-fitur baru yang diminta.




Baik, ini adalah pengembangan yang cukup signifikan. Saya akan mencoba memberikan kode selengkap mungkin dan menjelaskan langkah-langkahnya. Mengingat kompleksitasnya, terutama untuk integrasi Google dan Chatbot, Anda mungkin perlu melakukan beberapa penyesuaian dan debugging.

**Struktur Perubahan:**

1.  **Database:** Perubahan pada tabel `users` dan tabel baru untuk reset password.
2.  **Konfigurasi:** File baru untuk menyimpan kredensial API (Google, PHPMailer).
3.  **Google OAuth:**
    *   Tombol di `login.php` dan `register.php`.
    *   File callback `google_auth_callback.php`.
    *   Logika di `includes/db.php` atau file konfigurasi.
4.  **Lupa Password:**
    *   Link di `login.php`.
    *   File `forgot_password.php`.
    *   File `reset_password.php`.
    *   Fungsi pengiriman email.
5.  **Halaman Laporan (`laporan.php`):**
    *   Diagram batang (Chart.js).
    *   Kalender interaktif.
    *   Daftar tugas berdasarkan tanggal terpilih (AJAX).
    *   File AJAX `ajax_get_tasks_for_date.php`.
6.  **Chatbot:**
    *   Ikon chatbot di `includes/footer.php`.
    *   UI Chatbot (HTML/CSS/JS).
    *   File AJAX `ajax_chatbot_handler.php` (ini akan menjadi kerangka, karena logika inti bergantung pada API Anda).
7.  **CSS:** Penambahan dan modifikasi pada `css/style.css` dan `css/auth.css`.
8.  **File yang Dimodifikasi:** `login.php`, `register.php`, `includes/db.php`, `includes/header.php`, `includes/footer.php`, `includes/sidebar.php`, `css/style.css`, `css/auth.css`, dan `db_listin.sql`.
9.  **File Baru:** `config.php`, `google_auth_callback.php`, `forgot_password.php`, `reset_password.php`, `laporan.php`, `ajax_get_tasks_for_date.php`, `ajax_chatbot_handler.php`.

**PENTING: KUNCI API & KREDENSIAL**
Anda harus mendapatkan kunci API dan kredensial sendiri untuk:
*   **Google OAuth:** Dari Google Cloud Console.
*   **PHPMailer (untuk Lupa Password):** Kredensial SMTP dari penyedia email Anda (misalnya, Gmail App Password jika menggunakan Gmail).
*   **Chatbot API Key:** Dari penyedia layanan chatbot yang Anda pilih.

Mari kita mulai.

---

**LANGKAH 1: Perubahan Database**

Perbarui file `db_listin.sql` Anda.

**db_listin.sql (Diperbarui)**
```sql
-- --------------------------------------------------------
-- Host:                         127.0.0.1
-- Server version:               5.7.33 - MySQL Community Server (GPL)
-- Server OS:                    Win64
-- HeidiSQL Version:             11.2.0.6213
-- --------------------------------------------------------

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET NAMES utf8 */;
/*!50503 SET NAMES utf8mb4 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;


-- Dumping database structure for db_listin
CREATE DATABASE IF NOT EXISTS `db_listin` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;
USE `db_listin`;

-- Dumping structure for table db_listin.users
CREATE TABLE IF NOT EXISTS `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `google_id` varchar(255) DEFAULT NULL, -- Untuk Google OAuth
  `username` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) DEFAULT NULL, -- Bisa NULL jika login via Google
  `profile_image` varchar(255) DEFAULT 'images/placeholder-profile.png',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `email_verified_at` timestamp NULL DEFAULT NULL, -- Untuk verifikasi email jika diperlukan
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`),
  UNIQUE KEY `google_id` (`google_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Dumping structure for table db_listin.tasks
CREATE TABLE IF NOT EXISTS `tasks` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` text,
  `priority` enum('Low','Medium','High') NOT NULL DEFAULT 'Medium',
  `status` enum('Not Started','In Progress','Completed') NOT NULL DEFAULT 'Not Started',
  `due_date` date DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `last_overdue_notif_sent` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  KEY `status` (`status`),
  KEY `due_date` (`due_date`),
  CONSTRAINT `tasks_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Dumping structure for table db_listin.password_resets
CREATE TABLE IF NOT EXISTS `password_resets` (
  `email` varchar(100) NOT NULL,
  `token` varchar(255) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`email`),
  KEY `token_idx` (`token`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;


/*!40101 SET SQL_MODE=IFNULL(@OLD_SQL_MODE, '') */;
/*!40014 SET FOREIGN_KEY_CHECKS=IFNULL(@OLD_FOREIGN_KEY_CHECKS, 1) */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40111 SET SQL_NOTES=IFNULL(@OLD_SQL_NOTES, 1) */;
```
**Perubahan Utama:**
*   Tabel `users`:
    *   Tambah kolom `google_id` (unik) untuk menyimpan ID unik pengguna dari Google.
    *   Kolom `password` sekarang `DEFAULT NULL` karena pengguna Google mungkin tidak punya password lokal.
    *   Tambah kolom `email_verified_at` (opsional, tapi bagus untuk masa depan).
    *   Menggunakan `utf8mb4_unicode_ci` untuk dukungan karakter yang lebih baik.
*   Tabel `password_resets`: Tabel baru untuk menyimpan token reset password.

---

**LANGKAH 2: File Konfigurasi (`config.php`)**

Buat file `config.php` di root proyek Anda (sejajar dengan `login.php`).

**config.php (Baru)**
```php
<?php
// File: config.php

// Pengaturan Aplikasi
define('APP_URL', 'http://localhost/nama_folder_proyek_anda'); // Ganti dengan URL proyek Anda

// Konfigurasi Database (pindahkan dari db.php jika mau)
define('DB_HOST', '127.0.0.1');
define('DB_USER', 'root');
define('DB_PASS', '');
define('DB_NAME', 'db_listin');

// Pengaturan Google OAuth
define('GOOGLE_CLIENT_ID', 'MASUKKAN_CLIENT_ID_ANDA_DISINI');
define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_CLIENT_SECRET_ANDA_DISINI');
define('GOOGLE_REDIRECT_URI', APP_URL . '/google_auth_callback.php');

// Pengaturan PHPMailer (untuk Lupa Password)
define('SMTP_HOST', 'smtp.example.com'); // Ganti dengan host SMTP Anda
define('SMTP_USERNAME', 'user@example.com'); // Ganti dengan username SMTP Anda
define('SMTP_PASSWORD', 'password_smtp_anda'); // Ganti dengan password SMTP Anda
define('SMTP_PORT', 587); // Biasanya 587 untuk TLS, 465 untuk SSL
define('SMTP_SECURE', 'tls'); // 'tls' atau 'ssl'
define('EMAIL_FROM_ADDRESS', 'no-reply@example.com'); // Alamat email pengirim
define('EMAIL_FROM_NAME', 'List In App'); // Nama pengirim

// Pengaturan Chatbot (Ganti dengan API Key Anda)
define('CHATBOT_API_KEY', 'MASUKKAN_API_KEY_CHATBOT_ANDA_DISINI');
define('CHATBOT_API_ENDPOINT', 'URL_ENDPOINT_CHATBOT_ANDA'); // Jika ada endpoint spesifik

// Path untuk Library Google API Client (jika diinstal via Composer)
// Biasanya: dirname(__FILE__) . '/vendor/autoload.php'
// Jika manual, sesuaikan pathnya.
// define('GOOGLE_API_CLIENT_LIBRARY_PATH', dirname(__FILE__) . '/path/to/google-api-php-client/vendor/autoload.php');

// Set timezone
date_default_timezone_set('Asia/Jakarta');

// Mulai session di sini agar tersedia di semua file
if (session_status() == PHP_SESSION_NONE) {
    session_start();
}
?>
```
**Catatan:**
*   Ganti placeholder dengan nilai asli Anda.
*   Untuk `GOOGLE_API_CLIENT_LIBRARY_PATH`, Anda perlu menginstal Google API Client Library. Cara termudah adalah via Composer: `composer require google/apiclient:^2.0`. Jika ya, maka pathnya akan `dirname(__FILE__) . '/vendor/autoload.php'`.

---

**LANGKAH 3: Modifikasi `includes/db.php`**

File ini sekarang akan fokus pada koneksi DB dan memuat konfigurasi.

**includes/db.php (Dimodifikasi)**
```php
<?php
// File: includes/db.php
require_once dirname(__DIR__) . '/config.php'; // Memuat config.php dari root direktori

// Buat Koneksi
$conn = new mysqli(DB_HOST, DB_USER, DB_PASS, DB_NAME);

// Cek Koneksi
if ($conn->connect_error) {
    // Jangan tampilkan detail error di produksi
    // error_log("Koneksi Gagal: " . $conn->connect_error); // Log error
    die("Tidak dapat terhubung ke database. Silakan coba lagi nanti."); // Pesan umum untuk user
}

// Set character set (penting untuk UTF-8)
if (!$conn->set_charset("utf8mb4")) {
    // error_log("Error loading character set utf8mb4: " . $conn->error);
    // die("Kesalahan konfigurasi database.");
}

// Session sudah dimulai di config.php
?>
```

---

**LANGKAH 4: Integrasi Google OAuth**

Anda perlu Google API Client Library. Jika belum, install via Composer:
`composer require google/apiclient:^2.0`
Jika menggunakan Composer, pastikan Anda `require 'vendor/autoload.php';` di awal file yang membutuhkannya (`config.php` atau `google_auth_callback.php`).

**4.1. `google_auth_callback.php` (Baru - di root proyek)**
```php
<?php
// File: google_auth_callback.php
require_once 'config.php'; // Untuk GOOGLE_CLIENT_ID, dll. dan session_start()
require_once 'includes/db.php'; // Untuk koneksi $conn

// Pastikan Anda telah menginstal Google API Client Library
// Jika via Composer:
require_once __DIR__ . '/vendor/autoload.php';
// Jika manual, sesuaikan path ke autoload Google API Client
// require_once GOOGLE_API_CLIENT_LIBRARY_PATH;


$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI);
$client->addScope("email");
$client->addScope("profile");

if (isset($_GET['code'])) {
    $token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
    if (isset($token['error'])) {
        // Error saat mengambil access token
        $_SESSION['login_error_message'] = 'Gagal mendapatkan token dari Google: ' . htmlspecialchars($token['error_description'] ?? $token['error']);
        header('Location: login.php');
        exit();
    }
    $client->setAccessToken($token['access_token']);

    // Dapatkan info profil pengguna
    $google_oauth = new Google_Service_Oauth2($client);
    $google_account_info = $google_oauth->userinfo->get();
    
    $google_id = $google_account_info->id;
    $email = $google_account_info->email;
    $name = $google_account_info->name;
    $profile_pic_url = $google_account_info->picture;

    // Cek apakah pengguna sudah ada di database
    $stmt = $conn->prepare("SELECT id, username, profile_image FROM users WHERE google_id = ? OR email = ?");
    if (!$stmt) {
        $_SESSION['login_error_message'] = "Database error (prepare): " . $conn->error;
        header('Location: login.php');
        exit();
    }
    $stmt->bind_param("ss", $google_id, $email);
    $stmt->execute();
    $result = $stmt->get_result();
    $user = $result->fetch_assoc();
    $stmt->close();

    if ($user) {
        // Pengguna sudah ada, login
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username']; // Bisa jadi username lokalnya berbeda
        
        // Update google_id jika login via email tapi google_id belum ada
        if (empty($user['google_id']) && $user['email'] == $email) {
            $stmt_update_gid = $conn->prepare("UPDATE users SET google_id = ? WHERE id = ?");
            if($stmt_update_gid){
                $stmt_update_gid->bind_param("si", $google_id, $user['id']);
                $stmt_update_gid->execute();
                $stmt_update_gid->close();
            }
        }
        
        // Ambil gambar profil dari Google jika pengguna belum punya atau masih placeholder
        $current_db_image = $user['profile_image'];
        if ((empty($current_db_image) || $current_db_image == 'images/placeholder-profile.png') && !empty($profile_pic_url)) {
            // Coba unduh dan simpan gambar profil dari Google
            $image_data = @file_get_contents($profile_pic_url);
            if ($image_data !== false) {
                $upload_dir = 'uploads/profile_pictures/';
                if (!is_dir($upload_dir)) {
                    mkdir($upload_dir, 0755, true);
                }
                $filename = 'google_user_' . $user['id'] . '_' . time() . '.jpg'; // Asumsi jpg
                $filepath = $upload_dir . $filename;
                if (file_put_contents($filepath, $image_data)) {
                    $stmt_update_pic = $conn->prepare("UPDATE users SET profile_image = ? WHERE id = ?");
                    if($stmt_update_pic){
                        $stmt_update_pic->bind_param("si", $filepath, $user['id']);
                        $stmt_update_pic->execute();
                        $stmt_update_pic->close();
                        $_SESSION['profile_image'] = $filepath;
                    }
                }
            }
        } else {
            $_SESSION['profile_image'] = $current_db_image;
        }

        header('Location: dashboard.php');
        exit();
    } else {
        // Pengguna baru, daftarkan
        $default_profile_image_path = 'images/placeholder-profile.png'; // Default
        $new_user_image_path = $default_profile_image_path;

        // Coba unduh dan simpan gambar profil dari Google untuk pengguna baru
        if (!empty($profile_pic_url)) {
            $image_data_new = @file_get_contents($profile_pic_url);
            if ($image_data_new !== false) {
                $upload_dir_new = 'uploads/profile_pictures/';
                 if (!is_dir($upload_dir_new)) {
                    mkdir($upload_dir_new, 0755, true);
                }
                // Perlu ID user baru, jadi kita insert dulu baru update gambar
                // Atau, simpan sementara dan update setelah user ID didapat
            }
        }

        $stmt_insert = $conn->prepare("INSERT INTO users (google_id, username, email, profile_image, email_verified_at) VALUES (?, ?, ?, ?, NOW())");
        if (!$stmt_insert) {
             $_SESSION['login_error_message'] = "Database error (insert prepare): " . $conn->error;
            header('Location: login.php');
            exit();
        }
        // Gunakan $name sebagai username awal, $email, dan $new_user_image_path
        $stmt_insert->bind_param("ssss", $google_id, $name, $email, $new_user_image_path);
        if ($stmt_insert->execute()) {
            $new_user_id = $conn->insert_id;
            $_SESSION['user_id'] = $new_user_id;
            $_SESSION['username'] = $name;

            // Sekarang coba simpan gambar profil jika berhasil diunduh sebelumnya
            if (isset($image_data_new) && $image_data_new !== false) {
                $filename_new = 'google_user_' . $new_user_id . '_' . time() . '.jpg';
                $filepath_new = $upload_dir_new . $filename_new;
                if (file_put_contents($filepath_new, $image_data_new)) {
                    $stmt_update_new_pic = $conn->prepare("UPDATE users SET profile_image = ? WHERE id = ?");
                     if($stmt_update_new_pic){
                        $stmt_update_new_pic->bind_param("si", $filepath_new, $new_user_id);
                        $stmt_update_new_pic->execute();
                        $stmt_update_new_pic->close();
                        $_SESSION['profile_image'] = $filepath_new;
                     }
                } else {
                     $_SESSION['profile_image'] = $default_profile_image_path;
                }
            } else {
                $_SESSION['profile_image'] = $default_profile_image_path;
            }

            header('Location: dashboard.php'); // Arahkan ke dashboard
            exit();
        } else {
            $_SESSION['login_error_message'] = "Gagal mendaftarkan pengguna baru: " . $stmt_insert->error;
            header('Location: login.php');
            exit();
        }
        $stmt_insert->close();
    }
} else {
    // Tidak ada kode otorisasi, mungkin akses langsung atau error
    $_SESSION['login_error_message'] = 'Akses tidak sah atau otorisasi Google dibatalkan.';
    header('Location: login.php');
    exit();
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```
**4.2. Modifikasi `login.php`**

Tambahkan tombol login Google.

**login.php (Dimodifikasi)**
```php
<?php
require_once 'config.php'; // Untuk GOOGLE_CLIENT_ID, dll. dan session_start()
require_once 'includes/db.php';

// Setup Google Client untuk mendapatkan URL otorisasi
// Pastikan Anda telah menginstal Google API Client Library
// Jika via Composer:
require_once __DIR__ . '/vendor/autoload.php';
// Jika manual, sesuaikan path
// require_once GOOGLE_API_CLIENT_LIBRARY_PATH;


$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI);
$client->addScope("email");
$client->addScope("profile");
$google_login_url = $client->createAuthUrl();

$errors = [];

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

// Ambil pesan error dari session jika ada (misal dari google_auth_callback)
if (isset($_SESSION['login_error_message'])) {
    $errors[] = $_SESSION['login_error_message'];
    unset($_SESSION['login_error_message']);
}


if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['login_submit'])) { // Tambahkan name ke tombol submit
    $email = trim($_POST['email']);
    $password = $_POST['password'];

    if (empty($email)) $errors[] = "Alamat email wajib diisi.";
    if (empty($password)) $errors[] = "Kata sandi wajib diisi.";

    if (empty($errors)) {
        $stmt = $conn->prepare("SELECT id, username, password, profile_image FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($user = $result->fetch_assoc()) {
            // Hanya verifikasi password jika password di DB tidak NULL (bukan akun Google only)
            if ($user['password'] !== null && password_verify($password, $user['password'])) {
                $_SESSION['user_id'] = $user['id'];
                $_SESSION['username'] = $user['username'];
                $_SESSION['profile_image'] = $user['profile_image'];
                
                header("Location: dashboard.php");
                exit();
            } else {
                $errors[] = "Email atau password salah.";
            }
        } else {
            $errors[] = "Email atau password salah.";
        }
        $stmt->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>
<title>Masuk - List In</title>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Selamat Datang Kembali!</h2>
        <p class="subtitle">Masuk untuk melanjutkan dan mengatur tugas Anda.</p>

        <?php if (isset($_SESSION['success_message'])): ?>
            <div class="auth-message success">
                <p><?php echo $_SESSION['success_message']; ?></p>
            </div>
            <?php unset($_SESSION['success_message']); ?>
        <?php endif; ?>

        <?php if (isset($_SESSION['reset_success_message'])): ?>
            <div class="auth-message success">
                <p><?php echo $_SESSION['reset_success_message']; ?></p>
            </div>
            <?php unset($_SESSION['reset_success_message']); ?>
        <?php endif; ?>

        <?php if (!empty($errors)): ?>
            <div class="auth-message error">
                <?php foreach ($errors as $error): ?>
                    <p><?php echo htmlspecialchars($error); ?></p>
                <?php endforeach; ?>
            </div>
        <?php endif; ?>

        <form id="loginForm" method="POST" action="login.php">
            <div class="form-group">
                <label for="email">Alamat Email</label>
                <input type="email" id="email" name="email" required placeholder="cth: pengguna@email.com" value="<?php echo isset($_POST['email']) ? htmlspecialchars($_POST['email']) : ''; ?>">
            </div>
            <div class="form-group">
                <label for="password">Kata Sandi</label>
                <input type="password" id="password" name="password" required placeholder="Masukkan kata sandi Anda">
            </div>
            <div class="form-group forgot-password-link">
                <a href="forgot_password.php">Lupa Kata Sandi?</a>
            </div>
            <button type="submit" name="login_submit" class="btn-submit">Masuk Akun</button>
        </form>
        
        <div class="social-login-divider">
            <span>ATAU</span>
        </div>
        
        <a href="<?php echo htmlspecialchars($google_login_url); ?>" class="btn-social-login google">
            <i class="fab fa-google"></i> Lanjutkan dengan Google
        </a>

        <p class="auth-link">Belum punya akun? <a href="register.php">Daftar sekarang</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

**4.3. Modifikasi `register.php`**

Tambahkan tombol daftar dengan Google.

**register.php (Dimodifikasi)**
```php
<?php
require_once 'config.php'; // Untuk GOOGLE_CLIENT_ID, dll. dan session_start()
require_once 'includes/db.php';

// Setup Google Client untuk mendapatkan URL otorisasi
// Pastikan Anda telah menginstal Google API Client Library
// Jika via Composer:
require_once __DIR__ . '/vendor/autoload.php';
// Jika manual, sesuaikan path
// require_once GOOGLE_API_CLIENT_LIBRARY_PATH;


$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI); // Callback akan menangani pendaftaran
$client->addScope("email");
$client->addScope("profile");
$google_register_url = $client->createAuthUrl();


$errors = [];

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['register_submit'])) { // Tambahkan name ke tombol submit
    $username = trim($_POST['username']);
    $email = trim($_POST['email']);
    $password = $_POST['password'];
    $confirmPassword = $_POST['confirmPassword'];

    if (empty($username)) $errors[] = "Nama pengguna wajib diisi.";
    if (empty($email)) $errors[] = "Alamat email wajib diisi.";
    elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) $errors[] = "Format email tidak valid.";
    if (empty($password)) $errors[] = "Kata sandi wajib diisi.";
    elseif (strlen($password) < 6) $errors[] = "Kata sandi minimal 6 karakter.";
    if ($password !== $confirmPassword) $errors[] = "Password dan konfirmasi password tidak cocok.";

    if (empty($errors)) {
        $stmt_check_email = $conn->prepare("SELECT id FROM users WHERE email = ?");
        $stmt_check_email->bind_param("s", $email);
        $stmt_check_email->execute();
        $stmt_check_email->store_result();
        if ($stmt_check_email->num_rows > 0) {
            $errors[] = "Email sudah terdaftar. Gunakan email lain atau masuk.";
        }
        $stmt_check_email->close();
    }

    if (empty($errors)) {
        $hashed_password = password_hash($password, PASSWORD_DEFAULT);
        $default_profile_image = 'images/placeholder-profile.png';

        $stmt_insert = $conn->prepare("INSERT INTO users (username, email, password, profile_image, email_verified_at) VALUES (?, ?, ?, ?, NOW())"); // Asumsi email diverifikasi saat daftar manual
        $stmt_insert->bind_param("ssss", $username, $email, $hashed_password, $default_profile_image);
        
        if ($stmt_insert->execute()) {
            $_SESSION['success_message'] = "Registrasi berhasil! Silakan masuk dengan akun Anda.";
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Registrasi gagal. Silakan coba lagi. Error: " . $stmt_insert->error;
        }
        $stmt_insert->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>
<title>Daftar Akun - List In</title>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Buat Akun Baru Anda</h2>
        <p class="subtitle">Isi form di bawah untuk memulai perjalanan produktif Anda.</p>

        <?php if (!empty($errors)): ?>
            <div class="auth-message error">
                <?php foreach ($errors as $error): ?>
                    <p><?php echo htmlspecialchars($error); ?></p>
                <?php endforeach; ?>
            </div>
        <?php endif; ?>

        <form id="registerForm" method="POST" action="register.php">
            <div class="form-group">
                <label for="username">Nama Pengguna</label>
                <input type="text" id="username" name="username" required placeholder="cth: Amanuel" value="<?php echo isset($_POST['username']) ? htmlspecialchars($_POST['username']) : ''; ?>">
            </div>
            <div class="form-group">
                <label for="email">Alamat Email</label>
                <input type="email" id="email" name="email" required placeholder="cth: pengguna@email.com" value="<?php echo isset($_POST['email']) ? htmlspecialchars($_POST['email']) : ''; ?>">
            </div>
            <div class="form-group">
                <label for="password">Kata Sandi</label>
                <input type="password" id="password" name="password" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmPassword">Konfirmasi Kata Sandi</label>
                <input type="password" id="confirmPassword" name="confirmPassword" minlength="6" required placeholder="Ulangi kata sandi">
            </div>
            <button type="submit" name="register_submit" class="btn-submit">Daftar Akun</button>
        </form>

        <div class="social-login-divider">
            <span>ATAU</span>
        </div>
        
        <a href="<?php echo htmlspecialchars($google_register_url); ?>" class="btn-social-login google">
            <i class="fab fa-google"></i> Lanjutkan dengan Google
        </a>

        <p class="auth-link">Sudah punya akun? <a href="login.php">Masuk di sini</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

---

**LANGKAH 5: Fitur Lupa Password**

Anda perlu library PHPMailer. Install via Composer:
`composer require phpmailer/phpmailer`

**5.1. `forgot_password.php` (Baru - di root proyek)**
```php
<?php
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

require_once 'config.php'; // Untuk APP_URL, SMTP settings, dan session_start()
require_once 'includes/db.php'; // Untuk koneksi $conn

// Jika via Composer:
require __DIR__ . '/vendor/autoload.php'; // Untuk PHPMailer

$errors = [];
$success_message = '';

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $email = trim($_POST['email']);

    if (empty($email) || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Masukkan alamat email yang valid.";
    } else {
        $stmt = $conn->prepare("SELECT id FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($result->num_rows > 0) {
            // Email ditemukan, buat token
            $token = bin2hex(random_bytes(50)); // Token acak
            
            // Hapus token lama untuk email ini jika ada
            $stmt_delete_old = $conn->prepare("DELETE FROM password_resets WHERE email = ?");
            $stmt_delete_old->bind_param("s", $email);
            $stmt_delete_old->execute();
            $stmt_delete_old->close();

            // Simpan token baru ke database
            $stmt_insert_token = $conn->prepare("INSERT INTO password_resets (email, token) VALUES (?, ?)");
            $stmt_insert_token->bind_param("ss", $email, $token);
            
            if ($stmt_insert_token->execute()) {
                // Kirim email reset password
                $mail = new PHPMailer(true);
                try {
                    //Server settings
                    $mail->isSMTP();
                    $mail->Host       = SMTP_HOST;
                    $mail->SMTPAuth   = true;
                    $mail->Username   = SMTP_USERNAME;
                    $mail->Password   = SMTP_PASSWORD;
                    $mail->SMTPSecure = SMTP_SECURE; // PHPMailer::ENCRYPTION_SMTPS or PHPMailer::ENCRYPTION_STARTTLS
                    $mail->Port       = SMTP_PORT;

                    //Recipients
                    $mail->setFrom(EMAIL_FROM_ADDRESS, EMAIL_FROM_NAME);
                    $mail->addAddress($email);

                    //Content
                    $reset_link = APP_URL . "/reset_password.php?token=" . $token;
                    $mail->isHTML(true);
                    $mail->Subject = 'Reset Kata Sandi Akun List In Anda';
                    $mail->Body    = "Halo,<br><br>Kami menerima permintaan untuk mereset kata sandi akun Anda di List In.<br>"
                                   . "Silakan klik tautan di bawah ini untuk mengatur ulang kata sandi Anda:<br>"
                                   . "<a href='" . $reset_link . "'>" . $reset_link . "</a><br><br>"
                                   . "Jika Anda tidak meminta reset kata sandi, abaikan email ini.<br><br>"
                                   . "Salam,<br>Tim List In";
                    $mail->AltBody = "Halo,\n\nKami menerima permintaan untuk mereset kata sandi akun Anda di List In.\n"
                                   . "Silakan salin dan tempel tautan berikut di browser Anda untuk mengatur ulang kata sandi Anda:\n"
                                   . $reset_link . "\n\n"
                                   . "Jika Anda tidak meminta reset kata sandi, abaikan email ini.\n\n"
                                   . "Salam,\nTim List In";

                    $mail->send();
                    $success_message = 'Email instruksi reset kata sandi telah dikirim ke alamat email Anda. Silakan periksa kotak masuk (dan folder spam).';
                } catch (Exception $e) {
                    $errors[] = "Gagal mengirim email. Silakan coba lagi nanti. Mailer Error: {$mail->ErrorInfo}";
                    // error_log("Mailer Error: {$mail->ErrorInfo}");
                }
            } else {
                $errors[] = "Gagal menyimpan token reset. Silakan coba lagi.";
                // error_log("Token save error: " . $stmt_insert_token->error);
            }
            $stmt_insert_token->close();
        } else {
            // Email tidak ditemukan, tapi jangan beritahu secara eksplisit untuk keamanan
            $success_message = 'Jika alamat email Anda terdaftar, kami telah mengirimkan instruksi reset kata sandi. Silakan periksa kotak masuk (dan folder spam).';
        }
        $stmt->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>
<title>Lupa Kata Sandi - List In</title>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Lupa Kata Sandi?</h2>
    <p class="subtitle">Masukkan alamat email Anda di bawah ini. Kami akan mengirimkan tautan untuk mereset kata sandi Anda.</p>

    <?php if (!empty($success_message)): ?>
        <div class="auth-message success">
            <p><?php echo htmlspecialchars($success_message); ?></p>
        </div>
    <?php endif; ?>

    <?php if (!empty($errors)): ?>
        <div class="auth-message error">
            <?php foreach ($errors as $error): ?>
                <p><?php echo htmlspecialchars($error); ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <?php if (empty($success_message)): // Hanya tampilkan form jika belum ada pesan sukses ?>
    <form id="forgotPasswordForm" method="POST" action="forgot_password.php">
        <div class="form-group">
            <label for="email">Alamat Email</label>
            <input type="email" id="email" name="email" required placeholder="Masukkan email terdaftar Anda">
        </div>
        <button type="submit" class="btn-submit">Kirim Tautan Reset</button>
    </form>
    <?php endif; ?>

    <p class="auth-link">Ingat kata sandi Anda? <a href="login.php">Masuk di sini</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**5.2. `reset_password.php` (Baru - di root proyek)**
```php
<?php
require_once 'config.php'; // Untuk session_start()
require_once 'includes/db.php'; // Untuk koneksi $conn

$errors = [];
$token = $_GET['token'] ?? '';
$email_from_token = null;
$token_valid = false;

if (empty($token)) {
    $errors[] = "Token reset tidak valid atau tidak ditemukan.";
} else {
    $stmt_check_token = $conn->prepare("SELECT email, created_at FROM password_resets WHERE token = ?");
    $stmt_check_token->bind_param("s", $token);
    $stmt_check_token->execute();
    $result_token = $stmt_check_token->get_result();

    if ($reset_data = $result_token->fetch_assoc()) {
        $email_from_token = $reset_data['email'];
        $token_created_at = new DateTime($reset_data['created_at']);
        $now = new DateTime();
        $interval = $now->diff($token_created_at);
        $minutes_passed = ($interval->days * 24 * 60) + ($interval->h * 60) + $interval->i;

        if ($minutes_passed > 60) { // Token valid selama 1 jam
            $errors[] = "Token reset telah kedaluwarsa. Silakan minta tautan reset baru.";
            // Hapus token kedaluwarsa
            $stmt_delete_expired = $conn->prepare("DELETE FROM password_resets WHERE token = ?");
            $stmt_delete_expired->bind_param("s", $token);
            $stmt_delete_expired->execute();
            $stmt_delete_expired->close();
        } else {
            $token_valid = true;
        }
    } else {
        $errors[] = "Token reset tidak valid atau sudah digunakan.";
    }
    $stmt_check_token->close();
}

if ($_SERVER["REQUEST_METHOD"] == "POST" && $token_valid) {
    $new_password = $_POST['newPassword'];
    $confirm_new_password = $_POST['confirmNewPassword'];

    if (empty($new_password) || empty($confirm_new_password)) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak boleh kosong.";
    } elseif (strlen($new_password) < 6) {
        $errors[] = "Kata sandi baru minimal 6 karakter.";
    } elseif ($new_password !== $confirm_new_password) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak cocok.";
    } else {
        // Update password pengguna
        $hashed_password = password_hash($new_password, PASSWORD_DEFAULT);
        $stmt_update_pass = $conn->prepare("UPDATE users SET password = ? WHERE email = ?");
        $stmt_update_pass->bind_param("ss", $hashed_password, $email_from_token);
        
        if ($stmt_update_pass->execute()) {
            // Hapus token setelah berhasil digunakan
            $stmt_delete_used = $conn->prepare("DELETE FROM password_resets WHERE email = ?");
            $stmt_delete_used->bind_param("s", $email_from_token);
            $stmt_delete_used->execute();
            $stmt_delete_used->close();

            $_SESSION['reset_success_message'] = "Kata sandi Anda telah berhasil direset. Silakan masuk dengan kata sandi baru.";
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Gagal mereset kata sandi. Silakan coba lagi.";
            // error_log("Password update error: " . $stmt_update_pass->error);
        }
        $stmt_update_pass->close();
    }
}

?>
<?php require_once 'includes/header.php'; ?>
<title>Reset Kata Sandi - List In</title>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Reset Kata Sandi Anda</h2>

    <?php if (!empty($errors)): ?>
        <div class="auth-message error">
            <?php foreach ($errors as $error): ?>
                <p><?php echo htmlspecialchars($error); ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <?php if ($token_valid && empty($_SESSION['reset_success_message'])): ?>
        <p class="subtitle">Masukkan kata sandi baru Anda untuk akun terkait dengan <strong><?php echo htmlspecialchars($email_from_token); ?></strong>.</p>
        <form id="resetPasswordForm" method="POST" action="reset_password.php?token=<?php echo htmlspecialchars($token); ?>">
            <div class="form-group">
                <label for="newPassword">Kata Sandi Baru</label>
                <input type="password" id="newPassword" name="newPassword" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmNewPassword">Konfirmasi Kata Sandi Baru</label>
                <input type="password" id="confirmNewPassword" name="confirmNewPassword" minlength="6" required placeholder="Ulangi kata sandi baru">
            </div>
            <button type="submit" class="btn-submit">Reset Kata Sandi</button>
        </form>
    <?php elseif(!empty($_SESSION['reset_success_message'])): ?>
         <div class="auth-message success">
            <p><?php echo htmlspecialchars($_SESSION['reset_success_message']); ?></p>
            <?php unset($_SESSION['reset_success_message']); ?>
        </div>
        <p class="auth-link"><a href="login.php">Kembali ke Halaman Masuk</a></p>
    <?php else: ?>
        <p class="auth-link"><a href="forgot_password.php">Minta tautan reset baru</a> atau <a href="login.php">kembali ke Halaman Masuk</a>.</p>
    <?php endif; ?>
</div>

<?php require_once 'includes/footer.php'; ?>
```

---

**LANGKAH 6: Halaman Laporan (`laporan.php`)**

**6.1. `laporan.php` (Baru - di root proyek)**
```php
<?php
require_once 'includes/header.php';
require_once 'includes/sidebar.php';
require_once 'includes/task_helper.php'; // Untuk render_task_card

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];
$current_page_for_redirect = basename($_SERVER['SCRIPT_NAME']);

// --- DATA UNTUK DIAGRAM BATANG (Performa Pengerjaan - sama seperti dashboard) ---
$today_for_default_perf = new DateTimeImmutable();
$performance_start_date_val = $today_for_default_perf->modify('-6 days')->format('Y-m-d');
$performance_end_date_val = $today_for_default_perf->format('Y-m-d');
$active_preset_perf_val = 'last7days'; // Default

// Logika filter tanggal untuk diagram batang (mirip dashboard)
if (isset($_GET['filterReportChartSubmit'])) {
    if (isset($_GET['filterReportDateRange']) && !empty(trim($_GET['filterReportDateRange']))) {
        $range_perf = explode(' - ', $_GET['filterReportDateRange']);
        if (count($range_perf) >= 1) {
            $date_start_parts_perf = explode('/', trim($range_perf[0]));
            if (count($date_start_parts_perf) == 3 && checkdate((int)$date_start_parts_perf[1], (int)$date_start_parts_perf[0], (int)$date_start_parts_perf[2])) {
                $performance_start_date_val = $date_start_parts_perf[2] . '-' . $date_start_parts_perf[1] . '-' . $date_start_parts_perf[0];
            }
            if (count($range_perf) == 2) {
                $date_end_parts_perf = explode('/', trim($range_perf[1]));
                 if (count($date_end_parts_perf) == 3 && checkdate((int)$date_end_parts_perf[1], (int)$date_end_parts_perf[0], (int)$date_end_parts_perf[2])) {
                    $performance_end_date_val = $date_end_parts_perf[2] . '-' . $date_end_parts_perf[1] . '-' . $date_end_parts_perf[0];
                }
            } else { 
                if (DateTime::createFromFormat('Y-m-d', $performance_start_date_val)) { 
                    $performance_end_date_val = $performance_start_date_val;
                }
            }
        }
         $active_preset_perf_val = ''; // Kosongkan preset jika rentang kustom dipilih
    }
} elseif (isset($_GET['range_type_report_chart'])) { // Menggunakan nama unik untuk preset di laporan
    $active_preset_perf_val = $_GET['range_type_report_chart'];
    $today_for_preset = new DateTimeImmutable();
    switch ($active_preset_perf_val) {
        case 'today': $performance_start_date_val = $today_for_preset->format('Y-m-d'); $performance_end_date_val = $today_for_preset->format('Y-m-d'); break;
        case 'last7days': $performance_end_date_val = $today_for_preset->format('Y-m-d'); $performance_start_date_val = $today_for_preset->modify('-6 days')->format('Y-m-d'); break;
        case 'this_month': $performance_start_date_val = $today_for_preset->format('Y-m-01'); $performance_end_date_val = $today_for_preset->format('Y-m-t'); break;
    }
}

$bar_chart_labels_php = []; $bar_chart_data_completed_php = [];
if (isset($conn) && $conn instanceof mysqli) {
    try {
        $current_date_loop_obj = DateTime::createFromFormat('Y-m-d', $performance_start_date_val);
        $end_date_loop_obj = DateTime::createFromFormat('Y-m-d', $performance_end_date_val);
        if ($current_date_loop_obj && $end_date_loop_obj) {
            if ($current_date_loop_obj > $end_date_loop_obj) { list($current_date_loop_obj, $end_date_loop_obj) = [$end_date_loop_obj, $current_date_loop_obj]; }
            $loop_count = 0; $interval_one_day = new DateInterval('P1D');
            while ($current_date_loop_obj <= $end_date_loop_obj) {
                $date_str_loop = $current_date_loop_obj->format('Y-m-d');
                $bar_chart_labels_php[] = $current_date_loop_obj->format('d M'); // Format label
                $stmt_completed_on_date = $conn->prepare("SELECT COUNT(*) as count FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) = ?");
                $tasks_completed_on_day = 0;
                if($stmt_completed_on_date) {
                    $stmt_completed_on_date->bind_param("is", $user_id, $date_str_loop);
                    if($stmt_completed_on_date->execute()){ 
                        $result_completed_on_date = $stmt_completed_on_date->get_result();
                        if($result_completed_on_date && $result_completed_on_date->num_rows > 0) {
                            $row_completed = $result_completed_on_date->fetch_assoc();
                            $tasks_completed_on_day = isset($row_completed['count']) ? (int)$row_completed['count'] : 0;
                        }
                    }
                    $stmt_completed_on_date->close();
                }
                $bar_chart_data_completed_php[] = $tasks_completed_on_day;
                $current_date_loop_obj->add($interval_one_day); $loop_count++;
                if ($loop_count > 90 ) { if ($current_date_loop_obj <= $end_date_loop_obj) { $bar_chart_labels_php[] = "..."; $bar_chart_data_completed_php[] = null; } break; }
            }
        }
    } catch (Exception $e) { error_log("Laporan (Chart): Exception: " . $e->getMessage()); }
}
if (empty($bar_chart_labels_php)) $bar_chart_labels_php = ['Tidak Ada Data'];
if (empty($bar_chart_data_completed_php)) $bar_chart_data_completed_php = [0];

$report_bar_chart_data_php_final = [
    'labels' => $bar_chart_labels_php,
    'datasets' => [[
        'label' => 'Tugas Selesai','data' => $bar_chart_data_completed_php,
        'backgroundColor' => 'rgba(126, 71, 184, 0.6)', // Warna utama
        'borderColor' => 'rgba(126, 71, 184, 1)',
        'borderWidth' => 1,
        'borderRadius' => 8, // Untuk sudut yang halus
        'borderSkipped' => false,
    ]]
];

?>
<title>Laporan Tugas - List In</title>

<main class="main">
    <h2 class="page-title">Laporan Produktivitas</h2>

    <section class="widget report-chart-widget">
        <h3>Performa Pengerjaan Tugas</h3>
        <form method="GET" action="laporan.php" class="performance-filter-form report-performance-filter">
            <label>Rentang Waktu Diagram:</label>
            <div class="filter-preset-buttons">
                <button type="submit" name="range_type_report_chart" value="today" class="btn btn-sm <?php echo $active_preset_perf_val == 'today' ? 'active' : ''; ?>">Hari Ini</button>
                <button type="submit" name="range_type_report_chart" value="last7days" class="btn btn-sm <?php echo $active_preset_perf_val == 'last7days' ? 'active' : ''; ?>">7 Hari</button>
                <button type="submit" name="range_type_report_chart" value="this_month" class="btn btn-sm <?php echo $active_preset_perf_val == 'this_month' ? 'active' : ''; ?>">Bulan Ini</button>
            </div>
            <input type="text" id="filterReportDateRange" name="filterReportDateRange" placeholder="Kustom..." value="<?php echo htmlspecialchars($_GET['filterReportDateRange'] ?? ''); ?>" style="width:160px;">
            <div class="filter-action-buttons">
                <button type="submit" name="filterReportChartSubmit" value="1" class="btn btn-primary btn-apply-perf btn-sm">Lihat Diagram</button>
            </div>
        </form>
        <div class="widget-content-area chart-container" style="height: 250px; margin-top: 15px;">
             <?php
                $has_valid_bar_chart_data = false;
                if (isset($report_bar_chart_data_php_final['datasets'][0]['data']) && is_array($report_bar_chart_data_php_final['datasets'][0]['data'])) {
                    $filtered_data_bar = array_filter($report_bar_chart_data_php_final['datasets'][0]['data'], function($x) { return $x !== null && $x >=0; });
                    $has_valid_bar_chart_data = !empty($filtered_data_bar) && count($filtered_data_bar) > 0;
                }
                $has_valid_labels_bar = !empty($report_bar_chart_data_php_final['labels']) && !in_array("Error", $report_bar_chart_data_php_final['labels']) && !in_array("Tidak Ada Data", $report_bar_chart_data_php_final['labels']);

                if ($has_valid_bar_chart_data && $has_valid_labels_bar):
            ?>
            <canvas id="reportPerformanceBarChart"></canvas>
            <?php else: ?>
                <p class="no-tasks-message" style="text-align:center; padding-top:20px;">Tidak ada data tugas selesai untuk ditampilkan pada rentang waktu ini.</p>
            <?php endif; ?>
        </div>
    </section>

    <section class="report-tasks-by-date-section widget">
        <h3>Tugas Aktif Berdasarkan Tanggal</h3>
        <div class="report-interactive-area">
            <div class="report-task-list-container">
                <p id="selectedDateText" class="selected-date-indicator">Pilih tanggal di kalender untuk melihat tugas.</p>
                <div id="reportTasksList" class="widget-content-area scrollable-list">
                    <!-- Tugas akan dimuat di sini oleh AJAX -->
                    <p class="no-tasks-message">Pilih tanggal pada kalender.</p>
                </div>
            </div>
            <div class="report-calendar-container">
                <div id="reportCalendar"></div> <!-- Kalender akan dirender di sini oleh JS -->
            </div>
        </div>
    </section>

</main>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // --- Script untuk Diagram Batang Laporan ---
    const barCtx = document.getElementById('reportPerformanceBarChart');
    if (barCtx && typeof Chart !== 'undefined') {
        const reportBarData = <?php echo json_encode($report_bar_chart_data_php_final); ?>;
        
        let hasValidBarData = false;
         if (reportBarData && reportBarData.labels && Array.isArray(reportBarData.labels) &&
            reportBarData.datasets && Array.isArray(reportBarData.datasets) && reportBarData.datasets.length > 0 &&
            reportBarData.datasets[0].data && Array.isArray(reportBarData.datasets[0].data) ) {
            let validLabelsExistBar = reportBarData.labels.some(l => l !== 'Error' && l !== 'Tidak Ada Data' && l !== '...');
            let numericDataExistsBar = reportBarData.datasets[0].data.some(d => typeof d === 'number' && d >= 0);
            hasValidBarData = validLabelsExistBar && numericDataExistsBar;
        }

        if (hasValidBarData) {
            new Chart(barCtx, {
                type: 'bar',
                data: reportBarData,
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                stepSize: 1,
                                precision: 0, // Hanya integer di sumbu Y
                                callback: function(value) {if (Number.isInteger(value)) {return value;}}
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false // Sembunyikan legenda jika hanya satu dataset
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) { label += ': '; }
                                    if (context.parsed.y !== null) { label += context.parsed.y + ' tugas'; }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
        }
    }
    const reportPresetButtons = document.querySelectorAll('.report-performance-filter .filter-preset-buttons .btn');
    const reportDateRangeInput = document.getElementById('filterReportDateRange');
    reportPresetButtons.forEach(button => {
        button.addEventListener('click', function(e) {
            if (reportDateRangeInput) reportDateRangeInput.value = ''; 
        });
    });
     if (reportDateRangeInput) {
        reportDateRangeInput.addEventListener('input', function() { // atau 'change'
            if (this.value !== '') {
                const parentButtonsContainer = this.closest('.report-performance-filter').querySelector('.filter-preset-buttons');
                if(parentButtonsContainer){
                     parentButtonsContainer.querySelectorAll('.btn').forEach(btn => btn.classList.remove('active'));
                }
            }
        });
    }


    // --- Script untuk Kalender Interaktif dan Daftar Tugas ---
    const calendarContainer = document.getElementById('reportCalendar');
    const tasksListContainer = document.getElementById('reportTasksList');
    const selectedDateText = document.getElementById('selectedDateText');
    let currentYear, currentMonth;

    function renderCalendar(year, month) {
        currentYear = year;
        currentMonth = month;
        calendarContainer.innerHTML = ''; // Clear previous calendar

        const monthNames = ["Januari", "Februari", "Maret", "April", "Mei", "Juni", "Juli", "Agustus", "September", "Oktober", "November", "Desember"];
        const dayNames = ["Min", "Sen", "Sel", "Rab", "Kam", "Jum", "Sab"];

        const header = document.createElement('div');
        header.classList.add('calendar-header-report');
        header.innerHTML = `
            <button id="prevMonthBtn">&lt;</button>
            <span>${monthNames[month]} ${year}</span>
            <button id="nextMonthBtn">&gt;</button>
        `;
        calendarContainer.appendChild(header);

        const daysGrid = document.createElement('div');
        daysGrid.classList.add('calendar-days-grid-report');
        dayNames.forEach(day => {
            const dayNameCell = document.createElement('div');
            dayNameCell.classList.add('calendar-day-name-report');
            dayNameCell.textContent = day;
            daysGrid.appendChild(dayNameCell);
        });

        const firstDayOfMonth = new Date(year, month, 1).getDay();
        const daysInMonth = new Date(year, month + 1, 0).getDate();

        for (let i = 0; i < firstDayOfMonth; i++) {
            const emptyCell = document.createElement('div');
            daysGrid.appendChild(emptyCell);
        }

        for (let day = 1; day <= daysInMonth; day++) {
            const dayCell = document.createElement('div');
            dayCell.classList.add('calendar-day-report');
            dayCell.textContent = day;
            dayCell.dataset.date = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            
            const today = new Date();
            if (year === today.getFullYear() && month === today.getMonth() && day === today.getDate()) {
                dayCell.classList.add('today');
            }

            dayCell.addEventListener('click', function() {
                document.querySelectorAll('.calendar-day-report.selected').forEach(el => el.classList.remove('selected'));
                this.classList.add('selected');
                loadTasksForDate(this.dataset.date);
            });
            daysGrid.appendChild(dayCell);
        }
        calendarContainer.appendChild(daysGrid);

        document.getElementById('prevMonthBtn').addEventListener('click', () => {
            month--;
            if (month < 0) {
                month = 11;
                year--;
            }
            renderCalendar(year, month);
        });

        document.getElementById('nextMonthBtn').addEventListener('click', () => {
            month++;
            if (month > 11) {
                month = 0;
                year++;
            }
            renderCalendar(year, month);
        });
    }

    function loadTasksForDate(dateStr) { // dateStr format YYYY-MM-DD
        const dateObj = new Date(dateStr);
        const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
        selectedDateText.textContent = `Tugas Aktif untuk: ${dateObj.toLocaleDateString('id-ID', options)}`;
        tasksListContainer.innerHTML = '<p class="loading-message">Memuat tugas...</p>';

        fetch(`ajax_get_tasks_for_date.php?date=${dateStr}`)
            .then(response => {
                if (!response.ok) { throw new Error('Network response was not ok'); }
                return response.json();
            })
            .then(data => {
                tasksListContainer.innerHTML = ''; // Clear loading
                if (data.success && data.tasks.length > 0) {
                    data.tasks.forEach(taskHtml => {
                        // Karena taskHtml adalah string HTML dari render_task_card, kita bisa langsung append
                        tasksListContainer.insertAdjacentHTML('beforeend', taskHtml);
                    });
                } else if (data.success && data.tasks.length === 0) {
                    tasksListContainer.innerHTML = '<p class="no-tasks-message">Tidak ada tugas aktif untuk tanggal ini.</p>';
                } else {
                    tasksListContainer.innerHTML = `<p class="no-tasks-message">Gagal memuat tugas: ${data.message || 'Error tidak diketahui'}</p>`;
                }
            })
            .catch(error => {
                console.error('Error fetching tasks:', error);
                tasksListContainer.innerHTML = '<p class="no-tasks-message">Terjadi kesalahan saat memuat tugas.</p>';
            });
    }

    const today = new Date();
    renderCalendar(today.getFullYear(), today.getMonth());

    // Inisialisasi Flatpickr untuk filter diagram
    const reportDateRangePicker = document.getElementById('filterReportDateRange');
    if(reportDateRangePicker) {
        flatpickr(reportDateRangePicker, {
            mode: "range",
            dateFormat: "d/m/Y",
            locale: "id",
            allowInput: true
        });
    }
});
</script>
<?php require_once 'includes/footer.php'; ?>
```

**6.2. `ajax_get_tasks_for_date.php` (Baru - di root proyek)**
```php
<?php
// File: ajax_get_tasks_for_date.php
require_once 'includes/db.php'; // Untuk $conn dan session_start()
require_once 'includes/task_helper.php'; // Untuk render_task_card

header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'User not authenticated.', 'tasks' => []]);
    exit();
}
$user_id = $_SESSION['user_id'];
$selected_date_str = $_GET['date'] ?? null; // Format YYYY-MM-DD

if (!$selected_date_str) {
    echo json_encode(['success' => false, 'message' => 'Tanggal tidak disediakan.', 'tasks' => []]);
    exit();
}

// Validasi format tanggal YYYY-MM-DD
$date_parts = explode('-', $selected_date_str);
if (count($date_parts) !== 3 || !checkdate((int)$date_parts[1], (int)$date_parts[2], (int)$date_parts[0])) {
    echo json_encode(['success' => false, 'message' => 'Format tanggal tidak valid.', 'tasks' => []]);
    exit();
}

$tasks_html_array = [];
$sql = "SELECT id, title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted, due_date
        FROM tasks
        WHERE user_id = ? AND status != 'Completed' AND due_date = ?
        ORDER BY CASE priority WHEN 'High' THEN 1 WHEN 'Medium' THEN 2 WHEN 'Low' THEN 3 ELSE 4 END, created_at ASC";

$stmt = $conn->prepare($sql);
if ($stmt) {
    $stmt->bind_param("is", $user_id, $selected_date_str);
    $stmt->execute();
    $result = $stmt->get_result();
    
    $current_page_for_redirect = 'laporan.php'; // Atau halaman yang relevan jika ada aksi

    while ($task = $result->fetch_assoc()) {
        // Gunakan render_task_card. Untuk halaman laporan, mungkin tidak ada aksi,
        // jadi page_type bisa 'dashboardTodo' atau custom type jika perlu styling berbeda.
        // Jika ingin aksi (edit/delete) dari laporan, pastikan $current_page_for_redirect sesuai.
        $tasks_html_array[] = render_task_card($task, 'dashboardTodo', $current_page_for_redirect . '?selected_date=' . urlencode($selected_date_str));
    }
    $stmt->close();
    echo json_encode(['success' => true, 'tasks' => $tasks_html_array]);
} else {
    echo json_encode(['success' => false, 'message' => 'Database query error: ' . $conn->error, 'tasks' => []]);
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 7: Implementasi Chatbot (Kerangka)**

Ini adalah bagian yang paling bergantung pada API eksternal Anda. Saya akan berikan kerangka UI dan AJAX.

**7.1. `ajax_chatbot_handler.php` (Baru - di root proyek - KERANGKA)**
```php
<?php
// File: ajax_chatbot_handler.php (KERANGKA)
require_once 'config.php'; // Untuk CHATBOT_API_KEY, session_start()
require_once 'includes/db.php'; // Untuk $conn
require_once 'includes/header.php'; // Untuk add_notification (jika chatbot memberi notif internal)


header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'reply' => 'Autentikasi pengguna gagal.']);
    exit();
}
$user_id = $_SESSION['user_id'];
$user_message = trim($_POST['message'] ?? '');

if (empty($user_message)) {
    echo json_encode(['success' => false, 'reply' => 'Pesan tidak boleh kosong.']);
    exit();
}

$bot_reply = "Maaf, saya belum bisa memproses permintaan itu saat ini."; // Default reply

// ---------------------------------------------------------------------------
// BAGIAN INI ADALAH LOGIKA INTI CHATBOT ANDA
// Anda perlu:
// 1. Mengirim $user_message ke API Chatbot Anda (menggunakan CHATBOT_API_KEY).
// 2. Menerima respons dari API Chatbot.
// 3. Menganalisis respons untuk menentukan "intent" atau perintah.
// 4. Jika perintahnya adalah aksi internal (misal, "hapus tugas X", "bersihkan riwayat"),
//    lakukan aksi tersebut di database.
// 5. Format balasan yang sesuai.
// ---------------------------------------------------------------------------

// Contoh Placeholder untuk interaksi dengan API Chatbot eksternal:
/*
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, CHATBOT_API_ENDPOINT);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(['message' => $user_message, 'user_id_internal' => $user_id])); // Kirim ID user internal jika API Anda mendukungnya
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Content-Type: application/json',
    'Authorization: Bearer ' . CHATBOT_API_KEY
]);
$api_response_str = curl_exec($ch);
if (curl_errno($ch)) {
    // error_log('Chatbot API Curl error: ' . curl_error($ch));
    echo json_encode(['success' => false, 'reply' => 'Error menghubungi layanan chatbot.']);
    exit();
}
curl_close($ch);

$api_response = json_decode($api_response_str, true);
if ($api_response && isset($api_response['reply'])) {
    $bot_reply = $api_response['reply']; // Ambil balasan dari API

    // --- Analisis Intent Sederhana (CONTOH) ---
    // Ini sangat bergantung pada bagaimana API Anda mengembalikan intent
    $intent = strtolower($api_response['intent'] ?? ''); // Misal API Anda mengembalikan intent

    if (strpos(strtolower($user_message), "tampilkan tugas") !== false || $intent === 'show_tasks') {
        // Logika untuk mengambil dan menampilkan tugas
        $stmt_tasks = $conn->prepare("SELECT title, status, DATE_FORMAT(due_date, '%d %b %Y') as due_date_formatted FROM tasks WHERE user_id = ? AND status != 'Completed' ORDER BY due_date ASC LIMIT 5");
        if ($stmt_tasks) {
            $stmt_tasks->bind_param("i", $user_id);
            $stmt_tasks->execute();
            $result_tasks = $stmt_tasks->get_result();
            $task_list_reply = "Berikut tugas aktif Anda:\n";
            if ($result_tasks->num_rows > 0) {
                while ($task = $result_tasks->fetch_assoc()) {
                    $task_list_reply .= "- " . htmlspecialchars($task['title']) . " (Status: " . htmlspecialchars($task['status']) . ", Deadline: " . ($task['due_date_formatted'] ?: 'N/A') . ")\n";
                }
            } else {
                $task_list_reply = "Anda tidak memiliki tugas aktif saat ini.";
            }
            $bot_reply = $task_list_reply;
            $stmt_tasks->close();
        }
    } elseif (strpos(strtolower($user_message), "hapus notifikasi") !== false || $intent === 'clear_notifications') {
        if (isset($_SESSION['notification_messages'])) unset($_SESSION['notification_messages']);
        $_SESSION['has_unread_notifications_badge'] = false;
        $bot_reply = "Semua notifikasi telah dihapus.";
        // add_notification("Notifikasi dibersihkan via chatbot.", "info"); // Notif internal
    } elseif (preg_match('/hapus tugas "(.*?)"/i', $user_message, $matches) || ($intent === 'delete_task' && isset($api_response['task_title']))) {
        $task_title_to_delete = $matches[1] ?? ($api_response['task_title'] ?? '');
        if (!empty($task_title_to_delete)) {
            $stmt_delete = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND title = ? AND status != 'Completed'");
            if ($stmt_delete) {
                $stmt_delete->bind_param("is", $user_id, $task_title_to_delete);
                $stmt_delete->execute();
                if ($stmt_delete->affected_rows > 0) {
                    $bot_reply = "Tugas \"" . htmlspecialchars($task_title_to_delete) . "\" berhasil dihapus.";
                } else {
                    $bot_reply = "Tugas \"" . htmlspecialchars($task_title_to_delete) . "\" tidak ditemukan atau sudah selesai.";
                }
                $stmt_delete->close();
            }
        } else {
             $bot_reply = "Tolong sebutkan nama tugas yang ingin dihapus, contoh: hapus tugas \"Nama Tugas Ini\".";
        }
    } elseif (strpos(strtolower($user_message), "bersihkan riwayat") !== false || $intent === 'clear_history') {
        $stmt_clear_hist = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND (status = 'Completed' OR (due_date < CURDATE() AND status != 'Completed'))");
        if($stmt_clear_hist){
            $stmt_clear_hist->bind_param("i", $user_id);
            $stmt_clear_hist->execute();
            $bot_reply = "Semua riwayat tugas (selesai dan terlewat) telah dibersihkan.";
            $stmt_clear_hist->close();
        }
    } elseif (strpos(strtolower($user_message), "bersihkan semua tugas aktif") !== false || $intent === 'clear_active_tasks') {
        $stmt_clear_active = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND status != 'Completed'");
        if($stmt_clear_active){
            $stmt_clear_active->bind_param("i", $user_id);
            $stmt_clear_active->execute();
            $bot_reply = "Semua tugas aktif (yang belum selesai) telah dibersihkan.";
            $stmt_clear_active->close();
        }
    }
    // Tambahkan lebih banyak kondisi untuk perintah lain
} else {
    // error_log("Chatbot API response error or invalid format: " . $api_response_str);
    // $bot_reply tetap default
}
*/
// AKHIR BAGIAN LOGIKA INTI CHATBOT ANDA

echo json_encode(['success' => true, 'reply' => nl2br(htmlspecialchars($bot_reply))]);

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 8: Modifikasi File PHP yang Ada (`includes/header.php`, `includes/sidebar.php`, `includes/footer.php`)**

**8.1. `includes/header.php` (Dimodifikasi)**
```php
<?php
// File: includes/header.php
require_once dirname(__DIR__) . '/config.php'; // Memuat config.php dari root (sudah ada session_start)
require_once __DIR__ . '/db.php'; // Koneksi DB

$current_page = basename($_SERVER['SCRIPT_NAME']);
$current_user_id = $_SESSION['user_id'] ?? null;
$current_user_profile_image = 'images/placeholder-profile.png'; // Default

if ($current_user_id && isset($conn)) {
    $stmt_user_header = $conn->prepare("SELECT username, profile_image FROM users WHERE id = ?");
    if ($stmt_user_header) {
        $stmt_user_header->bind_param("i", $current_user_id);
        $stmt_user_header->execute();
        $result_user_header = $stmt_user_header->get_result();
        if ($user_data_header = $result_user_header->fetch_assoc()) {
            // Pastikan nama pengguna di session selalu update dari DB saat header load
            $_SESSION['username'] = $user_data_header['username']; 
            
            $current_user_profile_image_db = $user_data_header['profile_image'];
             // Cek jika path gambar dari DB valid dan file ada, jika tidak, gunakan placeholder
            if (!empty($current_user_profile_image_db) && file_exists(dirname(__DIR__) . '/' . $current_user_profile_image_db)) {
                $current_user_profile_image = $current_user_profile_image_db;
            } elseif (!empty($current_user_profile_image_db) && filter_var($current_user_profile_image_db, FILTER_VALIDATE_URL)) {
                // Jika itu URL (misal dari Google), gunakan langsung
                $current_user_profile_image = $current_user_profile_image_db;
            }
            // Update session profile image jika berbeda dari yang di DB (misal setelah login Google)
            if (($_SESSION['profile_image'] ?? '') !== $current_user_profile_image) {
                 $_SESSION['profile_image'] = $current_user_profile_image;
            }

        }
        $stmt_user_header->close();
    }
}


// Fungsi add_notification, pengecekan deadline, dan overdue tetap sama seperti sebelumnya
// ... (Salin fungsi add_notification, cek deadline H-1, cek overdue dari kode Anda sebelumnya) ...
function add_notification($message, $type = 'info') {
    if (!isset($_SESSION['notification_messages'])) {
        $_SESSION['notification_messages'] = [];
    }
    $new_notification = ['message' => $message, 'type' => $type, 'time' => time()];
    
    $is_duplicate_notif = false;
    if ($type === 'deadline_soon' || $type === 'overdue_task') { 
        $message_core_part = '';
        if (preg_match('/Tugas(?:-tugas)?\s*(".*?")\s*(akan jatuh tempo besok|telah melewati batas waktu)/', $new_notification['message'], $matches)) {
            $message_core_part = $matches[1]; 
        } elseif (preg_match('/(".*?")/', $new_notification['message'], $matches_generic)) {
            $message_core_part = $matches_generic[1];
        }

        foreach ($_SESSION['notification_messages'] as $existing_notif) {
            if (isset($existing_notif['message']) &&
                ($message_core_part && strpos($existing_notif['message'], $message_core_part) !== false) && 
                $existing_notif['type'] === $type &&
                (time() - ($existing_notif['time'] ?? 0)) < 3600 * 3) { 
                $is_duplicate_notif = true;
                break;
            }
        }
    }

    if (!$is_duplicate_notif) {
        $_SESSION['notification_messages'][] = $new_notification;
        if (count($_SESSION['notification_messages']) > 10) { 
            array_shift($_SESSION['notification_messages']);
        }
        $_SESSION['has_unread_notifications_badge'] = true;
    }
}

if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'])) {
    $tomorrow_date = date('Y-m-d', strtotime('+1 day'));
    $today_for_notif_check_h1 = date('Y-m-d');
    $notif_key_deadline_h1 = 'deadline_h1_notif_sent_' . $today_for_notif_check_h1 . '_uid' . $current_user_id;

    if (!isset($_SESSION[$notif_key_deadline_h1])) {
        $stmt_deadline_check_h1 = $conn->prepare("SELECT title FROM tasks WHERE user_id = ? AND due_date = ? AND status != 'Completed'");
        if ($stmt_deadline_check_h1) {
            $stmt_deadline_check_h1->bind_param("is", $current_user_id, $tomorrow_date);
            $stmt_deadline_check_h1->execute();
            $result_deadline_tasks_h1 = $stmt_deadline_check_h1->get_result();
            $tasks_deadline_tomorrow = [];
            while ($task_for_deadline_notif_h1 = $result_deadline_tasks_h1->fetch_assoc()) {
                $tasks_deadline_tomorrow[] = htmlspecialchars($task_for_deadline_notif_h1['title']);
            }
            $stmt_deadline_check_h1->close();

            if (!empty($tasks_deadline_tomorrow)) {
                $task_list_str_h1 = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_deadline_tomorrow));
                $message_plural_h1 = count($tasks_deadline_tomorrow) > 1 ? "Tugas-tugas" : "Tugas";
                $deadline_message_h1 = "<span class='message-content'><strong>PERHATIAN:</strong> $message_plural_h1 $task_list_str_h1 akan jatuh tempo besok!</span>";
                add_notification($deadline_message_h1, "deadline_soon");
                $_SESSION[$notif_key_deadline_h1] = true; 
            }
        } else {
            error_log("Failed to prepare statement for H-1 deadline check: " . $conn->error);
        }
    }
}

if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'])) {
    $today_for_overdue_check = date('Y-m-d');
    $notif_key_overdue = 'overdue_notif_sent_' . $today_for_overdue_check . '_uid' . $current_user_id;

    if (!isset($_SESSION[$notif_key_overdue])) {
        $stmt_overdue_check = $conn->prepare(
            "SELECT id, title FROM tasks 
             WHERE user_id = ? AND due_date < CURDATE() AND status != 'Completed' 
             AND (last_overdue_notif_sent IS NULL OR DATE(last_overdue_notif_sent) < CURDATE())"
        );
        
        if ($stmt_overdue_check) {
            $stmt_overdue_check->bind_param("i", $current_user_id);
            $stmt_overdue_check->execute();
            $result_overdue_tasks = $stmt_overdue_check->get_result();
            $tasks_overdue = [];
            $task_ids_to_update_notif_sent = [];

            while ($task_overdue = $result_overdue_tasks->fetch_assoc()) {
                $tasks_overdue[] = htmlspecialchars($task_overdue['title']);
                $task_ids_to_update_notif_sent[] = $task_overdue['id'];
            }
            $stmt_overdue_check->close();

            if (!empty($tasks_overdue)) {
                $task_list_str_overdue = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_overdue));
                $message_plural_overdue = count($tasks_overdue) > 1 ? "Tugas-tugas" : "Tugas";
                $overdue_message = "<span class='message-content'><strong>TERLEWAT:</strong> $message_plural_overdue $task_list_str_overdue telah melewati batas waktu!</span>";
                add_notification($overdue_message, "deadline_soon"); 

                if (!empty($task_ids_to_update_notif_sent)) {
                    $ids_placeholder = implode(',', array_fill(0, count($task_ids_to_update_notif_sent), '?'));
                    $stmt_update_notif_date = $conn->prepare(
                        "UPDATE tasks SET last_overdue_notif_sent = NOW() WHERE id IN ($ids_placeholder) AND user_id = ?"
                    );
                    if ($stmt_update_notif_date) {
                        $types_update = str_repeat('i', count($task_ids_to_update_notif_sent)) . 'i';
                        $params_update = array_merge($task_ids_to_update_notif_sent, [$current_user_id]);
                        $stmt_update_notif_date->bind_param($types_update, ...$params_update);
                        if (!$stmt_update_notif_date->execute()) {
                            error_log("Failed to update last_overdue_notif_sent: " . $stmt_update_notif_date->error);
                        }
                        $stmt_update_notif_date->close();
                    }
                }
                $_SESSION[$notif_key_overdue] = true; 
            }
        } else {
            error_log("Failed to prepare statement for overdue check: " . $conn->error);
        }
    }
}

$has_unread_badge_for_icon = $_SESSION['has_unread_notifications_badge'] ?? false;

function format_tanggal_indonesia_header($date_str) {
    if (empty($date_str) || $date_str == '0000-00-00') return 'N/A';
    try { $date = new DateTime($date_str); } catch (Exception $e) { return 'Invalid Date'; }
    $hari_arr = ['Min', 'Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab'];
    $bulan_arr = [1=>'Jan',2=>'Feb',3=>'Mar',4=>'Apr',5=>'Mei',6=>'Jun',7=>'Jul',8=>'Agu',9=>'Sep',10=>'Okt',11=>'Nov',12=>'Des'];
    $hari = $hari_arr[(int)$date->format('w')];
    $tanggal = $date->format('j');
    $bulan = $bulan_arr[(int)$date->format('n')];
    return $hari . ', ' . $tanggal . ' ' . $bulan;
}
$today_display_full = format_tanggal_indonesia_header(date('Y-m-d'));
$parts_today_display = explode(', ', $today_display_full);
$day_name_display = $parts_today_display[0] ?? '';
$date_str_display = $parts_today_display[1] ?? '';

// Halaman-halaman yang dianggap sebagai bagian "auth" flow
$auth_pages = ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'];
$is_auth_page = in_array($current_page, $auth_pages);

?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
      // Skrip FOUC Prevent
      (function() {
        const theme = localStorage.getItem('theme');
        const htmlEl = document.documentElement;
        if (theme === 'dark-theme' || (!theme && window.matchMedia?.('(prefers-color-scheme: dark)').matches)) {
          htmlEl.classList.add('dark-theme-active');
        }
        <?php if ($is_auth_page): ?>
          htmlEl.classList.add('auth-html');
        <?php endif; ?>
      })();
    </script>
    <link rel="stylesheet" href="css/style.css?v=<?php echo time(); ?>">
    <?php if ($is_auth_page): ?>
        <link rel="stylesheet" href="css/auth.css?v=<?php echo time(); ?>">
    <?php endif; ?>
</head>
<body class="<?php echo $is_auth_page ? 'auth-page' : ''; ?>">

    <?php if (!$is_auth_page): ?>
    <header class="header">
        <div class="header-left">
            <a href="profil.php">
                 <img src="<?php echo htmlspecialchars($current_user_profile_image); ?>?t=<?php echo time();?>" alt="Foto Profil" class="header-profile-pic" id="headerProfilePic">
            </a>
            <h2 class="app-title-toggle" id="appTitleToggle" title="Toggle Sidebar">List In</h2>
        </div>
        <?php
        $hide_search_bar_pages = ['profil.php', 'edit_profil.php', 'ubah_password.php', 'tambah_tugas.php', 'edit_tugas.php', 'dashboard.php', 'laporan.php']; // Tambah laporan.php
        $hide_search_bar = in_array($current_page, $hide_search_bar_pages);
        $search_action_page = 'manajemen_tugas.php';
        if ($current_page == 'riwayat.php') $search_action_page = 'riwayat.php';
        ?>
        <div class="search-bar" <?php if ($hide_search_bar) echo 'style="visibility: hidden;"'; ?>>
            <form action="<?php echo $search_action_page; ?>" method="GET" style="display:flex; width:100%;">
                <input type="text" name="search_term" id="searchInputGlobal" placeholder="Cari di <?php echo ($current_page == 'riwayat.php' ? 'Riwayat' : 'Manajemen'); ?>..." value="<?php echo isset($_GET['search_term']) ? htmlspecialchars($_GET['search_term']) : ''; ?>">
                <button type="submit" style="background:none; border:none; padding:0 0 0 8px; margin-left:auto; cursor:pointer;"><i class="fas fa-search"></i></button>
            </form>
        </div>
        <div class="header-right">
             <i class="fas fa-bell <?php if ($has_unread_badge_for_icon) echo 'has-notif'; ?>" id="bellIcon" title="Notifikasi"></i>
            <i class="fas fa-calendar-alt" id="calendarIcon" title="Kalender"></i>
            <div class="date-container">
                <p><?php echo htmlspecialchars($day_name_display); ?></p>
                <span><?php echo htmlspecialchars($date_str_display); ?></span>
            </div>
        </div>
    </header>
    <div class="content">
    <?php endif; ?>
```

**8.2. `includes/sidebar.php` (Dimodifikasi)**
```php
<?php
// $current_page sudah didefinisikan di header.php
$auth_pages_sidebar = ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'];
$is_auth_page_sidebar = in_array($current_page, $auth_pages_sidebar);
?>
<?php if (!$is_auth_page_sidebar): ?>
        <aside class="sidebar" id="sidebar">
            <nav class="menu">
                <a href="dashboard.php" class="<?php echo ($current_page == 'dashboard.php') ? 'active' : ''; ?>"><i class="fas fa-home"></i> <span>Dasbor</span></a>
                <a href="manajemen_tugas.php" class="<?php echo ($current_page == 'manajemen_tugas.php' || $current_page == 'edit_tugas.php') ? 'active' : ''; ?>"><i class="fas fa-tasks"></i> <span>Kelola Tugas</span></a>
                <a href="tambah_tugas.php" class="<?php echo ($current_page == 'tambah_tugas.php') ? 'active' : ''; ?>"><i class="fas fa-plus-circle"></i> <span>Tambah Tugas</span></a>
                <a href="laporan.php" class="<?php echo ($current_page == 'laporan.php') ? 'active' : ''; ?>"><i class="fas fa-chart-pie"></i> <span>Laporan</span></a>
                <a href="riwayat.php" class="<?php echo ($current_page == 'riwayat.php') ? 'active' : ''; ?>"><i class="fas fa-history"></i> <span>Riwayat</span></a>
            </nav>
            <a href="logout.php" class="logout" id="logoutButton"><i class="fas fa-sign-out-alt"></i> <span>Keluar</span></a>
        </aside>
<?php endif; ?>
```

**8.3. `includes/footer.php` (Dimodifikasi)**
```php
<?php
// $current_page dari header.php
// Halaman-halaman yang dianggap sebagai bagian "auth" flow atau halaman khusus
$no_standard_footer_pages = [
    'login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php',
    // Tambahkan halaman lain di sini jika tidak memerlukan notif popup, kalender popup, atau chatbot
    // 'laporan.php', // Jika laporan tidak butuh chatbot
];
$is_special_page_footer = in_array($current_page, $no_standard_footer_pages);

// Chatbot tidak ditampilkan di halaman tertentu
$no_chatbot_pages = [
    'login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php',
    'laporan.php', 'profil.php', 'edit_profil.php', 'ubah_password.php'
];
$show_chatbot = !in_array($current_page, $no_chatbot_pages);

?>
<?php if (!$is_special_page_footer): ?>
    </div> <!-- Penutup div.content dari header.php -->

    <div id="notification-popup">
        <div class="notification-header">
             <h4>Notifikasi</h4>
             <button id="clearAllNotificationsBtn" class="btn-clear-notif" title="Hapus semua notifikasi">Hapus Semua</button>
        </div>
        <ul id="notification-list">
            <?php 
            $current_session_notifications_for_popup = [];
            if (isset($_SESSION['notification_messages']) && is_array($_SESSION['notification_messages'])) {
                $current_session_notifications_for_popup = $_SESSION['notification_messages'];
                usort($current_session_notifications_for_popup, function($a, $b) {
                    return ($b['time'] ?? 0) - ($a['time'] ?? 0);
                });
            }
            $has_current_notifications_for_popup = !empty($current_session_notifications_for_popup);
            ?>
            <?php if ($has_current_notifications_for_popup): ?>
                <?php foreach ($current_session_notifications_for_popup as $notif_item): 
                    $time_ago_popup = 'Beberapa waktu lalu';
                    if (isset($notif_item['time'])) {
                        try {
                            $timestamp_popup = new DateTime('@' . $notif_item['time']);
                            $now_popup = new DateTime(); $interval_popup = $now_popup->diff($timestamp_popup);
                            if ($interval_popup->y > 0) $time_ago_popup = $interval_popup->y . " thn lalu";
                            elseif ($interval_popup->m > 0) $time_ago_popup = $interval_popup->m . " bln lalu";
                            elseif ($interval_popup->d > 0) $time_ago_popup = $interval_popup->d . " hr lalu";
                            elseif ($interval_popup->h > 0) $time_ago_popup = $interval_popup->h . " jam lalu";
                            elseif ($interval_popup->i > 0) $time_ago_popup = $interval_popup->i . " mnt lalu";
                            else $time_ago_popup = "Baru saja";
                        } catch (Exception $e) { /* Biarkan default */ }
                    }
                    $message_class_popup = '';
                    if(isset($notif_item['type'])) {
                        if($notif_item['type'] == 'success') $message_class_popup = 'notif-success';
                        else if($notif_item['type'] == 'error') $message_class_popup = 'notif-error';
                        else if($notif_item['type'] == 'info') $message_class_popup = 'notif-info';
                        else if($notif_item['type'] == 'deadline_soon') $message_class_popup = 'notif-deadline_soon';
                    }
                ?>
                    <li class="<?php echo $message_class_popup; ?>">
                        <?php echo $notif_item['message']; ?> 
                        <small class="notif-time"><?php echo $time_ago_popup; ?></small>
                    </li>
                <?php endforeach; ?>
            <?php else: ?>
                <li class="no-notifications">Tidak ada notifikasi baru.</li>
            <?php endif; ?>
        </ul>
    </div>
    <div id="calendar-popup">
        <div id="calendar-container-popup"></div>
    </div>

    <?php if ($show_chatbot): ?>
    <div id="chatbot-icon" title="Tanya ListIn Bot">
        <i class="fas fa-robot"></i>
    </div>
    <div id="chatbot-container">
        <div id="chatbot-header">
            ListIn Bot
            <button id="closeChatbotBtn">&times;</button>
        </div>
        <div id="chatbot-messages">
            <!-- Pesan chatbot akan muncul di sini -->
            <div class="chatbot-message bot">Halo! Ada yang bisa saya bantu terkait tugas Anda?</div>
        </div>
        <div id="chatbot-input-area">
            <input type="text" id="chatbotInput" placeholder="Ketik pesan Anda...">
            <button id="sendChatbotMessageBtn"><i class="fas fa-paper-plane"></i></button>
        </div>
    </div>
    <?php endif; ?>
    
    <script src="https://cdn.jsdelivr.net/npm/flatpickr"></script>
    <script src="https://npmcdn.com/flatpickr/dist/l10n/id.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const bodyElement = document.body; 
            const htmlElement = document.documentElement; 
            const bellIcon = document.getElementById('bellIcon');
            const notificationPopup = document.getElementById('notification-popup');
            const notificationListUl = document.getElementById('notification-list');
            const clearAllNotificationsBtn = document.getElementById('clearAllNotificationsBtn');

            const calendarIcon = document.getElementById('calendarIcon');
            const calendarPopup = document.getElementById('calendar-popup');
            
            // Variabel Chatbot (jika ditampilkan)
            const chatbotIcon = document.getElementById('chatbot-icon');
            const chatbotContainer = document.getElementById('chatbot-container');
            const closeChatbotBtn = document.getElementById('closeChatbotBtn');
            const chatbotMessagesDiv = document.getElementById('chatbot-messages');
            const chatbotInput = document.getElementById('chatbotInput');
            const sendChatbotMessageBtn = document.getElementById('sendChatbotMessageBtn');


            function togglePopup(popupElement, iconElement, otherPopupElement, anotherPopupElement = null) {
                if (otherPopupElement && otherPopupElement.classList.contains('show')) {
                    otherPopupElement.classList.remove('show');
                }
                if (anotherPopupElement && anotherPopupElement.classList.contains('show')) {
                    anotherPopupElement.classList.remove('show');
                }

                popupElement.classList.toggle('show');
                
                if (popupElement === notificationPopup && popupElement.classList.contains('show')) {
                    if (bellIcon && bellIcon.classList.contains('has-notif')) {
                        fetch('mark_notifications_viewed.php')
                            .then(response => response.json())
                            .then(data => {
                                if(data.success) {
                                     if(bellIcon) bellIcon.classList.remove('has-notif');
                                }
                            }).catch(error => console.error('Error marking notifications viewed:', error));
                    }
                }
            }

            if (bellIcon && notificationPopup) { 
                bellIcon.addEventListener('click', (e) => { 
                    e.stopPropagation(); 
                    togglePopup(notificationPopup, bellIcon, calendarPopup, chatbotContainer); 
                });
            }

            if (clearAllNotificationsBtn && notificationListUl) {
                clearAllNotificationsBtn.addEventListener('click', (e) => {
                    e.stopPropagation(); 
                    if (confirm('Anda yakin ingin menghapus semua notifikasi?')) {
                        fetch('ajax_clear_all_notifications.php') 
                            .then(response => response.json())
                            .then(data => {
                                if (data.success) {
                                    notificationListUl.innerHTML = '<li class="no-notifications">Tidak ada notifikasi baru.</li>';
                                    if (bellIcon && bellIcon.classList.contains('has-notif')) {
                                        bellIcon.classList.remove('has-notif'); 
                                    }
                                } else { alert('Gagal menghapus notifikasi.'); }
                            })
                            .catch(error => {
                                console.error('Error clearing all notifications:', error);
                                alert('Terjadi kesalahan saat menghapus notifikasi.');
                            });
                    }
                });
            }

            if (calendarIcon && calendarPopup) { 
                 flatpickr(document.getElementById('calendar-container-popup'), { inline: true, dateFormat: "d/m/Y", locale: "id" });
                calendarIcon.addEventListener('click', function (e) { 
                    e.stopPropagation(); 
                    togglePopup(calendarPopup, calendarIcon, notificationPopup, chatbotContainer); 
                });
            }

            // --- Chatbot Logic ---
            if (chatbotIcon && chatbotContainer) {
                chatbotIcon.addEventListener('click', (e) => {
                    e.stopPropagation();
                    // Sembunyikan popup lain jika terbuka
                    if (notificationPopup && notificationPopup.classList.contains('show')) notificationPopup.classList.remove('show');
                    if (calendarPopup && calendarPopup.classList.contains('show')) calendarPopup.classList.remove('show');
                    
                    chatbotContainer.classList.toggle('show');
                    chatbotIcon.style.display = chatbotContainer.classList.contains('show') ? 'none' : 'flex';
                    if(chatbotContainer.classList.contains('show')) chatbotInput.focus();
                });

                if (closeChatbotBtn) {
                    closeChatbotBtn.addEventListener('click', () => {
                        chatbotContainer.classList.remove('show');
                        chatbotIcon.style.display = 'flex';
                    });
                }

                function sendChatMessage() {
                    const message = chatbotInput.value.trim();
                    if (message === '') return;

                    appendMessageToChatbox(message, 'user');
                    chatbotInput.value = '';
                    chatbotInput.disabled = true;
                    sendChatbotMessageBtn.disabled = true;
                    appendMessageToChatbox('...', 'bot-typing'); // Typing indicator

                    fetch('ajax_chatbot_handler.php', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                        },
                        body: `message=${encodeURIComponent(message)}`
                    })
                    .then(response => response.json())
                    .then(data => {
                        removeTypingIndicator();
                        if (data.success) {
                            appendMessageToChatbox(data.reply, 'bot');
                        } else {
                            appendMessageToChatbox(data.reply || 'Maaf, terjadi kesalahan.', 'bot');
                        }
                    })
                    .catch(error => {
                        removeTypingIndicator();
                        console.error('Chatbot AJAX error:', error);
                        appendMessageToChatbox('Tidak dapat terhubung ke bot saat ini.', 'bot');
                    })
                    .finally(() => {
                        chatbotInput.disabled = false;
                        sendChatbotMessageBtn.disabled = false;
                        chatbotInput.focus();
                    });
                }
                
                if (sendChatbotMessageBtn) sendChatbotMessageBtn.addEventListener('click', sendChatMessage);
                if (chatbotInput) chatbotInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') sendChatMessage();
                });

                function appendMessageToChatbox(message, type) {
                    const messageEl = document.createElement('div');
                    messageEl.classList.add('chatbot-message', type);
                    messageEl.innerHTML = message; // Pesan dari bot mungkin sudah mengandung HTML (nl2br)
                    chatbotMessagesDiv.appendChild(messageEl);
                    chatbotMessagesDiv.scrollTop = chatbotMessagesDiv.scrollHeight;
                }
                function removeTypingIndicator() {
                    const typingIndicator = chatbotMessagesDiv.querySelector('.bot-typing');
                    if (typingIndicator) typingIndicator.remove();
                }
            }
            // --- End Chatbot Logic ---


            document.addEventListener('click', function (e) { 
                if (notificationPopup && notificationPopup.classList.contains('show') && !notificationPopup.contains(e.target) && e.target !== bellIcon) { 
                    notificationPopup.classList.remove('show'); 
                }
                if (calendarPopup && calendarPopup.classList.contains('show') && !calendarPopup.contains(e.target) && e.target !== calendarIcon) { 
                    calendarPopup.classList.remove('show'); 
                }
                // Jangan tutup chatbot jika klik di dalam chatbot container
                if (chatbotContainer && chatbotContainer.classList.contains('show') && !chatbotContainer.contains(e.target) && e.target !== chatbotIcon) {
                    // chatbotContainer.classList.remove('show'); 
                    // chatbotIcon.style.display = 'flex'; 
                    // Biarkan terbuka, tutup hanya via tombol close atau icon
                }
            });

            const dateInputs = document.querySelectorAll('input[type="text"][id$="Date"], input[type="text"][id$="DueDate"], input[type="text"][id$="DateRange"], input[type="text"][id="filterHistoryDateRange"], input[type="text"][id="filterReportDateRange"]');
            dateInputs.forEach(input => {
                let config = { dateFormat: "d/m/Y", locale: "id", allowInput: true };
                if (input.id === 'taskDueDate' || input.id === 'editTaskDueDate') {
                     config.minDate = "today";
                }
                if (input.id === 'filterHistoryDateRange' || input.id === 'filterPerformanceDateRange' || input.id === 'filterReportDateRange') { // Tambah filterReportDateRange
                    config.mode = "range";
                } else if (input.id === 'filterDate') { 
                    config.mode = "single";
                }
                flatpickr(input, config);
            });

            const deleteButtons = document.querySelectorAll('a.delete-btn');
            deleteButtons.forEach(button => {
                button.addEventListener('click', function(event) {
                    if (!confirm('Anda yakin ingin menghapus item ini?')) {
                        event.preventDefault();
                    }
                });
            });

            const presetButtons = document.querySelectorAll('.filter-preset-buttons .btn[data-range]');
            presetButtons.forEach(button => { /* ... (Logika preset button sama) ... */ });
            
            const editProfileImageInput = document.getElementById('editProfileImageFile');
            const currentImagePreview = document.getElementById('currentImagePreview');
            if (editProfileImageInput && currentImagePreview) {
                editProfileImageInput.addEventListener('change', function(event) {
                    const file = event.target.files[0];
                    if (file) {
                        const reader = new FileReader();
                        reader.onload = function(e) {
                            currentImagePreview.src = e.target.result;
                        }
                        reader.readAsDataURL(file);
                    }
                });
            }


            // Tema Gelap Logic (FOUC Fix - Target HTML) - Sama seperti sebelumnya
            const themeToggleCheckbox = document.getElementById('themeToggleCheckbox');
            function updateThemeOnPage(theme) {
                if (theme === 'dark-theme') {
                    htmlElement.classList.add('dark-theme-active');
                    if (themeToggleCheckbox) themeToggleCheckbox.checked = true;
                } else { 
                    htmlElement.classList.remove('dark-theme-active');
                    if (themeToggleCheckbox) themeToggleCheckbox.checked = false;
                }
            }
            const initialThemeIsDark = htmlElement.classList.contains('dark-theme-active');
            if (initialThemeIsDark) {
                if (themeToggleCheckbox) themeToggleCheckbox.checked = true;
            } else {
                if (themeToggleCheckbox) themeToggleCheckbox.checked = false;
            }
            if (themeToggleCheckbox) {
                themeToggleCheckbox.addEventListener('change', function() {
                    const newTheme = this.checked ? 'dark-theme' : 'light-theme';
                    updateThemeOnPage(newTheme); 
                    localStorage.setItem('theme', newTheme); 
                });
            }
            
            // Sidebar Toggle Logic - Sama seperti sebelumnya
            const appTitleToggle = document.getElementById('appTitleToggle');
            const sidebarElement = document.getElementById('sidebar'); 
            if (appTitleToggle && sidebarElement) {
                function setSidebarState(isHidden) {
                    if (isHidden) bodyElement.classList.add('sidebar-hidden');
                    else bodyElement.classList.remove('sidebar-hidden');
                }
                if (window.innerWidth > 768) {
                    const sidebarHiddenStored = localStorage.getItem('sidebarHidden') === 'true';
                    setSidebarState(sidebarHiddenStored);
                } else {
                    setSidebarState(false); 
                }
                appTitleToggle.addEventListener('click', () => {
                    if (window.innerWidth > 768) { 
                        const isNowHidden = bodyElement.classList.toggle('sidebar-hidden');
                        localStorage.setItem('sidebarHidden', isNowHidden);
                    }
                });
                window.addEventListener('resize', () => {
                    if (window.innerWidth <= 768) {
                        setSidebarState(false); 
                    } else {
                        const sidebarHiddenStored = localStorage.getItem('sidebarHidden') === 'true';
                        setSidebarState(sidebarHiddenStored);
                    }
                });
            }

            // Klik Card Tugas untuk Ubah Status (Manajemen) - Sama seperti sebelumnya
            if (document.querySelector('.main-content-manajemen')) {
                const taskListContainerManajemen = document.getElementById('managementTaskList');
                if (taskListContainerManajemen) {
                    taskListContainerManajemen.addEventListener('click', function(event) {
                        const card = event.target.closest('.task-item-card[data-task-id]');
                        if (!card) return; 
                        if (event.target.closest('.task-actions') || event.target.closest('a.task-title-link')) {
                            return;
                        }
                        const taskId = card.dataset.taskId;
                        const currentStatus = card.dataset.currentStatus;
                        let nextStatus = '';
                        let nextStatusTextForUI = '';
                        let currentStatusClass = 'status-' + currentStatus.toLowerCase().replace(/ /g, '-');
                        let nextStatusClass = '';

                        if (currentStatus === 'Not Started') {
                            nextStatus = 'In Progress';
                            nextStatusTextForUI = 'Dikerjakan';
                            nextStatusClass = 'status-in-progress';
                        } else if (currentStatus === 'In Progress') {
                            nextStatus = 'Completed';
                            nextStatusTextForUI = 'Selesai';
                            nextStatusClass = 'status-completed-history'; 
                        } else {
                            return; 
                        }
                        const statusTextElement = card.querySelector('.meta-info .task-status-text');
                        
                        card.classList.remove(currentStatusClass);
                        card.classList.add(nextStatusClass);
                        card.dataset.currentStatus = nextStatus; 
                        if (statusTextElement) {
                            statusTextElement.textContent = nextStatusTextForUI;
                        }

                        fetch('ajax_update_task_status.php', {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/x-www-form-urlencoded', },
                            body: `task_id=${taskId}&new_status=${nextStatus}`
                        })
                        .then(response => response.json())
                        .then(data => {
                            if (data.success) {
                                if (statusTextElement && data.new_status_text) {
                                    statusTextElement.textContent = data.new_status_text;
                                }
                                if (data.is_completed) {
                                    card.style.transition = 'opacity 0.4s ease-out, transform 0.4s ease-out, max-height 0.5s ease-in-out, padding 0.5s ease-in-out, margin 0.5s ease-in-out';
                                    card.style.opacity = '0';
                                    card.style.transform = 'scale(0.9)';
                                    card.style.maxHeight = '0px';
                                    card.style.paddingTop = '0px';
                                    card.style.paddingBottom = '0px';
                                    card.style.marginBottom = '0px';
                                    setTimeout(() => {
                                        card.remove();
                                        if (taskListContainerManajemen.children.length === 0) {
                                            if (!taskListContainerManajemen.querySelector('.no-tasks-message')) {
                                                taskListContainerManajemen.innerHTML = '<p class="no-tasks-message">Tidak ada tugas aktif yang sesuai.</p>';
                                            }
                                        }
                                    }, 500); 
                                }
                            } else { 
                                card.classList.remove(nextStatusClass);
                                card.classList.add(currentStatusClass);
                                card.dataset.currentStatus = currentStatus;
                                if (statusTextElement) {
                                     const originalStatusText = currentStatus === 'Not Started' ? 'Belum Mulai' : (currentStatus === 'In Progress' ? 'Dikerjakan' : 'Selesai');
                                     statusTextElement.textContent = originalStatusText;
                                }
                                alert('Gagal memperbarui status: ' + (data.message || 'Error tidak diketahui.'));
                            }
                        })
                        .catch(error => { 
                            console.error('Error AJAX:', error);
                            card.classList.remove(nextStatusClass);
                            card.classList.add(currentStatusClass);
                            card.dataset.currentStatus = currentStatus;
                            if (statusTextElement) {
                                const originalStatusText = currentStatus === 'Not Started' ? 'Belum Mulai' : (currentStatus === 'In Progress' ? 'Dikerjakan' : 'Selesai');
                                statusTextElement.textContent = originalStatusText;
                            }
                            alert('Terjadi kesalahan koneksi saat memperbarui status.');
                        });
                    });
                }
            }
        });
    </script>
<?php else: // Ini adalah penutup untuk if (!$is_special_page_footer)
    // Untuk halaman auth, kita tidak perlu script di atas, tapi tetap butuh penutup body/html
?>
<?php endif; ?>
</body>
</html>
<?php
if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 9: Pembaruan CSS (`css/auth.css` dan `css/style.css`)**

**9.1. `css/auth.css` (Tambahan/Modifikasi)**
```css
/* css/auth.css */
html.auth-html, body.auth-page {
    height: 100%;
    overflow: hidden; 
}

body.auth-page {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh; 
    background-image: url('../images/auth-bg.jpg'); 
    background-size: cover;
    background-position: center;
    font-family: 'Poppins', sans-serif;
    padding: 20px;
    position: relative;
}
body.auth-page::before { 
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background-color: rgba(0,0,0,0.5);
    z-index: 1;
    transition: background-color 0.3s ease;
}

.auth-container {
    background: rgba(255, 255, 255, 0.95); 
    backdrop-filter: blur(5px); 
    padding: 30px 35px; 
    border-radius: 10px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
    width: 100%;
    max-width: 400px; /* Sedikit lebih lebar untuk tombol Google */
    text-align: center;
    position: relative;
    z-index: 2;
    animation: fadeInScale 0.5s ease-out;
    transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

@keyframes fadeInScale {
    from { opacity: 0; transform: scale(0.95); }
    to { opacity: 1; transform: scale(1); }
}

.auth-container .logo-container {
    margin-bottom: 15px;
}
.auth-container .logo-container h1 {
    font-size: 2.5rem;
    color: #7e47b8; 
    margin:0;
    font-weight: 600;
    transition: color 0.3s ease;
}

.auth-container h2 { 
    color: #333; 
    margin-bottom: 8px;
    font-size: 1.4rem; 
    font-weight: 500;
    transition: color 0.3s ease;
}
.auth-container p.subtitle {
    color: #555; 
    margin-bottom: 25px;
    font-size: 0.9rem; 
    transition: color 0.3s ease;
}

.form-group {
    margin-bottom: 18px; 
    text-align: left;
    position: relative; 
}
.form-group label {
    display: block; margin-bottom: 6px; font-weight: 500;
    color: #444; 
    font-size: 0.85rem; 
    transition: color 0.3s ease;
}
.form-group input[type="text"],
.form-group input[type="email"],
.form-group input[type="password"] {
    width: 100%;
    padding: 10px 12px; 
    border: 1px solid #ccc; 
    background-color: #fff; 
    color: #333; 
    border-radius: 6px;
    font-size: 0.9rem; 
    box-sizing: border-box;
    transition: border-color 0.2s, box-shadow 0.2s, background-color 0.3s, color 0.3s;
}
.form-group input:focus {
    border-color: #7e47b8; 
    outline: none;
    box-shadow: 0 0 0 3px rgba(126, 71, 184, 0.2); 
}
.form-group.forgot-password-link {
    text-align: right;
    margin-top: -10px; /* Tarik ke atas sedikit */
    margin-bottom: 15px;
}
.form-group.forgot-password-link a {
    font-size: 0.8rem;
    color: #7e47b8;
    text-decoration: none;
}
.form-group.forgot-password-link a:hover {
    text-decoration: underline;
}


.btn-submit {
    background-color: #7e47b8; 
    color: white;
    padding: 11px 20px; 
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.95rem; 
    font-weight: 500;
    transition: background-color 0.2s ease, transform 0.1s ease;
    display: block;
    width: 100%;
    margin-top: 10px;
}
.btn-submit:hover {
    background-color: #6a3aa2; 
}
.btn-submit:active {
    transform: translateY(1px);
}

/* Tombol Login Sosial */
.social-login-divider {
    margin: 20px 0;
    text-align: center;
    position: relative;
    color: #777;
    font-size: 0.85rem;
}
.social-login-divider::before,
.social-login-divider::after {
    content: '';
    position: absolute;
    top: 50%;
    width: 40%;
    height: 1px;
    background-color: #ddd;
}
.social-login-divider::before { left: 0; }
.social-login-divider::after { right: 0; }

.btn-social-login {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    padding: 10px 15px;
    border-radius: 6px;
    text-decoration: none;
    font-size: 0.9rem;
    font-weight: 500;
    margin-bottom: 15px;
    transition: background-color 0.2s ease, box-shadow 0.1s ease;
    border: 1px solid #ddd;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}
.btn-social-login i {
    margin-right: 10px;
    font-size: 1.2em;
}
.btn-social-login.google {
    background-color: #fff;
    color: #444;
    border-color: #ccc;
}
.btn-social-login.google:hover {
    background-color: #f8f8f8;
    border-color: #bbb;
    box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}
html.dark-theme-active .btn-social-login.google {
    background-color: #333;
    color: #eee;
    border-color: #555;
}
html.dark-theme-active .btn-social-login.google:hover {
    background-color: #404040;
    border-color: #666;
}


.auth-link {
    margin-top: 20px;
    font-size: 0.85rem; 
    color: #444; 
    transition: color 0.3s ease;
}
.auth-link a {
    color: #7e47b8; 
    text-decoration: none;
    font-weight: 500;
    transition: color 0.3s ease;
}
.auth-link a:hover { text-decoration: underline; }

.error-message, /* Kelas ini dari kode lama Anda */
.auth-message.error /* Kelas baru untuk konsistensi */
 { 
    color: #e74c3c;
    font-size: 0.8rem;
    margin-top: 5px;
    display: block;
    text-align: left; /* error-message lama */
    /* Untuk .auth-message */
    background-color: #f8d7da; 
    border: 1px solid #f5c6cb;
    padding: 10px 15px;
    border-radius: 5px;
    margin-bottom: 15px;
    text-align: center; /* auth-message baru */
}
.auth-message.error p { margin: 0.3em 0; }

.auth-message.success {
    background-color: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
    padding: 10px 15px;
    border-radius: 5px;
    margin-bottom: 15px;
    text-align: center;
}
.auth-message.success p { margin: 0.3em 0; }


/* ==========================================================================
   DARK THEME STYLES FOR AUTH PAGE
   ========================================================================== */

html.dark-theme-active body.auth-page::before {
    background-color: rgba(0,0,0,0.7); 
}

html.dark-theme-active .auth-container {
    background: rgba(30, 30, 30, 0.92); 
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.5); 
}

html.dark-theme-active .auth-container .logo-container h1 {
    color: #bb86fc; 
}

html.dark-theme-active .auth-container h2 {
    color: #e0e0e0; 
}

html.dark-theme-active .auth-container p.subtitle {
    color: #b0b0b0; 
}

html.dark-theme-active .form-group label {
    color: #c0c0c0; 
}
html.dark-theme-active .form-group.forgot-password-link a {
    color: #bb86fc;
}

html.dark-theme-active .form-group input[type="text"],
html.dark-theme-active .form-group input[type="email"],
html.dark-theme-active .form-group input[type="password"] {
    background-color: #2c2c2c; 
    border-color: #555; 
    color: #e0e0e0; 
}

html.dark-theme-active .form-group input:focus {
    border-color: #bb86fc; 
    box-shadow: 0 0 0 3px rgba(187, 134, 252, 0.3); 
    background-color: #333; 
}

html.dark-theme-active .btn-submit {
    background-color: #bb86fc; 
    color: #121212; 
}
html.dark-theme-active .btn-submit:hover {
    background-color: #a06fec; 
}

html.dark-theme-active .auth-link {
    color: #b0b0b0; 
}
html.dark-theme-active .auth-link a {
    color: #bb86fc; 
}

html.dark-theme-active .social-login-divider {
    color: #aaa;
}
html.dark-theme-active .social-login-divider::before,
html.dark-theme-active .social-login-divider::after {
    background-color: #444;
}

html.dark-theme-active .auth-message.error {
    background-color: #4d2a2b; /* Lebih gelap untuk dark mode */
    color: #ffab91; /* Teks lebih terang */
    border-color: #8d4c47;
}
html.dark-theme-active .auth-message.success {
    background-color: #2a4d32;
    color: #a5d6a7;
    border-color: #4c8c4a;
}

/* (Pesan error biasanya sudah cukup kontras, tapi bisa disesuaikan jika perlu) */
/* html.dark-theme-active .error-message {
    color: #ff8a80;
} */
```

**9.2. `css/style.css` (Tambahan/Modifikasi)**
```css
/* ... (Semua kode CSS Anda sebelumnya) ... */

/* ==========================================================================
   Laporan Page Styles
   ========================================================================== */
.report-chart-widget {
    margin-bottom: 20px;
}
.report-performance-filter {
    display: flex;
    gap: 8px;
    align-items: center;
    margin-bottom: 10px;
    flex-wrap: nowrap; /* Coba nowrap untuk filter diagram */
    padding: 5px 0;
}
.report-performance-filter label {
    font-size: 0.85rem;
    white-space: nowrap;
    margin-right: 5px;
    color: #555;
}
html.dark-theme-active .report-performance-filter label { color: #ccc; }

.report-performance-filter .filter-preset-buttons {
    display: flex;
    gap: 5px;
}
.report-performance-filter .filter-preset-buttons .btn {
    padding: 5px 10px;
    font-size: 0.75rem;
    height: 30px;
    background-color: #f0f3f7;
    color: #555;
    border: 1px solid #dfe3e8;
}
.report-performance-filter .filter-preset-buttons .btn.active,
.report-performance-filter .filter-preset-buttons .btn:hover {
    background-color: #7e47b8;
    color: white;
    border-color: #7e47b8;
}
html.dark-theme-active .report-performance-filter .filter-preset-buttons .btn {
    background-color: #3a3a3a;
    color: #ccc;
    border-color: #555;
}
html.dark-theme-active .report-performance-filter .filter-preset-buttons .btn.active,
html.dark-theme-active .report-performance-filter .filter-preset-buttons .btn:hover {
    background-color: #bb86fc;
    color: #121212;
    border-color: #bb86fc;
}


.report-performance-filter input[type="text"] {
    padding: 5px 8px;
    font-size: 0.8rem;
    height: 30px;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    width: 150px; /* Atau sesuaikan */
    margin-left: 5px;
}
html.dark-theme-active .report-performance-filter input[type="text"]{
    background-color: #373737;
    border-color: #555;
    color: #e0e0e0;
}


.report-performance-filter .btn-apply-perf {
    padding: 5px 12px;
    font-size: 0.8rem;
    height: 30px;
    margin-left: 5px;
}


.report-tasks-by-date-section {
    /* height: calc(100vh - 55px - 30px - 40px - 250px - 20px - 60px); Sesuaikan jika perlu */
}

.report-interactive-area {
    display: flex;
    gap: 20px;
    height: 100%; /* Atau tinggi spesifik */
    max-height: 60vh; /* Batasi tinggi agar tidak terlalu panjang */
}

.report-task-list-container {
    flex: 2; /* Lebih banyak ruang untuk daftar tugas */
    display: flex;
    flex-direction: column;
    overflow: hidden;
}
.selected-date-indicator {
    font-size: 0.95rem;
    font-weight: 500;
    color: #34495e;
    margin-bottom: 10px;
    padding-bottom: 5px;
    border-bottom: 1px solid #eee;
}
html.dark-theme-active .selected-date-indicator {
    color: #f5f5f5;
    border-bottom-color: #444;
}

#reportTasksList {
    flex-grow: 1;
    overflow-y: auto;
    padding-right: 8px; /* Untuk scrollbar */
}
#reportTasksList .task-item-card { /* Style card tugas mungkin sudah ada, ini hanya contoh */
    font-size: 0.8rem; /* Lebih kecil di laporan */
    padding: 10px;
}
#reportTasksList .task-item-card .task-details strong { font-size: 0.9rem; }


.report-calendar-container {
    flex: 1;
    min-width: 280px; /* Agar kalender tidak terlalu sempit */
    background-color: #f9fafb;
    border-radius: 6px;
    padding: 15px;
    border: 1px solid #e7eaec;
    box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    height: fit-content; /* Agar tingginya pas dengan konten kalender */
}
html.dark-theme-active .report-calendar-container {
    background-color: #373737;
    border-color: #555;
}

.calendar-header-report {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
    font-weight: 600;
    font-size: 1.05rem;
    color: #2c3e50;
}
html.dark-theme-active .calendar-header-report { color: #f0f0f0; }
.calendar-header-report button {
    background: none;
    border: none;
    font-size: 1.2rem;
    color: #7e47b8;
    cursor: pointer;
    padding: 5px 8px;
    border-radius: 4px;
}
.calendar-header-report button:hover { background-color: #f0e9f7; }
html.dark-theme-active .calendar-header-report button { color: #bb86fc; }
html.dark-theme-active .calendar-header-report button:hover { background-color: rgba(187, 134, 252, 0.1); }

.calendar-days-grid-report {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 5px;
    text-align: center;
}
.calendar-day-name-report {
    font-weight: 500;
    font-size: 0.8rem;
    color: #555;
    padding-bottom: 5px;
}
html.dark-theme-active .calendar-day-name-report { color: #bbb; }

.calendar-day-report {
    padding: 8px 5px;
    font-size: 0.85rem;
    border-radius: 50%; /* Buat lingkaran */
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s;
    width: 32px; /* Ukuran tetap untuk lingkaran */
    height: 32px; /* Ukuran tetap untuk lingkaran */
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto; /* Pusatkan lingkaran */
    border: 1px solid transparent; /* Untuk hover border */
}
.calendar-day-report:hover {
    background-color: #eef1f5;
    border-color: #d1d5db;
}
.calendar-day-report.today {
    background-color: #7e47b8;
    color: white;
    font-weight: bold;
}
.calendar-day-report.selected {
    background-color: #34495e;
    color: white;
    border-color: #2c3e50;
}
html.dark-theme-active .calendar-day-report:hover {
    background-color: #4f4f4f;
    border-color: #666;
}
html.dark-theme-active .calendar-day-report.today {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active .calendar-day-report.selected {
    background-color: #5fcf80; /* Contoh warna selected dark mode */
    color: #121212;
    border-color: #5fcf80;
}


/* ==========================================================================
   Chatbot Styles
   ========================================================================== */
#chatbot-icon {
    position: fixed;
    bottom: 25px;
    right: 25px;
    width: 55px;
    height: 55px;
    background-color: #7e47b8;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.8rem;
    cursor: pointer;
    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    z-index: 1010;
    transition: transform 0.2s ease-in-out, background-color 0.2s;
}
#chatbot-icon:hover {
    transform: scale(1.1);
    background-color: #6a3aa2;
}
html.dark-theme-active #chatbot-icon {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #chatbot-icon:hover {
    background-color: #a06fec;
}


#chatbot-container {
    position: fixed;
    bottom: 25px;
    right: 25px;
    width: 350px;
    max-width: 90vw;
    height: 75vh;
    max-height: 500px;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 5px 20px rgba(0,0,0,0.2);
    display: none; /* Awalnya tersembunyi */
    flex-direction: column;
    overflow: hidden;
    z-index: 1009;
    border: 1px solid #ddd;
}
#chatbot-container.show {
    display: flex;
    animation: fadeInScaleChatbot 0.3s ease-out;
}
@keyframes fadeInScaleChatbot {
    from { opacity: 0; transform: scale(0.9) translateY(20px); }
    to { opacity: 1; transform: scale(1) translateY(0); }
}
html.dark-theme-active #chatbot-container {
    background-color: #2c2c2c;
    border-color: #444;
    box-shadow: 0 5px 25px rgba(0,0,0,0.4);
}


#chatbot-header {
    background-color: #7e47b8;
    color: white;
    padding: 12px 15px;
    font-weight: 600;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-shrink: 0;
}
html.dark-theme-active #chatbot-header {
    background-color: #bb86fc;
    color: #121212;
}
#closeChatbotBtn {
    background: none;
    border: none;
    color: white;
    font-size: 1.5rem;
    cursor: pointer;
    line-height: 1;
}
html.dark-theme-active #closeChatbotBtn { color: #121212; }

#chatbot-messages {
    flex-grow: 1;
    padding: 15px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 10px;
}
.chatbot-message {
    padding: 8px 12px;
    border-radius: 15px;
    max-width: 80%;
    word-wrap: break-word;
    line-height: 1.4;
    font-size: 0.9rem;
}
.chatbot-message.user {
    background-color: #e9eaf6; /* Warna user */
    color: #333;
    border-bottom-right-radius: 3px;
    align-self: flex-end;
}
html.dark-theme-active .chatbot-message.user {
    background-color: #555273; /* Dark user */
    color: #e0e0e0;
}
.chatbot-message.bot, .chatbot-message.bot-typing {
    background-color: #f1f0f0; /* Warna bot */
    color: #333;
    border-bottom-left-radius: 3px;
    align-self: flex-start;
}
html.dark-theme-active .chatbot-message.bot, html.dark-theme-active .chatbot-message.bot-typing {
    background-color: #3a3a3a; /* Dark bot */
    color: #e0e0e0;
}
.chatbot-message.bot-typing { font-style: italic; }


#chatbot-input-area {
    display: flex;
    padding: 10px;
    border-top: 1px solid #eee;
    background-color: #f9f9f9;
    flex-shrink: 0;
}
html.dark-theme-active #chatbot-input-area {
    border-top-color: #444;
    background-color: #333;
}
#chatbotInput {
    flex-grow: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 20px;
    margin-right: 8px;
    font-size: 0.9rem;
    outline: none;
}
html.dark-theme-active #chatbotInput {
    background-color: #252525;
    border-color: #555;
    color: #e0e0e0;
}
#sendChatbotMessageBtn {
    background-color: #7e47b8;
    color: white;
    border: none;
    border-radius: 50%;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 1.1rem;
}
#sendChatbotMessageBtn:hover { background-color: #6a3aa2; }
html.dark-theme-active #sendChatbotMessageBtn {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #sendChatbotMessageBtn:hover { background-color: #a06fec; }

/* Penyesuaian Responsive untuk Laporan & Chatbot */
@media (max-width: 992px) {
    .report-interactive-area {
        flex-direction: column;
        max-height: none; /* Hapus batasan tinggi di tablet/mobile */
    }
    .report-calendar-container {
        order: -1; /* Pindahkan kalender ke atas di tampilan sempit */
        margin-bottom: 15px;
    }
}
@media (max-width: 768px) {
    #chatbot-container {
        bottom: 0;
        right: 0;
        width: 100%;
        height: 100%;
        max-height: 100vh;
        border-radius: 0;
        border: none;
    }
}

/* ... (Sisa CSS Anda) ... */
```

---

**LANGKAH 10: Tutorial Integrasi**

**A. Integrasi Akun Google (Google OAuth)**

1.  **Buat Proyek di Google Cloud Console:**
    *   Buka [Google Cloud Console](https://console.cloud.google.com/).
    *   Buat proyek baru (atau pilih yang sudah ada).
    *   Di menu navigasi, buka "APIs & Services" > "Credentials".

2.  **Konfigurasi OAuth Consent Screen:**
    *   Jika ini pertama kali Anda membuat kredensial OAuth, Anda akan diminta untuk mengkonfigurasi "OAuth consent screen".
    *   Pilih "External" untuk User Type jika aplikasi Anda akan diakses oleh pengguna di luar organisasi Google Workspace Anda.
    *   Isi "App name", "User support email", dan "Developer contact information".
    *   Pada bagian "Scopes", klik "Add or Remove Scopes". Cari dan tambahkan:
        *   `.../auth/userinfo.email` (untuk mendapatkan alamat email)
        *   `.../auth/userinfo.profile` (untuk mendapatkan info profil dasar seperti nama dan gambar)
        *   Klik "Update".
    *   Pada bagian "Test users", tambahkan alamat email Anda sendiri untuk pengujian awal.
    *   Simpan dan lanjutkan.

3.  **Buat Kredensial OAuth 2.0 Client ID:**
    *   Di halaman "Credentials", klik "+ CREATE CREDENTIALS" > "OAuth client ID".
    *   Pilih "Web application" untuk Application type.
    *   Beri nama, misalnya "ListIn Web Client".
    *   Pada "Authorized JavaScript origins", tambahkan URL dasar aplikasi Anda (misal, `http://localhost` atau `http://nama_domain_anda.com`).
    *   Pada "Authorized redirect URIs", tambahkan URL callback yang telah kita buat: `http://localhost/nama_folder_proyek_anda/google_auth_callback.php` (sesuaikan dengan `APP_URL` di `config.php`). **Ini sangat penting dan harus cocok persis.**
    *   Klik "Create".
    *   Anda akan mendapatkan **Client ID** dan **Client Secret**. Salin keduanya.

4.  **Masukkan Kredensial ke `config.php`:**
    *   Buka file `config.php` Anda.
    *   Masukkan Client ID ke `define('GOOGLE_CLIENT_ID', 'MASUKKAN_CLIENT_ID_ANDA_DISINI');`
    *   Masukkan Client Secret ke `define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_CLIENT_SECRET_ANDA_DISINI');`
    *   Pastikan `GOOGLE_REDIRECT_URI` di `config.php` sama persis dengan yang Anda masukkan di Google Cloud Console.

5.  **Install Google API Client Library untuk PHP:**
    *   Cara termudah adalah menggunakan Composer. Buka terminal di root direktori proyek Anda dan jalankan:
        ```bash
        composer require google/apiclient:^2.0
        ```
    *   Ini akan membuat folder `vendor` dan file `vendor/autoload.php`.
    *   Pastikan baris `require_once __DIR__ . '/vendor/autoload.php';` ada dan benar di `google_auth_callback.php`, `login.php`, dan `register.php`.

6.  **Testing:**
    *   Coba login atau register menggunakan tombol Google.
    *   Periksa apakah Anda diarahkan ke halaman persetujuan Google dan kemudian kembali ke aplikasi Anda.
    *   Cek database `users` apakah `google_id` terisi.

**B. Integrasi PHPMailer (untuk Lupa Password)**

1.  **Install PHPMailer:**
    *   Gunakan Composer:
        ```bash
        composer require phpmailer/phpmailer
        ```
    *   Pastikan `require __DIR__ . '/vendor/autoload.php';` ada di `forgot_password.php`.

2.  **Konfigurasi SMTP di `config.php`:**
    *   Isi konstanta `SMTP_HOST`, `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_PORT`, `SMTP_SECURE`, `EMAIL_FROM_ADDRESS`, dan `EMAIL_FROM_NAME` dengan detail penyedia email Anda.
    *   **Untuk Gmail:**
        *   `SMTP_HOST`: `smtp.gmail.com`
        *   `SMTP_PORT`: `587`
        *   `SMTP_SECURE`: `tls`
        *   `SMTP_USERNAME`: Alamat Gmail Anda.
        *   `SMTP_PASSWORD`: Gunakan **App Password** jika Anda mengaktifkan 2-Step Verification di akun Google Anda. Jangan gunakan password utama Gmail Anda. Jika tidak pakai 2FA, Anda mungkin perlu mengaktifkan "Less secure app access" (tidak direkomendasikan).
    *   **Untuk penyedia lain:** Cek dokumentasi mereka untuk detail SMTP.

3.  **Testing:**
    *   Coba fitur "Lupa Kata Sandi".
    *   Periksa apakah email diterima dan tautan reset berfungsi.

**C. Integrasi API Key Chatbot Anda**

1.  **Dapatkan API Key:**
    *   Daftar ke layanan chatbot yang Anda pilih (misalnya, Dialogflow, Rasa, OpenAI API, atau layanan lain).
    *   Dapatkan API Key dan, jika ada, URL Endpoint API dari dashboard layanan tersebut.

2.  **Masukkan ke `config.php`:**
    *   Buka `config.php`.
    *   Masukkan API Key Anda ke `define('CHATBOT_API_KEY', 'MASUKKAN_API_KEY_CHATBOT_ANDA_DISINI');`
    *   Jika layanan chatbot Anda memiliki URL endpoint spesifik, masukkan ke `define('CHATBOT_API_ENDPOINT', 'URL_ENDPOINT_CHATBOT_ANDA');`

3.  **Sesuaikan `ajax_chatbot_handler.php`:**
    *   Buka `ajax_chatbot_handler.php`.
    *   Temukan bagian yang ditandai `// BAGIAN INI ADALAH LOGIKA INTI CHATBOT ANDA`.
    *   Modifikasi kode di sana untuk:
        *   Mengirim `user_message` ke `CHATBOT_API_ENDPOINT` menggunakan `CHATBOT_API_KEY` (biasanya via cURL atau GuzzleHTTP). Format request (JSON, form-data, header otorisasi) akan bergantung pada dokumentasi API chatbot Anda.
        *   Menerima dan mem-parse respons dari API chatbot. Respons ini mungkin berisi teks balasan langsung, atau data terstruktur yang berisi "intent" (maksud pengguna) dan "entities" (parameter).
        *   **Implementasi Aksi Internal:** Berdasarkan intent yang diterima dari API chatbot (atau analisis teks manual jika API-nya sederhana), implementasikan logika untuk melakukan aksi-aksi yang diminta (menampilkan tugas, menghapus notifikasi, menghapus tugas tertentu, membersihkan riwayat, dll.). Ini melibatkan query ke database Anda.
            *   **Keamanan:** Hati-hati saat melakukan aksi destruktif (seperti menghapus data) berdasarkan input chatbot. Pastikan Anda memvalidasi input dan hak akses pengguna.
        *   Format `$bot_reply` dengan respons yang sesuai.

4.  **Testing:**
    *   Buka chatbot di aplikasi Anda.
    *   Kirim berbagai perintah yang ingin Anda dukung.
    *   Periksa apakah chatbot merespons dengan benar dan apakah aksi internal (jika ada) berhasil dilakukan.
    *   Periksa log error PHP jika ada masalah.

---

Ini adalah implementasi yang sangat luas. Silakan implementasikan bagian per bagian dan uji secara menyeluruh. Jika Anda mengalami kendala spesifik pada bagian tertentu, jangan ragu untuk bertanya lebih lanjut dengan menyertakan kode yang relevan dan pesan error yang Anda dapatkan.





Baik, ini adalah pengembangan yang cukup signifikan. Saya akan mencoba memberikan kode selengkap mungkin dan menjelaskan langkah-langkahnya. Mengingat kompleksitasnya, terutama untuk integrasi Google dan Chatbot, Anda mungkin perlu melakukan beberapa penyesuaian dan debugging.

**Struktur Perubahan:**

1.  **Database:** Perubahan pada tabel `users` dan tabel baru untuk reset password.
2.  **Konfigurasi:** File baru untuk menyimpan kredensial API (Google, PHPMailer).
3.  **Google OAuth:**
    *   Tombol di `login.php` dan `register.php`.
    *   File callback `google_auth_callback.php`.
    *   Logika di `includes/db.php` atau file konfigurasi.
4.  **Lupa Password:**
    *   Link di `login.php`.
    *   File `forgot_password.php`.
    *   File `reset_password.php`.
    *   Fungsi pengiriman email.
5.  **Halaman Laporan (`laporan.php`):**
    *   Diagram batang (Chart.js).
    *   Kalender interaktif.
    *   Daftar tugas berdasarkan tanggal terpilih (AJAX).
    *   File AJAX `ajax_get_tasks_for_date.php`.
6.  **Chatbot:**
    *   Ikon chatbot di `includes/footer.php`.
    *   UI Chatbot (HTML/CSS/JS).
    *   File AJAX `ajax_chatbot_handler.php` (ini akan menjadi kerangka, karena logika inti bergantung pada API Anda).
7.  **CSS:** Penambahan dan modifikasi pada `css/style.css` dan `css/auth.css`.
8.  **File yang Dimodifikasi:** `login.php`, `register.php`, `includes/db.php`, `includes/header.php`, `includes/footer.php`, `includes/sidebar.php`, `css/style.css`, `css/auth.css`, dan `db_listin.sql`.
9.  **File Baru:** `config.php`, `google_auth_callback.php`, `forgot_password.php`, `reset_password.php`, `laporan.php`, `ajax_get_tasks_for_date.php`, `ajax_chatbot_handler.php`.

**PENTING: KUNCI API & KREDENSIAL**
Anda harus mendapatkan kunci API dan kredensial sendiri untuk:
*   **Google OAuth:** Dari Google Cloud Console.
*   **PHPMailer (untuk Lupa Password):** Kredensial SMTP dari penyedia email Anda (misalnya, Gmail App Password jika menggunakan Gmail).
*   **Chatbot API Key:** Dari penyedia layanan chatbot yang Anda pilih.

Mari kita mulai.

---

**LANGKAH 1: Perubahan Database**

Perbarui file `db_listin.sql` Anda.

**db_listin.sql (Diperbarui)**
```sql
-- --------------------------------------------------------
-- Host:                         127.0.0.1
-- Server version:               5.7.33 - MySQL Community Server (GPL)
-- Server OS:                    Win64
-- HeidiSQL Version:             11.2.0.6213
-- --------------------------------------------------------

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET NAMES utf8 */;
/*!50503 SET NAMES utf8mb4 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;


-- Dumping database structure for db_listin
CREATE DATABASE IF NOT EXISTS `db_listin` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;
USE `db_listin`;

-- Dumping structure for table db_listin.users
CREATE TABLE IF NOT EXISTS `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `google_id` varchar(255) DEFAULT NULL, -- Untuk Google OAuth
  `username` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) DEFAULT NULL, -- Bisa NULL jika login via Google
  `profile_image` varchar(255) DEFAULT 'images/placeholder-profile.png',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `email_verified_at` timestamp NULL DEFAULT NULL, -- Untuk verifikasi email jika diperlukan
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`),
  UNIQUE KEY `google_id` (`google_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Dumping structure for table db_listin.tasks
CREATE TABLE IF NOT EXISTS `tasks` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` text,
  `priority` enum('Low','Medium','High') NOT NULL DEFAULT 'Medium',
  `status` enum('Not Started','In Progress','Completed') NOT NULL DEFAULT 'Not Started',
  `due_date` date DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `last_overdue_notif_sent` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  KEY `status` (`status`),
  KEY `due_date` (`due_date`),
  CONSTRAINT `tasks_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Dumping structure for table db_listin.password_resets
CREATE TABLE IF NOT EXISTS `password_resets` (
  `email` varchar(100) NOT NULL,
  `token` varchar(255) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`email`),
  KEY `token_idx` (`token`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;


/*!40101 SET SQL_MODE=IFNULL(@OLD_SQL_MODE, '') */;
/*!40014 SET FOREIGN_KEY_CHECKS=IFNULL(@OLD_FOREIGN_KEY_CHECKS, 1) */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40111 SET SQL_NOTES=IFNULL(@OLD_SQL_NOTES, 1) */;
```
**Perubahan Utama:**
*   Tabel `users`:
    *   Tambah kolom `google_id` (unik) untuk menyimpan ID unik pengguna dari Google.
    *   Kolom `password` sekarang `DEFAULT NULL` karena pengguna Google mungkin tidak punya password lokal.
    *   Tambah kolom `email_verified_at` (opsional, tapi bagus untuk masa depan).
    *   Menggunakan `utf8mb4_unicode_ci` untuk dukungan karakter yang lebih baik.
*   Tabel `password_resets`: Tabel baru untuk menyimpan token reset password.

---

**LANGKAH 2: File Konfigurasi (`config.php`)**

Buat file `config.php` di root proyek Anda (sejajar dengan `login.php`).

**config.php (Baru)**
```php
<?php
// File: config.php

// Pengaturan Aplikasi
define('APP_URL', 'http://localhost/nama_folder_proyek_anda'); // Ganti dengan URL proyek Anda

// Konfigurasi Database (pindahkan dari db.php jika mau)
define('DB_HOST', '127.0.0.1');
define('DB_USER', 'root');
define('DB_PASS', '');
define('DB_NAME', 'db_listin');

// Pengaturan Google OAuth
define('GOOGLE_CLIENT_ID', 'MASUKKAN_CLIENT_ID_ANDA_DISINI');
define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_CLIENT_SECRET_ANDA_DISINI');
define('GOOGLE_REDIRECT_URI', APP_URL . '/google_auth_callback.php');

// Pengaturan PHPMailer (untuk Lupa Password)
define('SMTP_HOST', 'smtp.example.com'); // Ganti dengan host SMTP Anda
define('SMTP_USERNAME', 'user@example.com'); // Ganti dengan username SMTP Anda
define('SMTP_PASSWORD', 'password_smtp_anda'); // Ganti dengan password SMTP Anda
define('SMTP_PORT', 587); // Biasanya 587 untuk TLS, 465 untuk SSL
define('SMTP_SECURE', 'tls'); // 'tls' atau 'ssl'
define('EMAIL_FROM_ADDRESS', 'no-reply@example.com'); // Alamat email pengirim
define('EMAIL_FROM_NAME', 'List In App'); // Nama pengirim

// Pengaturan Chatbot (Ganti dengan API Key Anda)
define('CHATBOT_API_KEY', 'MASUKKAN_API_KEY_CHATBOT_ANDA_DISINI');
define('CHATBOT_API_ENDPOINT', 'URL_ENDPOINT_CHATBOT_ANDA'); // Jika ada endpoint spesifik

// Path untuk Library Google API Client (jika diinstal via Composer)
// Biasanya: dirname(__FILE__) . '/vendor/autoload.php'
// Jika manual, sesuaikan pathnya.
// define('GOOGLE_API_CLIENT_LIBRARY_PATH', dirname(__FILE__) . '/path/to/google-api-php-client/vendor/autoload.php');

// Set timezone
date_default_timezone_set('Asia/Jakarta');

// Mulai session di sini agar tersedia di semua file
if (session_status() == PHP_SESSION_NONE) {
    session_start();
}
?>
```
**Catatan:**
*   Ganti placeholder dengan nilai asli Anda.
*   Untuk `GOOGLE_API_CLIENT_LIBRARY_PATH`, Anda perlu menginstal Google API Client Library. Cara termudah adalah via Composer: `composer require google/apiclient:^2.0`. Jika ya, maka pathnya akan `dirname(__FILE__) . '/vendor/autoload.php'`.

---

**LANGKAH 3: Modifikasi `includes/db.php`**

File ini sekarang akan fokus pada koneksi DB dan memuat konfigurasi.

**includes/db.php (Dimodifikasi)**
```php
<?php
// File: includes/db.php
require_once dirname(__DIR__) . '/config.php'; // Memuat config.php dari root direktori

// Buat Koneksi
$conn = new mysqli(DB_HOST, DB_USER, DB_PASS, DB_NAME);

// Cek Koneksi
if ($conn->connect_error) {
    // Jangan tampilkan detail error di produksi
    // error_log("Koneksi Gagal: " . $conn->connect_error); // Log error
    die("Tidak dapat terhubung ke database. Silakan coba lagi nanti."); // Pesan umum untuk user
}

// Set character set (penting untuk UTF-8)
if (!$conn->set_charset("utf8mb4")) {
    // error_log("Error loading character set utf8mb4: " . $conn->error);
    // die("Kesalahan konfigurasi database.");
}

// Session sudah dimulai di config.php
?>
```

---

**LANGKAH 4: Integrasi Google OAuth**

Anda perlu Google API Client Library. Jika belum, install via Composer:
`composer require google/apiclient:^2.0`
Jika menggunakan Composer, pastikan Anda `require 'vendor/autoload.php';` di awal file yang membutuhkannya (`config.php` atau `google_auth_callback.php`).

**4.1. `google_auth_callback.php` (Baru - di root proyek)**
```php
<?php
// File: google_auth_callback.php
require_once 'config.php'; // Untuk GOOGLE_CLIENT_ID, dll. dan session_start()
require_once 'includes/db.php'; // Untuk koneksi $conn

// Pastikan Anda telah menginstal Google API Client Library
// Jika via Composer:
require_once __DIR__ . '/vendor/autoload.php';
// Jika manual, sesuaikan path ke autoload Google API Client
// require_once GOOGLE_API_CLIENT_LIBRARY_PATH;


$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI);
$client->addScope("email");
$client->addScope("profile");

if (isset($_GET['code'])) {
    $token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
    if (isset($token['error'])) {
        // Error saat mengambil access token
        $_SESSION['login_error_message'] = 'Gagal mendapatkan token dari Google: ' . htmlspecialchars($token['error_description'] ?? $token['error']);
        header('Location: login.php');
        exit();
    }
    $client->setAccessToken($token['access_token']);

    // Dapatkan info profil pengguna
    $google_oauth = new Google_Service_Oauth2($client);
    $google_account_info = $google_oauth->userinfo->get();
    
    $google_id = $google_account_info->id;
    $email = $google_account_info->email;
    $name = $google_account_info->name;
    $profile_pic_url = $google_account_info->picture;

    // Cek apakah pengguna sudah ada di database
    $stmt = $conn->prepare("SELECT id, username, profile_image FROM users WHERE google_id = ? OR email = ?");
    if (!$stmt) {
        $_SESSION['login_error_message'] = "Database error (prepare): " . $conn->error;
        header('Location: login.php');
        exit();
    }
    $stmt->bind_param("ss", $google_id, $email);
    $stmt->execute();
    $result = $stmt->get_result();
    $user = $result->fetch_assoc();
    $stmt->close();

    if ($user) {
        // Pengguna sudah ada, login
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username']; // Bisa jadi username lokalnya berbeda
        
        // Update google_id jika login via email tapi google_id belum ada
        if (empty($user['google_id']) && $user['email'] == $email) {
            $stmt_update_gid = $conn->prepare("UPDATE users SET google_id = ? WHERE id = ?");
            if($stmt_update_gid){
                $stmt_update_gid->bind_param("si", $google_id, $user['id']);
                $stmt_update_gid->execute();
                $stmt_update_gid->close();
            }
        }
        
        // Ambil gambar profil dari Google jika pengguna belum punya atau masih placeholder
        $current_db_image = $user['profile_image'];
        if ((empty($current_db_image) || $current_db_image == 'images/placeholder-profile.png') && !empty($profile_pic_url)) {
            // Coba unduh dan simpan gambar profil dari Google
            $image_data = @file_get_contents($profile_pic_url);
            if ($image_data !== false) {
                $upload_dir = 'uploads/profile_pictures/';
                if (!is_dir($upload_dir)) {
                    mkdir($upload_dir, 0755, true);
                }
                $filename = 'google_user_' . $user['id'] . '_' . time() . '.jpg'; // Asumsi jpg
                $filepath = $upload_dir . $filename;
                if (file_put_contents($filepath, $image_data)) {
                    $stmt_update_pic = $conn->prepare("UPDATE users SET profile_image = ? WHERE id = ?");
                    if($stmt_update_pic){
                        $stmt_update_pic->bind_param("si", $filepath, $user['id']);
                        $stmt_update_pic->execute();
                        $stmt_update_pic->close();
                        $_SESSION['profile_image'] = $filepath;
                    }
                }
            }
        } else {
            $_SESSION['profile_image'] = $current_db_image;
        }

        header('Location: dashboard.php');
        exit();
    } else {
        // Pengguna baru, daftarkan
        $default_profile_image_path = 'images/placeholder-profile.png'; // Default
        $new_user_image_path = $default_profile_image_path;

        // Coba unduh dan simpan gambar profil dari Google untuk pengguna baru
        if (!empty($profile_pic_url)) {
            $image_data_new = @file_get_contents($profile_pic_url);
            if ($image_data_new !== false) {
                $upload_dir_new = 'uploads/profile_pictures/';
                 if (!is_dir($upload_dir_new)) {
                    mkdir($upload_dir_new, 0755, true);
                }
                // Perlu ID user baru, jadi kita insert dulu baru update gambar
                // Atau, simpan sementara dan update setelah user ID didapat
            }
        }

        $stmt_insert = $conn->prepare("INSERT INTO users (google_id, username, email, profile_image, email_verified_at) VALUES (?, ?, ?, ?, NOW())");
        if (!$stmt_insert) {
             $_SESSION['login_error_message'] = "Database error (insert prepare): " . $conn->error;
            header('Location: login.php');
            exit();
        }
        // Gunakan $name sebagai username awal, $email, dan $new_user_image_path
        $stmt_insert->bind_param("ssss", $google_id, $name, $email, $new_user_image_path);
        if ($stmt_insert->execute()) {
            $new_user_id = $conn->insert_id;
            $_SESSION['user_id'] = $new_user_id;
            $_SESSION['username'] = $name;

            // Sekarang coba simpan gambar profil jika berhasil diunduh sebelumnya
            if (isset($image_data_new) && $image_data_new !== false) {
                $filename_new = 'google_user_' . $new_user_id . '_' . time() . '.jpg';
                $filepath_new = $upload_dir_new . $filename_new;
                if (file_put_contents($filepath_new, $image_data_new)) {
                    $stmt_update_new_pic = $conn->prepare("UPDATE users SET profile_image = ? WHERE id = ?");
                     if($stmt_update_new_pic){
                        $stmt_update_new_pic->bind_param("si", $filepath_new, $new_user_id);
                        $stmt_update_new_pic->execute();
                        $stmt_update_new_pic->close();
                        $_SESSION['profile_image'] = $filepath_new;
                     }
                } else {
                     $_SESSION['profile_image'] = $default_profile_image_path;
                }
            } else {
                $_SESSION['profile_image'] = $default_profile_image_path;
            }

            header('Location: dashboard.php'); // Arahkan ke dashboard
            exit();
        } else {
            $_SESSION['login_error_message'] = "Gagal mendaftarkan pengguna baru: " . $stmt_insert->error;
            header('Location: login.php');
            exit();
        }
        $stmt_insert->close();
    }
} else {
    // Tidak ada kode otorisasi, mungkin akses langsung atau error
    $_SESSION['login_error_message'] = 'Akses tidak sah atau otorisasi Google dibatalkan.';
    header('Location: login.php');
    exit();
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```
**4.2. Modifikasi `login.php`**

Tambahkan tombol login Google.

**login.php (Dimodifikasi)**
```php
<?php
require_once 'config.php'; // Untuk GOOGLE_CLIENT_ID, dll. dan session_start()
require_once 'includes/db.php';

// Setup Google Client untuk mendapatkan URL otorisasi
// Pastikan Anda telah menginstal Google API Client Library
// Jika via Composer:
require_once __DIR__ . '/vendor/autoload.php';
// Jika manual, sesuaikan path
// require_once GOOGLE_API_CLIENT_LIBRARY_PATH;


$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI);
$client->addScope("email");
$client->addScope("profile");
$google_login_url = $client->createAuthUrl();

$errors = [];

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

// Ambil pesan error dari session jika ada (misal dari google_auth_callback)
if (isset($_SESSION['login_error_message'])) {
    $errors[] = $_SESSION['login_error_message'];
    unset($_SESSION['login_error_message']);
}


if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['login_submit'])) { // Tambahkan name ke tombol submit
    $email = trim($_POST['email']);
    $password = $_POST['password'];

    if (empty($email)) $errors[] = "Alamat email wajib diisi.";
    if (empty($password)) $errors[] = "Kata sandi wajib diisi.";

    if (empty($errors)) {
        $stmt = $conn->prepare("SELECT id, username, password, profile_image FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($user = $result->fetch_assoc()) {
            // Hanya verifikasi password jika password di DB tidak NULL (bukan akun Google only)
            if ($user['password'] !== null && password_verify($password, $user['password'])) {
                $_SESSION['user_id'] = $user['id'];
                $_SESSION['username'] = $user['username'];
                $_SESSION['profile_image'] = $user['profile_image'];
                
                header("Location: dashboard.php");
                exit();
            } else {
                $errors[] = "Email atau password salah.";
            }
        } else {
            $errors[] = "Email atau password salah.";
        }
        $stmt->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>
<title>Masuk - List In</title>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Selamat Datang Kembali!</h2>
        <p class="subtitle">Masuk untuk melanjutkan dan mengatur tugas Anda.</p>

        <?php if (isset($_SESSION['success_message'])): ?>
            <div class="auth-message success">
                <p><?php echo $_SESSION['success_message']; ?></p>
            </div>
            <?php unset($_SESSION['success_message']); ?>
        <?php endif; ?>

        <?php if (isset($_SESSION['reset_success_message'])): ?>
            <div class="auth-message success">
                <p><?php echo $_SESSION['reset_success_message']; ?></p>
            </div>
            <?php unset($_SESSION['reset_success_message']); ?>
        <?php endif; ?>

        <?php if (!empty($errors)): ?>
            <div class="auth-message error">
                <?php foreach ($errors as $error): ?>
                    <p><?php echo htmlspecialchars($error); ?></p>
                <?php endforeach; ?>
            </div>
        <?php endif; ?>

        <form id="loginForm" method="POST" action="login.php">
            <div class="form-group">
                <label for="email">Alamat Email</label>
                <input type="email" id="email" name="email" required placeholder="cth: pengguna@email.com" value="<?php echo isset($_POST['email']) ? htmlspecialchars($_POST['email']) : ''; ?>">
            </div>
            <div class="form-group">
                <label for="password">Kata Sandi</label>
                <input type="password" id="password" name="password" required placeholder="Masukkan kata sandi Anda">
            </div>
            <div class="form-group forgot-password-link">
                <a href="forgot_password.php">Lupa Kata Sandi?</a>
            </div>
            <button type="submit" name="login_submit" class="btn-submit">Masuk Akun</button>
        </form>
        
        <div class="social-login-divider">
            <span>ATAU</span>
        </div>
        
        <a href="<?php echo htmlspecialchars($google_login_url); ?>" class="btn-social-login google">
            <i class="fab fa-google"></i> Lanjutkan dengan Google
        </a>

        <p class="auth-link">Belum punya akun? <a href="register.php">Daftar sekarang</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

**4.3. Modifikasi `register.php`**

Tambahkan tombol daftar dengan Google.

**register.php (Dimodifikasi)**
```php
<?php
require_once 'config.php'; // Untuk GOOGLE_CLIENT_ID, dll. dan session_start()
require_once 'includes/db.php';

// Setup Google Client untuk mendapatkan URL otorisasi
// Pastikan Anda telah menginstal Google API Client Library
// Jika via Composer:
require_once __DIR__ . '/vendor/autoload.php';
// Jika manual, sesuaikan path
// require_once GOOGLE_API_CLIENT_LIBRARY_PATH;


$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI); // Callback akan menangani pendaftaran
$client->addScope("email");
$client->addScope("profile");
$google_register_url = $client->createAuthUrl();


$errors = [];

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['register_submit'])) { // Tambahkan name ke tombol submit
    $username = trim($_POST['username']);
    $email = trim($_POST['email']);
    $password = $_POST['password'];
    $confirmPassword = $_POST['confirmPassword'];

    if (empty($username)) $errors[] = "Nama pengguna wajib diisi.";
    if (empty($email)) $errors[] = "Alamat email wajib diisi.";
    elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) $errors[] = "Format email tidak valid.";
    if (empty($password)) $errors[] = "Kata sandi wajib diisi.";
    elseif (strlen($password) < 6) $errors[] = "Kata sandi minimal 6 karakter.";
    if ($password !== $confirmPassword) $errors[] = "Password dan konfirmasi password tidak cocok.";

    if (empty($errors)) {
        $stmt_check_email = $conn->prepare("SELECT id FROM users WHERE email = ?");
        $stmt_check_email->bind_param("s", $email);
        $stmt_check_email->execute();
        $stmt_check_email->store_result();
        if ($stmt_check_email->num_rows > 0) {
            $errors[] = "Email sudah terdaftar. Gunakan email lain atau masuk.";
        }
        $stmt_check_email->close();
    }

    if (empty($errors)) {
        $hashed_password = password_hash($password, PASSWORD_DEFAULT);
        $default_profile_image = 'images/placeholder-profile.png';

        $stmt_insert = $conn->prepare("INSERT INTO users (username, email, password, profile_image, email_verified_at) VALUES (?, ?, ?, ?, NOW())"); // Asumsi email diverifikasi saat daftar manual
        $stmt_insert->bind_param("ssss", $username, $email, $hashed_password, $default_profile_image);
        
        if ($stmt_insert->execute()) {
            $_SESSION['success_message'] = "Registrasi berhasil! Silakan masuk dengan akun Anda.";
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Registrasi gagal. Silakan coba lagi. Error: " . $stmt_insert->error;
        }
        $stmt_insert->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>
<title>Daftar Akun - List In</title>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Buat Akun Baru Anda</h2>
        <p class="subtitle">Isi form di bawah untuk memulai perjalanan produktif Anda.</p>

        <?php if (!empty($errors)): ?>
            <div class="auth-message error">
                <?php foreach ($errors as $error): ?>
                    <p><?php echo htmlspecialchars($error); ?></p>
                <?php endforeach; ?>
            </div>
        <?php endif; ?>

        <form id="registerForm" method="POST" action="register.php">
            <div class="form-group">
                <label for="username">Nama Pengguna</label>
                <input type="text" id="username" name="username" required placeholder="cth: Amanuel" value="<?php echo isset($_POST['username']) ? htmlspecialchars($_POST['username']) : ''; ?>">
            </div>
            <div class="form-group">
                <label for="email">Alamat Email</label>
                <input type="email" id="email" name="email" required placeholder="cth: pengguna@email.com" value="<?php echo isset($_POST['email']) ? htmlspecialchars($_POST['email']) : ''; ?>">
            </div>
            <div class="form-group">
                <label for="password">Kata Sandi</label>
                <input type="password" id="password" name="password" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmPassword">Konfirmasi Kata Sandi</label>
                <input type="password" id="confirmPassword" name="confirmPassword" minlength="6" required placeholder="Ulangi kata sandi">
            </div>
            <button type="submit" name="register_submit" class="btn-submit">Daftar Akun</button>
        </form>

        <div class="social-login-divider">
            <span>ATAU</span>
        </div>
        
        <a href="<?php echo htmlspecialchars($google_register_url); ?>" class="btn-social-login google">
            <i class="fab fa-google"></i> Lanjutkan dengan Google
        </a>

        <p class="auth-link">Sudah punya akun? <a href="login.php">Masuk di sini</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

---

**LANGKAH 5: Fitur Lupa Password**

Anda perlu library PHPMailer. Install via Composer:
`composer require phpmailer/phpmailer`

**5.1. `forgot_password.php` (Baru - di root proyek)**
```php
<?php
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

require_once 'config.php'; // Untuk APP_URL, SMTP settings, dan session_start()
require_once 'includes/db.php'; // Untuk koneksi $conn

// Jika via Composer:
require __DIR__ . '/vendor/autoload.php'; // Untuk PHPMailer

$errors = [];
$success_message = '';

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $email = trim($_POST['email']);

    if (empty($email) || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Masukkan alamat email yang valid.";
    } else {
        $stmt = $conn->prepare("SELECT id FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($result->num_rows > 0) {
            // Email ditemukan, buat token
            $token = bin2hex(random_bytes(50)); // Token acak
            
            // Hapus token lama untuk email ini jika ada
            $stmt_delete_old = $conn->prepare("DELETE FROM password_resets WHERE email = ?");
            $stmt_delete_old->bind_param("s", $email);
            $stmt_delete_old->execute();
            $stmt_delete_old->close();

            // Simpan token baru ke database
            $stmt_insert_token = $conn->prepare("INSERT INTO password_resets (email, token) VALUES (?, ?)");
            $stmt_insert_token->bind_param("ss", $email, $token);
            
            if ($stmt_insert_token->execute()) {
                // Kirim email reset password
                $mail = new PHPMailer(true);
                try {
                    //Server settings
                    $mail->isSMTP();
                    $mail->Host       = SMTP_HOST;
                    $mail->SMTPAuth   = true;
                    $mail->Username   = SMTP_USERNAME;
                    $mail->Password   = SMTP_PASSWORD;
                    $mail->SMTPSecure = SMTP_SECURE; // PHPMailer::ENCRYPTION_SMTPS or PHPMailer::ENCRYPTION_STARTTLS
                    $mail->Port       = SMTP_PORT;

                    //Recipients
                    $mail->setFrom(EMAIL_FROM_ADDRESS, EMAIL_FROM_NAME);
                    $mail->addAddress($email);

                    //Content
                    $reset_link = APP_URL . "/reset_password.php?token=" . $token;
                    $mail->isHTML(true);
                    $mail->Subject = 'Reset Kata Sandi Akun List In Anda';
                    $mail->Body    = "Halo,<br><br>Kami menerima permintaan untuk mereset kata sandi akun Anda di List In.<br>"
                                   . "Silakan klik tautan di bawah ini untuk mengatur ulang kata sandi Anda:<br>"
                                   . "<a href='" . $reset_link . "'>" . $reset_link . "</a><br><br>"
                                   . "Jika Anda tidak meminta reset kata sandi, abaikan email ini.<br><br>"
                                   . "Salam,<br>Tim List In";
                    $mail->AltBody = "Halo,\n\nKami menerima permintaan untuk mereset kata sandi akun Anda di List In.\n"
                                   . "Silakan salin dan tempel tautan berikut di browser Anda untuk mengatur ulang kata sandi Anda:\n"
                                   . $reset_link . "\n\n"
                                   . "Jika Anda tidak meminta reset kata sandi, abaikan email ini.\n\n"
                                   . "Salam,\nTim List In";

                    $mail->send();
                    $success_message = 'Email instruksi reset kata sandi telah dikirim ke alamat email Anda. Silakan periksa kotak masuk (dan folder spam).';
                } catch (Exception $e) {
                    $errors[] = "Gagal mengirim email. Silakan coba lagi nanti. Mailer Error: {$mail->ErrorInfo}";
                    // error_log("Mailer Error: {$mail->ErrorInfo}");
                }
            } else {
                $errors[] = "Gagal menyimpan token reset. Silakan coba lagi.";
                // error_log("Token save error: " . $stmt_insert_token->error);
            }
            $stmt_insert_token->close();
        } else {
            // Email tidak ditemukan, tapi jangan beritahu secara eksplisit untuk keamanan
            $success_message = 'Jika alamat email Anda terdaftar, kami telah mengirimkan instruksi reset kata sandi. Silakan periksa kotak masuk (dan folder spam).';
        }
        $stmt->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>
<title>Lupa Kata Sandi - List In</title>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Lupa Kata Sandi?</h2>
    <p class="subtitle">Masukkan alamat email Anda di bawah ini. Kami akan mengirimkan tautan untuk mereset kata sandi Anda.</p>

    <?php if (!empty($success_message)): ?>
        <div class="auth-message success">
            <p><?php echo htmlspecialchars($success_message); ?></p>
        </div>
    <?php endif; ?>

    <?php if (!empty($errors)): ?>
        <div class="auth-message error">
            <?php foreach ($errors as $error): ?>
                <p><?php echo htmlspecialchars($error); ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <?php if (empty($success_message)): // Hanya tampilkan form jika belum ada pesan sukses ?>
    <form id="forgotPasswordForm" method="POST" action="forgot_password.php">
        <div class="form-group">
            <label for="email">Alamat Email</label>
            <input type="email" id="email" name="email" required placeholder="Masukkan email terdaftar Anda">
        </div>
        <button type="submit" class="btn-submit">Kirim Tautan Reset</button>
    </form>
    <?php endif; ?>

    <p class="auth-link">Ingat kata sandi Anda? <a href="login.php">Masuk di sini</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**5.2. `reset_password.php` (Baru - di root proyek)**
```php
<?php
require_once 'config.php'; // Untuk session_start()
require_once 'includes/db.php'; // Untuk koneksi $conn

$errors = [];
$token = $_GET['token'] ?? '';
$email_from_token = null;
$token_valid = false;

if (empty($token)) {
    $errors[] = "Token reset tidak valid atau tidak ditemukan.";
} else {
    $stmt_check_token = $conn->prepare("SELECT email, created_at FROM password_resets WHERE token = ?");
    $stmt_check_token->bind_param("s", $token);
    $stmt_check_token->execute();
    $result_token = $stmt_check_token->get_result();

    if ($reset_data = $result_token->fetch_assoc()) {
        $email_from_token = $reset_data['email'];
        $token_created_at = new DateTime($reset_data['created_at']);
        $now = new DateTime();
        $interval = $now->diff($token_created_at);
        $minutes_passed = ($interval->days * 24 * 60) + ($interval->h * 60) + $interval->i;

        if ($minutes_passed > 60) { // Token valid selama 1 jam
            $errors[] = "Token reset telah kedaluwarsa. Silakan minta tautan reset baru.";
            // Hapus token kedaluwarsa
            $stmt_delete_expired = $conn->prepare("DELETE FROM password_resets WHERE token = ?");
            $stmt_delete_expired->bind_param("s", $token);
            $stmt_delete_expired->execute();
            $stmt_delete_expired->close();
        } else {
            $token_valid = true;
        }
    } else {
        $errors[] = "Token reset tidak valid atau sudah digunakan.";
    }
    $stmt_check_token->close();
}

if ($_SERVER["REQUEST_METHOD"] == "POST" && $token_valid) {
    $new_password = $_POST['newPassword'];
    $confirm_new_password = $_POST['confirmNewPassword'];

    if (empty($new_password) || empty($confirm_new_password)) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak boleh kosong.";
    } elseif (strlen($new_password) < 6) {
        $errors[] = "Kata sandi baru minimal 6 karakter.";
    } elseif ($new_password !== $confirm_new_password) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak cocok.";
    } else {
        // Update password pengguna
        $hashed_password = password_hash($new_password, PASSWORD_DEFAULT);
        $stmt_update_pass = $conn->prepare("UPDATE users SET password = ? WHERE email = ?");
        $stmt_update_pass->bind_param("ss", $hashed_password, $email_from_token);
        
        if ($stmt_update_pass->execute()) {
            // Hapus token setelah berhasil digunakan
            $stmt_delete_used = $conn->prepare("DELETE FROM password_resets WHERE email = ?");
            $stmt_delete_used->bind_param("s", $email_from_token);
            $stmt_delete_used->execute();
            $stmt_delete_used->close();

            $_SESSION['reset_success_message'] = "Kata sandi Anda telah berhasil direset. Silakan masuk dengan kata sandi baru.";
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Gagal mereset kata sandi. Silakan coba lagi.";
            // error_log("Password update error: " . $stmt_update_pass->error);
        }
        $stmt_update_pass->close();
    }
}

?>
<?php require_once 'includes/header.php'; ?>
<title>Reset Kata Sandi - List In</title>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Reset Kata Sandi Anda</h2>

    <?php if (!empty($errors)): ?>
        <div class="auth-message error">
            <?php foreach ($errors as $error): ?>
                <p><?php echo htmlspecialchars($error); ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <?php if ($token_valid && empty($_SESSION['reset_success_message'])): ?>
        <p class="subtitle">Masukkan kata sandi baru Anda untuk akun terkait dengan <strong><?php echo htmlspecialchars($email_from_token); ?></strong>.</p>
        <form id="resetPasswordForm" method="POST" action="reset_password.php?token=<?php echo htmlspecialchars($token); ?>">
            <div class="form-group">
                <label for="newPassword">Kata Sandi Baru</label>
                <input type="password" id="newPassword" name="newPassword" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmNewPassword">Konfirmasi Kata Sandi Baru</label>
                <input type="password" id="confirmNewPassword" name="confirmNewPassword" minlength="6" required placeholder="Ulangi kata sandi baru">
            </div>
            <button type="submit" class="btn-submit">Reset Kata Sandi</button>
        </form>
    <?php elseif(!empty($_SESSION['reset_success_message'])): ?>
         <div class="auth-message success">
            <p><?php echo htmlspecialchars($_SESSION['reset_success_message']); ?></p>
            <?php unset($_SESSION['reset_success_message']); ?>
        </div>
        <p class="auth-link"><a href="login.php">Kembali ke Halaman Masuk</a></p>
    <?php else: ?>
        <p class="auth-link"><a href="forgot_password.php">Minta tautan reset baru</a> atau <a href="login.php">kembali ke Halaman Masuk</a>.</p>
    <?php endif; ?>
</div>

<?php require_once 'includes/footer.php'; ?>
```

---

**LANGKAH 6: Halaman Laporan (`laporan.php`)**

**6.1. `laporan.php` (Baru - di root proyek)**
```php
<?php
require_once 'includes/header.php';
require_once 'includes/sidebar.php';
require_once 'includes/task_helper.php'; // Untuk render_task_card

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];
$current_page_for_redirect = basename($_SERVER['SCRIPT_NAME']);

// --- DATA UNTUK DIAGRAM BATANG (Performa Pengerjaan - sama seperti dashboard) ---
$today_for_default_perf = new DateTimeImmutable();
$performance_start_date_val = $today_for_default_perf->modify('-6 days')->format('Y-m-d');
$performance_end_date_val = $today_for_default_perf->format('Y-m-d');
$active_preset_perf_val = 'last7days'; // Default

// Logika filter tanggal untuk diagram batang (mirip dashboard)
if (isset($_GET['filterReportChartSubmit'])) {
    if (isset($_GET['filterReportDateRange']) && !empty(trim($_GET['filterReportDateRange']))) {
        $range_perf = explode(' - ', $_GET['filterReportDateRange']);
        if (count($range_perf) >= 1) {
            $date_start_parts_perf = explode('/', trim($range_perf[0]));
            if (count($date_start_parts_perf) == 3 && checkdate((int)$date_start_parts_perf[1], (int)$date_start_parts_perf[0], (int)$date_start_parts_perf[2])) {
                $performance_start_date_val = $date_start_parts_perf[2] . '-' . $date_start_parts_perf[1] . '-' . $date_start_parts_perf[0];
            }
            if (count($range_perf) == 2) {
                $date_end_parts_perf = explode('/', trim($range_perf[1]));
                 if (count($date_end_parts_perf) == 3 && checkdate((int)$date_end_parts_perf[1], (int)$date_end_parts_perf[0], (int)$date_end_parts_perf[2])) {
                    $performance_end_date_val = $date_end_parts_perf[2] . '-' . $date_end_parts_perf[1] . '-' . $date_end_parts_perf[0];
                }
            } else { 
                if (DateTime::createFromFormat('Y-m-d', $performance_start_date_val)) { 
                    $performance_end_date_val = $performance_start_date_val;
                }
            }
        }
         $active_preset_perf_val = ''; // Kosongkan preset jika rentang kustom dipilih
    }
} elseif (isset($_GET['range_type_report_chart'])) { // Menggunakan nama unik untuk preset di laporan
    $active_preset_perf_val = $_GET['range_type_report_chart'];
    $today_for_preset = new DateTimeImmutable();
    switch ($active_preset_perf_val) {
        case 'today': $performance_start_date_val = $today_for_preset->format('Y-m-d'); $performance_end_date_val = $today_for_preset->format('Y-m-d'); break;
        case 'last7days': $performance_end_date_val = $today_for_preset->format('Y-m-d'); $performance_start_date_val = $today_for_preset->modify('-6 days')->format('Y-m-d'); break;
        case 'this_month': $performance_start_date_val = $today_for_preset->format('Y-m-01'); $performance_end_date_val = $today_for_preset->format('Y-m-t'); break;
    }
}

$bar_chart_labels_php = []; $bar_chart_data_completed_php = [];
if (isset($conn) && $conn instanceof mysqli) {
    try {
        $current_date_loop_obj = DateTime::createFromFormat('Y-m-d', $performance_start_date_val);
        $end_date_loop_obj = DateTime::createFromFormat('Y-m-d', $performance_end_date_val);
        if ($current_date_loop_obj && $end_date_loop_obj) {
            if ($current_date_loop_obj > $end_date_loop_obj) { list($current_date_loop_obj, $end_date_loop_obj) = [$end_date_loop_obj, $current_date_loop_obj]; }
            $loop_count = 0; $interval_one_day = new DateInterval('P1D');
            while ($current_date_loop_obj <= $end_date_loop_obj) {
                $date_str_loop = $current_date_loop_obj->format('Y-m-d');
                $bar_chart_labels_php[] = $current_date_loop_obj->format('d M'); // Format label
                $stmt_completed_on_date = $conn->prepare("SELECT COUNT(*) as count FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) = ?");
                $tasks_completed_on_day = 0;
                if($stmt_completed_on_date) {
                    $stmt_completed_on_date->bind_param("is", $user_id, $date_str_loop);
                    if($stmt_completed_on_date->execute()){ 
                        $result_completed_on_date = $stmt_completed_on_date->get_result();
                        if($result_completed_on_date && $result_completed_on_date->num_rows > 0) {
                            $row_completed = $result_completed_on_date->fetch_assoc();
                            $tasks_completed_on_day = isset($row_completed['count']) ? (int)$row_completed['count'] : 0;
                        }
                    }
                    $stmt_completed_on_date->close();
                }
                $bar_chart_data_completed_php[] = $tasks_completed_on_day;
                $current_date_loop_obj->add($interval_one_day); $loop_count++;
                if ($loop_count > 90 ) { if ($current_date_loop_obj <= $end_date_loop_obj) { $bar_chart_labels_php[] = "..."; $bar_chart_data_completed_php[] = null; } break; }
            }
        }
    } catch (Exception $e) { error_log("Laporan (Chart): Exception: " . $e->getMessage()); }
}
if (empty($bar_chart_labels_php)) $bar_chart_labels_php = ['Tidak Ada Data'];
if (empty($bar_chart_data_completed_php)) $bar_chart_data_completed_php = [0];

$report_bar_chart_data_php_final = [
    'labels' => $bar_chart_labels_php,
    'datasets' => [[
        'label' => 'Tugas Selesai','data' => $bar_chart_data_completed_php,
        'backgroundColor' => 'rgba(126, 71, 184, 0.6)', // Warna utama
        'borderColor' => 'rgba(126, 71, 184, 1)',
        'borderWidth' => 1,
        'borderRadius' => 8, // Untuk sudut yang halus
        'borderSkipped' => false,
    ]]
];

?>
<title>Laporan Tugas - List In</title>

<main class="main">
    <h2 class="page-title">Laporan Produktivitas</h2>

    <section class="widget report-chart-widget">
        <h3>Performa Pengerjaan Tugas</h3>
        <form method="GET" action="laporan.php" class="performance-filter-form report-performance-filter">
            <label>Rentang Waktu Diagram:</label>
            <div class="filter-preset-buttons">
                <button type="submit" name="range_type_report_chart" value="today" class="btn btn-sm <?php echo $active_preset_perf_val == 'today' ? 'active' : ''; ?>">Hari Ini</button>
                <button type="submit" name="range_type_report_chart" value="last7days" class="btn btn-sm <?php echo $active_preset_perf_val == 'last7days' ? 'active' : ''; ?>">7 Hari</button>
                <button type="submit" name="range_type_report_chart" value="this_month" class="btn btn-sm <?php echo $active_preset_perf_val == 'this_month' ? 'active' : ''; ?>">Bulan Ini</button>
            </div>
            <input type="text" id="filterReportDateRange" name="filterReportDateRange" placeholder="Kustom..." value="<?php echo htmlspecialchars($_GET['filterReportDateRange'] ?? ''); ?>" style="width:160px;">
            <div class="filter-action-buttons">
                <button type="submit" name="filterReportChartSubmit" value="1" class="btn btn-primary btn-apply-perf btn-sm">Lihat Diagram</button>
            </div>
        </form>
        <div class="widget-content-area chart-container" style="height: 250px; margin-top: 15px;">
             <?php
                $has_valid_bar_chart_data = false;
                if (isset($report_bar_chart_data_php_final['datasets'][0]['data']) && is_array($report_bar_chart_data_php_final['datasets'][0]['data'])) {
                    $filtered_data_bar = array_filter($report_bar_chart_data_php_final['datasets'][0]['data'], function($x) { return $x !== null && $x >=0; });
                    $has_valid_bar_chart_data = !empty($filtered_data_bar) && count($filtered_data_bar) > 0;
                }
                $has_valid_labels_bar = !empty($report_bar_chart_data_php_final['labels']) && !in_array("Error", $report_bar_chart_data_php_final['labels']) && !in_array("Tidak Ada Data", $report_bar_chart_data_php_final['labels']);

                if ($has_valid_bar_chart_data && $has_valid_labels_bar):
            ?>
            <canvas id="reportPerformanceBarChart"></canvas>
            <?php else: ?>
                <p class="no-tasks-message" style="text-align:center; padding-top:20px;">Tidak ada data tugas selesai untuk ditampilkan pada rentang waktu ini.</p>
            <?php endif; ?>
        </div>
    </section>

    <section class="report-tasks-by-date-section widget">
        <h3>Tugas Aktif Berdasarkan Tanggal</h3>
        <div class="report-interactive-area">
            <div class="report-task-list-container">
                <p id="selectedDateText" class="selected-date-indicator">Pilih tanggal di kalender untuk melihat tugas.</p>
                <div id="reportTasksList" class="widget-content-area scrollable-list">
                    <!-- Tugas akan dimuat di sini oleh AJAX -->
                    <p class="no-tasks-message">Pilih tanggal pada kalender.</p>
                </div>
            </div>
            <div class="report-calendar-container">
                <div id="reportCalendar"></div> <!-- Kalender akan dirender di sini oleh JS -->
            </div>
        </div>
    </section>

</main>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // --- Script untuk Diagram Batang Laporan ---
    const barCtx = document.getElementById('reportPerformanceBarChart');
    if (barCtx && typeof Chart !== 'undefined') {
        const reportBarData = <?php echo json_encode($report_bar_chart_data_php_final); ?>;
        
        let hasValidBarData = false;
         if (reportBarData && reportBarData.labels && Array.isArray(reportBarData.labels) &&
            reportBarData.datasets && Array.isArray(reportBarData.datasets) && reportBarData.datasets.length > 0 &&
            reportBarData.datasets[0].data && Array.isArray(reportBarData.datasets[0].data) ) {
            let validLabelsExistBar = reportBarData.labels.some(l => l !== 'Error' && l !== 'Tidak Ada Data' && l !== '...');
            let numericDataExistsBar = reportBarData.datasets[0].data.some(d => typeof d === 'number' && d >= 0);
            hasValidBarData = validLabelsExistBar && numericDataExistsBar;
        }

        if (hasValidBarData) {
            new Chart(barCtx, {
                type: 'bar',
                data: reportBarData,
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                stepSize: 1,
                                precision: 0, // Hanya integer di sumbu Y
                                callback: function(value) {if (Number.isInteger(value)) {return value;}}
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false // Sembunyikan legenda jika hanya satu dataset
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) { label += ': '; }
                                    if (context.parsed.y !== null) { label += context.parsed.y + ' tugas'; }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
        }
    }
    const reportPresetButtons = document.querySelectorAll('.report-performance-filter .filter-preset-buttons .btn');
    const reportDateRangeInput = document.getElementById('filterReportDateRange');
    reportPresetButtons.forEach(button => {
        button.addEventListener('click', function(e) {
            if (reportDateRangeInput) reportDateRangeInput.value = ''; 
        });
    });
     if (reportDateRangeInput) {
        reportDateRangeInput.addEventListener('input', function() { // atau 'change'
            if (this.value !== '') {
                const parentButtonsContainer = this.closest('.report-performance-filter').querySelector('.filter-preset-buttons');
                if(parentButtonsContainer){
                     parentButtonsContainer.querySelectorAll('.btn').forEach(btn => btn.classList.remove('active'));
                }
            }
        });
    }


    // --- Script untuk Kalender Interaktif dan Daftar Tugas ---
    const calendarContainer = document.getElementById('reportCalendar');
    const tasksListContainer = document.getElementById('reportTasksList');
    const selectedDateText = document.getElementById('selectedDateText');
    let currentYear, currentMonth;

    function renderCalendar(year, month) {
        currentYear = year;
        currentMonth = month;
        calendarContainer.innerHTML = ''; // Clear previous calendar

        const monthNames = ["Januari", "Februari", "Maret", "April", "Mei", "Juni", "Juli", "Agustus", "September", "Oktober", "November", "Desember"];
        const dayNames = ["Min", "Sen", "Sel", "Rab", "Kam", "Jum", "Sab"];

        const header = document.createElement('div');
        header.classList.add('calendar-header-report');
        header.innerHTML = `
            <button id="prevMonthBtn">&lt;</button>
            <span>${monthNames[month]} ${year}</span>
            <button id="nextMonthBtn">&gt;</button>
        `;
        calendarContainer.appendChild(header);

        const daysGrid = document.createElement('div');
        daysGrid.classList.add('calendar-days-grid-report');
        dayNames.forEach(day => {
            const dayNameCell = document.createElement('div');
            dayNameCell.classList.add('calendar-day-name-report');
            dayNameCell.textContent = day;
            daysGrid.appendChild(dayNameCell);
        });

        const firstDayOfMonth = new Date(year, month, 1).getDay();
        const daysInMonth = new Date(year, month + 1, 0).getDate();

        for (let i = 0; i < firstDayOfMonth; i++) {
            const emptyCell = document.createElement('div');
            daysGrid.appendChild(emptyCell);
        }

        for (let day = 1; day <= daysInMonth; day++) {
            const dayCell = document.createElement('div');
            dayCell.classList.add('calendar-day-report');
            dayCell.textContent = day;
            dayCell.dataset.date = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            
            const today = new Date();
            if (year === today.getFullYear() && month === today.getMonth() && day === today.getDate()) {
                dayCell.classList.add('today');
            }

            dayCell.addEventListener('click', function() {
                document.querySelectorAll('.calendar-day-report.selected').forEach(el => el.classList.remove('selected'));
                this.classList.add('selected');
                loadTasksForDate(this.dataset.date);
            });
            daysGrid.appendChild(dayCell);
        }
        calendarContainer.appendChild(daysGrid);

        document.getElementById('prevMonthBtn').addEventListener('click', () => {
            month--;
            if (month < 0) {
                month = 11;
                year--;
            }
            renderCalendar(year, month);
        });

        document.getElementById('nextMonthBtn').addEventListener('click', () => {
            month++;
            if (month > 11) {
                month = 0;
                year++;
            }
            renderCalendar(year, month);
        });
    }

    function loadTasksForDate(dateStr) { // dateStr format YYYY-MM-DD
        const dateObj = new Date(dateStr);
        const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
        selectedDateText.textContent = `Tugas Aktif untuk: ${dateObj.toLocaleDateString('id-ID', options)}`;
        tasksListContainer.innerHTML = '<p class="loading-message">Memuat tugas...</p>';

        fetch(`ajax_get_tasks_for_date.php?date=${dateStr}`)
            .then(response => {
                if (!response.ok) { throw new Error('Network response was not ok'); }
                return response.json();
            })
            .then(data => {
                tasksListContainer.innerHTML = ''; // Clear loading
                if (data.success && data.tasks.length > 0) {
                    data.tasks.forEach(taskHtml => {
                        // Karena taskHtml adalah string HTML dari render_task_card, kita bisa langsung append
                        tasksListContainer.insertAdjacentHTML('beforeend', taskHtml);
                    });
                } else if (data.success && data.tasks.length === 0) {
                    tasksListContainer.innerHTML = '<p class="no-tasks-message">Tidak ada tugas aktif untuk tanggal ini.</p>';
                } else {
                    tasksListContainer.innerHTML = `<p class="no-tasks-message">Gagal memuat tugas: ${data.message || 'Error tidak diketahui'}</p>`;
                }
            })
            .catch(error => {
                console.error('Error fetching tasks:', error);
                tasksListContainer.innerHTML = '<p class="no-tasks-message">Terjadi kesalahan saat memuat tugas.</p>';
            });
    }

    const today = new Date();
    renderCalendar(today.getFullYear(), today.getMonth());

    // Inisialisasi Flatpickr untuk filter diagram
    const reportDateRangePicker = document.getElementById('filterReportDateRange');
    if(reportDateRangePicker) {
        flatpickr(reportDateRangePicker, {
            mode: "range",
            dateFormat: "d/m/Y",
            locale: "id",
            allowInput: true
        });
    }
});
</script>
<?php require_once 'includes/footer.php'; ?>
```

**6.2. `ajax_get_tasks_for_date.php` (Baru - di root proyek)**
```php
<?php
// File: ajax_get_tasks_for_date.php
require_once 'includes/db.php'; // Untuk $conn dan session_start()
require_once 'includes/task_helper.php'; // Untuk render_task_card

header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'User not authenticated.', 'tasks' => []]);
    exit();
}
$user_id = $_SESSION['user_id'];
$selected_date_str = $_GET['date'] ?? null; // Format YYYY-MM-DD

if (!$selected_date_str) {
    echo json_encode(['success' => false, 'message' => 'Tanggal tidak disediakan.', 'tasks' => []]);
    exit();
}

// Validasi format tanggal YYYY-MM-DD
$date_parts = explode('-', $selected_date_str);
if (count($date_parts) !== 3 || !checkdate((int)$date_parts[1], (int)$date_parts[2], (int)$date_parts[0])) {
    echo json_encode(['success' => false, 'message' => 'Format tanggal tidak valid.', 'tasks' => []]);
    exit();
}

$tasks_html_array = [];
$sql = "SELECT id, title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted, due_date
        FROM tasks
        WHERE user_id = ? AND status != 'Completed' AND due_date = ?
        ORDER BY CASE priority WHEN 'High' THEN 1 WHEN 'Medium' THEN 2 WHEN 'Low' THEN 3 ELSE 4 END, created_at ASC";

$stmt = $conn->prepare($sql);
if ($stmt) {
    $stmt->bind_param("is", $user_id, $selected_date_str);
    $stmt->execute();
    $result = $stmt->get_result();
    
    $current_page_for_redirect = 'laporan.php'; // Atau halaman yang relevan jika ada aksi

    while ($task = $result->fetch_assoc()) {
        // Gunakan render_task_card. Untuk halaman laporan, mungkin tidak ada aksi,
        // jadi page_type bisa 'dashboardTodo' atau custom type jika perlu styling berbeda.
        // Jika ingin aksi (edit/delete) dari laporan, pastikan $current_page_for_redirect sesuai.
        $tasks_html_array[] = render_task_card($task, 'dashboardTodo', $current_page_for_redirect . '?selected_date=' . urlencode($selected_date_str));
    }
    $stmt->close();
    echo json_encode(['success' => true, 'tasks' => $tasks_html_array]);
} else {
    echo json_encode(['success' => false, 'message' => 'Database query error: ' . $conn->error, 'tasks' => []]);
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 7: Implementasi Chatbot (Kerangka)**

Ini adalah bagian yang paling bergantung pada API eksternal Anda. Saya akan berikan kerangka UI dan AJAX.

**7.1. `ajax_chatbot_handler.php` (Baru - di root proyek - KERANGKA)**
```php
<?php
// File: ajax_chatbot_handler.php (KERANGKA)
require_once 'config.php'; // Untuk CHATBOT_API_KEY, session_start()
require_once 'includes/db.php'; // Untuk $conn
require_once 'includes/header.php'; // Untuk add_notification (jika chatbot memberi notif internal)


header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'reply' => 'Autentikasi pengguna gagal.']);
    exit();
}
$user_id = $_SESSION['user_id'];
$user_message = trim($_POST['message'] ?? '');

if (empty($user_message)) {
    echo json_encode(['success' => false, 'reply' => 'Pesan tidak boleh kosong.']);
    exit();
}

$bot_reply = "Maaf, saya belum bisa memproses permintaan itu saat ini."; // Default reply

// ---------------------------------------------------------------------------
// BAGIAN INI ADALAH LOGIKA INTI CHATBOT ANDA
// Anda perlu:
// 1. Mengirim $user_message ke API Chatbot Anda (menggunakan CHATBOT_API_KEY).
// 2. Menerima respons dari API Chatbot.
// 3. Menganalisis respons untuk menentukan "intent" atau perintah.
// 4. Jika perintahnya adalah aksi internal (misal, "hapus tugas X", "bersihkan riwayat"),
//    lakukan aksi tersebut di database.
// 5. Format balasan yang sesuai.
// ---------------------------------------------------------------------------

// Contoh Placeholder untuk interaksi dengan API Chatbot eksternal:
/*
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, CHATBOT_API_ENDPOINT);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(['message' => $user_message, 'user_id_internal' => $user_id])); // Kirim ID user internal jika API Anda mendukungnya
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Content-Type: application/json',
    'Authorization: Bearer ' . CHATBOT_API_KEY
]);
$api_response_str = curl_exec($ch);
if (curl_errno($ch)) {
    // error_log('Chatbot API Curl error: ' . curl_error($ch));
    echo json_encode(['success' => false, 'reply' => 'Error menghubungi layanan chatbot.']);
    exit();
}
curl_close($ch);

$api_response = json_decode($api_response_str, true);
if ($api_response && isset($api_response['reply'])) {
    $bot_reply = $api_response['reply']; // Ambil balasan dari API

    // --- Analisis Intent Sederhana (CONTOH) ---
    // Ini sangat bergantung pada bagaimana API Anda mengembalikan intent
    $intent = strtolower($api_response['intent'] ?? ''); // Misal API Anda mengembalikan intent

    if (strpos(strtolower($user_message), "tampilkan tugas") !== false || $intent === 'show_tasks') {
        // Logika untuk mengambil dan menampilkan tugas
        $stmt_tasks = $conn->prepare("SELECT title, status, DATE_FORMAT(due_date, '%d %b %Y') as due_date_formatted FROM tasks WHERE user_id = ? AND status != 'Completed' ORDER BY due_date ASC LIMIT 5");
        if ($stmt_tasks) {
            $stmt_tasks->bind_param("i", $user_id);
            $stmt_tasks->execute();
            $result_tasks = $stmt_tasks->get_result();
            $task_list_reply = "Berikut tugas aktif Anda:\n";
            if ($result_tasks->num_rows > 0) {
                while ($task = $result_tasks->fetch_assoc()) {
                    $task_list_reply .= "- " . htmlspecialchars($task['title']) . " (Status: " . htmlspecialchars($task['status']) . ", Deadline: " . ($task['due_date_formatted'] ?: 'N/A') . ")\n";
                }
            } else {
                $task_list_reply = "Anda tidak memiliki tugas aktif saat ini.";
            }
            $bot_reply = $task_list_reply;
            $stmt_tasks->close();
        }
    } elseif (strpos(strtolower($user_message), "hapus notifikasi") !== false || $intent === 'clear_notifications') {
        if (isset($_SESSION['notification_messages'])) unset($_SESSION['notification_messages']);
        $_SESSION['has_unread_notifications_badge'] = false;
        $bot_reply = "Semua notifikasi telah dihapus.";
        // add_notification("Notifikasi dibersihkan via chatbot.", "info"); // Notif internal
    } elseif (preg_match('/hapus tugas "(.*?)"/i', $user_message, $matches) || ($intent === 'delete_task' && isset($api_response['task_title']))) {
        $task_title_to_delete = $matches[1] ?? ($api_response['task_title'] ?? '');
        if (!empty($task_title_to_delete)) {
            $stmt_delete = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND title = ? AND status != 'Completed'");
            if ($stmt_delete) {
                $stmt_delete->bind_param("is", $user_id, $task_title_to_delete);
                $stmt_delete->execute();
                if ($stmt_delete->affected_rows > 0) {
                    $bot_reply = "Tugas \"" . htmlspecialchars($task_title_to_delete) . "\" berhasil dihapus.";
                } else {
                    $bot_reply = "Tugas \"" . htmlspecialchars($task_title_to_delete) . "\" tidak ditemukan atau sudah selesai.";
                }
                $stmt_delete->close();
            }
        } else {
             $bot_reply = "Tolong sebutkan nama tugas yang ingin dihapus, contoh: hapus tugas \"Nama Tugas Ini\".";
        }
    } elseif (strpos(strtolower($user_message), "bersihkan riwayat") !== false || $intent === 'clear_history') {
        $stmt_clear_hist = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND (status = 'Completed' OR (due_date < CURDATE() AND status != 'Completed'))");
        if($stmt_clear_hist){
            $stmt_clear_hist->bind_param("i", $user_id);
            $stmt_clear_hist->execute();
            $bot_reply = "Semua riwayat tugas (selesai dan terlewat) telah dibersihkan.";
            $stmt_clear_hist->close();
        }
    } elseif (strpos(strtolower($user_message), "bersihkan semua tugas aktif") !== false || $intent === 'clear_active_tasks') {
        $stmt_clear_active = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND status != 'Completed'");
        if($stmt_clear_active){
            $stmt_clear_active->bind_param("i", $user_id);
            $stmt_clear_active->execute();
            $bot_reply = "Semua tugas aktif (yang belum selesai) telah dibersihkan.";
            $stmt_clear_active->close();
        }
    }
    // Tambahkan lebih banyak kondisi untuk perintah lain
} else {
    // error_log("Chatbot API response error or invalid format: " . $api_response_str);
    // $bot_reply tetap default
}
*/
// AKHIR BAGIAN LOGIKA INTI CHATBOT ANDA

echo json_encode(['success' => true, 'reply' => nl2br(htmlspecialchars($bot_reply))]);

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 8: Modifikasi File PHP yang Ada (`includes/header.php`, `includes/sidebar.php`, `includes/footer.php`)**

**8.1. `includes/header.php` (Dimodifikasi)**
```php
<?php
// File: includes/header.php
require_once dirname(__DIR__) . '/config.php'; // Memuat config.php dari root (sudah ada session_start)
require_once __DIR__ . '/db.php'; // Koneksi DB

$current_page = basename($_SERVER['SCRIPT_NAME']);
$current_user_id = $_SESSION['user_id'] ?? null;
$current_user_profile_image = 'images/placeholder-profile.png'; // Default

if ($current_user_id && isset($conn)) {
    $stmt_user_header = $conn->prepare("SELECT username, profile_image FROM users WHERE id = ?");
    if ($stmt_user_header) {
        $stmt_user_header->bind_param("i", $current_user_id);
        $stmt_user_header->execute();
        $result_user_header = $stmt_user_header->get_result();
        if ($user_data_header = $result_user_header->fetch_assoc()) {
            // Pastikan nama pengguna di session selalu update dari DB saat header load
            $_SESSION['username'] = $user_data_header['username']; 
            
            $current_user_profile_image_db = $user_data_header['profile_image'];
             // Cek jika path gambar dari DB valid dan file ada, jika tidak, gunakan placeholder
            if (!empty($current_user_profile_image_db) && file_exists(dirname(__DIR__) . '/' . $current_user_profile_image_db)) {
                $current_user_profile_image = $current_user_profile_image_db;
            } elseif (!empty($current_user_profile_image_db) && filter_var($current_user_profile_image_db, FILTER_VALIDATE_URL)) {
                // Jika itu URL (misal dari Google), gunakan langsung
                $current_user_profile_image = $current_user_profile_image_db;
            }
            // Update session profile image jika berbeda dari yang di DB (misal setelah login Google)
            if (($_SESSION['profile_image'] ?? '') !== $current_user_profile_image) {
                 $_SESSION['profile_image'] = $current_user_profile_image;
            }

        }
        $stmt_user_header->close();
    }
}


// Fungsi add_notification, pengecekan deadline, dan overdue tetap sama seperti sebelumnya
// ... (Salin fungsi add_notification, cek deadline H-1, cek overdue dari kode Anda sebelumnya) ...
function add_notification($message, $type = 'info') {
    if (!isset($_SESSION['notification_messages'])) {
        $_SESSION['notification_messages'] = [];
    }
    $new_notification = ['message' => $message, 'type' => $type, 'time' => time()];
    
    $is_duplicate_notif = false;
    if ($type === 'deadline_soon' || $type === 'overdue_task') { 
        $message_core_part = '';
        if (preg_match('/Tugas(?:-tugas)?\s*(".*?")\s*(akan jatuh tempo besok|telah melewati batas waktu)/', $new_notification['message'], $matches)) {
            $message_core_part = $matches[1]; 
        } elseif (preg_match('/(".*?")/', $new_notification['message'], $matches_generic)) {
            $message_core_part = $matches_generic[1];
        }

        foreach ($_SESSION['notification_messages'] as $existing_notif) {
            if (isset($existing_notif['message']) &&
                ($message_core_part && strpos($existing_notif['message'], $message_core_part) !== false) && 
                $existing_notif['type'] === $type &&
                (time() - ($existing_notif['time'] ?? 0)) < 3600 * 3) { 
                $is_duplicate_notif = true;
                break;
            }
        }
    }

    if (!$is_duplicate_notif) {
        $_SESSION['notification_messages'][] = $new_notification;
        if (count($_SESSION['notification_messages']) > 10) { 
            array_shift($_SESSION['notification_messages']);
        }
        $_SESSION['has_unread_notifications_badge'] = true;
    }
}

if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'])) {
    $tomorrow_date = date('Y-m-d', strtotime('+1 day'));
    $today_for_notif_check_h1 = date('Y-m-d');
    $notif_key_deadline_h1 = 'deadline_h1_notif_sent_' . $today_for_notif_check_h1 . '_uid' . $current_user_id;

    if (!isset($_SESSION[$notif_key_deadline_h1])) {
        $stmt_deadline_check_h1 = $conn->prepare("SELECT title FROM tasks WHERE user_id = ? AND due_date = ? AND status != 'Completed'");
        if ($stmt_deadline_check_h1) {
            $stmt_deadline_check_h1->bind_param("is", $current_user_id, $tomorrow_date);
            $stmt_deadline_check_h1->execute();
            $result_deadline_tasks_h1 = $stmt_deadline_check_h1->get_result();
            $tasks_deadline_tomorrow = [];
            while ($task_for_deadline_notif_h1 = $result_deadline_tasks_h1->fetch_assoc()) {
                $tasks_deadline_tomorrow[] = htmlspecialchars($task_for_deadline_notif_h1['title']);
            }
            $stmt_deadline_check_h1->close();

            if (!empty($tasks_deadline_tomorrow)) {
                $task_list_str_h1 = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_deadline_tomorrow));
                $message_plural_h1 = count($tasks_deadline_tomorrow) > 1 ? "Tugas-tugas" : "Tugas";
                $deadline_message_h1 = "<span class='message-content'><strong>PERHATIAN:</strong> $message_plural_h1 $task_list_str_h1 akan jatuh tempo besok!</span>";
                add_notification($deadline_message_h1, "deadline_soon");
                $_SESSION[$notif_key_deadline_h1] = true; 
            }
        } else {
            error_log("Failed to prepare statement for H-1 deadline check: " . $conn->error);
        }
    }
}

if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'])) {
    $today_for_overdue_check = date('Y-m-d');
    $notif_key_overdue = 'overdue_notif_sent_' . $today_for_overdue_check . '_uid' . $current_user_id;

    if (!isset($_SESSION[$notif_key_overdue])) {
        $stmt_overdue_check = $conn->prepare(
            "SELECT id, title FROM tasks 
             WHERE user_id = ? AND due_date < CURDATE() AND status != 'Completed' 
             AND (last_overdue_notif_sent IS NULL OR DATE(last_overdue_notif_sent) < CURDATE())"
        );
        
        if ($stmt_overdue_check) {
            $stmt_overdue_check->bind_param("i", $current_user_id);
            $stmt_overdue_check->execute();
            $result_overdue_tasks = $stmt_overdue_check->get_result();
            $tasks_overdue = [];
            $task_ids_to_update_notif_sent = [];

            while ($task_overdue = $result_overdue_tasks->fetch_assoc()) {
                $tasks_overdue[] = htmlspecialchars($task_overdue['title']);
                $task_ids_to_update_notif_sent[] = $task_overdue['id'];
            }
            $stmt_overdue_check->close();

            if (!empty($tasks_overdue)) {
                $task_list_str_overdue = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_overdue));
                $message_plural_overdue = count($tasks_overdue) > 1 ? "Tugas-tugas" : "Tugas";
                $overdue_message = "<span class='message-content'><strong>TERLEWAT:</strong> $message_plural_overdue $task_list_str_overdue telah melewati batas waktu!</span>";
                add_notification($overdue_message, "deadline_soon"); 

                if (!empty($task_ids_to_update_notif_sent)) {
                    $ids_placeholder = implode(',', array_fill(0, count($task_ids_to_update_notif_sent), '?'));
                    $stmt_update_notif_date = $conn->prepare(
                        "UPDATE tasks SET last_overdue_notif_sent = NOW() WHERE id IN ($ids_placeholder) AND user_id = ?"
                    );
                    if ($stmt_update_notif_date) {
                        $types_update = str_repeat('i', count($task_ids_to_update_notif_sent)) . 'i';
                        $params_update = array_merge($task_ids_to_update_notif_sent, [$current_user_id]);
                        $stmt_update_notif_date->bind_param($types_update, ...$params_update);
                        if (!$stmt_update_notif_date->execute()) {
                            error_log("Failed to update last_overdue_notif_sent: " . $stmt_update_notif_date->error);
                        }
                        $stmt_update_notif_date->close();
                    }
                }
                $_SESSION[$notif_key_overdue] = true; 
            }
        } else {
            error_log("Failed to prepare statement for overdue check: " . $conn->error);
        }
    }
}

$has_unread_badge_for_icon = $_SESSION['has_unread_notifications_badge'] ?? false;

function format_tanggal_indonesia_header($date_str) {
    if (empty($date_str) || $date_str == '0000-00-00') return 'N/A';
    try { $date = new DateTime($date_str); } catch (Exception $e) { return 'Invalid Date'; }
    $hari_arr = ['Min', 'Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab'];
    $bulan_arr = [1=>'Jan',2=>'Feb',3=>'Mar',4=>'Apr',5=>'Mei',6=>'Jun',7=>'Jul',8=>'Agu',9=>'Sep',10=>'Okt',11=>'Nov',12=>'Des'];
    $hari = $hari_arr[(int)$date->format('w')];
    $tanggal = $date->format('j');
    $bulan = $bulan_arr[(int)$date->format('n')];
    return $hari . ', ' . $tanggal . ' ' . $bulan;
}
$today_display_full = format_tanggal_indonesia_header(date('Y-m-d'));
$parts_today_display = explode(', ', $today_display_full);
$day_name_display = $parts_today_display[0] ?? '';
$date_str_display = $parts_today_display[1] ?? '';

// Halaman-halaman yang dianggap sebagai bagian "auth" flow
$auth_pages = ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'];
$is_auth_page = in_array($current_page, $auth_pages);

?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
      // Skrip FOUC Prevent
      (function() {
        const theme = localStorage.getItem('theme');
        const htmlEl = document.documentElement;
        if (theme === 'dark-theme' || (!theme && window.matchMedia?.('(prefers-color-scheme: dark)').matches)) {
          htmlEl.classList.add('dark-theme-active');
        }
        <?php if ($is_auth_page): ?>
          htmlEl.classList.add('auth-html');
        <?php endif; ?>
      })();
    </script>
    <link rel="stylesheet" href="css/style.css?v=<?php echo time(); ?>">
    <?php if ($is_auth_page): ?>
        <link rel="stylesheet" href="css/auth.css?v=<?php echo time(); ?>">
    <?php endif; ?>
</head>
<body class="<?php echo $is_auth_page ? 'auth-page' : ''; ?>">

    <?php if (!$is_auth_page): ?>
    <header class="header">
        <div class="header-left">
            <a href="profil.php">
                 <img src="<?php echo htmlspecialchars($current_user_profile_image); ?>?t=<?php echo time();?>" alt="Foto Profil" class="header-profile-pic" id="headerProfilePic">
            </a>
            <h2 class="app-title-toggle" id="appTitleToggle" title="Toggle Sidebar">List In</h2>
        </div>
        <?php
        $hide_search_bar_pages = ['profil.php', 'edit_profil.php', 'ubah_password.php', 'tambah_tugas.php', 'edit_tugas.php', 'dashboard.php', 'laporan.php']; // Tambah laporan.php
        $hide_search_bar = in_array($current_page, $hide_search_bar_pages);
        $search_action_page = 'manajemen_tugas.php';
        if ($current_page == 'riwayat.php') $search_action_page = 'riwayat.php';
        ?>
        <div class="search-bar" <?php if ($hide_search_bar) echo 'style="visibility: hidden;"'; ?>>
            <form action="<?php echo $search_action_page; ?>" method="GET" style="display:flex; width:100%;">
                <input type="text" name="search_term" id="searchInputGlobal" placeholder="Cari di <?php echo ($current_page == 'riwayat.php' ? 'Riwayat' : 'Manajemen'); ?>..." value="<?php echo isset($_GET['search_term']) ? htmlspecialchars($_GET['search_term']) : ''; ?>">
                <button type="submit" style="background:none; border:none; padding:0 0 0 8px; margin-left:auto; cursor:pointer;"><i class="fas fa-search"></i></button>
            </form>
        </div>
        <div class="header-right">
             <i class="fas fa-bell <?php if ($has_unread_badge_for_icon) echo 'has-notif'; ?>" id="bellIcon" title="Notifikasi"></i>
            <i class="fas fa-calendar-alt" id="calendarIcon" title="Kalender"></i>
            <div class="date-container">
                <p><?php echo htmlspecialchars($day_name_display); ?></p>
                <span><?php echo htmlspecialchars($date_str_display); ?></span>
            </div>
        </div>
    </header>
    <div class="content">
    <?php endif; ?>
```

**8.2. `includes/sidebar.php` (Dimodifikasi)**
```php
<?php
// $current_page sudah didefinisikan di header.php
$auth_pages_sidebar = ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php'];
$is_auth_page_sidebar = in_array($current_page, $auth_pages_sidebar);
?>
<?php if (!$is_auth_page_sidebar): ?>
        <aside class="sidebar" id="sidebar">
            <nav class="menu">
                <a href="dashboard.php" class="<?php echo ($current_page == 'dashboard.php') ? 'active' : ''; ?>"><i class="fas fa-home"></i> <span>Dasbor</span></a>
                <a href="manajemen_tugas.php" class="<?php echo ($current_page == 'manajemen_tugas.php' || $current_page == 'edit_tugas.php') ? 'active' : ''; ?>"><i class="fas fa-tasks"></i> <span>Kelola Tugas</span></a>
                <a href="tambah_tugas.php" class="<?php echo ($current_page == 'tambah_tugas.php') ? 'active' : ''; ?>"><i class="fas fa-plus-circle"></i> <span>Tambah Tugas</span></a>
                <a href="laporan.php" class="<?php echo ($current_page == 'laporan.php') ? 'active' : ''; ?>"><i class="fas fa-chart-pie"></i> <span>Laporan</span></a>
                <a href="riwayat.php" class="<?php echo ($current_page == 'riwayat.php') ? 'active' : ''; ?>"><i class="fas fa-history"></i> <span>Riwayat</span></a>
            </nav>
            <a href="logout.php" class="logout" id="logoutButton"><i class="fas fa-sign-out-alt"></i> <span>Keluar</span></a>
        </aside>
<?php endif; ?>
```

**8.3. `includes/footer.php` (Dimodifikasi)**
```php
<?php
// $current_page dari header.php
// Halaman-halaman yang dianggap sebagai bagian "auth" flow atau halaman khusus
$no_standard_footer_pages = [
    'login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php',
    // Tambahkan halaman lain di sini jika tidak memerlukan notif popup, kalender popup, atau chatbot
    // 'laporan.php', // Jika laporan tidak butuh chatbot
];
$is_special_page_footer = in_array($current_page, $no_standard_footer_pages);

// Chatbot tidak ditampilkan di halaman tertentu
$no_chatbot_pages = [
    'login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php',
    'laporan.php', 'profil.php', 'edit_profil.php', 'ubah_password.php'
];
$show_chatbot = !in_array($current_page, $no_chatbot_pages);

?>
<?php if (!$is_special_page_footer): ?>
    </div> <!-- Penutup div.content dari header.php -->

    <div id="notification-popup">
        <div class="notification-header">
             <h4>Notifikasi</h4>
             <button id="clearAllNotificationsBtn" class="btn-clear-notif" title="Hapus semua notifikasi">Hapus Semua</button>
        </div>
        <ul id="notification-list">
            <?php 
            $current_session_notifications_for_popup = [];
            if (isset($_SESSION['notification_messages']) && is_array($_SESSION['notification_messages'])) {
                $current_session_notifications_for_popup = $_SESSION['notification_messages'];
                usort($current_session_notifications_for_popup, function($a, $b) {
                    return ($b['time'] ?? 0) - ($a['time'] ?? 0);
                });
            }
            $has_current_notifications_for_popup = !empty($current_session_notifications_for_popup);
            ?>
            <?php if ($has_current_notifications_for_popup): ?>
                <?php foreach ($current_session_notifications_for_popup as $notif_item): 
                    $time_ago_popup = 'Beberapa waktu lalu';
                    if (isset($notif_item['time'])) {
                        try {
                            $timestamp_popup = new DateTime('@' . $notif_item['time']);
                            $now_popup = new DateTime(); $interval_popup = $now_popup->diff($timestamp_popup);
                            if ($interval_popup->y > 0) $time_ago_popup = $interval_popup->y . " thn lalu";
                            elseif ($interval_popup->m > 0) $time_ago_popup = $interval_popup->m . " bln lalu";
                            elseif ($interval_popup->d > 0) $time_ago_popup = $interval_popup->d . " hr lalu";
                            elseif ($interval_popup->h > 0) $time_ago_popup = $interval_popup->h . " jam lalu";
                            elseif ($interval_popup->i > 0) $time_ago_popup = $interval_popup->i . " mnt lalu";
                            else $time_ago_popup = "Baru saja";
                        } catch (Exception $e) { /* Biarkan default */ }
                    }
                    $message_class_popup = '';
                    if(isset($notif_item['type'])) {
                        if($notif_item['type'] == 'success') $message_class_popup = 'notif-success';
                        else if($notif_item['type'] == 'error') $message_class_popup = 'notif-error';
                        else if($notif_item['type'] == 'info') $message_class_popup = 'notif-info';
                        else if($notif_item['type'] == 'deadline_soon') $message_class_popup = 'notif-deadline_soon';
                    }
                ?>
                    <li class="<?php echo $message_class_popup; ?>">
                        <?php echo $notif_item['message']; ?> 
                        <small class="notif-time"><?php echo $time_ago_popup; ?></small>
                    </li>
                <?php endforeach; ?>
            <?php else: ?>
                <li class="no-notifications">Tidak ada notifikasi baru.</li>
            <?php endif; ?>
        </ul>
    </div>
    <div id="calendar-popup">
        <div id="calendar-container-popup"></div>
    </div>

    <?php if ($show_chatbot): ?>
    <div id="chatbot-icon" title="Tanya ListIn Bot">
        <i class="fas fa-robot"></i>
    </div>
    <div id="chatbot-container">
        <div id="chatbot-header">
            ListIn Bot
            <button id="closeChatbotBtn">&times;</button>
        </div>
        <div id="chatbot-messages">
            <!-- Pesan chatbot akan muncul di sini -->
            <div class="chatbot-message bot">Halo! Ada yang bisa saya bantu terkait tugas Anda?</div>
        </div>
        <div id="chatbot-input-area">
            <input type="text" id="chatbotInput" placeholder="Ketik pesan Anda...">
            <button id="sendChatbotMessageBtn"><i class="fas fa-paper-plane"></i></button>
        </div>
    </div>
    <?php endif; ?>
    
    <script src="https://cdn.jsdelivr.net/npm/flatpickr"></script>
    <script src="https://npmcdn.com/flatpickr/dist/l10n/id.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const bodyElement = document.body; 
            const htmlElement = document.documentElement; 
            const bellIcon = document.getElementById('bellIcon');
            const notificationPopup = document.getElementById('notification-popup');
            const notificationListUl = document.getElementById('notification-list');
            const clearAllNotificationsBtn = document.getElementById('clearAllNotificationsBtn');

            const calendarIcon = document.getElementById('calendarIcon');
            const calendarPopup = document.getElementById('calendar-popup');
            
            // Variabel Chatbot (jika ditampilkan)
            const chatbotIcon = document.getElementById('chatbot-icon');
            const chatbotContainer = document.getElementById('chatbot-container');
            const closeChatbotBtn = document.getElementById('closeChatbotBtn');
            const chatbotMessagesDiv = document.getElementById('chatbot-messages');
            const chatbotInput = document.getElementById('chatbotInput');
            const sendChatbotMessageBtn = document.getElementById('sendChatbotMessageBtn');


            function togglePopup(popupElement, iconElement, otherPopupElement, anotherPopupElement = null) {
                if (otherPopupElement && otherPopupElement.classList.contains('show')) {
                    otherPopupElement.classList.remove('show');
                }
                if (anotherPopupElement && anotherPopupElement.classList.contains('show')) {
                    anotherPopupElement.classList.remove('show');
                }

                popupElement.classList.toggle('show');
                
                if (popupElement === notificationPopup && popupElement.classList.contains('show')) {
                    if (bellIcon && bellIcon.classList.contains('has-notif')) {
                        fetch('mark_notifications_viewed.php')
                            .then(response => response.json())
                            .then(data => {
                                if(data.success) {
                                     if(bellIcon) bellIcon.classList.remove('has-notif');
                                }
                            }).catch(error => console.error('Error marking notifications viewed:', error));
                    }
                }
            }

            if (bellIcon && notificationPopup) { 
                bellIcon.addEventListener('click', (e) => { 
                    e.stopPropagation(); 
                    togglePopup(notificationPopup, bellIcon, calendarPopup, chatbotContainer); 
                });
            }

            if (clearAllNotificationsBtn && notificationListUl) {
                clearAllNotificationsBtn.addEventListener('click', (e) => {
                    e.stopPropagation(); 
                    if (confirm('Anda yakin ingin menghapus semua notifikasi?')) {
                        fetch('ajax_clear_all_notifications.php') 
                            .then(response => response.json())
                            .then(data => {
                                if (data.success) {
                                    notificationListUl.innerHTML = '<li class="no-notifications">Tidak ada notifikasi baru.</li>';
                                    if (bellIcon && bellIcon.classList.contains('has-notif')) {
                                        bellIcon.classList.remove('has-notif'); 
                                    }
                                } else { alert('Gagal menghapus notifikasi.'); }
                            })
                            .catch(error => {
                                console.error('Error clearing all notifications:', error);
                                alert('Terjadi kesalahan saat menghapus notifikasi.');
                            });
                    }
                });
            }

            if (calendarIcon && calendarPopup) { 
                 flatpickr(document.getElementById('calendar-container-popup'), { inline: true, dateFormat: "d/m/Y", locale: "id" });
                calendarIcon.addEventListener('click', function (e) { 
                    e.stopPropagation(); 
                    togglePopup(calendarPopup, calendarIcon, notificationPopup, chatbotContainer); 
                });
            }

            // --- Chatbot Logic ---
            if (chatbotIcon && chatbotContainer) {
                chatbotIcon.addEventListener('click', (e) => {
                    e.stopPropagation();
                    // Sembunyikan popup lain jika terbuka
                    if (notificationPopup && notificationPopup.classList.contains('show')) notificationPopup.classList.remove('show');
                    if (calendarPopup && calendarPopup.classList.contains('show')) calendarPopup.classList.remove('show');
                    
                    chatbotContainer.classList.toggle('show');
                    chatbotIcon.style.display = chatbotContainer.classList.contains('show') ? 'none' : 'flex';
                    if(chatbotContainer.classList.contains('show')) chatbotInput.focus();
                });

                if (closeChatbotBtn) {
                    closeChatbotBtn.addEventListener('click', () => {
                        chatbotContainer.classList.remove('show');
                        chatbotIcon.style.display = 'flex';
                    });
                }

                function sendChatMessage() {
                    const message = chatbotInput.value.trim();
                    if (message === '') return;

                    appendMessageToChatbox(message, 'user');
                    chatbotInput.value = '';
                    chatbotInput.disabled = true;
                    sendChatbotMessageBtn.disabled = true;
                    appendMessageToChatbox('...', 'bot-typing'); // Typing indicator

                    fetch('ajax_chatbot_handler.php', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                        },
                        body: `message=${encodeURIComponent(message)}`
                    })
                    .then(response => response.json())
                    .then(data => {
                        removeTypingIndicator();
                        if (data.success) {
                            appendMessageToChatbox(data.reply, 'bot');
                        } else {
                            appendMessageToChatbox(data.reply || 'Maaf, terjadi kesalahan.', 'bot');
                        }
                    })
                    .catch(error => {
                        removeTypingIndicator();
                        console.error('Chatbot AJAX error:', error);
                        appendMessageToChatbox('Tidak dapat terhubung ke bot saat ini.', 'bot');
                    })
                    .finally(() => {
                        chatbotInput.disabled = false;
                        sendChatbotMessageBtn.disabled = false;
                        chatbotInput.focus();
                    });
                }
                
                if (sendChatbotMessageBtn) sendChatbotMessageBtn.addEventListener('click', sendChatMessage);
                if (chatbotInput) chatbotInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') sendChatMessage();
                });

                function appendMessageToChatbox(message, type) {
                    const messageEl = document.createElement('div');
                    messageEl.classList.add('chatbot-message', type);
                    messageEl.innerHTML = message; // Pesan dari bot mungkin sudah mengandung HTML (nl2br)
                    chatbotMessagesDiv.appendChild(messageEl);
                    chatbotMessagesDiv.scrollTop = chatbotMessagesDiv.scrollHeight;
                }
                function removeTypingIndicator() {
                    const typingIndicator = chatbotMessagesDiv.querySelector('.bot-typing');
                    if (typingIndicator) typingIndicator.remove();
                }
            }
            // --- End Chatbot Logic ---


            document.addEventListener('click', function (e) { 
                if (notificationPopup && notificationPopup.classList.contains('show') && !notificationPopup.contains(e.target) && e.target !== bellIcon) { 
                    notificationPopup.classList.remove('show'); 
                }
                if (calendarPopup && calendarPopup.classList.contains('show') && !calendarPopup.contains(e.target) && e.target !== calendarIcon) { 
                    calendarPopup.classList.remove('show'); 
                }
                // Jangan tutup chatbot jika klik di dalam chatbot container
                if (chatbotContainer && chatbotContainer.classList.contains('show') && !chatbotContainer.contains(e.target) && e.target !== chatbotIcon) {
                    // chatbotContainer.classList.remove('show'); 
                    // chatbotIcon.style.display = 'flex'; 
                    // Biarkan terbuka, tutup hanya via tombol close atau icon
                }
            });

            const dateInputs = document.querySelectorAll('input[type="text"][id$="Date"], input[type="text"][id$="DueDate"], input[type="text"][id$="DateRange"], input[type="text"][id="filterHistoryDateRange"], input[type="text"][id="filterReportDateRange"]');
            dateInputs.forEach(input => {
                let config = { dateFormat: "d/m/Y", locale: "id", allowInput: true };
                if (input.id === 'taskDueDate' || input.id === 'editTaskDueDate') {
                     config.minDate = "today";
                }
                if (input.id === 'filterHistoryDateRange' || input.id === 'filterPerformanceDateRange' || input.id === 'filterReportDateRange') { // Tambah filterReportDateRange
                    config.mode = "range";
                } else if (input.id === 'filterDate') { 
                    config.mode = "single";
                }
                flatpickr(input, config);
            });

            const deleteButtons = document.querySelectorAll('a.delete-btn');
            deleteButtons.forEach(button => {
                button.addEventListener('click', function(event) {
                    if (!confirm('Anda yakin ingin menghapus item ini?')) {
                        event.preventDefault();
                    }
                });
            });

            const presetButtons = document.querySelectorAll('.filter-preset-buttons .btn[data-range]');
            presetButtons.forEach(button => { /* ... (Logika preset button sama) ... */ });
            
            const editProfileImageInput = document.getElementById('editProfileImageFile');
            const currentImagePreview = document.getElementById('currentImagePreview');
            if (editProfileImageInput && currentImagePreview) {
                editProfileImageInput.addEventListener('change', function(event) {
                    const file = event.target.files[0];
                    if (file) {
                        const reader = new FileReader();
                        reader.onload = function(e) {
                            currentImagePreview.src = e.target.result;
                        }
                        reader.readAsDataURL(file);
                    }
                });
            }


            // Tema Gelap Logic (FOUC Fix - Target HTML) - Sama seperti sebelumnya
            const themeToggleCheckbox = document.getElementById('themeToggleCheckbox');
            function updateThemeOnPage(theme) {
                if (theme === 'dark-theme') {
                    htmlElement.classList.add('dark-theme-active');
                    if (themeToggleCheckbox) themeToggleCheckbox.checked = true;
                } else { 
                    htmlElement.classList.remove('dark-theme-active');
                    if (themeToggleCheckbox) themeToggleCheckbox.checked = false;
                }
            }
            const initialThemeIsDark = htmlElement.classList.contains('dark-theme-active');
            if (initialThemeIsDark) {
                if (themeToggleCheckbox) themeToggleCheckbox.checked = true;
            } else {
                if (themeToggleCheckbox) themeToggleCheckbox.checked = false;
            }
            if (themeToggleCheckbox) {
                themeToggleCheckbox.addEventListener('change', function() {
                    const newTheme = this.checked ? 'dark-theme' : 'light-theme';
                    updateThemeOnPage(newTheme); 
                    localStorage.setItem('theme', newTheme); 
                });
            }
            
            // Sidebar Toggle Logic - Sama seperti sebelumnya
            const appTitleToggle = document.getElementById('appTitleToggle');
            const sidebarElement = document.getElementById('sidebar'); 
            if (appTitleToggle && sidebarElement) {
                function setSidebarState(isHidden) {
                    if (isHidden) bodyElement.classList.add('sidebar-hidden');
                    else bodyElement.classList.remove('sidebar-hidden');
                }
                if (window.innerWidth > 768) {
                    const sidebarHiddenStored = localStorage.getItem('sidebarHidden') === 'true';
                    setSidebarState(sidebarHiddenStored);
                } else {
                    setSidebarState(false); 
                }
                appTitleToggle.addEventListener('click', () => {
                    if (window.innerWidth > 768) { 
                        const isNowHidden = bodyElement.classList.toggle('sidebar-hidden');
                        localStorage.setItem('sidebarHidden', isNowHidden);
                    }
                });
                window.addEventListener('resize', () => {
                    if (window.innerWidth <= 768) {
                        setSidebarState(false); 
                    } else {
                        const sidebarHiddenStored = localStorage.getItem('sidebarHidden') === 'true';
                        setSidebarState(sidebarHiddenStored);
                    }
                });
            }

            // Klik Card Tugas untuk Ubah Status (Manajemen) - Sama seperti sebelumnya
            if (document.querySelector('.main-content-manajemen')) {
                const taskListContainerManajemen = document.getElementById('managementTaskList');
                if (taskListContainerManajemen) {
                    taskListContainerManajemen.addEventListener('click', function(event) {
                        const card = event.target.closest('.task-item-card[data-task-id]');
                        if (!card) return; 
                        if (event.target.closest('.task-actions') || event.target.closest('a.task-title-link')) {
                            return;
                        }
                        const taskId = card.dataset.taskId;
                        const currentStatus = card.dataset.currentStatus;
                        let nextStatus = '';
                        let nextStatusTextForUI = '';
                        let currentStatusClass = 'status-' + currentStatus.toLowerCase().replace(/ /g, '-');
                        let nextStatusClass = '';

                        if (currentStatus === 'Not Started') {
                            nextStatus = 'In Progress';
                            nextStatusTextForUI = 'Dikerjakan';
                            nextStatusClass = 'status-in-progress';
                        } else if (currentStatus === 'In Progress') {
                            nextStatus = 'Completed';
                            nextStatusTextForUI = 'Selesai';
                            nextStatusClass = 'status-completed-history'; 
                        } else {
                            return; 
                        }
                        const statusTextElement = card.querySelector('.meta-info .task-status-text');
                        
                        card.classList.remove(currentStatusClass);
                        card.classList.add(nextStatusClass);
                        card.dataset.currentStatus = nextStatus; 
                        if (statusTextElement) {
                            statusTextElement.textContent = nextStatusTextForUI;
                        }

                        fetch('ajax_update_task_status.php', {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/x-www-form-urlencoded', },
                            body: `task_id=${taskId}&new_status=${nextStatus}`
                        })
                        .then(response => response.json())
                        .then(data => {
                            if (data.success) {
                                if (statusTextElement && data.new_status_text) {
                                    statusTextElement.textContent = data.new_status_text;
                                }
                                if (data.is_completed) {
                                    card.style.transition = 'opacity 0.4s ease-out, transform 0.4s ease-out, max-height 0.5s ease-in-out, padding 0.5s ease-in-out, margin 0.5s ease-in-out';
                                    card.style.opacity = '0';
                                    card.style.transform = 'scale(0.9)';
                                    card.style.maxHeight = '0px';
                                    card.style.paddingTop = '0px';
                                    card.style.paddingBottom = '0px';
                                    card.style.marginBottom = '0px';
                                    setTimeout(() => {
                                        card.remove();
                                        if (taskListContainerManajemen.children.length === 0) {
                                            if (!taskListContainerManajemen.querySelector('.no-tasks-message')) {
                                                taskListContainerManajemen.innerHTML = '<p class="no-tasks-message">Tidak ada tugas aktif yang sesuai.</p>';
                                            }
                                        }
                                    }, 500); 
                                }
                            } else { 
                                card.classList.remove(nextStatusClass);
                                card.classList.add(currentStatusClass);
                                card.dataset.currentStatus = currentStatus;
                                if (statusTextElement) {
                                     const originalStatusText = currentStatus === 'Not Started' ? 'Belum Mulai' : (currentStatus === 'In Progress' ? 'Dikerjakan' : 'Selesai');
                                     statusTextElement.textContent = originalStatusText;
                                }
                                alert('Gagal memperbarui status: ' + (data.message || 'Error tidak diketahui.'));
                            }
                        })
                        .catch(error => { 
                            console.error('Error AJAX:', error);
                            card.classList.remove(nextStatusClass);
                            card.classList.add(currentStatusClass);
                            card.dataset.currentStatus = currentStatus;
                            if (statusTextElement) {
                                const originalStatusText = currentStatus === 'Not Started' ? 'Belum Mulai' : (currentStatus === 'In Progress' ? 'Dikerjakan' : 'Selesai');
                                statusTextElement.textContent = originalStatusText;
                            }
                            alert('Terjadi kesalahan koneksi saat memperbarui status.');
                        });
                    });
                }
            }
        });
    </script>
<?php else: // Ini adalah penutup untuk if (!$is_special_page_footer)
    // Untuk halaman auth, kita tidak perlu script di atas, tapi tetap butuh penutup body/html
?>
<?php endif; ?>
</body>
</html>
<?php
if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 9: Pembaruan CSS (`css/auth.css` dan `css/style.css`)**

**9.1. `css/auth.css` (Tambahan/Modifikasi)**
```css
/* css/auth.css */
html.auth-html, body.auth-page {
    height: 100%;
    overflow: hidden; 
}

body.auth-page {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh; 
    background-image: url('../images/auth-bg.jpg'); 
    background-size: cover;
    background-position: center;
    font-family: 'Poppins', sans-serif;
    padding: 20px;
    position: relative;
}
body.auth-page::before { 
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background-color: rgba(0,0,0,0.5);
    z-index: 1;
    transition: background-color 0.3s ease;
}

.auth-container {
    background: rgba(255, 255, 255, 0.95); 
    backdrop-filter: blur(5px); 
    padding: 30px 35px; 
    border-radius: 10px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
    width: 100%;
    max-width: 400px; /* Sedikit lebih lebar untuk tombol Google */
    text-align: center;
    position: relative;
    z-index: 2;
    animation: fadeInScale 0.5s ease-out;
    transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

@keyframes fadeInScale {
    from { opacity: 0; transform: scale(0.95); }
    to { opacity: 1; transform: scale(1); }
}

.auth-container .logo-container {
    margin-bottom: 15px;
}
.auth-container .logo-container h1 {
    font-size: 2.5rem;
    color: #7e47b8; 
    margin:0;
    font-weight: 600;
    transition: color 0.3s ease;
}

.auth-container h2 { 
    color: #333; 
    margin-bottom: 8px;
    font-size: 1.4rem; 
    font-weight: 500;
    transition: color 0.3s ease;
}
.auth-container p.subtitle {
    color: #555; 
    margin-bottom: 25px;
    font-size: 0.9rem; 
    transition: color 0.3s ease;
}

.form-group {
    margin-bottom: 18px; 
    text-align: left;
    position: relative; 
}
.form-group label {
    display: block; margin-bottom: 6px; font-weight: 500;
    color: #444; 
    font-size: 0.85rem; 
    transition: color 0.3s ease;
}
.form-group input[type="text"],
.form-group input[type="email"],
.form-group input[type="password"] {
    width: 100%;
    padding: 10px 12px; 
    border: 1px solid #ccc; 
    background-color: #fff; 
    color: #333; 
    border-radius: 6px;
    font-size: 0.9rem; 
    box-sizing: border-box;
    transition: border-color 0.2s, box-shadow 0.2s, background-color 0.3s, color 0.3s;
}
.form-group input:focus {
    border-color: #7e47b8; 
    outline: none;
    box-shadow: 0 0 0 3px rgba(126, 71, 184, 0.2); 
}
.form-group.forgot-password-link {
    text-align: right;
    margin-top: -10px; /* Tarik ke atas sedikit */
    margin-bottom: 15px;
}
.form-group.forgot-password-link a {
    font-size: 0.8rem;
    color: #7e47b8;
    text-decoration: none;
}
.form-group.forgot-password-link a:hover {
    text-decoration: underline;
}


.btn-submit {
    background-color: #7e47b8; 
    color: white;
    padding: 11px 20px; 
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.95rem; 
    font-weight: 500;
    transition: background-color 0.2s ease, transform 0.1s ease;
    display: block;
    width: 100%;
    margin-top: 10px;
}
.btn-submit:hover {
    background-color: #6a3aa2; 
}
.btn-submit:active {
    transform: translateY(1px);
}

/* Tombol Login Sosial */
.social-login-divider {
    margin: 20px 0;
    text-align: center;
    position: relative;
    color: #777;
    font-size: 0.85rem;
}
.social-login-divider::before,
.social-login-divider::after {
    content: '';
    position: absolute;
    top: 50%;
    width: 40%;
    height: 1px;
    background-color: #ddd;
}
.social-login-divider::before { left: 0; }
.social-login-divider::after { right: 0; }

.btn-social-login {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    padding: 10px 15px;
    border-radius: 6px;
    text-decoration: none;
    font-size: 0.9rem;
    font-weight: 500;
    margin-bottom: 15px;
    transition: background-color 0.2s ease, box-shadow 0.1s ease;
    border: 1px solid #ddd;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}
.btn-social-login i {
    margin-right: 10px;
    font-size: 1.2em;
}
.btn-social-login.google {
    background-color: #fff;
    color: #444;
    border-color: #ccc;
}
.btn-social-login.google:hover {
    background-color: #f8f8f8;
    border-color: #bbb;
    box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}
html.dark-theme-active .btn-social-login.google {
    background-color: #333;
    color: #eee;
    border-color: #555;
}
html.dark-theme-active .btn-social-login.google:hover {
    background-color: #404040;
    border-color: #666;
}


.auth-link {
    margin-top: 20px;
    font-size: 0.85rem; 
    color: #444; 
    transition: color 0.3s ease;
}
.auth-link a {
    color: #7e47b8; 
    text-decoration: none;
    font-weight: 500;
    transition: color 0.3s ease;
}
.auth-link a:hover { text-decoration: underline; }

.error-message, /* Kelas ini dari kode lama Anda */
.auth-message.error /* Kelas baru untuk konsistensi */
 { 
    color: #e74c3c;
    font-size: 0.8rem;
    margin-top: 5px;
    display: block;
    text-align: left; /* error-message lama */
    /* Untuk .auth-message */
    background-color: #f8d7da; 
    border: 1px solid #f5c6cb;
    padding: 10px 15px;
    border-radius: 5px;
    margin-bottom: 15px;
    text-align: center; /* auth-message baru */
}
.auth-message.error p { margin: 0.3em 0; }

.auth-message.success {
    background-color: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
    padding: 10px 15px;
    border-radius: 5px;
    margin-bottom: 15px;
    text-align: center;
}
.auth-message.success p { margin: 0.3em 0; }


/* ==========================================================================
   DARK THEME STYLES FOR AUTH PAGE
   ========================================================================== */

html.dark-theme-active body.auth-page::before {
    background-color: rgba(0,0,0,0.7); 
}

html.dark-theme-active .auth-container {
    background: rgba(30, 30, 30, 0.92); 
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.5); 
}

html.dark-theme-active .auth-container .logo-container h1 {
    color: #bb86fc; 
}

html.dark-theme-active .auth-container h2 {
    color: #e0e0e0; 
}

html.dark-theme-active .auth-container p.subtitle {
    color: #b0b0b0; 
}

html.dark-theme-active .form-group label {
    color: #c0c0c0; 
}
html.dark-theme-active .form-group.forgot-password-link a {
    color: #bb86fc;
}

html.dark-theme-active .form-group input[type="text"],
html.dark-theme-active .form-group input[type="email"],
html.dark-theme-active .form-group input[type="password"] {
    background-color: #2c2c2c; 
    border-color: #555; 
    color: #e0e0e0; 
}

html.dark-theme-active .form-group input:focus {
    border-color: #bb86fc; 
    box-shadow: 0 0 0 3px rgba(187, 134, 252, 0.3); 
    background-color: #333; 
}

html.dark-theme-active .btn-submit {
    background-color: #bb86fc; 
    color: #121212; 
}
html.dark-theme-active .btn-submit:hover {
    background-color: #a06fec; 
}

html.dark-theme-active .auth-link {
    color: #b0b0b0; 
}
html.dark-theme-active .auth-link a {
    color: #bb86fc; 
}

html.dark-theme-active .social-login-divider {
    color: #aaa;
}
html.dark-theme-active .social-login-divider::before,
html.dark-theme-active .social-login-divider::after {
    background-color: #444;
}

html.dark-theme-active .auth-message.error {
    background-color: #4d2a2b; /* Lebih gelap untuk dark mode */
    color: #ffab91; /* Teks lebih terang */
    border-color: #8d4c47;
}
html.dark-theme-active .auth-message.success {
    background-color: #2a4d32;
    color: #a5d6a7;
    border-color: #4c8c4a;
}

/* (Pesan error biasanya sudah cukup kontras, tapi bisa disesuaikan jika perlu) */
/* html.dark-theme-active .error-message {
    color: #ff8a80;
} */
```

**9.2. `css/style.css` (Tambahan/Modifikasi)**
```css
/* ... (Semua kode CSS Anda sebelumnya) ... */

/* ==========================================================================
   Laporan Page Styles
   ========================================================================== */
.report-chart-widget {
    margin-bottom: 20px;
}
.report-performance-filter {
    display: flex;
    gap: 8px;
    align-items: center;
    margin-bottom: 10px;
    flex-wrap: nowrap; /* Coba nowrap untuk filter diagram */
    padding: 5px 0;
}
.report-performance-filter label {
    font-size: 0.85rem;
    white-space: nowrap;
    margin-right: 5px;
    color: #555;
}
html.dark-theme-active .report-performance-filter label { color: #ccc; }

.report-performance-filter .filter-preset-buttons {
    display: flex;
    gap: 5px;
}
.report-performance-filter .filter-preset-buttons .btn {
    padding: 5px 10px;
    font-size: 0.75rem;
    height: 30px;
    background-color: #f0f3f7;
    color: #555;
    border: 1px solid #dfe3e8;
}
.report-performance-filter .filter-preset-buttons .btn.active,
.report-performance-filter .filter-preset-buttons .btn:hover {
    background-color: #7e47b8;
    color: white;
    border-color: #7e47b8;
}
html.dark-theme-active .report-performance-filter .filter-preset-buttons .btn {
    background-color: #3a3a3a;
    color: #ccc;
    border-color: #555;
}
html.dark-theme-active .report-performance-filter .filter-preset-buttons .btn.active,
html.dark-theme-active .report-performance-filter .filter-preset-buttons .btn:hover {
    background-color: #bb86fc;
    color: #121212;
    border-color: #bb86fc;
}


.report-performance-filter input[type="text"] {
    padding: 5px 8px;
    font-size: 0.8rem;
    height: 30px;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    width: 150px; /* Atau sesuaikan */
    margin-left: 5px;
}
html.dark-theme-active .report-performance-filter input[type="text"]{
    background-color: #373737;
    border-color: #555;
    color: #e0e0e0;
}


.report-performance-filter .btn-apply-perf {
    padding: 5px 12px;
    font-size: 0.8rem;
    height: 30px;
    margin-left: 5px;
}


.report-tasks-by-date-section {
    /* height: calc(100vh - 55px - 30px - 40px - 250px - 20px - 60px); Sesuaikan jika perlu */
}

.report-interactive-area {
    display: flex;
    gap: 20px;
    height: 100%; /* Atau tinggi spesifik */
    max-height: 60vh; /* Batasi tinggi agar tidak terlalu panjang */
}

.report-task-list-container {
    flex: 2; /* Lebih banyak ruang untuk daftar tugas */
    display: flex;
    flex-direction: column;
    overflow: hidden;
}
.selected-date-indicator {
    font-size: 0.95rem;
    font-weight: 500;
    color: #34495e;
    margin-bottom: 10px;
    padding-bottom: 5px;
    border-bottom: 1px solid #eee;
}
html.dark-theme-active .selected-date-indicator {
    color: #f5f5f5;
    border-bottom-color: #444;
}

#reportTasksList {
    flex-grow: 1;
    overflow-y: auto;
    padding-right: 8px; /* Untuk scrollbar */
}
#reportTasksList .task-item-card { /* Style card tugas mungkin sudah ada, ini hanya contoh */
    font-size: 0.8rem; /* Lebih kecil di laporan */
    padding: 10px;
}
#reportTasksList .task-item-card .task-details strong { font-size: 0.9rem; }


.report-calendar-container {
    flex: 1;
    min-width: 280px; /* Agar kalender tidak terlalu sempit */
    background-color: #f9fafb;
    border-radius: 6px;
    padding: 15px;
    border: 1px solid #e7eaec;
    box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    height: fit-content; /* Agar tingginya pas dengan konten kalender */
}
html.dark-theme-active .report-calendar-container {
    background-color: #373737;
    border-color: #555;
}

.calendar-header-report {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
    font-weight: 600;
    font-size: 1.05rem;
    color: #2c3e50;
}
html.dark-theme-active .calendar-header-report { color: #f0f0f0; }
.calendar-header-report button {
    background: none;
    border: none;
    font-size: 1.2rem;
    color: #7e47b8;
    cursor: pointer;
    padding: 5px 8px;
    border-radius: 4px;
}
.calendar-header-report button:hover { background-color: #f0e9f7; }
html.dark-theme-active .calendar-header-report button { color: #bb86fc; }
html.dark-theme-active .calendar-header-report button:hover { background-color: rgba(187, 134, 252, 0.1); }

.calendar-days-grid-report {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 5px;
    text-align: center;
}
.calendar-day-name-report {
    font-weight: 500;
    font-size: 0.8rem;
    color: #555;
    padding-bottom: 5px;
}
html.dark-theme-active .calendar-day-name-report { color: #bbb; }

.calendar-day-report {
    padding: 8px 5px;
    font-size: 0.85rem;
    border-radius: 50%; /* Buat lingkaran */
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s;
    width: 32px; /* Ukuran tetap untuk lingkaran */
    height: 32px; /* Ukuran tetap untuk lingkaran */
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto; /* Pusatkan lingkaran */
    border: 1px solid transparent; /* Untuk hover border */
}
.calendar-day-report:hover {
    background-color: #eef1f5;
    border-color: #d1d5db;
}
.calendar-day-report.today {
    background-color: #7e47b8;
    color: white;
    font-weight: bold;
}
.calendar-day-report.selected {
    background-color: #34495e;
    color: white;
    border-color: #2c3e50;
}
html.dark-theme-active .calendar-day-report:hover {
    background-color: #4f4f4f;
    border-color: #666;
}
html.dark-theme-active .calendar-day-report.today {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active .calendar-day-report.selected {
    background-color: #5fcf80; /* Contoh warna selected dark mode */
    color: #121212;
    border-color: #5fcf80;
}


/* ==========================================================================
   Chatbot Styles
   ========================================================================== */
#chatbot-icon {
    position: fixed;
    bottom: 25px;
    right: 25px;
    width: 55px;
    height: 55px;
    background-color: #7e47b8;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.8rem;
    cursor: pointer;
    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    z-index: 1010;
    transition: transform 0.2s ease-in-out, background-color 0.2s;
}
#chatbot-icon:hover {
    transform: scale(1.1);
    background-color: #6a3aa2;
}
html.dark-theme-active #chatbot-icon {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #chatbot-icon:hover {
    background-color: #a06fec;
}


#chatbot-container {
    position: fixed;
    bottom: 25px;
    right: 25px;
    width: 350px;
    max-width: 90vw;
    height: 75vh;
    max-height: 500px;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 5px 20px rgba(0,0,0,0.2);
    display: none; /* Awalnya tersembunyi */
    flex-direction: column;
    overflow: hidden;
    z-index: 1009;
    border: 1px solid #ddd;
}
#chatbot-container.show {
    display: flex;
    animation: fadeInScaleChatbot 0.3s ease-out;
}
@keyframes fadeInScaleChatbot {
    from { opacity: 0; transform: scale(0.9) translateY(20px); }
    to { opacity: 1; transform: scale(1) translateY(0); }
}
html.dark-theme-active #chatbot-container {
    background-color: #2c2c2c;
    border-color: #444;
    box-shadow: 0 5px 25px rgba(0,0,0,0.4);
}


#chatbot-header {
    background-color: #7e47b8;
    color: white;
    padding: 12px 15px;
    font-weight: 600;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-shrink: 0;
}
html.dark-theme-active #chatbot-header {
    background-color: #bb86fc;
    color: #121212;
}
#closeChatbotBtn {
    background: none;
    border: none;
    color: white;
    font-size: 1.5rem;
    cursor: pointer;
    line-height: 1;
}
html.dark-theme-active #closeChatbotBtn { color: #121212; }

#chatbot-messages {
    flex-grow: 1;
    padding: 15px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 10px;
}
.chatbot-message {
    padding: 8px 12px;
    border-radius: 15px;
    max-width: 80%;
    word-wrap: break-word;
    line-height: 1.4;
    font-size: 0.9rem;
}
.chatbot-message.user {
    background-color: #e9eaf6; /* Warna user */
    color: #333;
    border-bottom-right-radius: 3px;
    align-self: flex-end;
}
html.dark-theme-active .chatbot-message.user {
    background-color: #555273; /* Dark user */
    color: #e0e0e0;
}
.chatbot-message.bot, .chatbot-message.bot-typing {
    background-color: #f1f0f0; /* Warna bot */
    color: #333;
    border-bottom-left-radius: 3px;
    align-self: flex-start;
}
html.dark-theme-active .chatbot-message.bot, html.dark-theme-active .chatbot-message.bot-typing {
    background-color: #3a3a3a; /* Dark bot */
    color: #e0e0e0;
}
.chatbot-message.bot-typing { font-style: italic; }


#chatbot-input-area {
    display: flex;
    padding: 10px;
    border-top: 1px solid #eee;
    background-color: #f9f9f9;
    flex-shrink: 0;
}
html.dark-theme-active #chatbot-input-area {
    border-top-color: #444;
    background-color: #333;
}
#chatbotInput {
    flex-grow: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 20px;
    margin-right: 8px;
    font-size: 0.9rem;
    outline: none;
}
html.dark-theme-active #chatbotInput {
    background-color: #252525;
    border-color: #555;
    color: #e0e0e0;
}
#sendChatbotMessageBtn {
    background-color: #7e47b8;
    color: white;
    border: none;
    border-radius: 50%;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 1.1rem;
}
#sendChatbotMessageBtn:hover { background-color: #6a3aa2; }
html.dark-theme-active #sendChatbotMessageBtn {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #sendChatbotMessageBtn:hover { background-color: #a06fec; }

/* Penyesuaian Responsive untuk Laporan & Chatbot */
@media (max-width: 992px) {
    .report-interactive-area {
        flex-direction: column;
        max-height: none; /* Hapus batasan tinggi di tablet/mobile */
    }
    .report-calendar-container {
        order: -1; /* Pindahkan kalender ke atas di tampilan sempit */
        margin-bottom: 15px;
    }
}
@media (max-width: 768px) {
    #chatbot-container {
        bottom: 0;
        right: 0;
        width: 100%;
        height: 100%;
        max-height: 100vh;
        border-radius: 0;
        border: none;
    }
}

/* ... (Sisa CSS Anda) ... */
```

---

**LANGKAH 10: Tutorial Integrasi**

**A. Integrasi Akun Google (Google OAuth)**

1.  **Buat Proyek di Google Cloud Console:**
    *   Buka [Google Cloud Console](https://console.cloud.google.com/).
    *   Buat proyek baru (atau pilih yang sudah ada).
    *   Di menu navigasi, buka "APIs & Services" > "Credentials".

2.  **Konfigurasi OAuth Consent Screen:**
    *   Jika ini pertama kali Anda membuat kredensial OAuth, Anda akan diminta untuk mengkonfigurasi "OAuth consent screen".
    *   Pilih "External" untuk User Type jika aplikasi Anda akan diakses oleh pengguna di luar organisasi Google Workspace Anda.
    *   Isi "App name", "User support email", dan "Developer contact information".
    *   Pada bagian "Scopes", klik "Add or Remove Scopes". Cari dan tambahkan:
        *   `.../auth/userinfo.email` (untuk mendapatkan alamat email)
        *   `.../auth/userinfo.profile` (untuk mendapatkan info profil dasar seperti nama dan gambar)
        *   Klik "Update".
    *   Pada bagian "Test users", tambahkan alamat email Anda sendiri untuk pengujian awal.
    *   Simpan dan lanjutkan.

3.  **Buat Kredensial OAuth 2.0 Client ID:**
    *   Di halaman "Credentials", klik "+ CREATE CREDENTIALS" > "OAuth client ID".
    *   Pilih "Web application" untuk Application type.
    *   Beri nama, misalnya "ListIn Web Client".
    *   Pada "Authorized JavaScript origins", tambahkan URL dasar aplikasi Anda (misal, `http://localhost` atau `http://nama_domain_anda.com`).
    *   Pada "Authorized redirect URIs", tambahkan URL callback yang telah kita buat: `http://localhost/nama_folder_proyek_anda/google_auth_callback.php` (sesuaikan dengan `APP_URL` di `config.php`). **Ini sangat penting dan harus cocok persis.**
    *   Klik "Create".
    *   Anda akan mendapatkan **Client ID** dan **Client Secret**. Salin keduanya.

4.  **Masukkan Kredensial ke `config.php`:**
    *   Buka file `config.php` Anda.
    *   Masukkan Client ID ke `define('GOOGLE_CLIENT_ID', 'MASUKKAN_CLIENT_ID_ANDA_DISINI');`
    *   Masukkan Client Secret ke `define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_CLIENT_SECRET_ANDA_DISINI');`
    *   Pastikan `GOOGLE_REDIRECT_URI` di `config.php` sama persis dengan yang Anda masukkan di Google Cloud Console.

5.  **Install Google API Client Library untuk PHP:**
    *   Cara termudah adalah menggunakan Composer. Buka terminal di root direktori proyek Anda dan jalankan:
        ```bash
        composer require google/apiclient:^2.0
        ```
    *   Ini akan membuat folder `vendor` dan file `vendor/autoload.php`.
    *   Pastikan baris `require_once __DIR__ . '/vendor/autoload.php';` ada dan benar di `google_auth_callback.php`, `login.php`, dan `register.php`.

6.  **Testing:**
    *   Coba login atau register menggunakan tombol Google.
    *   Periksa apakah Anda diarahkan ke halaman persetujuan Google dan kemudian kembali ke aplikasi Anda.
    *   Cek database `users` apakah `google_id` terisi.

**B. Integrasi PHPMailer (untuk Lupa Password)**

1.  **Install PHPMailer:**
    *   Gunakan Composer:
        ```bash
        composer require phpmailer/phpmailer
        ```
    *   Pastikan `require __DIR__ . '/vendor/autoload.php';` ada di `forgot_password.php`.

2.  **Konfigurasi SMTP di `config.php`:**
    *   Isi konstanta `SMTP_HOST`, `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_PORT`, `SMTP_SECURE`, `EMAIL_FROM_ADDRESS`, dan `EMAIL_FROM_NAME` dengan detail penyedia email Anda.
    *   **Untuk Gmail:**
        *   `SMTP_HOST`: `smtp.gmail.com`
        *   `SMTP_PORT`: `587`
        *   `SMTP_SECURE`: `tls`
        *   `SMTP_USERNAME`: Alamat Gmail Anda.
        *   `SMTP_PASSWORD`: Gunakan **App Password** jika Anda mengaktifkan 2-Step Verification di akun Google Anda. Jangan gunakan password utama Gmail Anda. Jika tidak pakai 2FA, Anda mungkin perlu mengaktifkan "Less secure app access" (tidak direkomendasikan).
    *   **Untuk penyedia lain:** Cek dokumentasi mereka untuk detail SMTP.

3.  **Testing:**
    *   Coba fitur "Lupa Kata Sandi".
    *   Periksa apakah email diterima dan tautan reset berfungsi.

**C. Integrasi API Key Chatbot Anda**

1.  **Dapatkan API Key:**
    *   Daftar ke layanan chatbot yang Anda pilih (misalnya, Dialogflow, Rasa, OpenAI API, atau layanan lain).
    *   Dapatkan API Key dan, jika ada, URL Endpoint API dari dashboard layanan tersebut.

2.  **Masukkan ke `config.php`:**
    *   Buka `config.php`.
    *   Masukkan API Key Anda ke `define('CHATBOT_API_KEY', 'MASUKKAN_API_KEY_CHATBOT_ANDA_DISINI');`
    *   Jika layanan chatbot Anda memiliki URL endpoint spesifik, masukkan ke `define('CHATBOT_API_ENDPOINT', 'URL_ENDPOINT_CHATBOT_ANDA');`

3.  **Sesuaikan `ajax_chatbot_handler.php`:**
    *   Buka `ajax_chatbot_handler.php`.
    *   Temukan bagian yang ditandai `// BAGIAN INI ADALAH LOGIKA INTI CHATBOT ANDA`.
    *   Modifikasi kode di sana untuk:
        *   Mengirim `user_message` ke `CHATBOT_API_ENDPOINT` menggunakan `CHATBOT_API_KEY` (biasanya via cURL atau GuzzleHTTP). Format request (JSON, form-data, header otorisasi) akan bergantung pada dokumentasi API chatbot Anda.
        *   Menerima dan mem-parse respons dari API chatbot. Respons ini mungkin berisi teks balasan langsung, atau data terstruktur yang berisi "intent" (maksud pengguna) dan "entities" (parameter).
        *   **Implementasi Aksi Internal:** Berdasarkan intent yang diterima dari API chatbot (atau analisis teks manual jika API-nya sederhana), implementasikan logika untuk melakukan aksi-aksi yang diminta (menampilkan tugas, menghapus notifikasi, menghapus tugas tertentu, membersihkan riwayat, dll.). Ini melibatkan query ke database Anda.
            *   **Keamanan:** Hati-hati saat melakukan aksi destruktif (seperti menghapus data) berdasarkan input chatbot. Pastikan Anda memvalidasi input dan hak akses pengguna.
        *   Format `$bot_reply` dengan respons yang sesuai.

4.  **Testing:**
    *   Buka chatbot di aplikasi Anda.
    *   Kirim berbagai perintah yang ingin Anda dukung.
    *   Periksa apakah chatbot merespons dengan benar dan apakah aksi internal (jika ada) berhasil dilakukan.
    *   Periksa log error PHP jika ada masalah.

---

Ini adalah implementasi yang sangat luas. Silakan implementasikan bagian per bagian dan uji secara menyeluruh. Jika Anda mengalami kendala spesifik pada bagian tertentu, jangan ragu untuk bertanya lebih lanjut dengan menyertakan kode yang relevan dan pesan error yang Anda dapatkan.
