Tentu, saya akan membantu Anda menambahkan fitur-fitur yang diminta ke dalam proyek web Anda. Ini adalah perubahan yang cukup signifikan, jadi saya akan memberikan kode selengkap mungkin dan panduan untuk langkah-langkah eksternal.

**Penting:**
*   **Email & PDF (Poin 15):** Sesuai permintaan Anda, saya akan **membatalkan** fitur pengiriman email pengingat deadline dan pengiriman laporan PDF melalui email. Fokusnya adalah unduh PDF dan notifikasi dalam aplikasi.
*   **Google Login & Lupa Password (Email):** Fitur login dengan Google dan pengiriman email untuk lupa password memerlukan konfigurasi eksternal (Google Cloud Console, server SMTP). Saya akan menyediakan kode dengan placeholder dan tutorial di akhir.
*   **Library Eksternal:** Untuk login Google dan pembuatan PDF, kita akan menggunakan library yang diinstal melalui Composer.

Berikut adalah struktur file yang akan kita gunakan (file baru ditandai dengan `*`):

```
.
├── css/
│   ├── auth.css
│   └── style.css
├── images/
│   ├── auth-bg.jpg
│   └── placeholder-profile.png
├── includes/
│   ├── config.php           * (Untuk konfigurasi API Google, dll.)
│   ├── db.php
│   ├── footer.php
│   ├── header.php
│   ├── sidebar.php
│   └── task_helper.php
├── uploads/                 * (Untuk gambar profil, pastikan writable)
│   └── profile_pictures/    * (Pastikan writable)
├── vendor/                  * (Folder library dari Composer)
│   └── ...
├── ajax_clear_all_notifications.php (Harusnya ajax_clear_all_notifications.php)
├── ajax_get_tasks_for_date.php  * (Untuk kalender di halaman laporan)
├── ajax_handler.php
├── ajax_update_task_status.php
├── clear_notifications.php (Sebaiknya digabung ke ajax_handler atau dihapus jika duplikat fungsi)
├── composer.json            * (Untuk dependensi PHP)
├── dashboard.php
├── db_listin.sql            (Akan dimodifikasi)
├── edit_profil.php
├── edit_tugas.php
├── forgot_password.php      *
├── generate_report_pdf.php  *
├── google_auth_handler.php  *
├── index.php                * (Bisa redirect ke login atau dashboard)
├── laporan.php              *
├── login.php
├── logout.php
├── manajemen_tugas.php
├── mark_notifications_viewed.php
├── profil.php
├── register.php
├── reset_password.php       *
├── tambah_tugas.php
└── ubah_password.php
```

---

**Langkah 1: `composer.json` (Baru)**
Buat file `composer.json` di root proyek Anda:
```json
{
    "name": "yourname/listin-project",
    "description": "List In Task Management Application",
    "require": {
        "php": ">=7.4",
        "google/apiclient": "^2.15",
        "setasign/fpdf": "^1.8"
    },
    "authors": [
        {
            "name": "Your Name",
            "email": "your.email@example.com"
        }
    ],
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```
Setelah membuat file ini, jalankan `composer install` di terminal pada direktori root proyek Anda. Ini akan membuat folder `vendor`.

---

**Langkah 2: File Konfigurasi `includes/config.php` (Baru)**
```php
<?php
// includes/config.php

// Mulai session jika belum dimulai (penting untuk Google OAuth state)
if (session_status() == PHP_SESSION_NONE) {
    session_start();
}

// Pastikan vendor autoload dimuat
require_once __DIR__ . '/../vendor/autoload.php';

// Konfigurasi Aplikasi Dasar
define('SITE_URL', 'http://localhost/nama_folder_proyek_anda'); // Ganti dengan URL proyek Anda

// Konfigurasi Google OAuth
define('GOOGLE_CLIENT_ID', 'MASUKKAN_GOOGLE_CLIENT_ID_ANDA_DISINI');
define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_GOOGLE_CLIENT_SECRET_ANDA_DISINI');
define('GOOGLE_REDIRECT_URI', SITE_URL . '/google_auth_handler.php');

// Konfigurasi Database (jika belum ada di db.php atau untuk sentralisasi)
// Biasanya ini sudah ada di db.php, jadi bisa dikomentari jika duplikat
/*
define('DB_HOST', '127.0.0.1');
define('DB_USER', 'root');
define('DB_PASS', '');
define('DB_NAME', 'db_listin');
*/

// Email Aplikasi (Untuk "From" di email lupa password, dll.)
define('APP_EMAIL_ADDRESS', 'listinproject@gmail.com'); // Email "no-reply" Anda
define('APP_EMAIL_NAME', 'List In Support');

// Pengaturan Timezone
date_default_timezone_set('Asia/Jakarta');

?>
```

---
**Langkah 3: Modifikasi `db_listin.sql`**

Tambahkan kolom baru ke tabel `users` dan buat tabel `password_resets`.

```sql
-- File: db_listin.sql
-- (Tambahkan atau modifikasi dari file SQL Anda yang sudah ada)

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET NAMES utf8 */;
/*!50503 SET NAMES utf8mb4 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

CREATE DATABASE IF NOT EXISTS `db_listin` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;
USE `db_listin`;

-- Modifikasi tabel users
DROP TABLE IF EXISTS `users`;
CREATE TABLE IF NOT EXISTS `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `email` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL, -- Boleh NULL jika login via Google
  `profile_image` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT 'images/placeholder-profile.png',
  `oauth_provider` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `oauth_uid` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`),
  KEY `oauth_uid` (`oauth_uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;


-- Tabel untuk reset password
DROP TABLE IF EXISTS `password_resets`;
CREATE TABLE IF NOT EXISTS `password_resets` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `email` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `token` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `user_id` int(11) NOT NULL,
  `expires_at` timestamp NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `token` (`token`),
  KEY `email` (`email`),
  KEY `password_resets_user_id_foreign` (`user_id`),
  CONSTRAINT `password_resets_user_id_foreign` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;


-- Tabel tasks (Pastikan Foreign Key ke users.id benar)
DROP TABLE IF EXISTS `tasks`;
CREATE TABLE IF NOT EXISTS `tasks` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `description` text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
  `priority` enum('Low','Medium','High') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'Medium',
  `status` enum('Not Started','In Progress','Completed') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'Not Started',
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


/*!40101 SET SQL_MODE=IFNULL(@OLD_SQL_MODE, '') */;
/*!40014 SET FOREIGN_KEY_CHECKS=IFNULL(@OLD_FOREIGN_KEY_CHECKS, 1) */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40111 SET SQL_NOTES=IFNULL(@OLD_SQL_NOTES, 1) */;
```
**Catatan:** Jalankan SQL di atas pada phpMyAdmin atau tool database Anda. Ini akan **MENGHAPUS DATA LAMA** di tabel `users`, `password_resets`, dan `tasks` jika sudah ada. Jika Anda ingin mempertahankan data, lakukan `ALTER TABLE` secara manual untuk `users` dan `CREATE TABLE` untuk `password_resets`.

---
**Langkah 4: Modifikasi File PHP yang Ada dan Tambah File Baru**

**`index.php` (Baru atau Modifikasi)**
```php
<?php
// index.php
require_once 'includes/config.php'; // Memuat session_start() dan autoload
require_once 'includes/db.php';

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
} else {
    header("Location: login.php");
    exit();
}
?>
```

**`includes/db.php` (Revisi)**
Pastikan `session_start()` ada di awal dan aman.
```php
<?php
// includes/db.php

// Pastikan config.php sudah di-include untuk session_start() dan pengaturan lain
// Jika file ini di-include dari file yang sudah memanggil config.php, ini bisa jadi redundan,
// tapi aman untuk memastikan.
if (defined('SITE_URL') === false) { // Cek apakah config sudah dimuat
    // Jika ini adalah entry point atau config belum dimuat
    if (file_exists(__DIR__ . '/config.php')) {
        require_once __DIR__ . '/config.php';
    } elseif (file_exists(__DIR__ . '/../includes/config.php')) { // Jika dipanggil dari root folder
        require_once __DIR__ . '/../includes/config.php';
    } else {
        // Fallback jika config.php tidak ditemukan, penting untuk session
        if (session_status() == PHP_SESSION_NONE) {
            session_start();
        }
        date_default_timezone_set('Asia/Jakarta');
    }
}


// Konfigurasi Database
define('DB_HOST', '127.0.0.1');
define('DB_USER', 'root'); // Ganti jika berbeda
define('DB_PASS', '');     // Ganti jika berbeda
define('DB_NAME', 'db_listin'); // Ganti dengan nama database Anda

// Buat Koneksi
$conn = new mysqli(DB_HOST, DB_USER, DB_PASS, DB_NAME);

// Cek Koneksi
if ($conn->connect_error) {
    // Daripada die(), mungkin lebih baik log error dan tampilkan pesan ramah
    error_log("Koneksi Gagal: " . $conn->connect_error);
    // Untuk produksi, jangan tampilkan detail error ke pengguna
    die("Tidak dapat terhubung ke database. Silakan coba lagi nanti.");
}

// Set Charset (penting untuk UTF-8)
if (!$conn->set_charset("utf8mb4")) {
    error_log("Error loading character set utf8mb4: " . $conn->error);
}

// Session sudah dimulai di config.php atau di fallback di atas
?>
```

**`includes/header.php` (Modifikasi)**
*   Panggil `config.php` di awal.
*   Hapus logika notifikasi email deadline & overdue.
*   Pastikan path CSS dan JS benar.

```php
<?php
// includes/header.php
require_once __DIR__ . '/config.php'; // Memuat session_start(), autoload, dan konstanta
require_once __DIR__ . '/db.php';

$current_page = basename($_SERVER['SCRIPT_NAME']);
$current_user_id = $_SESSION['user_id'] ?? null;
$current_user_profile_image = 'images/placeholder-profile.png';
$user_logged_in_with_google = isset($_SESSION['oauth_provider']) && $_SESSION['oauth_provider'] == 'google';


if ($current_user_id && isset($conn)) {
    $stmt_user_header = $conn->prepare("SELECT username, profile_image, oauth_provider FROM users WHERE id = ?");
    if ($stmt_user_header) {
        $stmt_user_header->bind_param("i", $current_user_id);
        $stmt_user_header->execute();
        $result_user_header = $stmt_user_header->get_result();
        if ($user_data_header = $result_user_header->fetch_assoc()) {
            $current_user_profile_image = (!empty($user_data_header['profile_image']) && (filter_var($user_data_header['profile_image'], FILTER_VALIDATE_URL) || file_exists($user_data_header['profile_image'])))
                                        ? $user_data_header['profile_image']
                                        : 'images/placeholder-profile.png';
            $_SESSION['username'] = $user_data_header['username']; // Update session username
            if (!empty($user_data_header['oauth_provider'])) {
                 $_SESSION['oauth_provider'] = $user_data_header['oauth_provider'];
                 $user_logged_in_with_google = ($user_data_header['oauth_provider'] == 'google');
            }
        }
        $stmt_user_header->close();
    }
}

function add_notification($message, $type = 'info') {
    if (!isset($_SESSION['notification_messages'])) {
        $_SESSION['notification_messages'] = [];
    }
    // Cegah duplikasi pesan notifikasi yang persis sama dalam interval pendek (misal 1 menit)
    foreach ($_SESSION['notification_messages'] as $existing_notif) {
        if ($existing_notif['message'] === $message && $existing_notif['type'] === $type && (time() - ($existing_notif['time'] ?? 0)) < 60) {
            return; // Jangan tambahkan jika duplikat dalam 1 menit
        }
    }

    $new_notification = ['message' => $message, 'type' => $type, 'time' => time()];
    $_SESSION['notification_messages'][] = $new_notification;
    
    // Batasi jumlah notifikasi yang disimpan di session
    if (count($_SESSION['notification_messages']) > 15) { // Misalnya, batasi 15 notifikasi
        array_shift($_SESSION['notification_messages']);
    }
    $_SESSION['has_unread_notifications_badge'] = true;
}


// --- LOGIKA NOTIFIKASI DEADLINE & OVERDUE HANYA UNTUK IN-APP (EMAIL DIHAPUS SESUAI PERMINTAAN) ---
// Cek Tugas Deadline H-1 (In-App Notif)
if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])) {
    $tomorrow_date = date('Y-m-d', strtotime('+1 day'));
    $today_for_notif_check_h1 = date('Y-m-d');
    // Key notifikasi harian per user untuk mencegah spam notif yang sama setiap refresh
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
                $_SESSION[$notif_key_deadline_h1] = true; // Tandai notif sudah dikirim hari ini
            }
        } else { error_log("Failed to prepare statement for H-1 deadline check: " . $conn->error); }
    }
}

