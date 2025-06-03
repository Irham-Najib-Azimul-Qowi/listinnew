Baik, ini adalah permintaan yang cukup komprehensif! Saya akan mencoba menyusunnya selengkap mungkin. Ini akan menjadi proyek yang cukup besar.

**Penting:**
1.  **Backup:** Sebelum menerapkan perubahan besar ini, **SELALU BUAT BACKUP** dari seluruh proyek dan database Anda.
2.  **Composer:** Kita akan menggunakan Composer untuk mengelola pustaka PHP seperti PHPMailer dan Google API Client. Jika Anda belum menginstalnya, silakan instal dari [getcomposer.org](https://getcomposer.org).
3.  **Email & Google Setup:** Saya akan memberikan kode dengan placeholder atau contoh, tetapi Anda perlu mengkonfigurasi akun Google Cloud Console untuk OAuth 2.0 (Login dengan Google) dan SMTP server untuk pengiriman email (bisa Gmail, SendGrid, dll.). Tutorial untuk ini akan ada di akhir.

---

**Struktur File Proyek (Tambahan dan Perubahan):**

```
.
├── css/
│   ├── auth.css
│   └── style.css
├── images/
│   ├── auth-bg.jpg
│   └── placeholder-profile.png
│   └── listin-logo.png  (TAMBAHAN - untuk PDF dan email)
├── includes/
│   ├── db.php
│   ├── footer.php
│   ├── header.php
│   ├── sidebar.php
│   └── task_helper.php
├── uploads/
│   └── profile_pictures/ (Folder untuk foto profil pengguna)
├── vendor/  (Dibuat oleh Composer)
│   └── ... (PHPMailer, Google API Client, dll.)
├── ajax_clear_all_notifications.php (Harusnya ajax_clear_all_notifications.php)
├── ajax_get_tasks_for_date.php (BARU - Untuk kalender di laporan)
├── ajax_handler.php
├── ajax_update_task_status.php
├── clear_notifications.php (Duplikat? Mungkin maksudnya ajax_clear_all_notifications.php)
├── config/
│   └── config.php (BARU - Untuk konfigurasi global)
├── composer.json (BARU - Untuk dependensi PHP)
├── dashboard.php
├── db_listin.sql (Struktur DB, akan diupdate)
├── edit_profil.php
├── edit_tugas.php
├── forgot_password.php (BARU)
├── generate_pdf_report.php (BARU)
├── google_login_callback.php (BARU)
├── laporan.php (BARU)
├── login.php
├── logout.php
├── manajemen_tugas.php
├── mark_notifications_viewed.php
├── profil.php
├── register.php
├── reset_password.php (BARU)
├── riwayat.php
├── send_deadline_emails.php (BARU - Untuk cron job)
├── tambah_tugas.php
└── ubah_password.php
```

---
**Langkah 1: Persiapan & Konfigurasi Awal**

1.  **Buat `composer.json`** di root proyek:
    ```json
    {
        "name": "yourname/listin-project",
        "description": "Task management application",
        "require": {
            "phpmailer/phpmailer": "^6.8",
            "google/apiclient": "^2.15.0",
            "tecnickcom/tcpdf": "^6.6"
        },
        "authors": [
            {
                "name": "Your Name",
                "email": "your.email@example.com"
            }
        ]
    }
    ```
2.  Jalankan `composer install` di terminal pada direktori root proyek Anda. Ini akan membuat folder `vendor`.
3.  **Buat folder `config`** dan di dalamnya file `config.php`:

    ```php
    <?php
    // config/config.php

    // URL Aplikasi Anda (PENTING untuk redirect Google & link email)
    define('APP_URL', 'http://localhost/nama_folder_proyek_anda'); // Ganti sesuai URL Anda

    // Konfigurasi Database
    define('DB_HOST', '127.0.0.1');
    define('DB_USER', 'root');
    define('DB_PASS', '');
    define('DB_NAME', 'db_listin');

    // Konfigurasi Google OAuth 2.0
    define('GOOGLE_CLIENT_ID', 'MASUKKAN_CLIENT_ID_ANDA_DISINI');
    define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_CLIENT_SECRET_ANDA_DISINI');
    define('GOOGLE_REDIRECT_URI', APP_URL . '/google_login_callback.php');

    // Konfigurasi PHPMailer (Gunakan akun listinproject@gmail.com)
    define('MAIL_HOST', 'smtp.gmail.com');
    define('MAIL_USERNAME', 'listinproject@gmail.com');
    define('MAIL_PASSWORD', 'listin12312'); // Hati-hati! Sebaiknya gunakan App Password jika 2FA aktif
    define('MAIL_PORT', 587); // Atau 465 untuk SSL
    define('MAIL_ENCRYPTION', 'tls'); // Atau 'ssl'
    define('MAIL_FROM_ADDRESS', 'listinproject@gmail.com');
    define('MAIL_FROM_NAME', 'List In Notifikasi');

    // Path ke logo untuk PDF dan Email
    define('APP_LOGO_PATH', dirname(__FILE__) . '/../images/listin-logo.png'); // Sesuaikan jika perlu

    // Set timezone
    date_default_timezone_set('Asia/Jakarta');
    ?>
    ```
    *   *Catatan:* `listinproject@gmail.com` dan `listin12312` adalah contoh. Anda perlu memastikan akun ini bisa mengirim email via SMTP (mungkin perlu "Less Secure App Access" di Gmail atau App Password).

4.  **Buat `images/listin-logo.png`**: Siapkan file gambar logo untuk aplikasi Anda.

---
**Langkah 2: Update Struktur Database (`db_listin.sql`)**

Tambahkan kolom untuk Google ID, token reset password, dan timestamp untuk notifikasi email deadline.

```sql
-- db_listin.sql

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
CREATE DATABASE IF NOT EXISTS `db_listin` /*!40100 DEFAULT CHARACTER SET latin1 */;
USE `db_listin`;

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
  `last_deadline_email_sent` timestamp NULL DEFAULT NULL COMMENT 'Untuk notif H-1 via email',
  `last_overdue_email_sent` timestamp NULL DEFAULT NULL COMMENT 'Untuk notif overdue via email',
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  KEY `status` (`status`),
  KEY `due_date` (`due_date`),
  CONSTRAINT `tasks_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Dumping structure for table db_listin.users
CREATE TABLE IF NOT EXISTS `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `google_id` varchar(255) DEFAULT NULL,
  `username` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) DEFAULT NULL COMMENT 'Bisa NULL jika login via Google & belum set password manual',
  `profile_image` varchar(255) DEFAULT 'images/placeholder-profile.png',
  `reset_token` varchar(255) DEFAULT NULL,
  `reset_token_expiry` timestamp NULL DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`),
  UNIQUE KEY `google_id` (`google_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

/*!40101 SET SQL_MODE=IFNULL(@OLD_SQL_MODE, '') */;
/*!40014 SET FOREIGN_KEY_CHECKS=IFNULL(@OLD_FOREIGN_KEY_CHECKS, 1) */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40111 SET SQL_NOTES=IFNULL(@OLD_SQL_NOTES, 1) */;
```

Jalankan SQL ini di phpMyAdmin atau tool database Anda untuk memperbarui struktur tabel.

---
**Langkah 3: Modifikasi File PHP yang Ada & Penambahan File Baru**

Ini bagian terpanjang. Saya akan berikan kode lengkap untuk setiap file.

**`includes/db.php`**
```php
<?php
// includes/db.php
require_once dirname(__FILE__) . '/../config/config.php'; // Muat konfigurasi

// Buat Koneksi
$conn = new mysqli(DB_HOST, DB_USER, DB_PASS, DB_NAME);

// Cek Koneksi
if ($conn->connect_error) {
    // Jangan tampilkan error detail di produksi
    error_log("Koneksi Gagal: " . $conn->connect_error);
    die("Terjadi masalah koneksi ke database. Silakan coba lagi nanti.");
}

// Mulai session di sini agar tersedia di semua file yang meng-include db.php
if (session_status() == PHP_SESSION_NONE) {
    session_start();
}
?>
```

**`includes/header.php`** (Modifikasi signifikan untuk notifikasi email & Google Client)
```php
<?php
require_once 'db.php';
require_once dirname(__FILE__) . '/../vendor/autoload.php'; // Untuk Google API Client & PHPMailer

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

$current_page = basename($_SERVER['SCRIPT_NAME']);
$current_user_id = $_SESSION['user_id'] ?? null;
$current_user_profile_image = 'images/placeholder-profile.png'; // Default

if ($current_user_id && isset($conn)) {
    $stmt_user_header = $conn->prepare("SELECT username, email, profile_image FROM users WHERE id = ?");
    if ($stmt_user_header) {
        $stmt_user_header->bind_param("i", $current_user_id);
        $stmt_user_header->execute();
        $result_user_header = $stmt_user_header->get_result();
        if ($user_data_header = $result_user_header->fetch_assoc()) {
            $current_user_profile_image = (!empty($user_data_header['profile_image']) && (filter_var($user_data_header['profile_image'], FILTER_VALIDATE_URL) || file_exists($user_data_header['profile_image'])))
                                          ? $user_data_header['profile_image']
                                          : 'images/placeholder-profile.png';
            $_SESSION['username'] = $user_data_header['username']; // Pastikan username di session update
            $_SESSION['email'] = $user_data_header['email']; // Simpan email user di session
        }
        $stmt_user_header->close();
    }
}


function add_notification($message, $type = 'info') {
    if (!isset($_SESSION['notification_messages'])) {
        $_SESSION['notification_messages'] = [];
    }
    $new_notification = ['message' => $message, 'type' => $type, 'time' => time()];
    
    $is_duplicate_notif = false;
    // ... (logika duplikasi notifikasi session bisa tetap atau disederhanakan)
    if ($type === 'deadline_soon' || $type === 'overdue_task') {
        // (logika cek duplikat bisa disesuaikan atau dihapus jika email lebih utama)
    }

    if (!$is_duplicate_notif) {
        $_SESSION['notification_messages'][] = $new_notification;
        if (count($_SESSION['notification_messages']) > 10) {
            array_shift($_SESSION['notification_messages']);
        }
        $_SESSION['has_unread_notifications_badge'] = true;
    }
}

// --- Notifikasi Session untuk Deadline H-1 dan Terlewat ---
// Notifikasi ini bersifat UI/session, pengiriman email dilakukan oleh script cron terpisah.
// Cek Tugas Deadline H-1 (Notifikasi Session)
if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php'])) {
    $tomorrow_date = date('Y-m-d', strtotime('+1 day'));
    $today_for_notif_check_h1 = date('Y-m-d');
    $notif_key_deadline_h1_session = 'deadline_h1_session_notif_sent_' . $today_for_notif_check_h1 . '_uid' . $current_user_id;

    if (!isset($_SESSION[$notif_key_deadline_h1_session])) {
        $stmt_deadline_check_h1_sess = $conn->prepare("SELECT title FROM tasks WHERE user_id = ? AND due_date = ? AND status != 'Completed'");
        if ($stmt_deadline_check_h1_sess) {
            $stmt_deadline_check_h1_sess->bind_param("is", $current_user_id, $tomorrow_date);
            $stmt_deadline_check_h1_sess->execute();
            $result_deadline_tasks_h1_sess = $stmt_deadline_check_h1_sess->get_result();
            $tasks_deadline_tomorrow_sess = [];
            while ($task_for_deadline_notif_h1_sess = $result_deadline_tasks_h1_sess->fetch_assoc()) {
                $tasks_deadline_tomorrow_sess[] = htmlspecialchars($task_for_deadline_notif_h1_sess['title']);
            }
            $stmt_deadline_check_h1_sess->close();

            if (!empty($tasks_deadline_tomorrow_sess)) {
                $task_list_str_h1_sess = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_deadline_tomorrow_sess));
                $message_plural_h1_sess = count($tasks_deadline_tomorrow_sess) > 1 ? "Tugas-tugas" : "Tugas";
                $deadline_message_h1_sess = "<span class='message-content'><strong>PERHATIAN (UI):</strong> $message_plural_h1_sess $task_list_str_h1_sess akan jatuh tempo besok!</span>";
                add_notification($deadline_message_h1_sess, "deadline_soon");
                $_SESSION[$notif_key_deadline_h1_session] = true;
            }
        }
    }
}

// Cek Tugas Terlewat DL (Notifikasi Session)
if ($current_user_id && isset($conn) && !in_array($current_page, ['login.php', 'register.php'])) {
    $today_for_overdue_check_sess = date('Y-m-d');
    $notif_key_overdue_session = 'overdue_session_notif_sent_' . $today_for_overdue_check_sess . '_uid' . $current_user_id;

    if (!isset($_SESSION[$notif_key_overdue_session])) {
        $stmt_overdue_check_sess = $conn->prepare(
            "SELECT id, title FROM tasks
             WHERE user_id = ? AND due_date < CURDATE() AND status != 'Completed'
             AND (last_overdue_notif_sent IS NULL OR DATE(last_overdue_notif_sent) < CURDATE())" // Cek flag notif session
        );
        if ($stmt_overdue_check_sess) {
            $stmt_overdue_check_sess->bind_param("i", $current_user_id);
            $stmt_overdue_check_sess->execute();
            $result_overdue_tasks_sess = $stmt_overdue_check_sess->get_result();
            $tasks_overdue_sess = [];
            while ($task_overdue_s = $result_overdue_tasks_sess->fetch_assoc()) {
                $tasks_overdue_sess[] = htmlspecialchars($task_overdue_s['title']);
            }
            $stmt_overdue_check_sess->close();

            if (!empty($tasks_overdue_sess)) {
                $task_list_str_overdue_sess = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_overdue_sess));
                $message_plural_overdue_sess = count($tasks_overdue_sess) > 1 ? "Tugas-tugas" : "Tugas";
                $overdue_message_sess = "<span class='message-content'><strong>TERLEWAT (UI):</strong> $message_plural_overdue_sess $task_list_str_overdue_sess telah melewati batas waktu!</span>";
                add_notification($overdue_message_sess, "deadline_soon"); // Tipe "deadline_soon" untuk styling merah
                // Update flag session setelah notifikasi UI ditampilkan
                // Tidak perlu update kolom last_overdue_notif_sent di sini, itu untuk notifikasi email via cron
                $_SESSION[$notif_key_overdue_session] = true;
            }
        }
    }
}
// --- Akhir Notifikasi Session ---


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

// Google Client Initialization
$google_client = new Google_Client();
$google_client->setClientId(GOOGLE_CLIENT_ID);
$google_client->setClientSecret(GOOGLE_CLIENT_SECRET);
$google_client->setRedirectUri(GOOGLE_REDIRECT_URI);
$google_client->addScope("email");
$google_client->addScope("profile");

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
      })();
    </script>
    <link rel="stylesheet" href="css/style.css?v=<?php echo time(); ?>">
    <?php if (in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])): ?>
        <link rel="stylesheet" href="css/auth.css?v=<?php echo time(); ?>">
    <?php endif; ?>
    <title><?php echo isset($page_title) ? htmlspecialchars($page_title) . ' - ' : ''; ?>List In</title>
</head>
<body class="<?php echo in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php']) ? 'auth-page' : ''; ?>">
    <?php if (in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])): ?>
        <script> document.documentElement.classList.add('auth-html'); </script>
    <?php endif; ?>

    <?php if (!in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])): ?>
    <header class="header">
        <div class="header-left">
            <a href="profil.php">
                <img src="<?php echo htmlspecialchars($current_user_profile_image); ?>?t=<?php echo time();?>" alt="Foto Profil" class="header-profile-pic" id="headerProfilePic">
            </a>
            <h2 class="app-title-toggle" id="appTitleToggle" title="Toggle Sidebar">List In</h2>
        </div>
        <?php
        $hide_search_bar_pages = ['profil.php', 'edit_profil.php', 'ubah_password.php', 'tambah_tugas.php', 'edit_tugas.php', 'dashboard.php', 'laporan.php'];
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

**`includes/sidebar.php`** (Tambah link Laporan)
```php
<?php
// $current_page sudah didefinisikan di header.php
?>
<?php if (!in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])): ?>
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

**`includes/footer.php`** (Penyesuaian untuk halaman auth yang baru)
```php
<?php
// $current_page dari header.php
?>
<?php if (!in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])): ?>
    </div> <!-- Penutup div.content -->

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
                // Sortir notifikasi, terbaru di atas
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
                        <?php echo $notif_item['message']; // Pesan sudah HTML dari header.php ?> 
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
        // Variabel global untuk halaman laporan jika diperlukan
        var tasksForCalendar = {}; // Untuk menyimpan tugas yang diambil via AJAX

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
                        fetch('mark_notifications_viewed.php') // AJAX untuk menandai notifikasi UI telah dilihat
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
                    if (confirm('Anda yakin ingin menghapus semua notifikasi ini?')) {
                        fetch('ajax_clear_all_notifications.php') // AJAX untuk menghapus notifikasi dari session
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
            
            const calendarContainerPopup = document.getElementById('calendar-container-popup');
            if (calendarIcon && calendarPopup && calendarContainerPopup) { 
                 flatpickr(calendarContainerPopup, { inline: true, dateFormat: "d/m/Y", locale: "id" });
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

            // Inisialisasi Flatpickr untuk input tanggal
            const dateInputs = document.querySelectorAll(
                'input[type="text"][id$="Date"], input[type="text"][id$="DueDate"], ' +
                'input[type="text"][id$="DateRange"], input[type="text"][id="filterHistoryDateRange"], ' +
                'input[type="text"][id="reportDateRange"]' // Tambahan untuk laporan
            );
            dateInputs.forEach(input => {
                let config = { dateFormat: "d/m/Y", locale: "id", allowInput: true };
                if (input.id === 'taskDueDate' || input.id === 'editTaskDueDate') {
                     config.minDate = "today";
                }
                if (input.id === 'filterHistoryDateRange' || input.id === 'filterPerformanceDateRange' || input.id === 'reportDateRange') {
                    config.mode = "range";
                } else if (input.id === 'filterDate') { // Untuk filter di manajemen_tugas
                    config.mode = "single";
                }
                flatpickr(input, config);
            });

            // Konfirmasi hapus
            const deleteButtons = document.querySelectorAll('a.delete-btn');
            deleteButtons.forEach(button => {
                button.addEventListener('click', function(event) {
                    if (!confirm('Anda yakin ingin menghapus item ini?')) {
                        event.preventDefault();
                    }
                });
            });

            // Tema Gelap Logic (dari kode Anda, sedikit penyesuaian jika perlu)
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

            // Sidebar Toggle Logic (dari kode Anda)
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

            // Klik Card Tugas untuk Ubah Status (Manajemen - dari kode Anda)
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
                        
                        card.classList.remove(currentStatusClass); card.classList.add(nextStatusClass);
                        card.dataset.currentStatus = nextStatus; 
                        if (statusTextElement) { statusTextElement.textContent = nextStatusTextForUI; }

                        fetch('ajax_update_task_status.php', {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/x-www-form-urlencoded', },
                            body: `task_id=${taskId}&new_status=${nextStatus}`
                        })
                        .then(response => response.json())
                        .then(data => {
                            if (data.success) {
                                if (statusTextElement && data.new_status_text) { statusTextElement.textContent = data.new_status_text; }
                                if (data.is_completed) {
                                    card.style.transition = 'opacity 0.4s ease-out, transform 0.4s ease-out, max-height 0.5s ease-in-out, padding 0.5s ease-in-out, margin 0.5s ease-in-out';
                                    card.style.opacity = '0'; card.style.transform = 'scale(0.9)';
                                    card.style.maxHeight = '0px'; card.style.paddingTop = '0px'; card.style.paddingBottom = '0px'; card.style.marginBottom = '0px';
                                    setTimeout(() => {
                                        card.remove();
                                        if (taskListContainerManajemen.children.length === 0 && !taskListContainerManajemen.querySelector('.no-tasks-message')) {
                                            taskListContainerManajemen.innerHTML = '<p class="no-tasks-message">Tidak ada tugas aktif yang sesuai.</p>';
                                        }
                                    }, 500);
                                }
                            } else { // Rollback
                                card.classList.remove(nextStatusClass); card.classList.add(currentStatusClass);
                                card.dataset.currentStatus = currentStatus;
                                if (statusTextElement) {
                                     const originalStatusText = currentStatus === 'Not Started' ? 'Belum Mulai' : (currentStatus === 'In Progress' ? 'Dikerjakan' : 'Selesai');
                                     statusTextElement.textContent = originalStatusText;
                                }
                                alert('Gagal memperbarui status: ' + (data.message || 'Error tidak diketahui.'));
                            }
                        })
                        .catch(error => { // Rollback
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

             // Preview gambar profil saat edit
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

        });
    </script>
<?php else: // Untuk halaman auth, tidak ada skrip JS kompleks ini ?>
    <script>
        // Skrip minimal jika diperlukan untuk halaman auth
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

**`login.php`** (Modifikasi untuk Google Login & Lupa Password)
```php
<?php
$page_title = "Masuk";
require_once 'includes/header.php'; // header.php sudah mengandung db.php dan session_start()

$errors = [];

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

// Link login Google
$google_login_url = $google_client->createAuthUrl();

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if (isset($_POST['login_manual'])) { // Form login manual
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
}
?>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Selamat Datang Kembali!</h2>
    <p class="subtitle">Masuk untuk melanjutkan dan mengatur tugas Anda.</p>

    <?php if (isset($_SESSION['success_message'])): ?>
        <div style="background-color: #d4edda; color: #155724; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
            <?php echo $_SESSION['success_message']; ?>
        </div>
        <?php unset($_SESSION['success_message']); ?>
    <?php endif; ?>
    <?php if (isset($_SESSION['error_message'])): // Untuk error dari reset password ?>
        <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
            <?php echo $_SESSION['error_message']; ?>
        </div>
        <?php unset($_SESSION['error_message']); ?>
    <?php endif; ?>

    <?php if (!empty($errors)): ?>
        <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
            <?php foreach ($errors as $error): ?>
                <p><?php echo $error; ?></p>
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
        <div style="text-align: right; margin-bottom: 15px; font-size: 0.8rem;">
            <a href="forgot_password.php" style="color: #7e47b8; text-decoration: none;">Lupa Kata Sandi?</a>
        </div>
        <button type="submit" name="login_manual" class="btn-submit">Masuk Akun</button>
    </form>

    <div style="margin-top: 20px; text-align: center; color: #555;">
        Atau masuk dengan
    </div>
    <a href="<?php echo htmlspecialchars($google_login_url); ?>" class="btn-submit" style="background-color: #DB4437; margin-top:10px; display:flex; align-items:center; justify-content:center;">
        <i class="fab fa-google" style="margin-right: 8px;"></i> Google
    </a>
    
    <p class="auth-link">Belum punya akun? <a href="register.php">Daftar sekarang</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**`google_login_callback.php` (BARU)**