// Cek Tugas Terlewat DL (In-App Notif)
if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])) {
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
                add_notification($overdue_message, "deadline_soon"); // Menggunakan tipe "deadline_soon" untuk styling

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
        } else { error_log("Failed to prepare statement for overdue check: " . $conn->error); }
    }
}
// --- AKHIR LOGIKA NOTIFIKASI ---


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

$is_auth_page = in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php']);
?>
<!DOCTYPE html>
<html lang="id" class="<?php if ($is_auth_page) echo 'auth-html'; ?>">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
      // Skrip FOUC Prevent (Tema Gelap)
      (function() {
        const theme = localStorage.getItem('theme');
        const htmlEl = document.documentElement;
        if (theme === 'dark-theme' || (!theme && window.matchMedia?.('(prefers-color-scheme: dark)').matches)) {
          htmlEl.classList.add('dark-theme-active');
        }
      })();
    </script>
    <link rel="stylesheet" href="css/style.css?v=<?php echo time(); ?>">
    <?php if ($is_auth_page): ?>
        <link rel="stylesheet" href="css/auth.css?v=<?php echo time(); ?>">
    <?php endif; ?>
</head>
<body class="<?php if ($is_auth_page) echo 'auth-page'; ?>">

    <?php if (!$is_auth_page): ?>
    <header class="header">
        <div class="header-left">
            <a href="profil.php">
                <img src="<?php echo htmlspecialchars($current_user_profile_image); ?>?t=<?php echo time();?>" alt="Foto Profil" class="header-profile-pic" id="headerProfilePic">
            </a>
            <h2 class="app-title-toggle" id="appTitleToggle" title="Toggle Sidebar">List In</h2>
        </div>
        <?php
        // Halaman yang tidak menampilkan search bar utama
        $hide_search_bar_pages = ['profil.php', 'edit_profil.php', 'ubah_password.php', 'tambah_tugas.php', 'edit_tugas.php', 'dashboard.php', 'laporan.php'];
        $hide_search_bar = in_array($current_page, $hide_search_bar_pages);
        
        $search_action_page = 'manajemen_tugas.php'; // Default
        if ($current_page == 'riwayat.php') $search_action_page = 'riwayat.php';
        // Untuk halaman lain, search bar tidak relevan atau disembunyikan.
        ?>
        <div class="search-bar" <?php if ($hide_search_bar) echo 'style="visibility: hidden;"'; ?>>
            <?php if (!$hide_search_bar): // Hanya tampilkan form jika search bar tidak disembunyikan ?>
            <form action="<?php echo $search_action_page; ?>" method="GET" style="display:flex; width:100%;">
                <input type="text" name="search_term" id="searchInputGlobal" placeholder="Cari di <?php echo ($current_page == 'riwayat.php' ? 'Riwayat' : 'Manajemen'); ?>..." value="<?php echo isset($_GET['search_term']) ? htmlspecialchars($_GET['search_term']) : ''; ?>">
                <button type="submit" style="background:none; border:none; padding:0 0 0 8px; margin-left:auto; cursor:pointer;"><i class="fas fa-search"></i></button>
            </form>
            <?php endif; ?>
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

**`includes/sidebar.php` (Modifikasi)**
Tambahkan link ke `laporan.php`.
```php
<?php
// includes/sidebar.php
// $current_page sudah didefinisikan di header.php
$is_auth_page_sidebar = in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php']);
?>
<?php if (!$is_auth_page_sidebar): ?>
        <aside class="sidebar" id="sidebar">
            <nav class="menu">
                <a href="dashboard.php" class="<?php echo ($current_page == 'dashboard.php') ? 'active' : ''; ?>"><i class="fas fa-home"></i> <span>Dasbor</span></a>
                <a href="manajemen_tugas.php" class="<?php echo ($current_page == 'manajemen_tugas.php' || $current_page == 'edit_tugas.php') ? 'active' : ''; ?>"><i class="fas fa-tasks"></i> <span>Kelola Tugas</span></a>
                <a href="tambah_tugas.php" class="<?php echo ($current_page == 'tambah_tugas.php') ? 'active' : ''; ?>"><i class="fas fa-plus-circle"></i> <span>Tambah Tugas</span></a>
                <a href="riwayat.php" class="<?php echo ($current_page == 'riwayat.php') ? 'active' : ''; ?>"><i class="fas fa-history"></i> <span>Riwayat</span></a>
                <a href="laporan.php" class="<?php echo ($current_page == 'laporan.php') ? 'active' : ''; ?>"><i class="fas fa-chart-pie"></i> <span>Laporan</span></a>
            </nav>
            <a href="logout.php" class="logout" id="logoutButton"><i class="fas fa-sign-out-alt"></i> <span>Keluar</span></a>
        </aside>
<?php endif; ?>
```

**`login.php` (Modifikasi)**
Tambahkan tombol login Google dan link Lupa Password.
```php
<?php
// login.php
require_once 'includes/config.php';
require_once 'includes/db.php';

$errors = [];

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

// Google Login
$google_client = new Google_Client();
$google_client->setClientId(GOOGLE_CLIENT_ID);
$google_client->setClientSecret(GOOGLE_CLIENT_SECRET);
$google_client->setRedirectUri(GOOGLE_REDIRECT_URI);
$google_client->addScope("email");
$google_client->addScope("profile");
$google_login_url = $google_client->createAuthUrl();


if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if (isset($_POST['login_submit'])) { // Cek apakah submit dari form login biasa
        $email = trim($_POST['email']);
        $password = $_POST['password'];

        if (empty($email)) $errors[] = "Alamat email wajib diisi.";
        if (empty($password)) $errors[] = "Kata sandi wajib diisi.";

        if (empty($errors)) {
            $stmt = $conn->prepare("SELECT id, username, password, profile_image, oauth_provider FROM users WHERE email = ?");
            $stmt->bind_param("s", $email);
            $stmt->execute();
            $result = $stmt->get_result();

            if ($user = $result->fetch_assoc()) {
                // Cek apakah user terdaftar via Google dan mencoba login manual tanpa password
                if (!empty($user['oauth_provider']) && $user['oauth_provider'] == 'google' && empty($user['password'])) {
                    $errors[] = "Akun ini terdaftar menggunakan Google. Silakan login dengan Google.";
                }
                // Cek password jika ada (untuk login manual atau user Google yg sudah set password)
                elseif (empty($user['password']) || !password_verify($password, $user['password'])) {
                     $errors[] = "Email atau kata sandi salah.";
                } else {
                    // Password cocok
                    $_SESSION['user_id'] = $user['id'];
                    $_SESSION['username'] = $user['username'];
                    $_SESSION['profile_image'] = $user['profile_image'];
                    if (!empty($user['oauth_provider'])) {
                        $_SESSION['oauth_provider'] = $user['oauth_provider'];
                    } else {
                        unset($_SESSION['oauth_provider']);
                    }
                    
                    add_notification("Selamat datang kembali, " . htmlspecialchars($user['username']) . "!", "success");
                    header("Location: dashboard.php");
                    exit();
                }
            } else {
                $errors[] = "Email atau kata sandi salah.";
            }
            $stmt->close();
        }
    }
}
?>
<?php require_once 'includes/header.php'; // Ini akan memanggil config.php juga ?>
<title>Masuk - List In</title>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Selamat Datang Kembali!</h2>
        <p class="subtitle">Masuk untuk melanjutkan dan mengatur tugas Anda.</p>

        <?php if (isset($_SESSION['success_message'])): // Untuk pesan dari register atau reset password ?>
            <div class="auth-message success">
                <?php echo htmlspecialchars($_SESSION['success_message']); ?>
            </div>
            <?php unset($_SESSION['success_message']); ?>
        <?php endif; ?>
        
        <?php if (isset($_SESSION['error_message'])): // Untuk pesan error umum dari halaman lain ?>
            <div class="auth-message error">
                <?php echo htmlspecialchars($_SESSION['error_message']); ?>
            </div>
            <?php unset($_SESSION['error_message']); ?>
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
            <div class="form-group" style="text-align: right; margin-bottom: 10px;">
                <a href="forgot_password.php" class="auth-forgot-link">Lupa Kata Sandi?</a>
            </div>
            <button type="submit" name="login_submit" class="btn-submit">Masuk Akun</button>
        </form>
        
        <div class="auth-separator">
            <span>ATAU</span>
        </div>
        
        <a href="<?php echo htmlspecialchars($google_login_url); ?>" class="btn-google-login">
            <img src="https://developers.google.com/identity/images/g-logo.png" alt="Google logo">
            Masuk dengan Google
        </a>
        
        <p class="auth-link">Belum punya akun? <a href="register.php">Daftar sekarang</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

**`google_auth_handler.php` (Baru)**
```php
<?php
// google_auth_handler.php
require_once 'includes/config.php'; // Memuat session_start, autoload, konstanta
require_once 'includes/db.php';    // Memuat koneksi DB

$google_client = new Google_Client();
$google_client->setClientId(GOOGLE_CLIENT_ID);
$google_client->setClientSecret(GOOGLE_CLIENT_SECRET);
$google_client->setRedirectUri(GOOGLE_REDIRECT_URI);
$google_client->addScope("email");
$google_client->addScope("profile");

if (isset($_GET['code'])) {
    $token = $google_client->fetchAccessTokenWithAuthCode($_GET['code']);
    if (!isset($token['error'])) {
        $google_client->setAccessToken($token['access_token']);
        $_SESSION['access_token'] = $token['access_token'];

        $google_service = new Google_Service_Oauth2($google_client);
        $data = $google_service->userinfo->get();

        $oauth_uid = $data['id'] ?? null;
        $email = $data['email'] ?? null;
        $first_name = $data['givenName'] ?? '';
        $last_name = $data['familyName'] ?? '';
        $username = trim($first_name . ' ' . $last_name);
        if (empty($username)) { // Fallback jika nama tidak ada
            $username = explode('@', $email)[0];
        }
        $profile_image_url = $data['picture'] ?? null;

        if ($oauth_uid && $email) {
            // Cek apakah user sudah ada
            $stmt_check = $conn->prepare("SELECT id, username, profile_image FROM users WHERE email = ? OR (oauth_provider = 'google' AND oauth_uid = ?)");
            $stmt_check->bind_param("ss", $email, $oauth_uid);
            $stmt_check->execute();
            $result_check = $stmt_check->get_result();
            $user = $result_check->fetch_assoc();
            $stmt_check->close();

            if ($user) { // User ada
                // Update oauth_uid jika login dengan email yang sudah ada tapi belum terhubung Google
                if (empty($user['oauth_provider']) || $user['oauth_provider'] !== 'google' || empty($user['oauth_uid'])) {
                    $stmt_update_oauth = $conn->prepare("UPDATE users SET oauth_provider = 'google', oauth_uid = ?, profile_image = IF(profile_image = 'images/placeholder-profile.png' OR profile_image IS NULL, ?, profile_image) WHERE id = ?");
                    $stmt_update_oauth->bind_param("ssi", $oauth_uid, $profile_image_url, $user['id']);
                    $stmt_update_oauth->execute();
                    $stmt_update_oauth->close();
                }
                $_SESSION['user_id'] = $user['id'];
                $_SESSION['username'] = $user['username']; // Gunakan username dari DB
                $_SESSION['profile_image'] = (!empty($user['profile_image']) && $user['profile_image'] !== 'images/placeholder-profile.png') ? $user['profile_image'] : ($profile_image_url ?? 'images/placeholder-profile.png');
                $_SESSION['oauth_provider'] = 'google';

            } else { // User baru, daftarkan
                $stmt_insert = $conn->prepare("INSERT INTO users (username, email, oauth_provider, oauth_uid, profile_image) VALUES (?, ?, 'google', ?, ?)");
                $stmt_insert->bind_param("ssss", $username, $email, $oauth_uid, $profile_image_url);
                if ($stmt_insert->execute()) {
                    $_SESSION['user_id'] = $stmt_insert->insert_id;
                    $_SESSION['username'] = $username;
                    $_SESSION['profile_image'] = $profile_image_url ?? 'images/placeholder-profile.png';
                    $_SESSION['oauth_provider'] = 'google';
                } else {
                    $_SESSION['error_message'] = "Gagal mendaftarkan akun Google Anda: " . $stmt_insert->error;
                    header('Location: login.php');
                    exit();
                }
                $stmt_insert->close();
            }
            
            // Pesan selamat datang untuk pengguna yang baru login/daftar via Google
            if (isset($_SESSION['username'])) {
                 add_notification("Login dengan Google berhasil! Selamat datang, " . htmlspecialchars($_SESSION['username']) . ".", "success");
            }
            header('Location: dashboard.php');
            exit();

        } else {
            $_SESSION['error_message'] = "Gagal mendapatkan data dari Google.";
            header('Location: login.php');
            exit();
        }
    } else {
        $_SESSION['error_message'] = "Terjadi kesalahan saat otentikasi Google: " . htmlspecialchars($token['error_description'] ?? 'Unknown error');
        header('Location: login.php');
        exit();
    }
} else {
    $_SESSION['error_message'] = "Kode otentikasi Google tidak ditemukan.";
    header('Location: login.php');
    exit();
}
?>
```

**`forgot_password.php` (Baru)**
```php
<?php
// forgot_password.php
require_once 'includes/config.php';
require_once 'includes/db.php';