```php
<?php
require_once 'includes/header.php'; // Untuk $google_client, $conn, session_start()

if (isset($_GET['code'])) {
    $token = $google_client->fetchAccessTokenWithAuthCode($_GET['code']);
    if (isset($token['error'])) {
        // Redirect ke login dengan pesan error
        $_SESSION['error_message'] = 'Gagal otentikasi dengan Google: ' . $token['error_description'];
        header('Location: login.php');
        exit();
    }
    $google_client->setAccessToken($token['access_token']);

    $google_oauth = new Google_Service_Oauth2($google_client);
    $google_account_info = $google_oauth->userinfo->get();
    
    $google_id = $google_account_info->id;
    $email = $google_account_info->email;
    $name = $google_account_info->name;
    $picture = $google_account_info->picture;

    // Cek apakah user dengan google_id ini sudah ada
    $stmt = $conn->prepare("SELECT id, username, profile_image FROM users WHERE google_id = ?");
    $stmt->bind_param("s", $google_id);
    $stmt->execute();
    $result = $stmt->get_result();
    $user = $result->fetch_assoc();
    $stmt->close();

    if ($user) { // User ditemukan dengan google_id
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        // Update foto profil jika dari Google lebih baru atau user belum punya
        if ($picture && ($user['profile_image'] == 'images/placeholder-profile.png' || $user['profile_image'] != $picture)) {
            $stmt_update_pic = $conn->prepare("UPDATE users SET profile_image = ? WHERE id = ?");
            $stmt_update_pic->bind_param("si", $picture, $user['id']);
            $stmt_update_pic->execute();
            $stmt_update_pic->close();
            $_SESSION['profile_image'] = $picture;
        } else {
            $_SESSION['profile_image'] = $user['profile_image'];
        }
    } else {
        // User dengan google_id tidak ada, cek berdasarkan email
        $stmt = $conn->prepare("SELECT id, username, profile_image FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();
        $user_by_email = $result->fetch_assoc();
        $stmt->close();

        if ($user_by_email) { // User dengan email ini ada, link-kan google_id
            $stmt_link = $conn->prepare("UPDATE users SET google_id = ?, profile_image = ? WHERE id = ?");
            $new_profile_image = ($user_by_email['profile_image'] == 'images/placeholder-profile.png' && $picture) ? $picture : $user_by_email['profile_image'];
            if ($picture && $user_by_email['profile_image'] != $picture) $new_profile_image = $picture;

            $stmt_link->bind_param("ssi", $google_id, $new_profile_image, $user_by_email['id']);
            $stmt_link->execute();
            $stmt_link->close();

            $_SESSION['user_id'] = $user_by_email['id'];
            $_SESSION['username'] = $user_by_email['username'];
            $_SESSION['profile_image'] = $new_profile_image;

        } else { // User baru, daftarkan
            // Password bisa dikosongkan atau di-generate random (tidak bisa login manual sampai diset)
            $stmt_insert = $conn->prepare("INSERT INTO users (google_id, username, email, profile_image, password) VALUES (?, ?, ?, ?, NULL)");
            $stmt_insert->bind_param("ssss", $google_id, $name, $email, $picture);
            $stmt_insert->execute();
            $new_user_id = $stmt_insert->insert_id;
            $stmt_insert->close();

            $_SESSION['user_id'] = $new_user_id;
            $_SESSION['username'] = $name;
            $_SESSION['profile_image'] = $picture;
        }
    }
    header('Location: dashboard.php');
    exit();

} else {
    header('Location: login.php');
    exit();
}
?>
```

**`forgot_password.php` (BARU)**
```php
<?php
$page_title = "Lupa Kata Sandi";
require_once 'includes/header.php'; // Untuk $conn, PHPMailer, config

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;
use PHPMailer\PHPMailer\Exception;

$errors = [];
$success_message = '';

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $email = trim($_POST['email']);

    if (empty($email) || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Format email tidak valid atau kosong.";
    } else {
        $stmt = $conn->prepare("SELECT id, username FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();
        $user = $result->fetch_assoc();
        $stmt->close();

        if ($user) {
            $token = bin2hex(random_bytes(50)); // Token unik
            $expiry_time = date("Y-m-d H:i:s", time() + 3600); // Token berlaku 1 jam

            $stmt_update = $conn->prepare("UPDATE users SET reset_token = ?, reset_token_expiry = ? WHERE id = ?");
            $stmt_update->bind_param("ssi", $token, $expiry_time, $user['id']);
            
            if ($stmt_update->execute()) {
                $reset_link = APP_URL . "/reset_password.php?token=" . $token;
                
                $mail = new PHPMailer(true);
                try {
                    //Server settings
                    // $mail->SMTPDebug = SMTP::DEBUG_SERVER; // Enable verbose debug output untuk testing
                    $mail->isSMTP();
                    $mail->Host       = MAIL_HOST;
                    $mail->SMTPAuth   = true;
                    $mail->Username   = MAIL_USERNAME;
                    $mail->Password   = MAIL_PASSWORD;
                    $mail->SMTPSecure = MAIL_ENCRYPTION;
                    $mail->Port       = MAIL_PORT;

                    //Recipients
                    $mail->setFrom(MAIL_FROM_ADDRESS, MAIL_FROM_NAME);
                    $mail->addAddress($email, $user['username']);

                    //Content
                    $mail->isHTML(true);
                    $mail->Subject = 'Reset Kata Sandi Akun List In Anda';
                    $mail->Body    = "
                        <div style='font-family: Arial, sans-serif; line-height: 1.6;'>
                            <div style='max-width: 600px; margin: 20px auto; padding: 20px; border: 1px solid #ddd; border-radius: 5px;'>
                                <div style='text-align: center; margin-bottom: 20px;'>
                                    <img src='cid:logo_listin' alt='List In Logo' style='max-height: 50px;'>
                                    <h2 style='color: #7e47b8;'>List In - Permintaan Reset Kata Sandi</h2>
                                </div>
                                <p>Halo " . htmlspecialchars($user['username']) . ",</p>
                                <p>Kami menerima permintaan untuk mereset kata sandi akun List In Anda. Jika Anda tidak melakukan permintaan ini, abaikan email ini.</p>
                                <p>Untuk mereset kata sandi Anda, silakan klik tautan di bawah ini:</p>
                                <p style='text-align: center; margin: 20px 0;'>
                                    <a href='" . $reset_link . "' style='background-color: #7e47b8; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; display: inline-block;'>Reset Kata Sandi Saya</a>
                                </p>
                                <p>Tautan ini akan kedaluwarsa dalam 1 jam. Jika tautan tidak berfungsi, salin dan tempel URL berikut di browser Anda:</p>
                                <p><a href='" . $reset_link . "'>" . $reset_link . "</a></p>
                                <hr style='border: none; border-top: 1px solid #eee; margin: 20px 0;'>
                                <p style='font-size: 0.9em; color: #777; text-align: center;'>Email ini dikirim secara otomatis. Mohon untuk tidak membalas email ini.<br>
                                &copy; " . date("Y") . " List In. Hak Cipta Dilindungi.</p>
                            </div>
                        </div>";
                    
                    // Embed logo (pastikan path APP_LOGO_PATH di config.php benar)
                    if (defined('APP_LOGO_PATH') && file_exists(APP_LOGO_PATH)) {
                         $mail->addEmbeddedImage(APP_LOGO_PATH, 'logo_listin');
                    }

                    $mail->send();
                    $success_message = 'Tautan reset kata sandi telah dikirim ke email Anda. Silakan periksa kotak masuk (dan folder spam).';
                } catch (Exception $e) {
                    $errors[] = "Gagal mengirim email. Mailer Error: {$mail->ErrorInfo}. Coba lagi nanti atau hubungi administrator.";
                    // error_log("Mailer Error: {$mail->ErrorInfo}");
                }
            } else {
                $errors[] = "Gagal menyimpan token reset. Coba lagi nanti.";
            }
            $stmt_update->close();
        } else {
            $errors[] = "Email tidak terdaftar di sistem kami.";
        }
    }
}
?>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Lupa Kata Sandi Anda?</h2>
    <p class="subtitle">Masukkan alamat email Anda di bawah ini. Kami akan mengirimkan tautan untuk mereset kata sandi.</p>

    <?php if (!empty($errors)): ?>
        <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
            <?php foreach ($errors as $error): ?>
                <p><?php echo $error; ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <?php if ($success_message): ?>
        <div style="background-color: #d4edda; color: #155724; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
            <p><?php echo $success_message; ?></p>
        </div>
    <?php else: // Tampilkan form hanya jika belum ada pesan sukses ?>
        <form id="forgotPasswordForm" method="POST" action="forgot_password.php">
            <div class="form-group">
                <label for="email">Alamat Email</label>
                <input type="email" id="email" name="email" required placeholder="Masukkan email terdaftar Anda" value="<?php echo isset($_POST['email']) ? htmlspecialchars($_POST['email']) : ''; ?>">
            </div>
            <button type="submit" class="btn-submit">Kirim Tautan Reset</button>
        </form>
    <?php endif; ?>
    <p class="auth-link">Ingat kata sandi? <a href="login.php">Masuk di sini</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**`reset_password.php` (BARU)**
```php
<?php
$page_title = "Reset Kata Sandi";
require_once 'includes/header.php'; // Untuk $conn

$errors = [];
$success_message = '';
$token = $_GET['token'] ?? null;
$valid_token = false;
$user_id_for_reset = null;

if (!$token) {
    $_SESSION['error_message'] = "Token reset tidak valid atau tidak ditemukan.";
    header("Location: login.php");
    exit();
}

// Validasi token
$stmt_check_token = $conn->prepare("SELECT id FROM users WHERE reset_token = ? AND reset_token_expiry > NOW()");
$stmt_check_token->bind_param("s", $token);
$stmt_check_token->execute();
$result_token = $stmt_check_token->get_result();
if ($user_for_reset = $result_token->fetch_assoc()) {
    $valid_token = true;
    $user_id_for_reset = $user_for_reset['id'];
} else {
    $_SESSION['error_message'] = "Token reset tidak valid, kedaluwarsa, atau sudah digunakan.";
    header("Location: login.php");
    exit();
}
$stmt_check_token->close();


if ($_SERVER["REQUEST_METHOD"] == "POST" && $valid_token) {
    $new_password = $_POST['newPassword'];
    $confirm_new_password = $_POST['confirmNewPassword'];

    if (empty($new_password) || empty($confirm_new_password)) {
        $errors[] = "Kata sandi baru dan konfirmasi wajib diisi.";
    } elseif (strlen($new_password) < 6) {
        $errors[] = "Kata sandi baru minimal 6 karakter.";
    } elseif ($new_password !== $confirm_new_password) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak cocok.";
    }

    if (empty($errors)) {
        $hashed_password = password_hash($new_password, PASSWORD_DEFAULT);
        // Update password dan hapus token
        $stmt_update_pass = $conn->prepare("UPDATE users SET password = ?, reset_token = NULL, reset_token_expiry = NULL WHERE id = ?");
        $stmt_update_pass->bind_param("si", $hashed_password, $user_id_for_reset);
        
        if ($stmt_update_pass->execute()) {
            $_SESSION['success_message'] = "Kata sandi Anda telah berhasil direset. Silakan masuk dengan kata sandi baru.";
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Gagal mereset kata sandi. Silakan coba lagi.";
        }
        $stmt_update_pass->close();
    }
}
?>

<div class="auth-container">
    <div class="logo-container">
        <h1>List In</h1>
    </div>
    <h2>Reset Kata Sandi Anda</h2>
    <p class="subtitle">Masukkan kata sandi baru untuk akun Anda.</p>

    <?php if (!empty($errors)): ?>
        <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
            <?php foreach ($errors as $error): ?>
                <p><?php echo $error; ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <form id="resetPasswordForm" method="POST" action="reset_password.php?token=<?php echo htmlspecialchars($token); ?>">
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
    <p class="auth-link">Ingat kata sandi? <a href="login.php">Masuk di sini</a></p>
</div>

<?php require_once 'includes/footer.php'; ?>
```

**`ubah_password.php`** (Penyesuaian, user bisa saja tidak punya password jika dari Google)
```php
<?php
$page_title = "Ubah Kata Sandi";
require_once 'includes/header.php';
require_once 'includes/sidebar.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];

$stmt_user_pass_check = $conn->prepare("SELECT password FROM users WHERE id = ?");
$stmt_user_pass_check->bind_param("i", $user_id);
$stmt_user_pass_check->execute();
$result_user_pass = $stmt_user_pass_check->get_result();
$user_pass_data = $result_user_pass->fetch_assoc();
$stmt_user_pass_check->close();

$user_has_password = ($user_pass_data && $user_pass_data['password'] !== null);

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $currentPassword = $_POST['currentPassword'] ?? null; // Bisa kosong jika user belum punya password
    $newPassword = $_POST['newPassword'];
    $confirmNewPassword = $_POST['confirmNewPassword'];
    $validation_errors_found = false;

    if ($user_has_password && empty($currentPassword)) {
        add_notification("Kata sandi saat ini wajib diisi jika Anda sudah memilikinya.", "error");
        $validation_errors_found = true;
    }
    if (empty($newPassword) || empty($confirmNewPassword)) {
        add_notification("Kata sandi baru dan konfirmasi wajib diisi.", "error");
        $validation_errors_found = true;
    }
    
    if (!$validation_errors_found) {
        if ($user_has_password) { // Jika user punya password, verifikasi password lama
            if (!password_verify($currentPassword, $user_pass_data['password'])) {
                add_notification("Kata sandi saat ini salah.", "error");
                $validation_errors_found = true;
            }
        }
        // Lanjutkan validasi password baru
        if (strlen($newPassword) < 6) {
            add_notification("Kata sandi baru minimal 6 karakter.", "error");
            $validation_errors_found = true;
        } elseif ($newPassword !== $confirmNewPassword) {
            add_notification("Kata sandi baru dan konfirmasi tidak cocok.", "error");
            $validation_errors_found = true;
        }
            
        if (!$validation_errors_found) {
            $hashed_new_password = password_hash($newPassword, PASSWORD_DEFAULT);
            $stmt_update = $conn->prepare("UPDATE users SET password = ? WHERE id = ?");
            $stmt_update->bind_param("si", $hashed_new_password, $user_id);
            if ($stmt_update->execute()) {
                add_notification("Kata sandi berhasil diubah!", "success");
                // Jika user sebelumnya tidak punya password, mungkin beri notif tambahan
                if (!$user_has_password) {
                    add_notification("Anda sekarang dapat masuk menggunakan email dan kata sandi ini.", "info");
                }
                header("Location: profil.php");
                exit();
            } else {
                add_notification("Gagal mengubah kata sandi: " . $stmt_update->error, "error");
            }
            $stmt_update->close();
        }
    }
}
?>
<main class="main">
    <h2 class="page-title">Ubah Kata Sandi Akun</h2>

    <div class="form-container">
        <?php if (!$user_has_password): ?>
            <div style="background-color: #e2f3fe; color: #0c5464; padding: 10px; border-radius: 5px; margin-bottom: 15px; text-align:left;">
                Anda saat ini masuk menggunakan akun Google dan belum mengatur kata sandi manual. Mengisi form ini akan membuat kata sandi baru untuk akun Anda.
            </div>
        <?php endif; ?>

        <form id="changePasswordForm" method="POST" action="ubah_password.php">
            <?php if ($user_has_password): ?>
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
                <button type="submit" class="btn btn-primary">Simpan Kata Sandi</button>
                 <a href="profil.php" class="btn btn-secondary">Batal</a>
            </div>
        </form>
    </div>
</main>
<?php require_once 'includes/footer.php'; ?>
```

**`send_deadline_emails.php` (BARU - Untuk Cron Job)**
```php
<?php
// send_deadline_emails.php
// Skrip ini HARUS dipanggil oleh cron job, bukan diakses via browser langsung.
// Untuk keamanan, Anda bisa menambahkan cek IP atau secret key jika perlu.

require_once dirname(__FILE__) . '/includes/db.php'; // Mengandung config.php
require_once dirname(__FILE__) . '/vendor/autoload.php';

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

// --- Kirim Email untuk Tugas Deadline H-1 ---
$tomorrow_date = date('Y-m-d', strtotime('+1 day'));
$today_for_email_check = date('Y-m-d');

$stmt_h1 = $conn->prepare(
    "SELECT t.id as task_id, t.title as task_title, DATE_FORMAT(t.due_date, '%d %M %Y') as task_due_date_formatted, 
            u.id as user_id, u.email as user_email, u.username as user_username
     FROM tasks t
     JOIN users u ON t.user_id = u.id
     WHERE t.due_date = ? AND t.status != 'Completed'
     AND (t.last_deadline_email_sent IS NULL OR DATE(t.last_deadline_email_sent) < ?)"
);
if (!$stmt_h1) {
    error_log("Send Deadline Email (H-1) - Prepare failed: " . $conn->error);
    // exit(); // Jangan exit jika ada error prepare, lanjutkan ke overdue
} else {
    $stmt_h1->bind_param("ss", $tomorrow_date, $today_for_email_check);
    $stmt_h1->execute();
    $result_h1 = $stmt_h1->get_result();
    $tasks_due_tomorrow_by_user = [];
    while ($row = $result_h1->fetch_assoc()) {
        $tasks_due_tomorrow_by_user[$row['user_id']]['user_info'] = ['email' => $row['user_email'], 'username' => $row['user_username']];
        $tasks_due_tomorrow_by_user[$row['user_id']]['tasks'][] = ['id' => $row['task_id'], 'title' => $row['task_title'], 'due_date' => $row['task_due_date_formatted']];
    }
    $stmt_h1->close();

    foreach ($tasks_due_tomorrow_by_user as $user_id_h1 => $data_h1) {
        $user_info_h1 = $data_h1['user_info'];
        $task_list_html_h1 = "<ul>";
        $task_ids_to_update_h1 = [];
        foreach ($data_h1['tasks'] as $task_h1) {
            $task_list_html_h1 .= "<li><strong>" . htmlspecialchars($task_h1['title']) . "</strong> (Jatuh tempo: " . htmlspecialchars($task_h1['due_date']) . ")</li>";
            $task_ids_to_update_h1[] = $task_h1['id'];
        }
        $task_list_html_h1 .= "</ul>";

        $mail_h1 = new PHPMailer(true);
        try {
            $mail_h1->isSMTP(); $mail_h1->Host = MAIL_HOST; $mail_h1->SMTPAuth = true;
            $mail_h1->Username = MAIL_USERNAME; $mail_h1->Password = MAIL_PASSWORD;
            $mail_h1->SMTPSecure = MAIL_ENCRYPTION; $mail_h1->Port = MAIL_PORT;
            $mail_h1->setFrom(MAIL_FROM_ADDRESS, MAIL_FROM_NAME);
            $mail_h1->addAddress($user_info_h1['email'], $user_info_h1['username']);
            $mail_h1->isHTML(true);
            $mail_h1->Subject = 'Pengingat: Tugas Akan Jatuh Tempo Besok - List In';
            $body_h1 = file_get_contents(dirname(__FILE__) . '/email_templates/deadline_h1_template.html');
            $body_h1 = str_replace('{{USERNAME}}', htmlspecialchars($user_info_h1['username']), $body_h1);
            $body_h1 = str_replace('{{TASK_LIST}}', $task_list_html_h1, $body_h1);
            $body_h1 = str_replace('{{APP_URL}}', APP_URL, $body_h1);
            $body_h1 = str_replace('{{CURRENT_YEAR}}', date("Y"), $body_h1);
            $mail_h1->Body = $body_h1;
            if (defined('APP_LOGO_PATH') && file_exists(APP_LOGO_PATH)) {
                $mail_h1->addEmbeddedImage(APP_LOGO_PATH, 'logo_listin_email');
            }
            $mail_h1->send();
            echo "Email pengingat H-1 dikirim ke " . $user_info_h1['email'] . "\n";

            // Update last_deadline_email_sent untuk tugas yang sudah dinotifikasi
            if (!empty($task_ids_to_update_h1)) {
                $ids_placeholder_h1 = implode(',', array_fill(0, count($task_ids_to_update_h1), '?'));
                $stmt_update_sent_h1 = $conn->prepare("UPDATE tasks SET last_deadline_email_sent = NOW() WHERE id IN ($ids_placeholder_h1) AND user_id = ?");
                if ($stmt_update_sent_h1) {
                    $types_update_h1 = str_repeat('i', count($task_ids_to_update_h1)) . 'i';
                    $params_update_h1 = array_merge($task_ids_to_update_h1, [$user_id_h1]);
                    $stmt_update_sent_h1->bind_param($types_update_h1, ...$params_update_h1);
                    $stmt_update_sent_h1->execute();
                    $stmt_update_sent_h1->close();
                }
            }
        } catch (Exception $e) {
            echo "Gagal mengirim email H-1 ke " . $user_info_h1['email'] . ". Mailer Error: {$mail_h1->ErrorInfo}\n";
            error_log("Send Deadline Email (H-1) - Mailer Error for " . $user_info_h1['email'] . ": " . $mail_h1->ErrorInfo);
        }
    }
}

// --- Kirim Email untuk Tugas Terlewat (Overdue) ---
$stmt_overdue = $conn->prepare(
    "SELECT t.id as task_id, t.title as task_title, DATE_FORMAT(t.due_date, '%d %M %Y') as task_due_date_formatted, 
            u.id as user_id, u.email as user_email, u.username as user_username
     FROM tasks t
     JOIN users u ON t.user_id = u.id
     WHERE t.due_date < CURDATE() AND t.status != 'Completed'
     AND (t.last_overdue_email_sent IS NULL OR DATE(t.last_overdue_email_sent) < ?)"
);
if (!$stmt_overdue) {
    error_log("Send Deadline Email (Overdue) - Prepare failed: " . $conn->error);
    // exit();
} else {
    $stmt_overdue->bind_param("s", $today_for_email_check);
    $stmt_overdue->execute();
    $result_overdue = $stmt_overdue->get_result();
    $tasks_overdue_by_user = [];
    while ($row_ov = $result_overdue->fetch_assoc()) {
        $tasks_overdue_by_user[$row_ov['user_id']]['user_info'] = ['email' => $row_ov['user_email'], 'username' => $row_ov['user_username']];
        $tasks_overdue_by_user[$row_ov['user_id']]['tasks'][] = ['id' => $row_ov['task_id'], 'title' => $row_ov['task_title'], 'due_date' => $row_ov['task_due_date_formatted']];
    }
    $stmt_overdue->close();

    foreach ($tasks_overdue_by_user as $user_id_ov => $data_ov) {
        $user_info_ov = $data_ov['user_info'];
        $task_list_html_ov = "<ul>";
        $task_ids_to_update_ov = [];
        foreach ($data_ov['tasks'] as $task_ov) {
            $task_list_html_ov .= "<li><strong>" . htmlspecialchars($task_ov['title']) . "</strong> (Seharusnya selesai pada: " . htmlspecialchars($task_ov['due_date']) . ")</li>";
            $task_ids_to_update_ov[] = $task_ov['id'];
        }
        $task_list_html_ov .= "</ul>";

        $mail_ov = new PHPMailer(true);
        try {
            $mail_ov->isSMTP(); $mail_ov->Host = MAIL_HOST; $mail_ov->SMTPAuth = true;
            $mail_ov->Username = MAIL_USERNAME; $mail_ov->Password = MAIL_PASSWORD;
            $mail_ov->SMTPSecure = MAIL_ENCRYPTION; $mail_ov->Port = MAIL_PORT;
            $mail_ov->setFrom(MAIL_FROM_ADDRESS, MAIL_FROM_NAME);
            $mail_ov->addAddress($user_info_ov['email'], $user_info_ov['username']);
            $mail_ov->isHTML(true);
            $mail_ov->Subject = 'Pemberitahuan: Ada Tugas yang Terlewat Batas Waktu - List In';

            $body_ov = file_get_contents(dirname(__FILE__) . '/email_templates/overdue_task_template.html');
            $body_ov = str_replace('{{USERNAME}}', htmlspecialchars($user_info_ov['username']), $body_ov);
            $body_ov = str_replace('{{TASK_LIST}}', $task_list_html_ov, $body_ov);
            $body_ov = str_replace('{{APP_URL}}', APP_URL, $body_ov);
            $body_ov = str_replace('{{CURRENT_YEAR}}', date("Y"), $body_ov);
            $mail_ov->Body = $body_ov;
             if (defined('APP_LOGO_PATH') && file_exists(APP_LOGO_PATH)) {
                $mail_ov->addEmbeddedImage(APP_LOGO_PATH, 'logo_listin_email');
            }

            $mail_ov->send();
            echo "Email pengingat overdue dikirim ke " . $user_info_ov['email'] . "\n";

            // Update last_overdue_email_sent
            if (!empty($task_ids_to_update_ov)) {
                $ids_placeholder_ov = implode(',', array_fill(0, count($task_ids_to_update_ov), '?'));
                $stmt_update_sent_ov = $conn->prepare("UPDATE tasks SET last_overdue_email_sent = NOW() WHERE id IN ($ids_placeholder_ov) AND user_id = ?");
                if ($stmt_update_sent_ov) {
                    $types_update_ov = str_repeat('i', count($task_ids_to_update_ov)) . 'i';
                    $params_update_ov = array_merge($task_ids_to_update_ov, [$user_id_ov]);
                    $stmt_update_sent_ov->bind_param($types_update_ov, ...$params_update_ov);
                    $stmt_update_sent_ov->execute();
                    $stmt_update_sent_ov->close();
                }
            }
        } catch (Exception $e) {
            echo "Gagal mengirim email overdue ke " . $user_info_ov['email'] . ". Mailer Error: {$mail_ov->ErrorInfo}\n";
            error_log("Send Deadline Email (Overdue) - Mailer Error for " . $user_info_ov['email'] . ": " . $mail_ov->ErrorInfo);
        }
    }
}