$errors = [];
$success_message = '';

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $email = trim($_POST['email']);

    if (empty($email)) {
        $errors[] = "Alamat email wajib diisi.";
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Format email tidak valid.";
    } else {
        $stmt = $conn->prepare("SELECT id, username, oauth_provider FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();
        $user = $result->fetch_assoc();
        $stmt->close();

        if ($user) {
            if (!empty($user['oauth_provider']) && $user['oauth_provider'] == 'google' && empty($user['password'])) {
                 $errors[] = "Akun ini terdaftar menggunakan Google. Silakan coba login dengan Google atau reset password akun Google Anda.";
            } else {
                // Generate token
                $token = bin2hex(random_bytes(32));
                $expires_at = date('Y-m-d H:i:s', time() + 3600); // Token berlaku 1 jam

                $stmt_insert_token = $conn->prepare("INSERT INTO password_resets (email, token, user_id, expires_at) VALUES (?, ?, ?, ?)");
                $stmt_insert_token->bind_param("ssis", $email, $token, $user['id'], $expires_at);
                
                if ($stmt_insert_token->execute()) {
                    $reset_link = SITE_URL . "/reset_password.php?token=" . $token;
                    
                    // --- SIMULASI PENGIRIMAN EMAIL ---
                    // Di aplikasi nyata, gunakan PHPMailer atau library sejenis
                    $email_subject = "Reset Kata Sandi Akun List In Anda";
                    $email_body = "Halo " . htmlspecialchars($user['username']) . ",\n\n";
                    $email_body .= "Anda menerima email ini karena ada permintaan untuk mereset kata sandi akun Anda di List In.\n";
                    $email_body .= "Silakan klik link berikut untuk mereset kata sandi Anda:\n";
                    $email_body .= $reset_link . "\n\n";
                    $email_body .= "Jika Anda tidak meminta reset kata sandi, abaikan email ini.\n";
                    $email_body .= "Link ini akan kedaluwarsa dalam 1 jam.\n\n";
                    $email_body .= "Terima kasih,\nTim List In";
                    
                    // Untuk demonstrasi, kita tampilkan linknya saja di sini.
                    // Di produksi, ini akan dikirim via email.
                    $success_message = "Link reset kata sandi telah dikirim (simulasi). Cek email Anda. <br><strong>Link (untuk tes):</strong> <a href='$reset_link' target='_blank'>$reset_link</a>";
                    
                    // Ini contoh jika menggunakan mail():
                    // $headers = 'From: ' . APP_EMAIL_NAME . ' <' . APP_EMAIL_ADDRESS . '>' . "\r\n";
                    // if (mail($email, $email_subject, $email_body, $headers)) {
                    //     $success_message = "Link reset kata sandi telah dikirim ke email Anda. Silakan periksa kotak masuk (dan folder spam).";
                    // } else {
                    //     $errors[] = "Gagal mengirim email reset. Silakan coba lagi atau hubungi support.";
                    //     // Mungkin perlu hapus token jika email gagal
                    //     $conn->query("DELETE FROM password_resets WHERE token = '$token'");
                    // }

                } else {
                    $errors[] = "Gagal menyimpan token reset. Silakan coba lagi. " . $stmt_insert_token->error;
                }
                $stmt_insert_token->close();
            }
        } else {
            $errors[] = "Email tidak terdaftar di sistem kami.";
        }
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
    <p class="subtitle">Masukkan alamat email Anda untuk menerima link reset kata sandi.</p>

    <?php if (!empty($success_message)): ?>
        <div class="auth-message success" style="text-align:left; line-height: 1.6;">
            <?php echo $success_message; // Menggunakan echo karena $success_message sudah mengandung HTML <a> ?>
        </div>
    <?php endif; ?>

    <?php if (!empty($errors)): ?>
        <div class="auth-message error">
            <?php foreach ($errors as $error): ?>
                <p><?php echo htmlspecialchars($error); ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <?php if (empty($success_message)) : // Sembunyikan form jika sukses ?>
    <form id="forgotPasswordForm" method="POST" action="forgot_password.php">
        <div class="form-group">
            <label for="email">Alamat Email Terdaftar</label>
            <input type="email" id="email" name="email" required placeholder="cth: pengguna@email.com" value="<?php echo isset($_POST['email']) ? htmlspecialchars($_POST['email']) : ''; ?>">
        </div>
        <button type="submit" class="btn-submit">Kirim Link Reset</button>
    </form>
    <?php endif; ?>

    <p class="auth-link">Ingat kata sandi Anda? <a href="login.php">Masuk di sini</a></p>
    <p class="auth-link" style="margin-top:10px;">Belum punya akun? <a href="register.php">Daftar sekarang</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**`reset_password.php` (Baru)**
```php
<?php
// reset_password.php
require_once 'includes/config.php';
require_once 'includes/db.php';

$errors = [];
$token_is_valid = false;
$token_from_url = $_GET['token'] ?? null;
$user_id_from_token = null;

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

if ($token_from_url) {
    $stmt_check_token = $conn->prepare("SELECT user_id, expires_at FROM password_resets WHERE token = ?");
    $stmt_check_token->bind_param("s", $token_from_url);
    $stmt_check_token->execute();
    $result_token = $stmt_check_token->get_result();
    $token_data = $result_token->fetch_assoc();
    $stmt_check_token->close();

    if ($token_data) {
        if (strtotime($token_data['expires_at']) > time()) {
            $token_is_valid = true;
            $user_id_from_token = $token_data['user_id'];
        } else {
            $errors[] = "Token reset kata sandi sudah kedaluwarsa. Silakan minta link baru.";
            // Hapus token yang kedaluwarsa
            $conn->query("DELETE FROM password_resets WHERE token = '$token_from_url'");
        }
    } else {
        $errors[] = "Token reset kata sandi tidak valid atau sudah digunakan.";
    }
} else {
    $errors[] = "Token reset tidak ditemukan. Pastikan Anda menggunakan link yang benar dari email.";
}


if ($_SERVER["REQUEST_METHOD"] == "POST" && $token_is_valid && $user_id_from_token) {
    $new_password = $_POST['newPassword'];
    $confirm_new_password = $_POST['confirmNewPassword'];

    if (empty($new_password) || empty($confirm_new_password)) {
        $errors[] = "Kata sandi baru dan konfirmasinya wajib diisi.";
    } elseif (strlen($new_password) < 6) {
        $errors[] = "Kata sandi baru minimal 6 karakter.";
    } elseif ($new_password !== $confirm_new_password) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak cocok.";
    }

    if (empty($errors)) {
        $hashed_new_password = password_hash($new_password, PASSWORD_DEFAULT);
        $stmt_update_pass = $conn->prepare("UPDATE users SET password = ?, oauth_provider = NULL, oauth_uid = NULL WHERE id = ?");
        // Menghapus oauth_provider dan oauth_uid jika ada, karena user kini memiliki password lokal
        $stmt_update_pass->bind_param("si", $hashed_new_password, $user_id_from_token);

        if ($stmt_update_pass->execute()) {
            // Hapus token setelah berhasil digunakan
            $conn->query("DELETE FROM password_resets WHERE token = '$token_from_url'");
            
            $_SESSION['success_message'] = "Kata sandi Anda berhasil diubah! Silakan masuk dengan kata sandi baru.";
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Gagal mengubah kata sandi. Silakan coba lagi. " . $stmt_update_pass->error;
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

    <?php if ($token_is_valid): ?>
        <p class="subtitle">Masukkan kata sandi baru Anda di bawah ini.</p>
        <form id="resetPasswordForm" method="POST" action="reset_password.php?token=<?php echo htmlspecialchars($token_from_url); ?>">
            <div class="form-group">
                <label for="newPassword">Kata Sandi Baru</label>
                <input type="password" id="newPassword" name="newPassword" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmNewPassword">Konfirmasi Kata Sandi Baru</label>
                <input type="password" id="confirmNewPassword" name="confirmNewPassword" minlength="6" required placeholder="Ulangi kata sandi baru">
            </div>
            <button type="submit" class="btn-submit">Simpan Kata Sandi Baru</button>
        </form>
    <?php else: ?>
         <p class="subtitle">Jika Anda mengalami masalah, silakan coba minta link reset baru dari <a href="forgot_password.php">halaman lupa kata sandi</a>.</p>
    <?php endif; ?>
    <p class="auth-link" style="margin-top: 20px;">Kembali ke <a href="login.php">Halaman Masuk</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**`ubah_password.php` (Modifikasi)**
Handle jika user login dengan Google dan belum punya password lokal.
```php
<?php
// ubah_password.php
require_once 'includes/header.php'; // Mengandung config.php dan db.php
require_once 'includes/sidebar.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];
$user_logged_in_with_google = isset($_SESSION['oauth_provider']) && $_SESSION['oauth_provider'] == 'google';
$user_has_local_password = false;

// Cek apakah user punya password lokal
$stmt_check_pass_exist = $conn->prepare("SELECT password FROM users WHERE id = ?");
$stmt_check_pass_exist->bind_param("i", $user_id);
$stmt_check_pass_exist->execute();
$result_pass_exist = $stmt_check_pass_exist->get_result();
$user_data_for_pass_check = $result_pass_exist->fetch_assoc();
$stmt_check_pass_exist->close();

if ($user_data_for_pass_check && !empty($user_data_for_pass_check['password'])) {
    $user_has_local_password = true;
}


if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $currentPassword = $_POST['currentPassword'] ?? null; // Mungkin tidak ada jika user Google set password pertama kali
    $newPassword = $_POST['newPassword'];
    $confirmNewPassword = $_POST['confirmNewPassword'];
    $validation_errors_found = false;

    // Validasi dasar
    if (empty($newPassword) || empty($confirmNewPassword)) {
        add_notification("Kata sandi baru dan konfirmasinya wajib diisi.", "error");
        $validation_errors_found = true;
    } elseif (strlen($newPassword) < 6) {
        add_notification("Kata sandi baru minimal 6 karakter.", "error");
        $validation_errors_found = true;
    } elseif ($newPassword !== $confirmNewPassword) {
        add_notification("Kata sandi baru dan konfirmasi tidak cocok.", "error");
        $validation_errors_found = true;
    }

    // Jika user punya password lokal, validasi password saat ini
    if ($user_has_local_password && !$validation_errors_found) {
        if (empty($currentPassword)) {
            add_notification("Kata sandi saat ini wajib diisi.", "error");
            $validation_errors_found = true;
        } elseif (!password_verify($currentPassword, $user_data_for_pass_check['password'])) {
            add_notification("Kata sandi saat ini salah.", "error");
            $validation_errors_found = true;
        }
    }
    // Jika user login via Google dan belum punya password, $currentPassword tidak divalidasi (karena tidak ada)

    if (!$validation_errors_found) {
        $hashed_new_password = password_hash($newPassword, PASSWORD_DEFAULT);
        // Saat user Google set password pertama kali, kita bisa hapus tanda oauth providernya
        // agar ia bisa login manual juga. Atau biarkan saja jika ingin tetap ada history login Google.
        // Untuk kasus ini, kita update password saja.
        $stmt_update = $conn->prepare("UPDATE users SET password = ? WHERE id = ?");
        $stmt_update->bind_param("si", $hashed_new_password, $user_id);
        if ($stmt_update->execute()) {
            add_notification("Kata sandi berhasil diubah/disetel!", "success");
            // Jika sebelumnya login Google dan kini set password, user mungkin ingin logout dan login manual.
            // Atau biarkan saja, behaviornya konsisten.
            header("Location: profil.php");
            exit();
        } else {
            add_notification("Gagal mengubah kata sandi: " . $stmt_update->error, "error");
        }
        $stmt_update->close();
    }
}
?>
<title>Ubah Kata Sandi - List In</title>
<main class="main">
    <h2 class="page-title">
        <?php echo ($user_logged_in_with_google && !$user_has_local_password) ? "Setel Kata Sandi Akun" : "Ubah Kata Sandi Akun"; ?>
    </h2>

    <div class="form-container">
        <?php if ($user_logged_in_with_google && !$user_has_local_password) : ?>
            <p class="auth-message info" style="text-align: left; margin-bottom:15px;">
                Anda saat ini login menggunakan akun Google. Anda dapat menyetel kata sandi lokal untuk akun ini agar dapat login secara manual juga.
            </p>
        <?php endif; ?>

        <form id="changePasswordForm" method="POST" action="ubah_password.php">
            <?php if ($user_has_local_password): // Hanya tampilkan field password saat ini jika ada password lokal ?>
            <div class="form-group">
                <label for="currentPassword">Kata Sandi Saat Ini</label>
                <input type="password" id="currentPassword" name="currentPassword" required placeholder="Masukkan kata sandi Anda saat ini">
            </div>
            <?php endif; ?>

            <div class="form-group">
                <label for="newPassword">Kata Sandi Baru</label>
                <input type="password" id="newPassword" name="newPassword" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmNewPassword">Konfirmasi Kata Sandi Baru</label>
                <input type="password" id="confirmNewPassword" name="confirmNewPassword" minlength="6" required placeholder="Ulangi kata sandi baru">
            </div>
            <div class="form-actions">
                <button type="submit" class="btn btn-primary">
                     <?php echo ($user_logged_in_with_google && !$user_has_local_password) ? "Setel Kata Sandi" : "Simpan Kata Sandi"; ?>
                </button>
                <a href="profil.php" class="btn btn-secondary">Batal</a>
            </div>
        </form>
    </div>
</main>
<?php require_once 'includes/footer.php'; ?>
```

**`register.php` (Modifikasi)**
Pastikan `oauth_provider` dan `oauth_uid` bisa NULL.
```php
<?php
// register.php
require_once 'includes/config.php';
require_once 'includes/db.php';

$errors = [];

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
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
            $errors[] = "Email sudah terdaftar. Gunakan email lain atau <a href='login.php'>masuk</a>.";
        }
        $stmt_check_email->close();
    }

    if (empty($errors)) {
        $hashed_password = password_hash($password, PASSWORD_DEFAULT);
        $default_profile_image = 'images/placeholder-profile.png';

        // oauth_provider dan oauth_uid akan NULL untuk registrasi manual
        $stmt_insert = $conn->prepare("INSERT INTO users (username, email, password, profile_image, oauth_provider, oauth_uid) VALUES (?, ?, ?, ?, NULL, NULL)");
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
                    <p><?php echo $error; // Izinkan HTML untuk link 'masuk' ?></p>
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
            <button type="submit" class="btn-submit">Daftar Akun</button>
        </form>
        <p class="auth-link">Sudah punya akun? <a href="login.php">Masuk di sini</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

---
**Langkah 5: Halaman Laporan (`laporan.php`, `generate_report_pdf.php`, `ajax_get_tasks_for_date.php`)**

**`laporan.php` (Baru)**
```php
<?php
require_once 'includes/header.php';
require_once 'includes/sidebar.php';
require_once 'includes/task_helper.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];

// --- Default Date Range & Filter Logic ---
$today_for_report = new DateTimeImmutable();
$report_start_date_val = $today_for_report->modify('-6 days')->format('Y-m-d'); // Default 7 hari terakhir
$report_end_date_val = $today_for_report->format('Y-m-d');
$active_preset_report_val = 'last7days'; // Default preset

if (isset($_GET['filterReportSubmit'])) {
    if (isset($_GET['filterReportDateRange']) && !empty(trim($_GET['filterReportDateRange']))) {
        $range_report_str = trim($_GET['filterReportDateRange']);
        $range_report_arr = explode(' - ', $range_report_str);
        
        if (count($range_report_arr) >= 1) {
            $date_start_parts_report = explode('/', trim($range_report_arr[0]));
            if (count($date_start_parts_report) == 3 && checkdate((int)$date_start_parts_report[1], (int)$date_start_parts_report[0], (int)$date_start_parts_report[2])) {
                $report_start_date_val = $date_start_parts_report[2] . '-' . $date_start_parts_report[1] . '-' . $date_start_parts_report[0];
                $active_preset_report_val = ''; // Reset preset jika custom range
            }
            if (count($range_report_arr) == 2) {
                $date_end_parts_report = explode('/', trim($range_report_arr[1]));
                if (count($date_end_parts_report) == 3 && checkdate((int)$date_end_parts_report[1], (int)$date_end_parts_report[0], (int)$date_end_parts_report[2])) {
                    $report_end_date_val = $date_end_parts_report[2] . '-' . $date_end_parts_report[1] . '-' . $date_end_parts_report[0];
                }
            } else { // Jika hanya satu tanggal di custom range
                $report_end_date_val = $report_start_date_val;
            }
        }
    }
} elseif (isset($_GET['range_type_report'])) {
    $active_preset_report_val = $_GET['range_type_report'];
    switch ($active_preset_report_val) {
        case 'today':
            $report_start_date_val = $today_for_report->format('Y-m-d');
            $report_end_date_val = $today_for_report->format('Y-m-d');
            break;
        case 'last7days':
            $report_end_date_val = $today_for_report->format('Y-m-d');
            $report_start_date_val = $today_for_report->modify('-6 days')->format('Y-m-d');
            break;
        case 'this_month':
            $report_start_date_val = $today_for_report->format('Y-m-01');
            $report_end_date_val = $today_for_report->format('Y-m-t');
            break;
        // default: // last7days is default
    }
}

// --- DATA UNTUK STATUS TUGAS (Progress Circles) dalam rentang waktu ---
$total_tasks_in_range = 0; $completed_in_range = 0; $in_progress_in_range = 0; $not_started_in_range = 0;
if (isset($conn)) {
    // Tugas yang relevan: dibuat atau jatuh tempo atau diselesaikan dalam rentang waktu
    $stmt_stats_range = $conn->prepare(
       "SELECT status, COUNT(*) as count FROM tasks 
        WHERE user_id = ? 
        AND (
            (status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?) OR 
            (status != 'Completed' AND (DATE(created_at) BETWEEN ? AND ? OR DATE(due_date) BETWEEN ? AND ?))
        )
        GROUP BY status"
    );
    if ($stmt_stats_range) {
        $stmt_stats_range->bind_param("issssss", $user_id, $report_start_date_val, $report_end_date_val, $report_start_date_val, $report_end_date_val, $report_start_date_val, $report_end_date_val);
        if ($stmt_stats_range->execute()) {
            $result_stats_range = $stmt_stats_range->get_result();
            while ($row_stat_range = $result_stats_range->fetch_assoc()) {
                $count_val = (int)$row_stat_range['count'];
                $total_tasks_in_range += $count_val;
                if ($row_stat_range['status'] == 'Completed') $completed_in_range = $count_val;
                elseif ($row_stat_range['status'] == 'In Progress') $in_progress_in_range = $count_val;
                elseif ($row_stat_range['status'] == 'Not Started') $not_started_in_range = $count_val;
            }
        }
        $stmt_stats_range->close();
    }
}
$progress_circles_data_report = [
    'Selesai' => ['count' => $completed_in_range, 'percent' => ($total_tasks_in_range > 0) ? round(($completed_in_range / $total_tasks_in_range) * 100) : 0, 'color' => '#4caf50'],
    'Dikerjakan' => ['count' => $in_progress_in_range, 'percent' => ($total_tasks_in_range > 0) ? round(($in_progress_in_range / $total_tasks_in_range) * 100) : 0, 'color' => '#2196f3'],
    'Belum Mulai' => ['count' => $not_started_in_range, 'percent' => ($total_tasks_in_range > 0) ? round(($not_started_in_range / $total_tasks_in_range) * 100) : 0, 'color' => '#f44336'],
];


// --- DATA UNTUK DIAGRAM GARIS (Performa Pengerjaan) ---
$line_chart_labels_report = []; $line_chart_data_completed_report = [];
if (isset($conn)) {
    try {
        $current_date_loop_obj = DateTime::createFromFormat('Y-m-d', $report_start_date_val);
        $end_date_loop_obj = DateTime::createFromFormat('Y-m-d', $report_end_date_val);
        if ($current_date_loop_obj && $end_date_loop_obj) {
            if ($current_date_loop_obj > $end_date_loop_obj) { // Tukar jika salah urutan
                 list($current_date_loop_obj, $end_date_loop_obj) = [$end_date_loop_obj, $current_date_loop_obj];
            }
            $loop_count = 0; $interval_one_day = new DateInterval('P1D');
            
            $all_completed_counts_on_date = [];
            $stmt_all_completed_range = $conn->prepare("SELECT DATE(updated_at) as completion_date, COUNT(*) as count FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ? GROUP BY DATE(updated_at)");
            if ($stmt_all_completed_range) {
                $stmt_all_completed_range->bind_param("iss", $user_id, $report_start_date_val, $report_end_date_val);
                $stmt_all_completed_range->execute();
                $result_all_completed = $stmt_all_completed_range->get_result();
                while ($row_completed = $result_all_completed->fetch_assoc()) {
                    $all_completed_counts_on_date[$row_completed['completion_date']] = (int)$row_completed['count'];
                }
                $stmt_all_completed_range->close();
            }

            while ($current_date_loop_obj <= $end_date_loop_obj) {
                $date_str_loop = $current_date_loop_obj->format('Y-m-d');
                $line_chart_labels_report[] = $current_date_loop_obj->format('d M');
                $line_chart_data_completed_report[] = $all_completed_counts_on_date[$date_str_loop] ?? 0;
                
                $current_date_loop_obj->add($interval_one_day); $loop_count++;
                if ($loop_count > 90 ) { // Batasi loop untuk performa
                    if ($current_date_loop_obj <= $end_date_loop_obj) { 
                        $line_chart_labels_report[] = "..."; $line_chart_data_completed_report[] = null; 
                    }
                    break; 
                }
            }
        }
    } catch (Exception $e) { error_log("Laporan (Performa): Exception: " . $e->getMessage()); }
}
if (empty($line_chart_labels_report)) $line_chart_labels_report = ['Tidak Ada Data'];
if (empty($line_chart_data_completed_report)) $line_chart_data_completed_report = [0];
$performance_chart_data_report_final = [
    'labels' => $line_chart_labels_report,
    'datasets' => [[
        'label' => 'Tugas Selesai','data' => $line_chart_data_completed_report,
        'borderColor' => '#7e47b8','backgroundColor' => 'rgba(126, 71, 184, 0.1)',
        'fill' => true,'tension' => 0.2
    ]]
];

// Tentukan rentang tanggal untuk kalender Flatpickr
$calendar_default_date = $today_for_report->format("Y-m-d");
$calendar_min_date = $report_start_date_val;
$calendar_max_date = $report_end_date_val;

?>
<title>Laporan Tugas - List In</title>

<main class="main">
    <div class="report-page-header">
        <h2 class="page-title">Laporan Tugas</h2>
        <a href="generate_report_pdf.php?start_date=<?php echo urlencode($report_start_date_val); ?>&end_date=<?php echo urlencode($report_end_date_val); ?>&preset=<?php echo urlencode($active_preset_report_val); ?>" target="_blank" class="btn btn-primary"><i class="fas fa-file-pdf"></i> Unduh Laporan PDF</a>
    </div>
    
    <div class="filters-container report-filters">
        <form method="GET" action="laporan.php" class="performance-filter-form" style="width:100%; justify-content: flex-start;">
            <label style="margin-right: 10px;">Filter Rentang Waktu:</label>
            <div class="filter-preset-buttons">
                <button type="submit" name="range_type_report" value="today" class="btn btn-sm <?php echo $active_preset_report_val == 'today' ? 'active' : ''; ?>">Hari Ini</button>
                <button type="submit" name="range_type_report" value="last7days" class="btn btn-sm <?php echo $active_preset_report_val == 'last7days' ? 'active' : ''; ?>">7 Hari Terakhir</button>
                <button type="submit" name="range_type_report" value="this_month" class="btn btn-sm <?php echo $active_preset_report_val == 'this_month' ? 'active' : ''; ?>">Bulan Ini</button>
            </div>
            <input type="text" id="filterReportDateRange" name="filterReportDateRange" placeholder="Pilih Rentang Kustom..." value="<?php echo htmlspecialchars($_GET['filterReportDateRange'] ?? ''); ?>" style="width: 200px; margin-left:10px;">
            <div class="filter-action-buttons" style="margin-left:10px;">
                <button type="submit" name="filterReportSubmit" value="1" class="btn btn-primary btn-apply-perf btn-sm">Terapkan Filter</button>
                 <a href="laporan.php" class="btn btn-secondary btn-sm">Reset</a>
            </div>
        </form>
    </div>

    <div class="report-grid">
        <div class="report-grid-left">
            <section class="widget report-widget">
                <h3>Status Tugas (Rentang: <?php echo date("d M Y", strtotime($report_start_date_val)) . " - " . date("d M Y", strtotime($report_end_date_val)); ?>)</h3>
                <div class="widget-content-area status-progress-container">
                    <?php if ($total_tasks_in_range > 0): ?>
                        <?php foreach ($progress_circles_data_report as $label => $data): ?>
                        <div class="progress-item">
                            <div class="progress-circle"
                                 style="--progress-color: <?php echo $data['color']; ?>; --progress-percent: <?php echo $data['percent']; ?>%;"
                                 data-progress="<?php echo $data['percent']; ?>">
                            </div>
                            <p><?php echo $label; ?> (<?php echo $data['count']; ?>)</p>
                        </div>
                        <?php endforeach; ?>
                    <?php else: ?>
                        <p class="no-tasks-message" style="text-align:center; width:100%; padding: 20px 0;">Tidak ada data tugas untuk status pada rentang ini.</p>
                    <?php endif; ?>
                </div>
            </section>

            <section class="widget report-widget">
                <h3>Performa Pengerjaan (Rentang: <?php echo date("d M Y", strtotime($report_start_date_val)) . " - " . date("d M Y", strtotime($report_end_date_val)); ?>)</h3>
                <div class="widget-content-area chart-container">
                     <?php
                     $has_valid_line_chart_data_report = false;
                     if (isset($performance_chart_data_report_final['datasets'][0]['data']) && is_array($performance_chart_data_report_final['datasets'][0]['data'])) {
                         $filtered_data_report = array_filter($performance_chart_data_report_final['datasets'][0]['data'], function($x) { return $x !== null && $x >=0; });
                         $has_valid_line_chart_data_report = !empty($filtered_data_report) && count($filtered_data_report) > 0;
                     }
                     $has_valid_labels_report = !empty($performance_chart_data_report_final['labels']) && !in_array("Error", $performance_chart_data_report_final['labels']) && !in_array("Tidak Ada Data", $performance_chart_data_report_final['labels']);
                     
                     if ($has_valid_line_chart_data_report && $has_valid_labels_report):
                     ?>
                        <canvas id="reportPerformanceLineChart"></canvas>
                    <?php else: ?>
                        <p class="no-tasks-message" style="text-align:center; padding: 20px 0;">Tidak ada data tugas selesai untuk ditampilkan pada rentang waktu ini.</p>
                    <?php endif; ?>
                </div>
            </section>
        </div>

        <div class="report-grid-right">
            <section class="widget report-widget report-calendar-widget">
                <h3>Kalender Tugas (Rentang Terfilter)</h3>
                <div class="report-calendar-container">
                    <div id="reportCalendarInteractive"></div>
                    <div id="reportCalendarTasksList">
                        <p class="no-tasks-message">Pilih tanggal pada kalender untuk melihat tugas.</p>
                    </div>
                </div>
            </section>
        </div>
    </div>
</main>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // Chart untuk Performa di Laporan
    const reportLineCtx = document.getElementById('reportPerformanceLineChart');
    if (reportLineCtx && typeof Chart !== 'undefined') {
        const reportPerformanceData = <?php echo json_encode($performance_chart_data_report_final); ?>;
        let hasValidReportLineData = false;
        if (reportPerformanceData && reportPerformanceData.labels && Array.isArray(reportPerformanceData.labels) &&
            reportPerformanceData.datasets && Array.isArray(reportPerformanceData.datasets) && reportPerformanceData.datasets.length > 0 &&
            reportPerformanceData.datasets[0].data && Array.isArray(reportPerformanceData.datasets[0].data) ) {
            
            let validLabelsExistReport = reportPerformanceData.labels.some(l => l !== 'Error' && l !== 'Tidak Ada Data' && l !== '...');
            let numericDataExistsReport = reportPerformanceData.datasets[0].data.some(d => typeof d === 'number' && d >= 0);
            hasValidReportLineData = validLabelsExistReport && numericDataExistsReport;
        }

        if (hasValidReportLineData) {
            new Chart(reportLineCtx, {
                type: 'line', data: reportPerformanceData,
                options: { responsive: true, maintainAspectRatio: false,
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1, precision: 0, callback: function(value) {if (Number.isInteger(value)) {return value;}}} } },
                    plugins: { legend: { display: false }, tooltip: { callbacks: { label: function(context) { let label = context.dataset.label || ''; if (label) { label += ': '; } if (context.parsed.y !== null) { label += context.parsed.y; } return label; }}}}
                }
            });
        }
    }

    // Logika Tombol Preset Filter Laporan
    const reportPresetButtons = document.querySelectorAll('.report-filters .filter-preset-buttons .btn');
    const reportDateRangeInput = document.getElementById('filterReportDateRange');
    reportPresetButtons.forEach(button => {
        button.addEventListener('click', function(e) {
            // e.preventDefault(); // Hentikan submit form bawaan agar bisa set input dulu
            if (reportDateRangeInput) reportDateRangeInput.value = ''; 
            // Hapus kelas active dari semua tombol preset
            const parentButtonsContainer = this.closest('.filter-preset-buttons');
            if(parentButtonsContainer){
                parentButtonsContainer.querySelectorAll('.btn').forEach(btn => btn.classList.remove('active'));
            }
            this.classList.add('active'); // Tambahkan active ke yang diklik
            // Form akan tersubmit karena tipe tombol adalah submit
        });
    });
    if (reportDateRangeInput) { // Jika input tanggal kustom diisi, hapus status active dari preset
        reportDateRangeInput.addEventListener('input', function() {
            if (this.value !== '') {
                 const parentButtonsContainer = this.closest('.performance-filter-form').querySelector('.filter-preset-buttons');
                 if(parentButtonsContainer){
                     parentButtonsContainer.querySelectorAll('.btn').forEach(btn => btn.classList.remove('active'));
                 }
            }
        });
         flatpickr(reportDateRangeInput, {
            mode: "range",
            dateFormat: "d/m/Y",
            locale: "id",
            allowInput: true
        });
    }


    // Kalender Interaktif di Laporan
    const calendarEl = document.getElementById('reportCalendarInteractive');
    const tasksListEl = document.getElementById('reportCalendarTasksList');

    if (calendarEl && tasksListEl) {
        const fpCalendar = flatpickr(calendarEl, {
            inline: true,
            dateFormat: "Y-m-d", // Format untuk value
            locale: "id",
            defaultDate: "<?php echo $calendar_default_date; ?>",
            minDate: "<?php echo $calendar_min_date; ?>",
            maxDate: "<?php echo $calendar_max_date; ?>",
            onChange: function(selectedDates, dateStr, instance) {
                if (selectedDates.length > 0) {
                    loadTasksForDate(dateStr);
                }
            },
            onReady: function(selectedDates, dateStr, instance) { // Muat tugas untuk tanggal default saat kalender siap
                const defaultDateOnInit = instance.config.defaultDate || instance.now;
                const formattedDefaultDate = instance.formatDate(defaultDateOnInit, "Y-m-d");
                loadTasksForDate(formattedDefaultDate);
            }
        });

        function loadTasksForDate(dateStr_YMD) {
            tasksListEl.innerHTML = '<p class="loading-message">Memuat tugas...</p>';
            fetch(`ajax_get_tasks_for_date.php?date=${dateStr_YMD}`)
                .then(response => response.json())
                .then(data => {
                    tasksListEl.innerHTML = ''; // Bersihkan dulu
                    if (data.success && data.tasks.length > 0) {
                        data.tasks.forEach(task => {
                            // Menggunakan fungsi render_task_card dari PHP via JS (jika kompleks)
                            // atau buat template JS sederhana di sini
                            const taskCard = document.createElement('div');
                            taskCard.className = `task-item-card report-task-item ${task.status_class || ''}`; // Tambah kelas status
                            taskCard.innerHTML = `
                                <div class="task-details">
                                    <strong>${task.title || 'Tanpa Judul'}</strong>
                                    <p class="description">${task.description || 'Tidak ada deskripsi.'}</p>
                                    <p class="meta-info">
                                        Prioritas: <span class="priority-${(task.priority || 'medium').toLowerCase()}">${task.priority || 'Medium'}</span> | 
                                        Status: ${task.status_text || 'N/A'} | 
                                        Deadline: ${task.due_date_formatted || 'N/A'}
                                    </p>
                                </div>
                                ${task.is_active ? 
                                    `<div class="task-actions">
                                        <a href="edit_tugas.php?id=${task.id}&from=laporan.php" class="edit-btn" title="Edit Tugas"><i class="fas fa-edit"></i></a>
                                    </div>` : 
                                    (task.status === 'Completed' ? `<div class="task-actions"><i class="fas fa-check-circle" style="color: green;" title="Selesai"></i></div>` : '')
                                }
                            `;
                            tasksListEl.appendChild(taskCard);
                        });
                    } else if (data.success && data.tasks.length === 0) {
                        tasksListEl.innerHTML = '<p class="no-tasks-message">Tidak ada tugas pada tanggal ini.</p>';
                    } else {
                        tasksListEl.innerHTML = `<p class="no-tasks-message error-message">${data.message || 'Gagal memuat tugas.'}</p>`;
                    }
                })
                .catch(error => {
                    console.error('Error fetching tasks:', error);
                    tasksListEl.innerHTML = '<p class="no-tasks-message error-message">Terjadi kesalahan saat memuat tugas.</p>';
                });
        }
    }
});
</script>
<?php require_once 'includes/footer.php'; ?>
```

**`ajax_get_tasks_for_date.php` (Baru)**
```php
<?php
// ajax_get_tasks_for_date.php
require_once 'includes/config.php';
require_once 'includes/db.php';

header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'User not authenticated.']);
    exit();
}
$user_id = $_SESSION['user_id'];
$selected_date_str = $_GET['date'] ?? null; // Format Y-m-d

if (!$selected_date_str || !DateTime::createFromFormat('Y-m-d', $selected_date_str)) {
    echo json_encode(['success' => false, 'message' => 'Format tanggal tidak valid.']);
    exit();
}

$tasks_on_date = [];
// Ambil tugas yang DUE pada tanggal tersebut ATAU COMPLETED pada tanggal tersebut
$sql = "SELECT id, title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted, DATE(updated_at) as completion_date
        FROM tasks
        WHERE user_id = ? AND 
              (DATE(due_date) = ? OR (status = 'Completed' AND DATE(updated_at) = ?))
        ORDER BY CASE priority WHEN 'High' THEN 1 WHEN 'Medium' THEN 2 WHEN 'Low' THEN 3 ELSE 4 END, due_date ASC";

$stmt = $conn->prepare($sql);
if ($stmt) {
    $stmt->bind_param("iss", $user_id, $selected_date_str, $selected_date_str);
    $stmt->execute();
    $result = $stmt->get_result();
    while ($row = $result->fetch_assoc()) {
        $status_text = '';
        $status_class = '';
        $is_active = false;
        switch ($row['status']) {
            case 'Not Started': $status_text = 'Belum Mulai'; $status_class = 'status-not-started'; $is_active = true; break;
            case 'In Progress': $status_text = 'Dikerjakan'; $status_class = 'status-in-progress'; $is_active = true; break;
            case 'Completed': $status_text = 'Selesai'; $status_class = 'status-completed-history'; break;
            default: $status_text = $row['status'];
        }
        // Cek apakah tugas overdue dan belum selesai
        if ($row['status'] !== 'Completed' && !empty($row['due_date']) && strtotime($row['due_date']) < strtotime(date('Y-m-d')) && $row['due_date'] == $selected_date_str ) {
            $status_class = 'status-overdue-uncompleted-history';
            // $status_text .= ' (Terlewat)'; // Tambahan info jika mau
        }


        $tasks_on_date[] = [
            'id' => $row['id'],
            'title' => htmlspecialchars($row['title']),
            'description' => nl2br(htmlspecialchars($row['description'] ?: '')),
            'priority' => htmlspecialchars($row['priority']),
            'status' => $row['status'],
            'status_text' => $status_text,
            'status_class' => $status_class,
            'is_active' => $is_active,
            'due_date_formatted' => $row['due_date_formatted'],
            'completion_date_formatted' => ($row['status'] == 'Completed' && $row['completion_date']) ? date('d/m/Y', strtotime($row['completion_date'])) : null
        ];
    }
    $stmt->close();
    echo json_encode(['success' => true, 'tasks' => $tasks_on_date]);
} else {
    echo json_encode(['success' => false, 'message' => 'Gagal mempersiapkan query: ' . $conn->error]);
}
exit();
?>
```

**`generate_report_pdf.php` (Baru)**
```php
<?php
// generate_report_pdf.php
require_once 'includes/config.php'; // Untuk autoload FPDF, dll.
require_once 'includes/db.php';

if (!isset($_SESSION['user_id'])) {
    // Seharusnya tidak bisa diakses langsung tanpa login, tapi sebagai pengaman
    die("Akses ditolak. Silakan login terlebih dahulu.");
}
$user_id = $_SESSION['user_id'];
$username = $_SESSION['username'] ?? 'Pengguna';

$start_date_str = $_GET['start_date'] ?? date('Y-m-d', strtotime('-6 days'));
$end_date_str = $_GET['end_date'] ?? date('Y-m-d');
$preset = $_GET['preset'] ?? 'Kustom';

// Validasi tanggal
if (!DateTime::createFromFormat('Y-m-d', $start_date_str) || !DateTime::createFromFormat('Y-m-d', $end_date_str)) {
    die("Format tanggal tidak valid.");
}
if (strtotime($start_date_str) > strtotime($end_date_str)) {
    list($start_date_str, $end_date_str) = [$end_date_str, $start_date_str]; // Tukar jika salah
}

// Ambil data tugas untuk laporan
$tasks_for_report = [];
$sql_report_tasks = "SELECT title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted, DATE_FORMAT(updated_at, '%d/%m/%Y') as completion_date_formatted
                     FROM tasks
                     WHERE user_id = ? AND 
                           ( (status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?) OR
                             (status != 'Completed' AND (DATE(created_at) BETWEEN ? AND ? OR DATE(due_date) BETWEEN ? AND ?)) )
                     ORDER BY due_date ASC, FIELD(priority, 'High', 'Medium', 'Low')";
$stmt_report = $conn->prepare($sql_report_tasks);
$stmt_report->bind_param("issssss", $user_id, $start_date_str, $end_date_str, $start_date_str, $end_date_str, $start_date_str, $end_date_str);
$stmt_report->execute();
$result_report = $stmt_report->get_result();
while($row = $result_report->fetch_assoc()){
    $tasks_for_report[] = $row;
}
$stmt_report->close();

// Ambil data statistik
$total_tasks_report = 0; $completed_report = 0; $in_progress_report = 0; $not_started_report = 0;
$stmt_stats_report_pdf = $conn->prepare(
   "SELECT status, COUNT(*) as count FROM tasks 
    WHERE user_id = ? 
    AND (
        (status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?) OR 
        (status != 'Completed' AND (DATE(created_at) BETWEEN ? AND ? OR DATE(due_date) BETWEEN ? AND ?))
    )
    GROUP BY status"
);
if ($stmt_stats_report_pdf) {
    $stmt_stats_report_pdf->bind_param("issssss", $user_id, $start_date_str, $end_date_str, $start_date_str, $end_date_str, $start_date_str, $end_date_str);
    if ($stmt_stats_report_pdf->execute()) {
        $result_stats_pdf = $stmt_stats_report_pdf->get_result();
        while ($row_stat_pdf = $result_stats_pdf->fetch_assoc()) {
            $count_val_pdf = (int)$row_stat_pdf['count'];
            $total_tasks_report += $count_val_pdf;
            if ($row_stat_pdf['status'] == 'Completed') $completed_report = $count_val_pdf;
            elseif ($row_stat_pdf['status'] == 'In Progress') $in_progress_report = $count_val_pdf;
            elseif ($row_stat_pdf['status'] == 'Not Started') $not_started_report = $count_val_pdf;
        }
    }
    $stmt_stats_report_pdf->close();
}


class PDF extends FPDF {
    function Header() {
        // Logo (jika ada)
        // $this->Image('images/logo.png',10,6,30);
        $this->SetFont('Arial','B',15);
        $this->Cell(80); // Pindah ke tengah
        $this->Cell(30,10,'List In - Laporan Tugas',0,0,'C');
        $this->Ln(15); // Line break
    }

    function Footer() {
        $this->SetY(-15); // Posisi 1.5 cm dari bawah
        $this->SetFont('Arial','I',8);
        $this->Cell(0,10,'Halaman '.$this->PageNo().'/{nb}',0,0,'C'); // Nomor halaman
        $this->SetX(-50);
        $this->Cell(0,10, 'Dicetak pada: '.date('d/m/Y H:i'),0,0,'R');
    }

    function ChapterTitle($label) {
        $this->SetFont('Arial','B',12);
        $this->SetFillColor(230,230,230); // Warna abu-abu muda
        $this->Cell(0,8,$label,0,1,'L',true);
        $this->Ln(2);
    }

    function BodyText($text) {
        $this->SetFont('Arial','',10);
        $this->MultiCell(0,6, $text);
        $this->Ln();
    }
    
    function TaskTable($header, $data) {
        // Lebar kolom
        $w = array(70, 30, 30, 60); // Judul, Prioritas, Status, Deadline/Selesai
        // Header
        $this->SetFillColor(200,220,255);
        $this->SetTextColor(0);
        $this->SetDrawColor(128,0,0);
        $this->SetLineWidth(.3);
        $this->SetFont('','B', 9);
        for($i=0;$i<count($header);$i++)
            $this->Cell($w[$i],7,$header[$i],1,0,'C',true);
        $this->Ln();
        // Data
        $this->SetFillColor(224,235,255);
        $this->SetTextColor(0);
        $this->SetFont('','', 8);
        $fill = false;
        foreach($data as $row)
        {
            $this->Cell($w[0],6, substr($row['title'],0,45) . (strlen($row['title'])>45 ? '...':''),'LR',0,'L',$fill); // Batasi panjang judul
            $this->Cell($w[1],6,$row['priority'],'LR',0,'C',$fill);
            $this->Cell($w[2],6,$row['status'],'LR',0,'C',$fill);
            $date_display = ($row['status'] == 'Completed') ? ($row['completion_date_formatted'] ?? 'N/A') : ($row['due_date_formatted'] ?? 'N/A');
            $this->Cell($w[3],6,$date_display,'LR',0,'C',$fill);
            $this->Ln();
            $fill = !$fill;
        }
        // Closing line
        $this->Cell(array_sum($w),0,'','T');
        $this->Ln(5);
    }
}

$pdf = new PDF();
$pdf->AliasNbPages(); // Untuk total halaman {nb}
$pdf->AddPage();
$pdf->SetFont('Arial','',11);

$pdf->Cell(0,8,'Pengguna: ' . htmlspecialchars($username),0,1);
$pdf->Cell(0,8,'Periode Laporan: ' . date("d M Y", strtotime($start_date_str)) . ' - ' . date("d M Y", strtotime($end_date_str)),0,1);
$pdf->Cell(0,8,'Preset Filter: ' . ucfirst(str_replace('_', ' ', $preset)),0,1);
$pdf->Ln(5);

$pdf->ChapterTitle('Ringkasan Status Tugas');
$pdf->BodyText("Total tugas dalam rentang waktu ini: " . $total_tasks_report);
$pdf->BodyText("- Selesai: " . $completed_report . " (" . ($total_tasks_report > 0 ? round(($completed_report/$total_tasks_report)*100) : 0) . "%)");
$pdf->BodyText("- Dikerjakan (In Progress): " . $in_progress_report . " (" . ($total_tasks_report > 0 ? round(($in_progress_report/$total_tasks_report)*100) : 0) . "%)");
$pdf->BodyText("- Belum Mulai: " . $not_started_report . " (" . ($total_tasks_report > 0 ? round(($not_started_report/$total_tasks_report)*100) : 0) . "%)");
$pdf->Ln(5);


if (!empty($tasks_for_report)) {
    $pdf->ChapterTitle('Daftar Tugas');
    $header_tabel = array('Judul Tugas', 'Prioritas', 'Status', 'Deadline/Selesai');
    $pdf->TaskTable($header_tabel, $tasks_for_report);
} else {
    $pdf->BodyText("Tidak ada data tugas untuk ditampilkan dalam rentang waktu ini.");
}

$filename = "Laporan_ListIn_" . date("Ymd", strtotime($start_date_str)) . "_" . date("Ymd", strtotime($end_date_str)) . ".pdf";
$pdf->Output('D', $filename); // D: Download, I: Inline
exit;
?>
```

---
**Langkah 6: Modifikasi CSS (`css/style.css` dan `css/auth.css`)**

**`css/auth.css` (Tambahkan/Modifikasi)**
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

.auth-container .logo-container { margin-bottom: 15px; }
.auth-container .logo-container h1 {
    font-size: 2.5rem; color: #7e47b8; margin:0; font-weight: 600;
    transition: color 0.3s ease;
}

.auth-container h2 {
    color: #333; margin-bottom: 8px; font-size: 1.4rem; font-weight: 500;
    transition: color 0.3s ease;
}
.auth-container p.subtitle {
    color: #555; margin-bottom: 25px; font-size: 0.9rem;
    transition: color 0.3s ease;
}

.form-group {
    margin-bottom: 18px; text-align: left; position: relative;
}
.form-group label {
    display: block; margin-bottom: 6px; font-weight: 500;
    color: #444; font-size: 0.85rem;
    transition: color 0.3s ease;
}
.form-group input[type="text"],
.form-group input[type="email"],
.form-group input[type="password"] {
    width: 100%; padding: 10px 12px; border: 1px solid #ccc;
    background-color: #fff; color: #333; border-radius: 6px;
    font-size: 0.9rem; box-sizing: border-box;
    transition: border-color 0.2s, box-shadow 0.2s, background-color 0.3s, color 0.3s;
}
.form-group input:focus {
    border-color: #7e47b8; outline: none;
    box-shadow: 0 0 0 3px rgba(126, 71, 184, 0.2);
}
.auth-forgot-link {
    font-size: 0.8rem;
    color: #7e47b8;
    text-decoration: none;
}
.auth-forgot-link:hover { text-decoration: underline; }


.btn-submit {
    background-color: #7e47b8; color: white; padding: 11px 20px;
    border: none; border-radius: 6px; cursor: pointer;
    font-size: 0.95rem; font-weight: 500;
    transition: background-color 0.2s ease, transform 0.1s ease;
    display: block; width: 100%; margin-top: 10px;
}
.btn-submit:hover { background-color: #6a3aa2; }
.btn-submit:active { transform: translateY(1px); }

.auth-separator {
    display: flex; align-items: center; text-align: center;
    color: #aaa; margin: 20px 0; font-size: 0.8rem;
}
.auth-separator::before, .auth-separator::after {
    content: ''; flex: 1; border-bottom: 1px solid #ddd;
}
.auth-separator:not(:empty)::before { margin-right: .5em; }
.auth-separator:not(:empty)::after { margin-left: .5em; }


.btn-google-login {
    display: flex; align-items: center; justify-content: center;
    width: 100%; padding: 10px 15px; margin-bottom: 20px;
    background-color: #fff; color: #444;
    border: 1px solid #ddd; border-radius: 6px;
    font-size: 0.9rem; font-weight: 500; text-decoration: none;
    cursor: pointer; transition: background-color 0.2s, box-shadow 0.2s;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}
.btn-google-login:hover {
    background-color: #f8f8f8;
    box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}
.btn-google-login img {
    width: 18px; height: 18px; margin-right: 10px;
}

.auth-link {
    margin-top: 20px; font-size: 0.85rem; color: #444;
    transition: color 0.3s ease;
}
.auth-link a {
    color: #7e47b8; text-decoration: none; font-weight: 500;
    transition: color 0.3s ease;
}
.auth-link a:hover { text-decoration: underline; }

.auth-message {
    padding: 12px 15px;
    margin-bottom: 20px;
    border-radius: 6px;
    font-size: 0.88rem;
    line-height: 1.5;
}
.auth-message.success {
    background-color: #d1e7dd; color: #0f5132; border: 1px solid #badbcc;
}
.auth-message.error {
    background-color: #f8d7da; color: #842029; border: 1px solid #f5c2c7;
}
.auth-message.info {
    background-color: #cff4fc; color: #055160; border: 1px solid #b6effb;
}
.auth-message p { margin: 0 0 5px 0; }
.auth-message p:last-child { margin-bottom: 0; }
.auth-message a { color: inherit; font-weight: bold; text-decoration: underline; }


/* DARK THEME FOR AUTH PAGE */
html.dark-theme-active body.auth-page::before {
    background-color: rgba(0,0,0,0.7);
}
html.dark-theme-active .auth-container {
    background: rgba(30, 30, 30, 0.92);
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.5);
}
html.dark-theme-active .auth-container .logo-container h1 { color: #bb86fc; }
html.dark-theme-active .auth-container h2 { color: #e0e0e0; }
html.dark-theme-active .auth-container p.subtitle { color: #b0b0b0; }
html.dark-theme-active .form-group label { color: #c0c0c0; }
html.dark-theme-active .form-group input[type="text"],
html.dark-theme-active .form-group input[type="email"],
html.dark-theme-active .form-group input[type="password"] {
    background-color: #2c2c2c; border-color: #555; color: #e0e0e0;
}
html.dark-theme-active .form-group input:focus {
    border-color: #bb86fc; box-shadow: 0 0 0 3px rgba(187, 134, 252, 0.3);
    background-color: #333;
}
html.dark-theme-active .auth-forgot-link { color: #bb86fc; }

html.dark-theme-active .btn-submit {
    background-color: #bb86fc; color: #121212;
}
html.dark-theme-active .btn-submit:hover { background-color: #a06fec; }

html.dark-theme-active .auth-separator { color: #777; }
html.dark-theme-active .auth-separator::before, html.dark-theme-active .auth-separator::after { border-bottom-color: #444; }

html.dark-theme-active .btn-google-login {
    background-color: #333; color: #e0e0e0; border-color: #555;
}
html.dark-theme-active .btn-google-login:hover { background-color: #3e3e3e; }

html.dark-theme-active .auth-link { color: #b0b0b0; }
html.dark-theme-active .auth-link a { color: #bb86fc; }

html.dark-theme-active .auth-message.success {
    background-color: rgba(102, 187, 106, 0.15); color: #a5d6a7; border-color: #66bb6a;
}
html.dark-theme-active .auth-message.error {
    background-color: rgba(239, 83, 80, 0.15); color: #ef9a9a; border-color: #ef5350;
}
html.dark-theme-active .auth-message.info {
    background-color: rgba(66, 165, 245, 0.15); color: #90caf9; border-color: #42a5f5;
}
```

**`css/style.css` (Tambahkan/Modifikasi)**
Perlu ditambahkan styling untuk halaman laporan, kalender, dll.
```css
/* css/style.css */
/* ... (Semua CSS yang sudah ada sebelumnya) ... */

/* ==========================================================================
   HALAMAN LAPORAN
   ========================================================================== */
.report-page-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
}
.report-page-header .page-title {
    margin-bottom: 0;
    border-bottom: none;
}
.report-filters {
    margin-bottom: 20px;
}
.report-filters .performance-filter-form {
    padding: 10px 15px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
}
html.dark-theme-active .report-filters .performance-filter-form {
    background: #2c2c2c;
    box-shadow: 0 1px 3px rgba(0,0,0,0.3);
    border: 1px solid #444;
}


.report-grid {
    display: grid;
    grid-template-columns: 1fr; /* Default 1 kolom */
    gap: 20px;
}
@media (min-width: 992px) { /* 2 kolom untuk layar lebih besar */
    .report-grid {
        grid-template-columns: 2fr 1.2fr; /* Kolom kiri lebih besar */
    }
}

.report-grid-left, .report-grid-right {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.widget.report-widget {
    /* Styling umum widget sudah ada, ini untuk penyesuaian jika perlu */
}
.report-widget .widget-content-area.status-progress-container {
    padding-top: 15px; /* Beri ruang sedikit */
}

.report-calendar-widget .widget-content-area {
    padding: 0; /* Hapus padding default jika kalender dan list di dalamnya */
}
.report-calendar-container {
    display: flex;
    flex-direction: column; /* Default untuk mobile */
    gap: 15px;
    padding: 15px; /* Padding untuk container ini */
}
@media (min-width: 768px) { /* Layout berdampingan untuk tablet ke atas */
    .report-calendar-container {
        flex-direction: row;
        align-items: flex-start; /* Kalender dan list mulai dari atas */
    }
}

#reportCalendarInteractive {
    flex: 1 1 auto; /* Biarkan kalender mengambil ruang yang cukup */
    min-width: 280px; /* Agar kalender tidak terlalu kecil */
    background-color: #fff;
    border-radius: 6px;
    padding:10px;
    box-shadow: 0 0 10px rgba(0,0,0,0.05);
}
html.dark-theme-active #reportCalendarInteractive {
    background-color: #252525; /* Warna sedikit beda dari widget */
}
/* Styling Flatpickr di dalam #reportCalendarInteractive (jika perlu override) */
html.dark-theme-active #reportCalendarInteractive .flatpickr-calendar {
    background: #252525;
    border-color: #444;
    box-shadow: 0 3px 13px rgba(255, 255, 255, 0.08);
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-months .flatpickr-month,
html.dark-theme-active #reportCalendarInteractive .flatpickr-weekday,
html.dark-theme-active #reportCalendarInteractive .flatpickr-weekdays {
    color: #e0e0e0;
    fill: #e0e0e0;
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-day {
    color: #bdbdbd;
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-day:hover,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day:focus {
    background: #3e3e3e;
    border-color: #3e3e3e;
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.selected,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.startRange,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.endRange,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.selected:focus,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.startRange:focus,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.endRange:focus,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.selected:hover,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.startRange:hover,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.endRange:hover {
    background: #bb86fc;
    border-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.today {
    border-color: #bb86fc;
    color: #bb86fc;
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.today:hover {
    background: #bb86fc;
    border-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.flatpickr-disabled,
html.dark-theme-active #reportCalendarInteractive .flatpickr-day.flatpickr-disabled:hover {
    color: #555;
}


#reportCalendarTasksList {
    flex: 1 1 300px; /* List tugas mengambil sisa ruang atau minimal 300px */
    max-height: 450px; /* Batasi tinggi agar tidak terlalu panjang */
    overflow-y: auto;
    padding-right: 8px; /* Ruang untuk scrollbar */
}
.task-item-card.report-task-item {
    font-size: 0.8rem; /* Sedikit lebih kecil untuk daftar */
    padding: 10px;
}
.task-item-card.report-task-item .task-details strong { font-size: 0.9rem; }
.task-item-card.report-task-item .task-details .description { font-size: 0.75rem; margin-bottom: 3px; }
.task-item-card.report-task-item .task-details .meta-info { font-size: 0.7rem; }

#reportCalendarTasksList .loading-message,
#reportCalendarTasksList .no-tasks-message {
    text-align: center;
    padding: 20px;
    color: #777;
    font-style: italic;
}
html.dark-theme-active #reportCalendarTasksList .loading-message,
html.dark-theme-active #reportCalendarTasksList .no-tasks-message {
    color: #aaa;
}
#reportCalendarTasksList .error-message {
    color: #e74c3c;
}
html.dark-theme-active #reportCalendarTasksList .error-message {
    color: #ef9a9a;
}

/* Responsive untuk Halaman Laporan */
@media (max-width: 767px) {
    .report-page-header {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
    }
    .report-page-header .btn {
        width: 100%;
    }
    .report-filters .performance-filter-form {
        flex-direction: column;
        align-items: stretch;
    }
    .report-filters .performance-filter-form .filter-preset-buttons,
    .report-filters .performance-filter-form input[type="text"],
    .report-filters .performance-filter-form .filter-action-buttons {
        width: 100%;
        margin-left: 0;
    }
    .report-filters .performance-filter-form .filter-action-buttons {
        margin-top:10px;
        flex-direction: column;
    }
     .report-filters .performance-filter-form .filter-action-buttons .btn {
        width: 100%;
    }
    .report-calendar-container {
        padding: 10px;
    }
    #reportCalendarTasksList {
        max-height: 300px;
    }
}

/* ... (Sisa CSS yang sudah ada sebelumnya) ... */

/* Pastikan .main, .widget, dll. juga memiliki style dark theme yang konsisten */
html.dark-theme-active .main {
    /* Jika belum ada, background main content juga perlu disesuaikan */
    /* background-color: #181818; */ /* contoh */
}

html.dark-theme-active .filters-container,
html.dark-theme-active .performance-filter-container {
    background: #2c2c2c;
    box-shadow: 0 1px 3px rgba(0,0,0,0.3);
    border: 1px solid #444;
}
html.dark-theme-active .filters-container label,
html.dark-theme-active .performance-filter-container label {
    color: #ccc;
}
html.dark-theme-active .filters-container select,
html.dark-theme-active .filters-container input[type="text"],
html.dark-theme-active .performance-filter-container select,
html.dark-theme-active .performance-filter-container input[type="text"] {
    background-color: #373737;
    border-color: #555;
    color: #e0e0e0;
}
html.dark-theme-active .filters-container select:focus,
html.dark-theme-active .filters-container input[type="text"]:focus,
html.dark-theme-active .performance-filter-container select:focus,
html.dark-theme-active .performance-filter-container input[type="text"]:focus {
    border-color: #bb86fc;
    box-shadow: 0 0 0 2.5px rgba(187, 134, 252, 0.3);
    background-color: #3c3c3c;
}
html.dark-theme-active .filter-preset-buttons .btn {
    background-color: #3a3a3a;
    color: #ccc;
    border-color: #555;
}
html.dark-theme-active .filter-preset-buttons .btn.active,
html.dark-theme-active .filter-preset-buttons .btn:hover {
    background-color: #bb86fc;
    color: #121212;
    border-color: #bb86fc;
}
html.dark-theme-active .performance-filter-form .filter-preset-buttons .btn {
    background-color: #3a3a3a;
    color: #ccc;
    border-color: #555;
}
html.dark-theme-active .performance-filter-form .filter-preset-buttons .btn.active,
html.dark-theme-active .performance-filter-form .filter-preset-buttons .btn:hover {
    background-color: #bb86fc;
    color: #121212;
    border-color: #bb86fc;
}
html.dark-theme-active .performance-filter-form label {
    color: #bdbdbd;
}
```

---
**Langkah 7: Modifikasi `includes/footer.php`**
Pastikan Flatpickr untuk `#filterReportDateRange` dan logika kalender laporan ditambahkan.
```php
<?php
// includes/footer.php
// $current_page dari header.php
$is_auth_page_footer = in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php']);
?>
<?php if (!$is_auth_page_footer): ?>
    </div> <!-- Penutup .content -->

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
                // Urutkan notifikasi, yang terbaru di atas
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
                            $now_popup = new DateTime('now', new DateTimeZone('Asia/Jakarta')); // Pastikan timezone
                            $interval_popup = $now_popup->diff($timestamp_popup);
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
                        <div class="notif-message-content"><?php echo $notif_item['message']; ?></div>
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

            function togglePopup(popupElement, iconElement, otherPopupElement) {
                if (otherPopupElement && otherPopupElement.classList.contains('show')) {
                    otherPopupElement.classList.remove('show');
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
                    togglePopup(notificationPopup, bellIcon, calendarPopup); 
                });
            }

            if (clearAllNotificationsBtn && notificationListUl) {
                clearAllNotificationsBtn.addEventListener('click', (e) => {
                    e.stopPropagation(); 
                    if (confirm('Anda yakin ingin menghapus semua notifikasi?')) {
                        // Gunakan path yang benar untuk ajax_clear_all_notifications.php
                        // Jika file ada di root, path 'ajax_clear_all_notifications.php' sudah benar.
                        // Jika file ada di includes/, pathnya 'includes/ajax_clear_all_notifications.php'
                        // Berdasarkan struktur file, sepertinya ada di root.
                        fetch('ajax_clear_all_notifications.php') // Sesuaikan path jika perlu
                            .then(response => response.json())
                            .then(data => {
                                if (data.success) {
                                    notificationListUl.innerHTML = '<li class="no-notifications">Tidak ada notifikasi baru.</li>';
                                    if (bellIcon && bellIcon.classList.contains('has-notif')) {
                                        bellIcon.classList.remove('has-notif'); 
                                    }
                                    // Secara opsional, bisa juga update badge di session server-side jika perlu.
                                    // Tapi karena ini hanya client-side, badge akan hilang sampai refresh/navigasi berikutnya.
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
                const fpPopup = flatpickr(document.getElementById('calendar-container-popup'), { 
                    inline: true, 
                    dateFormat: "d/m/Y", 
                    locale: "id" 
                });
                calendarIcon.addEventListener('click', function (e) { 
                    e.stopPropagation(); 
                    togglePopup(calendarPopup, calendarIcon, notificationPopup); 
                });
            }

            document.addEventListener('click', function (e) { 
                if (notificationPopup && notificationPopup.classList.contains('show') && !notificationPopup.contains(e.target) && e.target !== bellIcon) { 
                    notificationPopup.classList.remove('show'); 
                }
                if (calendarPopup && calendarPopup.classList.contains('show') && !calendarPopup.contains(e.target) && e.target !== calendarIcon) { 
                    calendarPopup.classList.remove('show'); 
                }
            });

            // Initialize Flatpickr untuk semua input tanggal yang relevan
            const dateInputs = document.querySelectorAll('input[type="text"][id$="Date"], input[type="text"][id$="DueDate"], input[type="text"][id$="DateRange"], input[type="text"][id="filterHistoryDateRange"], input[type="text"][id="filterReportDateRange"]');
            dateInputs.forEach(input => {
                let config = { dateFormat: "d/m/Y", locale: "id", allowInput: true };
                if (input.id === 'taskDueDate' || input.id === 'editTaskDueDate') {
                     config.minDate = "today";
                }
                // Untuk range picker
                if (input.id === 'filterHistoryDateRange' || input.id === 'filterPerformanceDateRange' || input.id === 'filterReportDateRange') {
                    config.mode = "range";
                } else if (input.id === 'filterDate') { // Untuk filter single date di manajemen_tugas
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

            // --- Tema Gelap Logic ---
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
            // --- Akhir Tema Gelap Logic ---

            // --- Sidebar Toggle Logic ---
            const appTitleToggle = document.getElementById('appTitleToggle');
            const sidebarElement = document.getElementById('sidebar'); 
            if (appTitleToggle && sidebarElement) {
                function setSidebarState(isHidden) {
                    if (window.innerWidth > 768) { // Hanya berlaku di desktop
                        if (isHidden) bodyElement.classList.add('sidebar-hidden');
                        else bodyElement.classList.remove('sidebar-hidden');
                    } else { // Di mobile, sidebar selalu terlihat (atau di-handle CSS responsif)
                         bodyElement.classList.remove('sidebar-hidden');
                    }
                }
                // Initial state
                const sidebarHiddenStored = localStorage.getItem('sidebarHidden') === 'true';
                setSidebarState(sidebarHiddenStored);

                appTitleToggle.addEventListener('click', () => {
                    if (window.innerWidth > 768) {
                        const isNowHidden = bodyElement.classList.toggle('sidebar-hidden');
                        localStorage.setItem('sidebarHidden', isNowHidden);
                    } else {
                        // Di mobile, klik judul mungkin bisa toggle menu dropdown horizontal jika ada
                        // Untuk sekarang, tidak ada aksi khusus di mobile.
                    }
                });
                window.addEventListener('resize', () => {
                    const sidebarHiddenStoredResize = localStorage.getItem('sidebarHidden') === 'true';
                    setSidebarState(sidebarHiddenStoredResize); // Terapkan ulang state saat resize
                });
            }
            // --- Akhir Sidebar Toggle Logic ---


            // --- Klik Card Tugas untuk Ubah Status (Manajemen) ---
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
                            nextStatus = 'In Progress'; nextStatusTextForUI = 'Dikerjakan'; nextStatusClass = 'status-in-progress';
                        } else if (currentStatus === 'In Progress') {
                            nextStatus = 'Completed'; nextStatusTextForUI = 'Selesai'; nextStatusClass = 'status-completed-history'; 
                        } else { return; }

                        const statusTextElement = card.querySelector('.meta-info .task-status-text');
                        
                        // Optimistic UI Update
                        card.classList.remove(currentStatusClass);
                        card.classList.add(nextStatusClass);
                        card.dataset.currentStatus = nextStatus;
                        if (statusTextElement) statusTextElement.textContent = nextStatusTextForUI;

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
                                if (data.is_completed) { // Jika tugas selesai, animasikan dan hapus
                                    card.style.transition = 'opacity 0.4s ease-out, transform 0.4s ease-out, max-height 0.5s ease-in-out, padding 0.5s ease-in-out, margin 0.5s ease-in-out';
                                    card.style.opacity = '0'; card.style.transform = 'scale(0.9)';
                                    card.style.maxHeight = '0px'; card.style.paddingTop = '0px';
                                    card.style.paddingBottom = '0px'; card.style.marginBottom = '0px';
                                    setTimeout(() => {
                                        card.remove();
                                        if (taskListContainerManajemen.children.length === 0 && !taskListContainerManajemen.querySelector('.no-tasks-message')) {
                                            taskListContainerManajemen.innerHTML = '<p class="no-tasks-message">Tidak ada tugas aktif yang sesuai.</p>';
                                        }
                                    }, 500);
                                }
                            } else { // Rollback jika AJAX gagal
                                card.classList.remove(nextStatusClass); card.classList.add(currentStatusClass);
                                card.dataset.currentStatus = currentStatus;
                                if (statusTextElement) {
                                     const originalStatusText = currentStatus === 'Not Started' ? 'Belum Mulai' : (currentStatus === 'In Progress' ? 'Dikerjakan' : 'Selesai');
                                     statusTextElement.textContent = originalStatusText;
                                }
                                alert('Gagal memperbarui status: ' + (data.message || 'Error tidak diketahui.'));
                            }
                        })
                        .catch(error => { // Rollback jika koneksi error
                            console.error('Error AJAX:', error);
                            card.classList.remove(nextStatusClass); card.classList.add(currentStatusClass);
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
            // --- Akhir Klik Card Tugas ---
        });
    </script>
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
**Langkah 8: File PHP Lainnya (Tidak Ada Perubahan Signifikan atau Sudah Dicakup)**
File seperti `dashboard.php`, `edit_profil.php`, `edit_tugas.php`, `manajemen_tugas.php`, `profil.php`, `riwayat.php`, `tambah_tugas.php`, `logout.php`, `mark_notifications_viewed.php`, `ajax_handler.php`, `ajax_update_task_status.php` dan `includes/task_helper.php` sebagian besar logikanya tetap sama atau perubahannya minor dan sudah diakomodasi oleh modifikasi di `header.php` atau `db.php`.

**ajax_clear_all_notifications.php** (seharusnya `ajax_clear_all_notifications.php`)
```php
<?php
// ajax_clear_all_notifications.php
require_once 'includes/config.php'; // Untuk session_start()

// Hapus semua pesan notifikasi dari session
if (isset($_SESSION['notification_messages'])) {
    unset($_SESSION['notification_messages']);
}
// Juga pastikan badge tidak aktif lagi
$_SESSION['has_unread_notifications_badge'] = false;

header('Content-Type: application/json');
echo json_encode(['success' => true]);
exit();
?>
```
File `clear_notifications.php` bisa dihapus jika fungsinya sama dengan `ajax_clear_all_notifications.php`.

---
**Langkah 9: Tutorial untuk Konfigurasi Eksternal**

Setelah semua kode di atas ditempatkan dengan benar:

1.  **Instalasi Composer & Dependensi:**
    *   Jika Anda belum memiliki Composer, unduh dan instal dari [getcomposer.org](https://getcomposer.org).
    *   Buka terminal/command prompt di direktori root proyek Anda (tempat Anda meletakkan `composer.json`).
    *   Jalankan perintah: `composer install`
    *   Ini akan mengunduh library Google API Client dan FPDF ke dalam folder `vendor/`.

2.  **Konfigurasi Google Cloud Console untuk OAuth 2.0 (Login dengan Google):**
    *   **Buat Proyek Baru:**
        *   Buka [Google Cloud Console](https://console.cloud.google.com/).
        *   Buat proyek baru (misalnya, "ListIn App").
    *   **Aktifkan API:**
        *   Setelah proyek dibuat dan dipilih, pergi ke "APIs & Services" > "Library".
        *   Cari "Google People API" dan aktifkan (Enable). Anda mungkin juga memerlukan "OAuth2 API" jika belum aktif.
    *   **Konfigurasi Layar Persetujuan OAuth (OAuth consent screen):**
        *   Pergi ke "APIs & Services" > "OAuth consent screen".
        *   Pilih "External" dan klik "Create".
        *   Isi nama aplikasi (misalnya, "List In"), email support Anda, logo (opsional).
        *   Di bagian "Authorized domains", tambahkan domain aplikasi Anda (misalnya, jika `SITE_URL` adalah `http://localhost`, Anda mungkin tidak perlu menambahkan apa pun untuk localhost, tetapi untuk domain publik, tambahkan domainnya).
        *   Simpan dan lanjutkan.
        *   **Scopes:** Klik "Add or Remove Scopes". Tambahkan scope `.../auth/userinfo.email` (untuk alamat email) dan `.../auth/userinfo.profile` (untuk info profil dasar). Klik "Update".
        *   Simpan dan lanjutkan.
        *   **Test users:** Tambahkan alamat email Google Anda sebagai test user saat aplikasi masih dalam mode testing.
        *   Kembali ke dashboard OAuth consent screen.
    *   **Buat Kredensial OAuth 2.0:**
        *   Pergi ke "APIs & Services" > "Credentials".
        *   Klik "+ CREATE CREDENTIALS" > "OAuth client ID".
        *   Pilih "Web application" sebagai Application type.
        *   Beri nama (misalnya, "ListIn Web Client").
        *   Di bagian "Authorized JavaScript origins", tambahkan URL dasar aplikasi Anda (misalnya, `http://localhost` atau `http://nama_folder_proyek_anda` jika di subfolder localhost).
        *   Di bagian "Authorized redirect URIs", tambahkan URI yang ada di `includes/config.php` untuk `GOOGLE_REDIRECT_URI`. Contoh: `http://localhost/nama_folder_proyek_anda/google_auth_handler.php`.
        *   Klik "Create".
    *   **Salin Client ID dan Client Secret:**
        *   Setelah dibuat, Anda akan melihat "Client ID" dan "Client Secret". Salin nilai-nilai ini.
        *   Buka file `includes/config.php` di proyek Anda dan masukkan Client ID dan Client Secret yang baru Anda dapatkan ke konstanta `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET`.
        *   Juga pastikan `SITE_URL` dan `GOOGLE_REDIRECT_URI` di `includes/config.php` sudah benar sesuai dengan setup Anda.

3.  **Konfigurasi Email untuk Lupa Kata Sandi (Simulasi):**
    *   Kode yang saya berikan untuk `forgot_password.php` **mensimulasikan** pengiriman email dengan menampilkan link reset langsung di halaman.
    *   **Untuk pengiriman email nyata:**
        *   Anda memerlukan server SMTP (misalnya, dari Gmail, SendGrid, Mailgun, atau hosting Anda).
        *   Gunakan library PHP seperti PHPMailer:
            *   Tambahkan ke `composer.json`: `"phpmailer/phpmailer": "^6.5"` lalu jalankan `composer update`.
            *   Modifikasi bagian pengiriman email di `forgot_password.php` untuk menggunakan PHPMailer dengan konfigurasi SMTP Anda.
            *   ```php
              // Contoh dengan PHPMailer (instal dulu via composer)
              use PHPMailer\PHPMailer\PHPMailer;
              use PHPMailer\PHPMailer\SMTP;
              use PHPMailer\PHPMailer\Exception;

              // require '../vendor/autoload.php'; // Jika belum di autoload utama

              $mail = new PHPMailer(true);
              try {
                  //Server settings
                  // $mail->SMTPDebug = SMTP::DEBUG_SERVER; // Enable verbose debug output
                  $mail->isSMTP();
                  $mail->Host       = 'smtp.example.com'; // Ganti dengan host SMTP Anda
                  $mail->SMTPAuth   = true;
                  $mail->Username   = 'user@example.com'; // Ganti username SMTP
                  $mail->Password   = 'password_smtp';    // Ganti password SMTP
                  $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS; // Atau ENCRYPTION_SMTPS
                  $mail->Port       = 587; // Atau 465 untuk SMTPS

                  //Recipients
                  $mail->setFrom(APP_EMAIL_ADDRESS, APP_EMAIL_NAME);
                  $mail->addAddress($email, $user['username']);

                  // Content
                  $mail->isHTML(false); // Kirim sebagai plain text
                  $mail->Subject = $email_subject;
                  $mail->Body    = $email_body;

                  $mail->send();
                  $success_message = "Link reset kata sandi telah dikirim ke email Anda.";
              } catch (Exception $e) {
                  $errors[] = "Gagal mengirim email reset. Mailer Error: {$mail->ErrorInfo}";
                  // Log error: error_log("Mailer Error: " . $mail->ErrorInfo);
                  $conn->query("DELETE FROM password_resets WHERE token = '$token'");
              }
              ```
        *   **Email Aplikasi:** Gunakan `listinproject@gmail.com` dan password `listin12312` **HANYA** jika Anda membuat akun Gmail tersebut khusus untuk ini dan mengaktifkan "Akses aplikasi kurang aman" atau membuat "App Password" di pengaturan keamanan akun Google tersebut. **Sangat tidak disarankan menggunakan akun email pribadi utama Anda untuk ini.**

4.  **Struktur Folder dan Izin:**
    *   Pastikan folder `uploads/profile_pictures/` ada dan **dapat ditulis (writable)** oleh server web Anda (misalnya, izin `755` atau `777` untuk development, tetapi perhatikan keamanan untuk produksi).

5.  **Database:**
    *   Pastikan Anda sudah membuat database `db_listin` dan menjalankan file `db_listin.sql` yang telah dimodifikasi untuk membuat tabel dengan struktur yang benar.

6.  **URL Aplikasi:**
    *   Sesuaikan `SITE_URL` di `includes/config.php` dengan URL root aplikasi Anda. Jika Anda menjalankan di `http://localhost/listinapp/`, maka `SITE_URL` harus `http://localhost/listinapp`.

Dengan mengikuti langkah-langkah ini dan kode yang diberikan, Anda seharusnya bisa mengimplementasikan fitur-fitur yang diminta. Pastikan untuk menguji setiap fitur secara menyeluruh.