echo "Proses pengiriman email selesai.\n";
if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```
*Buat folder `email_templates` di root proyek dan di dalamnya dua file HTML template:*

**`email_templates/deadline_h1_template.html`**
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pengingat Tugas Jatuh Tempo Besok</title>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; background-color: #f4f4f4; margin: 0; padding: 0; }
        .container { max-width: 600px; margin: 20px auto; padding: 20px; background-color: #fff; border: 1px solid #ddd; border-radius: 5px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        .header { text-align: center; margin-bottom: 20px; padding-bottom: 15px; border-bottom: 1px solid #eee; }
        .header img { max-height: 50px; margin-bottom: 10px; }
        .header h2 { color: #7e47b8; margin: 0; font-size: 1.5em; }
        .content p { margin-bottom: 15px; }
        .content ul { list-style-type: disc; margin-left: 20px; padding-left: 0; }
        .content li { margin-bottom: 8px; }
        .button-container { text-align: center; margin: 25px 0; }
        .button { background-color: #7e47b8; color: white !important; padding: 12px 25px; text-decoration: none; border-radius: 5px; display: inline-block; font-weight: bold; }
        .footer { font-size: 0.9em; color: #777; text-align: center; margin-top: 20px; padding-top: 15px; border-top: 1px solid #eee; }
        .footer a { color: #7e47b8; text-decoration: none; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <img src="cid:logo_listin_email" alt="List In Logo">
            <h2>Pengingat Tugas Jatuh Tempo</h2>
        </div>
        <div class="content">
            <p>Halo {{USERNAME}},</p>
            <p>Ini adalah pengingat otomatis dari List In bahwa Anda memiliki beberapa tugas yang akan jatuh tempo besok. Mohon periksa dan selesaikan tugas-tugas berikut:</p>
            {{TASK_LIST}}
            <p>Segera selesaikan tugas-tugas ini untuk menjaga produktivitas Anda. Anda dapat melihat detail tugas dan mengelolanya melalui aplikasi List In.</p>
            <div class="button-container">
                <a href="{{APP_URL}}/manajemen_tugas.php" class="button">Lihat Tugas Saya</a>
            </div>
            <p>Jika Anda telah menyelesaikan tugas ini, Anda dapat mengabaikan email ini atau menandainya sebagai 'Selesai' di aplikasi.</p>
            <p>Terima kasih atas perhatiannya dan semoga hari Anda produktif!</p>
            <p>Salam Hormat,<br>Tim List In</p>
        </div>
        <div class="footer">
            <p>Email ini dikirim secara otomatis. Mohon untuk tidak membalas email ini.<br>
            Kunjungi <a href="{{APP_URL}}">List In</a> untuk informasi lebih lanjut.<br>
            &copy; {{CURRENT_YEAR}} List In. Hak Cipta Dilindungi.</p>
        </div>
    </div>
</body>
</html>
```

**`email_templates/overdue_task_template.html`**
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pemberitahuan Tugas Terlewat Batas Waktu</title>
     <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; background-color: #f4f4f4; margin: 0; padding: 0; }
        .container { max-width: 600px; margin: 20px auto; padding: 20px; background-color: #fff; border: 1px solid #ddd; border-radius: 5px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        .header { text-align: center; margin-bottom: 20px; padding-bottom: 15px; border-bottom: 1px solid #eee; }
        .header img { max-height: 50px; margin-bottom: 10px; }
        .header h2 { color: #d9534f; margin: 0; font-size: 1.5em; } /* Warna merah untuk overdue */
        .content p { margin-bottom: 15px; }
        .content ul { list-style-type: disc; margin-left: 20px; padding-left: 0; }
        .content li { margin-bottom: 8px; }
        .content strong { color: #c9302c; } /* Penekanan warna merah */
        .button-container { text-align: center; margin: 25px 0; }
        .button { background-color: #7e47b8; color: white !important; padding: 12px 25px; text-decoration: none; border-radius: 5px; display: inline-block; font-weight: bold; }
        .footer { font-size: 0.9em; color: #777; text-align: center; margin-top: 20px; padding-top: 15px; border-top: 1px solid #eee; }
        .footer a { color: #7e47b8; text-decoration: none; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <img src="cid:logo_listin_email" alt="List In Logo">
            <h2>Pemberitahuan Tugas Terlewat</h2>
        </div>
        <div class="content">
            <p>Halo {{USERNAME}},</p>
            <p>Sistem kami mendeteksi bahwa Anda memiliki beberapa tugas yang telah <strong>melewati batas waktu penyelesaian</strong>. Mohon segera periksa dan tindak lanjuti tugas-tugas berikut:</p>
            {{TASK_LIST}}
            <p>Menyelesaikan tugas yang terlewat penting untuk menjaga alur kerja dan pencapaian target Anda. Silakan akses aplikasi List In untuk memperbarui status atau menjadwalkan ulang tugas-tugas ini.</p>
            <div class="button-container">
                <a href="{{APP_URL}}/manajemen_tugas.php" class="button">Kelola Tugas Sekarang</a>
            </div>
            <p>Jika ada kendala atau Anda memerlukan bantuan, jangan ragu untuk merujuk pada sumber daya yang tersedia atau menghubungi tim support (jika ada).</p>
            <p>Terima kasih atas perhatian dan kerjasamanya.</p>
            <p>Salam Hormat,<br>Tim List In</p>
        </div>
        <div class="footer">
            <p>Email ini dikirim secara otomatis. Mohon untuk tidak membalas email ini.<br>
            Kunjungi <a href="{{APP_URL}}">List In</a> untuk informasi lebih lanjut.<br>
            &copy; {{CURRENT_YEAR}} List In. Hak Cipta Dilindungi.</p>
        </div>
    </div>
</body>
</html>
```

**`laporan.php` (BARU)**
```php
<?php
$page_title = "Laporan Tugas";
require_once 'includes/header.php';
require_once 'includes/sidebar.php';
require_once 'includes/task_helper.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];

// Inisialisasi filter tanggal untuk PDF dan tampilan halaman
$report_date_range_val = '';
$report_start_date_sql = null;
$report_end_date_sql = null;

if (isset($_GET['filterReport'])) {
    $report_date_range_val = trim($_GET['reportDateRange'] ?? '');
    if (!empty($report_date_range_val)) {
        $dates = explode(' - ', $report_date_range_val);
        if (count($dates) >= 1 && !empty(trim($dates[0]))) {
            $date_parts_start = explode('/', trim($dates[0]));
            if (count($date_parts_start) == 3 && checkdate((int)$date_parts_start[1], (int)$date_parts_start[0], (int)$date_parts_start[2])) {
                $report_start_date_sql = $date_parts_start[2] . '-' . $date_parts_start[1] . '-' . $date_parts_start[0];
            }
        }
        if (count($dates) == 2 && !empty(trim($dates[1]))) {
            $date_parts_end = explode('/', trim($dates[1]));
             if (count($date_parts_end) == 3 && checkdate((int)$date_parts_end[1], (int)$date_parts_end[0], (int)$date_parts_end[2])) {
                $report_end_date_sql = $date_parts_end[2] . '-' . $date_parts_end[1] . '-' . $date_parts_end[0];
            }
        } elseif (count($dates) == 1 && $report_start_date_sql) {
            $report_end_date_sql = $report_start_date_sql; // Jika hanya satu tanggal, rentangnya adalah hari itu
        }
        if ($report_start_date_sql && $report_end_date_sql && strtotime($report_start_date_sql) > strtotime($report_end_date_sql)) {
            list($report_start_date_sql, $report_end_date_sql) = [$report_end_date_sql, $report_start_date_sql];
        }
    }
} else { // Default: 7 hari terakhir
    $today_for_report = new DateTimeImmutable();
    $report_end_date_sql = $today_for_report->format('Y-m-d');
    $report_start_date_sql = $today_for_report->modify('-6 days')->format('Y-m-d');
    $report_date_range_val = date('d/m/Y', strtotime($report_start_date_sql)) . ' - ' . date('d/m/Y', strtotime($report_end_date_sql));
}


// Data untuk Progress Circles (menggunakan filter tanggal jika ada)
$total_tasks_report = 0; $completed_report = 0; $in_progress_report = 0; $not_started_report = 0;
$sql_stats_report = "SELECT status, COUNT(*) as count FROM tasks WHERE user_id = ?";
$params_stats_report = [$user_id];
$types_stats_report = "i";

if ($report_start_date_sql && $report_end_date_sql) {
    // Filter berdasarkan tanggal pembuatan atau update TUGAS dalam rentang, BUKAN tanggal deadline
    // Ini mungkin perlu disesuaikan tergantung definisi "laporan dalam rentang waktu"
    $sql_stats_report .= " AND ( (DATE(created_at) BETWEEN ? AND ?) OR (status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?) )";
    $params_stats_report[] = $report_start_date_sql; $params_stats_report[] = $report_end_date_sql;
    $params_stats_report[] = $report_start_date_sql; $params_stats_report[] = $report_end_date_sql;
    $types_stats_report .= "ssss";
}
$sql_stats_report .= " GROUP BY status";

$stmt_stats_report = $conn->prepare($sql_stats_report);
if($stmt_stats_report){
    $stmt_stats_report->bind_param($types_stats_report, ...$params_stats_report);
    $stmt_stats_report->execute();
    $result_stats_report = $stmt_stats_report->get_result();
    while ($row_stat_report = $result_stats_report->fetch_assoc()) {
        $total_tasks_report += (int)$row_stat_report['count'];
        if ($row_stat_report['status'] == 'Completed') $completed_report = (int)$row_stat_report['count'];
        if ($row_stat_report['status'] == 'In Progress') $in_progress_report = (int)$row_stat_report['count'];
        if ($row_stat_report['status'] == 'Not Started') $not_started_report = (int)$row_stat_report['count'];
    }
    $stmt_stats_report->close();
}
$progress_circles_data_report = [
    'Selesai' => ['count' => $completed_report, 'percent' => ($total_tasks_report > 0) ? round(($completed_report / $total_tasks_report) * 100) : 0, 'color' => '#4caf50'],
    'Dikerjakan' => ['count' => $in_progress_report, 'percent' => ($total_tasks_report > 0) ? round(($in_progress_report / $total_tasks_report) * 100) : 0, 'color' => '#2196f3'],
    'Belum Mulai' => ['count' => $not_started_report, 'percent' => ($total_tasks_report > 0) ? round(($not_started_report / $total_tasks_report) * 100) : 0, 'color' => '#f44336'],
];


// Data untuk Diagram Performa (menggunakan filter tanggal)
$line_chart_labels_report = []; $line_chart_data_completed_report = [];
if ($report_start_date_sql && $report_end_date_sql) {
    try {
        $current_date_loop_obj_report = DateTime::createFromFormat('Y-m-d', $report_start_date_sql);
        $end_date_loop_obj_report = DateTime::createFromFormat('Y-m-d', $report_end_date_sql);

        if ($current_date_loop_obj_report && $end_date_loop_obj_report) {
            $loop_count_report = 0; $interval_one_day_report = new DateInterval('P1D');
            while ($current_date_loop_obj_report <= $end_date_loop_obj_report) {
                $date_str_loop_report = $current_date_loop_obj_report->format('Y-m-d');
                $line_chart_labels_report[] = $current_date_loop_obj_report->format('d M');
                
                $stmt_completed_on_date_report = $conn->prepare("SELECT COUNT(*) as count FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) = ?");
                $tasks_completed_on_day_report = 0;
                if($stmt_completed_on_date_report) {
                    $stmt_completed_on_date_report->bind_param("is", $user_id, $date_str_loop_report);
                    $stmt_completed_on_date_report->execute();
                    $result_completed_on_date_report = $stmt_completed_on_date_report->get_result();
                    if($result_completed_on_date_report && $result_completed_on_date_report->num_rows > 0) {
                        $row_completed_report = $result_completed_on_date_report->fetch_assoc();
                        $tasks_completed_on_day_report = (int)($row_completed_report['count'] ?? 0);
                    }
                    $stmt_completed_on_date_report->close();
                }
                $line_chart_data_completed_report[] = $tasks_completed_on_day_report;
                $current_date_loop_obj_report->add($interval_one_day_report); $loop_count_report++;
                if ($loop_count_report > 90 ) { // Batasi loop untuk performa
                    if ($current_date_loop_obj_report <= $end_date_loop_obj_report) {
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

$performance_chart_data_report = [
    'labels' => $line_chart_labels_report,
    'datasets' => [[
        'label' => 'Tugas Selesai','data' => $line_chart_data_completed_report,
        'borderColor' => '#7e47b8','backgroundColor' => 'rgba(126, 71, 184, 0.1)',
        'fill' => true,'tension' => 0.2
    ]]
];

?>
<main class="main">
    <div class="page-title-container" style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; padding-bottom: 10px; border-bottom: 1px solid #dfe3e8;">
        <h2 class="page-title" style="margin-bottom:0; border-bottom:none;">Laporan Tugas</h2>
        <form method="GET" action="generate_pdf_report.php" target="_blank" style="display: inline-block;">
             <?php if ($report_start_date_sql && $report_end_date_sql): ?>
                <input type="hidden" name="start_date" value="<?php echo htmlspecialchars($report_start_date_sql); ?>">
                <input type="hidden" name="end_date" value="<?php echo htmlspecialchars($report_end_date_sql); ?>">
             <?php endif; ?>
            <button type="submit" class="btn btn-primary"><i class="fas fa-file-pdf"></i> Unduh Laporan PDF</button>
        </form>
    </div>

    <div class="filters-container" style="margin-bottom: 20px;">
        <form method="GET" action="laporan.php" style="display: flex; gap: 15px; align-items: flex-end; width:100%;">
            <div class="form-group" style="flex-grow:1;">
                <label for="reportDateRange">Pilih Rentang Waktu Laporan</label>
                <input type="text" id="reportDateRange" name="reportDateRange" placeholder="Pilih rentang tanggal..." value="<?php echo htmlspecialchars($report_date_range_val); ?>">
            </div>
            <button type="submit" name="filterReport" value="1" class="btn btn-primary">Tampilkan Laporan</button>
            <a href="laporan.php" class="btn btn-secondary">Reset Filter</a>
        </form>
    </div>

    <div class="dashboard-grid" style="height: auto;"> <!-- Adaptasi dari dashboard grid -->
        <div class="dashboard-left-column"> <!-- Kolom untuk Status & Performa -->
            <section class="widget">
                <h3>Status Tugas (Rentang Dipilih)</h3><br>
                <div class="widget-content-area status-progress-container">
                    <?php if ($total_tasks_report > 0): ?>
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
                        <p class="no-tasks-message" style="text-align:center; width:100%;">Tidak ada data tugas pada rentang ini.</p>
                    <?php endif; ?>
                </div>
            </section>

            <section class="widget" style="margin-top:15px;">
                <h3>Performa Pengerjaan (Rentang Dipilih)</h3>
                <div class="widget-content-area chart-container">
                     <?php
                     $has_valid_line_chart_data_report = false;
                     if (isset($performance_chart_data_report['datasets'][0]['data']) && is_array($performance_chart_data_report['datasets'][0]['data'])) {
                         $filtered_data_report = array_filter($performance_chart_data_report['datasets'][0]['data'], function($x) { return $x !== null && $x >=0; });
                         $has_valid_line_chart_data_report = !empty($filtered_data_report);
                     }
                     $has_valid_labels_report = !empty($performance_chart_data_report['labels']) && !in_array("Tidak Ada Data", $performance_chart_data_report['labels']);

                     if ($has_valid_line_chart_data_report && $has_valid_labels_report):
                     ?>
                        <canvas id="reportPerformanceLineChart"></canvas>
                    <?php else: ?>
                        <p class="no-tasks-message" style="text-align:center; padding-top:20px;">Tidak ada data tugas selesai untuk ditampilkan pada rentang waktu ini.</p>
                    <?php endif; ?>
                </div>
            </section>
        </div>

        <div class="dashboard-right-column"> <!-- Kolom untuk Kalender & Detail Tugas -->
            <section class="widget" style="height:100%;">
                <h3>Kalender Tugas (Deadline)</h3>
                <div class="report-calendar-area" style="display: flex; flex-direction: column; height: 100%;">
                    <div id="reportCalendarContainer" style="margin-bottom: 15px;">
                        <!-- Kalender akan diinisialisasi oleh Flatpickr di sini -->
                    </div>
                    <h4 id="selectedDateTitle" style="margin-bottom: 10px; color: #34495e; border-bottom: 1px solid #f0f0f0; padding-bottom: 5px;">Tugas untuk Tanggal: -</h4>
                    <div id="tasksForSelectedDate" class="widget-content-area" style="flex-grow:1; overflow-y:auto; padding-right:5px;">
                        <p class="no-tasks-message">Pilih tanggal pada kalender untuk melihat tugas.</p>
                        <!-- Tugas akan dimuat di sini oleh JavaScript -->
                    </div>
                </div>
            </section>
        </div>
    </div>
</main>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // Chart untuk laporan
    const reportLineCtx = document.getElementById('reportPerformanceLineChart');
    if (reportLineCtx && typeof Chart !== 'undefined') {
        const reportPerformanceData = <?php echo json_encode($performance_chart_data_report); ?>;
        let hasValidReportLineData = false;
        if (reportPerformanceData && reportPerformanceData.labels && Array.isArray(reportPerformanceData.labels) &&
            reportPerformanceData.datasets && Array.isArray(reportPerformanceData.datasets) && reportPerformanceData.datasets.length > 0 &&
            reportPerformanceData.datasets[0].data && Array.isArray(reportPerformanceData.datasets[0].data) ) {
            
            let validLabelsExistReport = reportPerformanceData.labels.some(l => l !== 'Error' && l !== 'Tidak Ada Data' && l !== '...');
            let numericDataExistsReport = reportPerformanceData.datasets[0].data.some(d => typeof d === 'number' && d >= 0);
            hasValidReportLineData = validLabelsExistReport && numericDataExistsReport;
        }

        if(hasValidReportLineData){
            new Chart(reportLineCtx, {
                type: 'line', data: reportPerformanceData,
                options: { responsive: true, maintainAspectRatio: false,
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1, precision: 0, callback: function(value) {if (Number.isInteger(value)) {return value;}}} } },
                    plugins: { legend: { display: false }, tooltip: { callbacks: { label: function(context) { let label = context.dataset.label || ''; if (label) { label += ': '; } if (context.parsed.y !== null) { label += context.parsed.y; } return label; }}}}
                }
            });
        }
    }

    // Kalender di halaman laporan
    const reportCalendarEl = document.getElementById('reportCalendarContainer');
    const tasksForSelectedDateEl = document.getElementById('tasksForSelectedDate');
    const selectedDateTitleEl = document.getElementById('selectedDateTitle');

    if (reportCalendarEl && tasksForSelectedDateEl && selectedDateTitleEl) {
        flatpickr(reportCalendarEl, {
            inline: true,
            dateFormat: "Y-m-d", // Format untuk AJAX
            locale: "id",
            onChange: function(selectedDates, dateStr, instance) {
                if (selectedDates.length > 0) {
                    const selectedDate = dateStr;
                    const displayDate = instance.formatDate(selectedDates[0], "d F Y");
                    selectedDateTitleEl.textContent = "Tugas untuk Tanggal: " + displayDate;
                    tasksForSelectedDateEl.innerHTML = '<p class="no-tasks-message">Memuat tugas...</p>';
                    
                    fetch(`ajax_get_tasks_for_date.php?date=${selectedDate}`)
                        .then(response => response.json())
                        .then(data => {
                            tasksForSelectedDateEl.innerHTML = ''; // Kosongkan dulu
                            if (data.success && data.tasks.length > 0) {
                                data.tasks.forEach(task => {
                                    // Membuat card mirip dengan task_helper.php tapi di JS
                                    // Ini bisa disederhanakan atau memanggil fungsi render_task_card versi JS jika ada
                                    const card = document.createElement('div');
                                    card.className = 'task-item-card status-' + task.status.toLowerCase().replace(/ /g, '-');
                                    
                                    let priorityClass = 'priority-' + (task.priority ? task.priority.toLowerCase() : 'medium');
                                    let statusText = task.status_display || task.status;

                                    card.innerHTML = `
                                        <div class="task-details">
                                            <strong>${task.title || 'Tanpa Judul'}</strong>
                                            <p class="description">${task.description || 'Tidak ada deskripsi.'}</p>
                                            <p class="meta-info">
                                                Prioritas: <span class="${priorityClass}">${task.priority || 'Medium'}</span> | 
                                                Status: <span class="task-status-text">${statusText}</span> | 
                                                Deadline: ${task.due_date_formatted || 'N/A'}
                                            </p>
                                        </div>
                                        <div class="task-actions">
                                            <a href="edit_tugas.php?id=${task.id}&from=laporan.php" class="edit-btn" title="Edit Tugas"><i class="fas fa-edit"></i></a>
                                        </div>
                                    `;
                                    tasksForSelectedDateEl.appendChild(card);
                                });
                            } else if (data.success && data.tasks.length === 0) {
                                tasksForSelectedDateEl.innerHTML = '<p class="no-tasks-message">Tidak ada tugas dengan deadline pada tanggal ini.</p>';
                            } else {
                                tasksForSelectedDateEl.innerHTML = `<p class="no-tasks-message">Gagal memuat tugas: ${data.message || 'Error tidak diketahui.'}</p>`;
                            }
                        })
                        .catch(error => {
                            console.error('Error fetching tasks for date:', error);
                            tasksForSelectedDateEl.innerHTML = '<p class="no-tasks-message">Terjadi kesalahan saat mengambil data tugas.</p>';
                        });
                }
            }
        });
    }
});
</script>
<?php require_once 'includes/footer.php'; ?>
```

**`ajax_get_tasks_for_date.php` (BARU)**
```php
<?php
require_once 'includes/db.php';

header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'User not authenticated.']);
    exit();
}
$user_id = $_SESSION['user_id'];
$selected_date = $_GET['date'] ?? null;

if (!$selected_date || !DateTime::createFromFormat('Y-m-d', $selected_date)) {
    echo json_encode(['success' => false, 'message' => 'Format tanggal tidak valid.']);
    exit();
}

$tasks = [];
$stmt = $conn->prepare("SELECT id, title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted 
                        FROM tasks 
                        WHERE user_id = ? AND due_date = ? 
                        ORDER BY CASE priority WHEN 'High' THEN 1 WHEN 'Medium' THEN 2 WHEN 'Low' THEN 3 ELSE 4 END, created_at ASC");

if ($stmt) {
    $stmt->bind_param("is", $user_id, $selected_date);
    $stmt->execute();
    $result = $stmt->get_result();
    $status_map = [
        'Not Started' => 'Belum Mulai',
        'In Progress' => 'Dikerjakan',
        'Completed' => 'Selesai'
    ];
    while ($row = $result->fetch_assoc()) {
        $row['status_display'] = $status_map[$row['status']] ?? $row['status'];
        $tasks[] = $row;
    }
    $stmt->close();
    echo json_encode(['success' => true, 'tasks' => $tasks]);
} else {
    echo json_encode(['success' => false, 'message' => 'Gagal mempersiapkan query: ' . $conn->error]);
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

**`generate_pdf_report.php` (BARU)**
```php
<?php
require_once 'includes/db.php'; // Untuk session_start(), $conn, config
require_once 'vendor/autoload.php'; // Untuk TCPDF

if (!isset($_SESSION['user_id'])) {
    // Seharusnya tidak bisa diakses tanpa login, tapi sebagai pengaman
    die("Akses ditolak. Silakan login terlebih dahulu.");
}
$user_id = $_SESSION['user_id'];
$username = $_SESSION['username'] ?? 'Pengguna';

$start_date_filter = $_GET['start_date'] ?? null;
$end_date_filter = $_GET['end_date'] ?? null;

$date_range_text = "Semua Waktu";
$sql_conditions = ["user_id = ?"];
$params = [$user_id];
$types = "i";

if ($start_date_filter && $end_date_filter) {
    // Asumsi format YYYY-MM-DD dari GET request
    $sql_conditions[] = "((DATE(created_at) BETWEEN ? AND ?) OR (status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?))";
    $params[] = $start_date_filter; $params[] = $end_date_filter;
    $params[] = $start_date_filter; $params[] = $end_date_filter;
    $types .= "ssss";
    try {
        $start_dt = new DateTime($start_date_filter);
        $end_dt = new DateTime($end_date_filter);
        $date_range_text = $start_dt->format('d M Y') . " - " . $end_dt->format('d M Y');
    } catch (Exception $e) { $date_range_text = "Rentang Tidak Valid"; }

} elseif ($start_date_filter) { // Hanya tanggal mulai (satu hari)
    $sql_conditions[] = "((DATE(created_at) = ?) OR (status = 'Completed' AND DATE(updated_at) = ?))";
    $params[] = $start_date_filter; $params[] = $start_date_filter;
    $types .= "ss";
    try {
        $start_dt = new DateTime($start_date_filter);
        $date_range_text = $start_dt->format('d M Y');
    } catch (Exception $e) { $date_range_text = "Tanggal Tidak Valid"; }
}

$sql_where = implode(" AND ", $sql_conditions);
$sql_tasks = "SELECT title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_f, DATE_FORMAT(created_at, '%d/%m/%Y') as created_at_f, DATE_FORMAT(updated_at, '%d/%m/%Y') as updated_at_f FROM tasks WHERE $sql_where ORDER BY created_at DESC";

$stmt_tasks = $conn->prepare($sql_tasks);
$tasks_for_pdf = [];
if ($stmt_tasks) {
    $stmt_tasks->bind_param($types, ...$params);
    $stmt_tasks->execute();
    $result = $stmt_tasks->get_result();
    while ($row = $result->fetch_assoc()) {
        $tasks_for_pdf[] = $row;
    }
    $stmt_tasks->close();
}

// Statistik
$total_count = 0; $completed_count = 0; $inprogress_count = 0; $notstarted_count = 0;
$sql_stats = "SELECT status, COUNT(*) as count FROM tasks WHERE $sql_where GROUP BY status";
$stmt_stats = $conn->prepare($sql_stats);
if($stmt_stats){
    $stmt_stats->bind_param($types, ...$params);
    $stmt_stats->execute();
    $res_stats = $stmt_stats->get_result();
    while($r_stat = $res_stats->fetch_assoc()){
        $total_count += (int)$r_stat['count'];
        if($r_stat['status'] == 'Completed') $completed_count = (int)$r_stat['count'];
        if($r_stat['status'] == 'In Progress') $inprogress_count = (int)$r_stat['count'];
        if($r_stat['status'] == 'Not Started') $notstarted_count = (int)$r_stat['count'];
    }
    $stmt_stats->close();
}

// Buat instance PDF baru (TCPDF)
class MYPDF extends TCPDF {
    public function Header() {
        if (defined('APP_LOGO_PATH') && file_exists(APP_LOGO_PATH)) {
            $this->Image(APP_LOGO_PATH, 10, 10, 25, '', 'PNG', '', 'T', false, 300, '', false, false, 0, false, false, false);
        }
        $this->SetFont('helvetica', 'B', 18);
        $this->SetTextColor(126, 71, 184); // Warna ungu ListIn
        $this->Cell(0, 15, 'List In - Laporan Tugas', 0, false, 'C', 0, '', 0, false, 'M', 'M');
        $this->Ln(5);
        $this->SetFont('helvetica', '', 10);
        $this->SetTextColor(0,0,0); // Reset warna teks
        $this->Cell(0, 10, 'Dihasilkan oleh Sistem Manajemen Tugas List In', 0, false, 'C');
        $this->Ln(5);
        $this->Line(10, 30, $this->getPageWidth()-10, 30); // Garis bawah header
    }

    public function Footer() {
        $this->SetY(-15);
        $this->SetFont('helvetica', 'I', 8);
        $this->Cell(0, 10, 'Halaman '.$this->getAliasNumPage().' dari '.$this->getAliasNbPages() . ' | List In Report &copy; ' . date('Y'), 0, false, 'C', 0, '', 0, false, 'T', 'M');
    }
}

$pdf = new MYPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);

// Set informasi dokumen
$pdf->SetCreator(PDF_CREATOR);
$pdf->SetAuthor('List In Application');
$pdf->SetTitle('Laporan Tugas - List In');
$pdf->SetSubject('Laporan Detail Tugas Produktivitas');
$pdf->SetKeywords('List In, Tugas, Laporan, Produktivitas');

// Set header dan footer
$pdf->setHeaderFont(Array(PDF_FONT_NAME_MAIN, '', PDF_FONT_SIZE_MAIN));
$pdf->setFooterFont(Array(PDF_FONT_NAME_DATA, '', PDF_FONT_SIZE_DATA));
$pdf->SetDefaultMonospacedFont(PDF_FONT_MONOSPACED);
$pdf->SetMargins(PDF_MARGIN_LEFT, 35, PDF_MARGIN_RIGHT); // Top margin 35mm
$pdf->SetHeaderMargin(PDF_MARGIN_HEADER);
$pdf->SetFooterMargin(PDF_MARGIN_FOOTER);
$pdf->SetAutoPageBreak(TRUE, PDF_MARGIN_BOTTOM);
$pdf->setImageScale(PDF_IMAGE_SCALE_RATIO);
if (@file_exists(dirname(__FILE__).'/vendor/tecnickcom/tcpdf/examples/lang/ind.php')) {
    require_once(dirname(__FILE__).'/vendor/tecnickcom/tcpdf/examples/lang/ind.php');
    $pdf->setLanguageArray($l);
}
$pdf->SetFont('helvetica', '', 10);
$pdf->AddPage();

// Judul Konten Laporan
$pdf->SetFont('helvetica', 'B', 14);
$pdf->Cell(0, 10, 'Ringkasan Laporan Tugas', 0, 1, 'L');
$pdf->SetFont('helvetica', '', 10);
$pdf->Cell(0, 6, 'Pengguna: ' . htmlspecialchars($username), 0, 1, 'L');
$pdf->Cell(0, 6, 'Rentang Waktu: ' . htmlspecialchars($date_range_text), 0, 1, 'L');
$pdf->Cell(0, 6, 'Tanggal Laporan: ' . date('d F Y, H:i:s'), 0, 1, 'L');
$pdf->Ln(5);

// Statistik Ringkas
$pdf->SetFont('helvetica', 'B', 11);
$pdf->Cell(0, 8, 'Statistik Tugas:', 0, 1, 'L');
$pdf->SetFont('helvetica', '', 10);
$html_stats = '<table border="0" cellpadding="4">
    <tr><td width="150">Total Tugas dalam Rentang</td><td width="10">:</td><td>' . $total_count . '</td></tr>
    <tr><td>Tugas Selesai</td><td>:</td><td>' . $completed_count . ' (' . ($total_count > 0 ? round(($completed_count/$total_count)*100) : 0) . '%)</td></tr>
    <tr><td>Tugas Dikerjakan</td><td>:</td><td>' . $inprogress_count . ' (' . ($total_count > 0 ? round(($inprogress_count/$total_count)*100) : 0) . '%)</td></tr>
    <tr><td>Tugas Belum Mulai</td><td>:</td><td>' . $notstarted_count . ' (' . ($total_count > 0 ? round(($notstarted_count/$total_count)*100) : 0) . '%)</td></tr>
</table>';
$pdf->writeHTML($html_stats, true, false, true, false, '');
$pdf->Ln(8);

// Tabel Detail Tugas
$pdf->SetFont('helvetica', 'B', 11);
$pdf->Cell(0, 10, 'Detail Tugas:', 0, 1, 'L');
$pdf->SetFont('helvetica', '', 9);

$html_tasks = '<table border="1" cellpadding="5" cellspacing="0" style="border-collapse: collapse;">
    <thead>
        <tr style="background-color: #7e47b8; color: white; font-weight:bold; text-align:center;">
            <th width="5%">No.</th>
            <th width="25%">Judul</th>
            <th width="15%">Prioritas</th>
            <th width="15%">Status</th>
            <th width="15%">Deadline</th>
            <th width="25%">Tgl. Diperbarui/Selesai</th>
        </tr>
    </thead>
    <tbody>';

if (empty($tasks_for_pdf)) {
    $html_tasks .= '<tr><td colspan="6" style="text-align:center; font-style:italic;">Tidak ada data tugas untuk ditampilkan pada rentang waktu ini.</td></tr>';
} else {
    $no = 1;
    foreach ($tasks_for_pdf as $task) {
        $status_display = $task['status'];
        if ($task['status'] == 'Not Started') $status_display = 'Belum Mulai';
        elseif ($task['status'] == 'In Progress') $status_display = 'Dikerjakan';

        // Tanggal yang relevan: updated_at jika completed, jika tidak created_at (atau due_date)
        $relevant_date = ($task['status'] == 'Completed') ? $task['updated_at_f'] : $task['created_at_f'];

        $html_tasks .= '<tr>
            <td style="text-align:center;">' . $no++ . '</td>
            <td>' . htmlspecialchars($task['title']) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($task['priority']) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($status_display) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($task['due_date_f'] ?? 'N/A') . '</td>
            <td style="text-align:center;">' . htmlspecialchars($relevant_date) . '</td>
        </tr>';
        if (!empty($task['description'])) {
             $html_tasks .= '<tr><td colspan="6" style="font-size:8pt; font-style:italic; background-color:#f9f9f9;">Deskripsi: ' . nl2br(htmlspecialchars($task['description'])) . '</td></tr>';
        }
    }
}
$html_tasks .= '</tbody></table>';
$pdf->writeHTML($html_tasks, true, false, true, false, '');
$pdf->Ln(5);

$pdf->SetFont('helvetica', 'I', 8);
$pdf->MultiCell(0, 10, "Dokumen ini adalah laporan resmi yang dihasilkan oleh sistem List In. Informasi yang terkandung di dalamnya bersifat rahasia dan hanya ditujukan untuk pengguna yang bersangkutan. Dilarang menyebarluaskan tanpa izin.", 0, 'L', 0, 1, '', '', true);

// Tutup dan output PDF
$pdf_filename = 'Laporan_Tugas_ListIn_' . date('Ymd_His') . '.pdf';
$pdf->Output($pdf_filename, 'I'); // I: tampilkan di browser, D: download

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
exit; // Penting untuk menghentikan eksekusi skrip lain
?>
```

**`css/style.css`** (Tambahkan ini di akhir file yang ada)
```css
/* styles.css - TAMBAHAN UNTUK HALAMAN LAPORAN & AUTH */

/* Tombol Login Google */
.btn-google {
    background-color: #DB4437 !important; /* Warna Google */
    color: white;
}
.btn-google:hover {
    background-color: #c23321 !important;
}
.btn-submit .fab.fa-google {
    margin-right: 8px;
    font-size: 1.1em;
}


/* Halaman Laporan */
.report-calendar-area {
    display: flex;
    gap: 20px; /* Jarak antara kalender dan list tugas */
    align-items: flex-start; /* Kalender dan list tugas align top */
}

#reportCalendarContainer {
    flex: 1 1 400px; /* Kalender bisa membesar, basis 400px */
    min-width: 300px; /* Lebar minimal kalender */
    background: #fff;
    padding: 15px;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
}
html.dark-theme-active #reportCalendarContainer {
    background: #2c2c2c;
    border: 1px solid #444;
}
/* Styling Flatpickr di dalam #reportCalendarContainer jika diperlukan (sudah ada styling umum flatpickr) */
html.dark-theme-active #reportCalendarContainer .flatpickr-calendar {
    background: #2c2c2c;
    border-color: #444;
    color: #e0e0e0;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-month,
html.dark-theme-active #reportCalendarContainer .flatpickr-weekday,
html.dark-theme-active #reportCalendarContainer .flatpickr-day {
    color: #e0e0e0;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-day:hover,
html.dark-theme-active #reportCalendarContainer .flatpickr-day:focus {
    background: #3a3a3a;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-day.today {
    border-color: #bb86fc;
    color: #bb86fc;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-day.selected,
html.dark-theme-active #reportCalendarContainer .flatpickr-day.startRange,
html.dark-theme-active #reportCalendarContainer .flatpickr-day.endRange {
    background: #bb86fc;
    border-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-day.flatpickr-disabled,
html.dark-theme-active #reportCalendarContainer .flatpickr-day.prevMonthDay,
html.dark-theme-active #reportCalendarContainer .flatpickr-day.nextMonthDay {
    color: #757575;
}
html.dark-theme-active #reportCalendarContainer .arrowUp,
html.dark-theme-active #reportCalendarContainer .arrowDown {
    border-bottom-color: #e0e0e0; /* atau border-top-color */
}


#tasksForSelectedDate {
    flex: 1 1 500px; /* List tugas bisa membesar, basis 500px */
    max-height: 400px; /* Batasi tinggi dan buat scrollable jika perlu */
    overflow-y: auto;
    padding: 10px;
    background: #f9fafb; /* Sedikit beda dari widget utama */
    border-radius: 6px;
    border: 1px solid #e7eaec;
}
html.dark-theme-active #tasksForSelectedDate {
    background: #373737;
    border-color: #555;
}

#selectedDateTitle {
    font-size: 1.1rem;
    color: #34495e;
    margin-bottom: 10px;
    padding-bottom: 8px;
    border-bottom: 1px solid #f0f0f0;
}
html.dark-theme-active #selectedDateTitle {
    color: #f5f5f5;
    border-bottom-color: #444;
}


/* Responsive untuk Halaman Laporan */
@media (max-width: 992px) { /* Samakan dengan breakpoint dashboard */
    .report-calendar-area {
        flex-direction: column; /* Susun vertikal di layar kecil */
    }
    #reportCalendarContainer,
    #tasksForSelectedDateContainer { /* Jika Anda membungkus tasksForSelectedDate */
        flex-basis: auto; /* Reset flex-basis */
        width: 100%;
    }
    #tasksForSelectedDate {
        max-height: 300px; /* Kurangi tinggi di mobile */
    }
}

/* Penyesuaian untuk .dashboard-grid di laporan.php jika diperlukan */
.main .dashboard-grid { /* Target spesifik untuk laporan */
    /* ... bisa tambahkan style jika default dashboard-grid tidak cocok ... */
}

```

**`css/auth.css`** (Tambahkan untuk tombol google, sesuaikan jika ada yang bertabrakan)
```css
/* css/auth.css - TAMBAHAN */

.auth-container .btn-submit.btn-google {
    background-color: #DB4437; /* Warna Google */
    margin-top: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
}
.auth-container .btn-submit.btn-google:hover {
    background-color: #c23321;
}

.auth-container .btn-submit .fab.fa-google {
    margin-right: 8px;
    font-size: 1.1em; /* Sesuaikan ukuran ikon */
}

html.dark-theme-active .auth-container .btn-submit.btn-google {
    background-color: #e57373; /* Warna Google lebih terang untuk dark mode */
    color: #121212; /* Teks gelap agar kontras */
}
html.dark-theme-active .auth-container .btn-submit.btn-google:hover {
    background-color: #ef5350;
}
```

---
**Langkah 4: Tutorial Tambahan**

1.  **Setup Google Cloud Console untuk OAuth 2.0 (Login dengan Google):**
    *   Buka [Google Cloud Console](https://console.cloud.google.com/).
    *   Buat Proyek Baru (atau pilih yang sudah ada).
    *   Navigasi ke "APIs & Services" > "Credentials".
    *   Klik "Create Credentials" > "OAuth client ID".
    *   Jika diminta, konfigurasikan "OAuth consent screen" terlebih dahulu:
        *   Pilih "External" dan klik "Create".
        *   Isi "App name" (misal: List In Anda), "User support email", dan "Developer contact information". Klik "Save and Continue".
        *   Di bagian "Scopes", klik "Add or Remove Scopes". Cari dan tambahkan scope `.../auth/userinfo.email` dan `.../auth/userinfo.profile`. Klik "Update", lalu "Save and Continue".
        *   Di bagian "Test users", tambahkan alamat email Gmail Anda untuk pengujian. Klik "Save and Continue".
        *   Kembali ke "Credentials".
    *   Pilih "Web application" untuk Application type.
    *   Beri nama (misal: ListIn Web Client).
    *   Di "Authorized JavaScript origins", tambahkan URL dasar aplikasi Anda (misal: `http://localhost`).
    *   Di "Authorized redirect URIs", tambahkan URL callback Anda: `http://localhost/nama_folder_proyek_anda/google_login_callback.php`. **Pastikan ini sama persis dengan `GOOGLE_REDIRECT_URI` di `config.php`**.
    *   Klik "Create". Anda akan mendapatkan Client ID dan Client Secret.
    *   Salin Client ID dan Client Secret ini ke `config/config.php` pada `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET`.
    *   Pastikan "Google People API" dan "Identity Toolkit API" (atau cukup "Google Sign-In API") aktif di "APIs & Services" > "Library".

2.  **Setup Pengiriman Email dengan Gmail (PHPMailer):**
    *   Email `listinproject@gmail.com` dengan password `listin12312` yang Anda sebutkan.
    *   **Keamanan Rendah (Less Secure App Access):**
        *   Buka akun Gmail `listinproject@gmail.com`.
        *   Pergi ke [myaccount.google.com/lesssecureapps](https://myaccount.google.com/lesssecureapps).
        *   Aktifkan "Allow less secure apps". **PERINGATAN:** Ini kurang aman.
    *   **App Password (Lebih Direkomendasikan jika Akun Memiliki 2-Step Verification):**
        *   Aktifkan 2-Step Verification untuk akun `listinproject@gmail.com`.
        *   Pergi ke [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).
        *   Pilih "Mail" untuk aplikasi dan "Other (Custom name)" untuk perangkat, beri nama (misal: ListIn App).
        *   Google akan generate 16-digit App Password. Gunakan password ini di `MAIL_PASSWORD` dalam `config.php` **BUKAN** password login Gmail Anda. Simpan App Password ini dengan aman karena hanya ditampilkan sekali.
    *   Pastikan konfigurasi SMTP di `config.php` (MAIL\_HOST, MAIL\_PORT, MAIL\_ENCRYPTION) sudah benar untuk Gmail.

3.  **Setup Cron Job untuk `send_deadline_emails.php`:**
    *   Cron job adalah task scheduler di sistem berbasis Unix/Linux. Jika Anda menggunakan shared hosting, biasanya ada panel kontrol (cPanel, Plesk) untuk mengatur cron job. Jika VPS, Anda atur via command line.
    *   Tujuannya adalah menjalankan skrip `send_deadline_emails.php` secara periodik (misal, sekali sehari pada tengah malam atau pagi hari).
    *   **Contoh Perintah Cron (Linux):**
        ```bash
        0 1 * * * /usr/bin/php /path/to/your/project/send_deadline_emails.php > /dev/null 2>&1
        ```
        *   `0 1 * * *`: Artinya jalankan setiap hari jam 01:00 pagi.
        *   `/usr/bin/php`: Path ke PHP CLI interpreter Anda (bisa berbeda).
        *   `/path/to/your/project/send_deadline_emails.php`: Path absolut ke skrip Anda.
        *   `> /dev/null 2>&1`: Mengarahkan output (standar dan error) ke null agar tidak membuat file log yang besar (opsional, bisa diarahkan ke file log jika ingin).
    *   **Di cPanel:** Cari "Cron Jobs". Pilih interval (misal, "Once Per Day"). Masukkan perintah: `php -q /home/your_cpanel_user/public_html/nama_folder_proyek_anda/send_deadline_emails.php`. Path bisa berbeda, cek dokumentasi hosting Anda.
    *   **Windows Task Scheduler:** Jika server Anda Windows, gunakan Task Scheduler untuk menjalankan `php.exe C:\path\to\your\project\send_deadline_emails.php`.

---
Ini adalah kerangka besar. Setiap bagian memerlukan pengujian yang cermat. Pastikan path, URL, dan kredensial di `config.php` sudah benar. Mulai dengan satu fitur, uji, lalu lanjut ke fitur berikutnya.

**Catatan Penting Terakhir:**
*   Kode di atas belum termasuk penanganan error yang sangat mendalam atau validasi sisi klien yang canggih untuk setiap field. Ini adalah dasar fungsional.
*   Keamanan adalah aspek penting. Selalu sanitasi input dan output (htmlspecialchars), gunakan prepared statements.
*   Untuk `ajax_clear_all_notifications.php`, pastikan nama filenya benar. Di daftar file Anda ada `ajax_clear_all_notifictions.php` (typo).

Semoga berhasil dengan implementasinya!
