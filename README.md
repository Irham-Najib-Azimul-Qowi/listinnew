Oke, ini adalah permintaan yang sangat besar dan kompleks, melibatkan banyak integrasi dan fitur baru. Saya akan berusaha sebaik mungkin untuk memberikan kerangka kerja dan kode inti yang Anda butuhkan. Namun, penting untuk dipahami bahwa implementasi penuh dan debugging fitur-fitur ini (terutama yang melibatkan layanan eksternal seperti Google OAuth, pengiriman email, dan cron job) akan memerlukan waktu dan upaya yang signifikan dari sisi Anda.

Saya akan memberikan kode selengkap mungkin. Untuk bagian yang memerlukan konfigurasi eksternal (Google API, SMTP, Cron Job), saya akan berikan panduan di akhir.

**PENTING:**
1.  **Backup Proyek Anda:** Sebelum melakukan perubahan besar, selalu buat backup dari seluruh proyek dan database Anda.
2.  **Composer:** Anda akan memerlukan Composer untuk mengelola dependensi PHP seperti Google API Client, PHPMailer, dan TCPDF. Jika belum terinstal, silakan instal terlebih dahulu.
3.  **Testing Bertahap:** Implementasikan dan uji setiap fitur satu per satu. Jangan coba mengintegrasikan semuanya sekaligus.

---

**Struktur File yang Akan Ditambahkan/Dimodifikasi:**

```
.
├── css/
│   ├── auth.css
│   └── style.css
├── images/
│   ├── auth-bg.jpg
│   └── placeholder-profile.png
├── includes/
│   ├── db.php
│   ├── footer.php
│   ├── header.php
│   ├── sidebar.php
│   └── task_helper.php
├── uploads/
│   └── profile_pictures/ (buat folder ini, pastikan writable)
├── vendor/ (dibuat oleh Composer)
├── .htaccess (OPSIONAL, untuk URL cantik jika diinginkan nanti)
├── ajax_clear_all_notifications.php  (revisi nama file dari permintaan asli)
├── ajax_handler.php
├── ajax_update_task_status.php
├── clear_notifications.php (sepertinya duplikat dengan ajax_clear_all_notifications.php, pilih salah satu)
├── config.php (BARU - untuk konfigurasi Google & Email)
├── cron_send_reports.php (BARU - untuk cron job laporan email)
├── dashboard.php
├── db_listin.sql
├── edit_profil.php
├── edit_tugas.php
├── forgot_password.php (BARU)
├── generate_report_pdf.php (BARU - atau bisa jadi fungsi di dalam laporan.php/cron_send_reports.php)
├── google_callback.php (BARU)
├── google_login.php (BARU - skrip untuk redirect ke Google)
├── laporan.php (BARU)
├── login.php
├── logout.php
├── manajemen_tugas.php
├── mark_notifications_viewed.php
├── profil.php
├── register.php
├── reset_password.php (BARU)
├── riwayat.php
├── send_email.php (BARU - helper untuk kirim email)
├── tambah_tugas.php
├── ubah_password.php
└── composer.json (BARU - untuk dependensi PHP)
```

---
**Langkah 1: Persiapan & Instalasi Library**

1.  Buat file `composer.json` di root proyek Anda:
    ```json
    {
        "name": "yourname/listin-project",
        "description": "A to-do list application with advanced features.",
        "require": {
            "phpmailer/phpmailer": "^6.8",
            "google/apiclient": "^2.15",
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

2.  Buka terminal/command prompt di root proyek Anda, lalu jalankan:
    ```bash
    composer install
    ```
    Ini akan mengunduh library yang dibutuhkan ke dalam folder `vendor/`.

3.  Buat folder `uploads/profile_pictures/`. Pastikan folder ini dapat ditulis oleh server web Anda (misalnya, `chmod 775 uploads/profile_pictures`).

---

**Langkah 2: Modifikasi Database (`db_listin.sql`)**

Tambahkan kolom dan tabel baru ke `db_listin.sql` Anda, lalu impor ulang atau jalankan query ALTER TABLE di phpMyAdmin/alat database Anda.

```sql
-- db_listin.sql (Tambahan & Modifikasi)

-- Hapus dulu constraint foreign key jika ada, untuk memodifikasi users
-- ALTER TABLE tasks DROP FOREIGN KEY tasks_ibfk_1; (Jika error saat modifikasi users, jalankan ini dulu)

-- Modifikasi tabel users
ALTER TABLE `users`
  ADD COLUMN `google_id` VARCHAR(255) DEFAULT NULL AFTER `profile_image`,
  ADD COLUMN `is_google_verified` TINYINT(1) DEFAULT 0 AFTER `google_id`,
  ADD COLUMN `reset_token` VARCHAR(255) DEFAULT NULL AFTER `is_google_verified`,
  ADD COLUMN `reset_token_expires_at` DATETIME DEFAULT NULL AFTER `reset_token`,
  ADD COLUMN `report_frequency` ENUM('none','daily','weekly','monthly') NOT NULL DEFAULT 'none' AFTER `reset_token_expires_at`,
  ADD COLUMN `last_report_sent_at` TIMESTAMP NULL DEFAULT NULL AFTER `report_frequency`,
  ADD UNIQUE INDEX `google_id_unique` (`google_id`);

-- Jika Anda menjalankan ALTER TABLE tasks DROP FOREIGN KEY sebelumnya, tambahkan kembali:
-- ALTER TABLE `tasks` ADD CONSTRAINT `tasks_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE;


-- Tidak perlu tabel password_resets terpisah jika token disimpan di users table
-- Tabel report_schedules juga tidak perlu jika info disimpan di users table

-- Pastikan kolom last_overdue_notif_sent dan last_deadline_email_sent ada di tasks
ALTER TABLE `tasks`
  ADD COLUMN `last_deadline_email_sent` TIMESTAMP NULL DEFAULT NULL AFTER `last_overdue_notif_sent`;

-- (Struktur tabel tasks dan users lainnya tetap sama dari file asli Anda)
-- Pastikan untuk menjalankan kembali CREATE TABLE jika Anda membuat database baru.
-- Jika memodifikasi yang sudah ada, cukup ALTER TABLE di atas.

-- Contoh struktur lengkap jika membuat ulang (gabungkan dengan SQL Anda):
DROP TABLE IF EXISTS `tasks`;
DROP TABLE IF EXISTS `users`;

CREATE TABLE IF NOT EXISTS `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) DEFAULT NULL, /* Bisa NULL jika login via Google */
  `profile_image` varchar(255) DEFAULT 'images/placeholder-profile.png',
  `google_id` VARCHAR(255) DEFAULT NULL,
  `is_google_verified` TINYINT(1) DEFAULT 0,
  `reset_token` VARCHAR(255) DEFAULT NULL,
  `reset_token_expires_at` DATETIME DEFAULT NULL,
  `report_frequency` ENUM('none','daily','weekly','monthly') NOT NULL DEFAULT 'none',
  `last_report_sent_at` TIMESTAMP NULL DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`),
  UNIQUE KEY `google_id_unique` (`google_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

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
  `last_deadline_email_sent` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  KEY `status` (`status`),
  KEY `due_date` (`due_date`),
  CONSTRAINT `tasks_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

```

---
**Langkah 3: File Konfigurasi Baru (`config.php`)**

Buat file `includes/config.php`:
```php
<?php
// includes/config.php

// Google API Configuration
define('GOOGLE_CLIENT_ID', 'MASUKKAN_CLIENT_ID_ANDA_DISINI'); // Ganti dengan Client ID Anda
define('GOOGLE_CLIENT_SECRET', 'MASUKKAN_CLIENT_SECRET_ANDA_DISINI'); // Ganti dengan Client Secret Anda
define('GOOGLE_REDIRECT_URI', 'http://localhost/nama_folder_proyek_anda/google_callback.php'); // Sesuaikan URL

// Email Configuration (PHPMailer)
define('SMTP_HOST', 'smtp.gmail.com'); // Atau SMTP host lain
define('SMTP_USERNAME', 'listinproject@gmail.com'); // Email pengirim
define('SMTP_PASSWORD', 'listin12312'); // Password email (atau App Password jika pakai Gmail 2FA)
define('SMTP_PORT', 587); // Umumnya 587 untuk TLS, 465 untuk SSL
define('SMTP_SECURE', 'tls'); // 'tls' atau 'ssl'
define('EMAIL_FROM_ADDRESS', 'listinproject@gmail.com');
define('EMAIL_FROM_NAME', 'List In Application');

// Site URL (untuk link di email, dll)
define('BASE_URL', 'http://localhost/nama_folder_proyek_anda'); // Sesuaikan

// Pastikan session dimulai jika belum
if (session_status() == PHP_SESSION_NONE) {
    session_start();
}

// Autoload Composer dependencies
require_once __DIR__ . '/../vendor/autoload.php';
?>
```
**PENTING:** Ganti `MASUKKAN_CLIENT_ID_ANDA_DISINI`, `MASUKKAN_CLIENT_SECRET_ANDA_DISINI`, `http://localhost/nama_folder_proyek_anda/` dengan nilai yang benar.

---
**Langkah 4: Helper Pengiriman Email Baru (`send_email.php`)**

Buat file `includes/send_email.php`:
```php
<?php
// includes/send_email.php
require_once __DIR__ . '/config.php'; // Untuk konstanta SMTP dan autoload PHPMailer

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

function send_email($to, $subject, $htmlBody, $altBody = '', $attachments = []) {
    $mail = new PHPMailer(true);
    try {
        // Server settings
        // $mail->SMTPDebug = SMTP::DEBUG_SERVER; // Enable verbose debug output untuk development
        $mail->isSMTP();
        $mail->Host       = SMTP_HOST;
        $mail->SMTPAuth   = true;
        $mail->Username   = SMTP_USERNAME;
        $mail->Password   = SMTP_PASSWORD;
        $mail->SMTPSecure = SMTP_SECURE; // PHPMailer::ENCRYPTION_SMTPS jika pakai SMTPSecure const
        $mail->Port       = SMTP_PORT;
        $mail->CharSet    = 'UTF-8';

        // Recipients
        $mail->setFrom(EMAIL_FROM_ADDRESS, EMAIL_FROM_NAME);
        $mail->addAddress($to);

        // Attachments
        foreach ($attachments as $attachment) {
            if (isset($attachment['path']) && isset($attachment['name'])) {
                $mail->addAttachment($attachment['path'], $attachment['name']);
            }
        }

        // Content
        $mail->isHTML(true);
        $mail->Subject = $subject;
        $mail->Body    = $htmlBody;
        $mail->AltBody = $altBody ?: strip_tags($htmlBody);

        $mail->send();
        return true;
    } catch (Exception $e) {
        error_log("Message could not be sent. Mailer Error: {$mail->ErrorInfo}"); // Log error
        // add_notification("Gagal mengirim email: {$mail->ErrorInfo}", "error"); // Mungkin tidak cocok di sini, karena ini helper
        return false;
    }
}

function get_email_template_header() {
    return "<!DOCTYPE html>
    <html lang='id'>
    <head>
        <meta charset='UTF-8'>
        <meta name='viewport' content='width=device-width, initial-scale=1.0'>
        <style>
            body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; background-color: #f4f4f4; margin: 0; padding: 0; }
            .container { max-width: 600px; margin: 20px auto; padding: 20px; background-color: #fff; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
            .header { background-color: #7e47b8; color: white; padding: 15px; text-align: center; border-top-left-radius: 8px; border-top-right-radius: 8px; }
            .header h1 { margin: 0; font-size: 24px; }
            .content { padding: 20px; }
            .content h2 { color: #7e47b8; }
            .content p { margin-bottom: 15px; }
            .task-list { list-style: none; padding: 0; }
            .task-list li { background-color: #f9f9f9; border: 1px solid #eee; padding: 10px; margin-bottom: 8px; border-radius: 4px; }
            .task-list li strong { color: #555; }
            .button { display: inline-block; background-color: #7e47b8; color: white; padding: 10px 15px; text-decoration: none; border-radius: 5px; margin-top: 15px; }
            .footer { text-align: center; margin-top: 20px; padding: 15px; font-size: 0.9em; color: #777; border-top: 1px solid #eee; }
        </style>
    </head>
    <body>
        <div class='container'>
            <div class='header'><h1>List In</h1></div>";
}

function get_email_template_footer() {
    return "<div class='footer'>
                <p>&copy; " . date('Y') . " List In. Semua hak cipta dilindungi.</p>
                <p>Ini adalah email otomatis, mohon untuk tidak membalas.</p>
            </div>
        </div>
    </body>
    </html>";
}

?>
```

---
**Langkah 5: Modifikasi File yang Ada**

**5.1. `includes/db.php`**
Ganti `session_start();` dengan kode yang ada di `config.php` (karena `config.php` sudah memulainya). Cukup pastikan `require_once __DIR__ . '/config.php';` ada di awal file yang butuh koneksi DB dan session.
Isi `db.php` Anda sudah baik.

**5.2. `includes/header.php`**
Ini akan jadi sangat panjang.
*   Tambahkan `require_once __DIR__ . '/config.php';` dan `require_once __DIR__ . '/send_email.php';` di paling atas.
*   Modifikasi fungsi `add_notification`.
*   Modifikasi logika cek deadline & overdue untuk mengirim email.

```php
<?php
// includes/header.php
require_once __DIR__ . '/config.php'; // Mengandung session_start() dan autoload
require_once __DIR__ . '/db.php';     // Untuk koneksi DB $conn
require_once __DIR__ . '/send_email.php'; // Untuk fungsi send_email()

$current_page = basename($_SERVER['SCRIPT_NAME']);
$current_user_id = $_SESSION['user_id'] ?? null;
$current_user_email = ''; // Untuk pengiriman email
$current_user_profile_image = BASE_URL . '/images/placeholder-profile.png'; // Gunakan BASE_URL

if ($current_user_id && isset($conn)) {
    $stmt_user_header = $conn->prepare("SELECT username, email, profile_image FROM users WHERE id = ?");
    if ($stmt_user_header) {
        $stmt_user_header->bind_param("i", $current_user_id);
        $stmt_user_header->execute();
        $result_user_header = $stmt_user_header->get_result();
        if ($user_data_header = $result_user_header->fetch_assoc()) {
            $current_user_profile_image = !empty($user_data_header['profile_image']) && file_exists(__DIR__ . '/../' . $user_data_header['profile_image'])
                ? BASE_URL . '/' . $user_data_header['profile_image']
                : BASE_URL . '/images/placeholder-profile.png';
            $_SESSION['username'] = $user_data_header['username']; // Update session username
            $current_user_email = $user_data_header['email'];
        }
        $stmt_user_header->close();
    }
}

function add_notification($message, $type = 'info', $send_email_flag = false, $email_details = []) {
    if (!isset($_SESSION['notification_messages'])) {
        $_SESSION['notification_messages'] = [];
    }
    $new_notification = ['message' => $message, 'type' => $type, 'time' => time()];

    $is_duplicate_notif = false;
    if ($type === 'deadline_soon' || $type === 'overdue_task') {
        // ... (logika duplikasi notifikasi Anda sudah baik) ...
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

        // Kirim email jika flag true dan ada detail email
        if ($send_email_flag && !empty($email_details) && !empty($email_details['to'])) {
            send_email($email_details['to'], $email_details['subject'], $email_details['body']);
        }
    }
}

// Cek Tugas Deadline H-1
if ($current_user_id && $current_user_email && isset($conn) && !in_array($current_page, ['login.php', 'register.php'])) {
    $tomorrow_date = date('Y-m-d', strtotime('+1 day'));
    $today_for_notif_check_h1 = date('Y-m-d');
    // Gunakan kolom last_deadline_email_sent untuk email, session untuk notif web
    $notif_key_deadline_h1_web = 'deadline_h1_notif_sent_web_' . $today_for_notif_check_h1 . '_uid' . $current_user_id;

    // Cek untuk notifikasi web (via session)
    if (!isset($_SESSION[$notif_key_deadline_h1_web])) {
        $stmt_deadline_check_h1_web = $conn->prepare("SELECT title FROM tasks WHERE user_id = ? AND due_date = ? AND status != 'Completed'");
        // ... (logika notifikasi web Anda yang sudah ada untuk H-1) ...
        if ($stmt_deadline_check_h1_web) {
            $stmt_deadline_check_h1_web->bind_param("is", $current_user_id, $tomorrow_date);
            $stmt_deadline_check_h1_web->execute();
            $result_deadline_tasks_h1_web = $stmt_deadline_check_h1_web->get_result();
            $tasks_deadline_tomorrow_web = [];
            while ($task_for_deadline_notif_h1_web = $result_deadline_tasks_h1_web->fetch_assoc()) {
                $tasks_deadline_tomorrow_web[] = htmlspecialchars($task_for_deadline_notif_h1_web['title']);
            }
            $stmt_deadline_check_h1_web->close();

            if (!empty($tasks_deadline_tomorrow_web)) {
                $task_list_str_h1_web = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_deadline_tomorrow_web));
                $message_plural_h1_web = count($tasks_deadline_tomorrow_web) > 1 ? "Tugas-tugas" : "Tugas";
                $deadline_message_h1_web = "<span class='message-content'><strong>PERHATIAN:</strong> $message_plural_h1_web $task_list_str_h1_web akan jatuh tempo besok!</span>";
                add_notification($deadline_message_h1_web, "deadline_soon"); // Hanya notif web
                $_SESSION[$notif_key_deadline_h1_web] = true;
            }
        }
    }

    // Cek untuk pengiriman email H-1 (gunakan kolom last_deadline_email_sent)
    $stmt_deadline_email = $conn->prepare(
        "SELECT id, title, description, DATE_FORMAT(due_date, '%W, %e %M %Y') as due_date_formatted_email
         FROM tasks WHERE user_id = ? AND due_date = ? AND status != 'Completed'
         AND (last_deadline_email_sent IS NULL OR DATE(last_deadline_email_sent) < CURDATE())"
    );
    if ($stmt_deadline_email) {
        $stmt_deadline_email->bind_param("is", $current_user_id, $tomorrow_date);
        $stmt_deadline_email->execute();
        $result_deadline_tasks_email = $stmt_deadline_email->get_result();
        $tasks_for_email_h1 = [];
        $task_ids_to_update_deadline_email = [];
        while ($task_email = $result_deadline_tasks_email->fetch_assoc()) {
            $tasks_for_email_h1[] = $task_email;
            $task_ids_to_update_deadline_email[] = $task_email['id'];
        }
        $stmt_deadline_email->close();

        if (!empty($tasks_for_email_h1)) {
            $email_subject_h1 = "Pengingat: Tugas Akan Jatuh Tempo Besok di List In";
            $email_body_h1 = get_email_template_header();
            $email_body_h1 .= "<div class='content'>
                                <h2>Halo " . htmlspecialchars($_SESSION['username']) . ",</h2>
                                <p>Ini adalah pengingat bahwa tugas-tugas berikut akan jatuh tempo besok, <strong>" . date('d M Y', strtotime($tomorrow_date)) . "</strong>. Mohon segera selesaikan atau perbarui statusnya di aplikasi List In.</p>
                                <ul class='task-list'>";
            foreach ($tasks_for_email_h1 as $task_item_email) {
                $email_body_h1 .= "<li><strong>" . htmlspecialchars($task_item_email['title']) . "</strong><br>" .
                                  "<small>Deskripsi: " . (empty($task_item_email['description']) ? '<em>Tidak ada</em>' : nl2br(htmlspecialchars($task_item_email['description']))) . "</small></li>";
            }
            $email_body_h1 .= "</ul>
                               <p>Anda dapat mengelola tugas-tugas Anda dengan mengunjungi tautan berikut:</p>
                               <a href='" . BASE_URL . "/manajemen_tugas.php' class='button'>Kelola Tugas Saya</a>
                               <p>Terima kasih telah menggunakan List In untuk membantu produktivitas Anda!</p>
                               <p>Salam,<br>Tim List In</p>
                               </div>";
            $email_body_h1 .= get_email_template_footer();

            if (send_email($current_user_email, $email_subject_h1, $email_body_h1)) {
                if (!empty($task_ids_to_update_deadline_email)) {
                    $ids_placeholder_h1 = implode(',', array_fill(0, count($task_ids_to_update_deadline_email), '?'));
                    $stmt_update_deadline_sent = $conn->prepare(
                        "UPDATE tasks SET last_deadline_email_sent = NOW() WHERE id IN ($ids_placeholder_h1) AND user_id = ?"
                    );
                    if ($stmt_update_deadline_sent) {
                        $types_update_h1 = str_repeat('i', count($task_ids_to_update_deadline_email)) . 'i';
                        $params_update_h1 = array_merge($task_ids_to_update_deadline_email, [$current_user_id]);
                        $stmt_update_deadline_sent->bind_param($types_update_h1, ...$params_update_h1);
                        $stmt_update_deadline_sent->execute();
                        $stmt_update_deadline_sent->close();
                    }
                }
            }
        }
    }
}


// Cek Tugas Terlewat DL (Overdue)
if ($current_user_id && $current_user_email && isset($conn) && !in_array($current_page, ['login.php', 'register.php'])) {
    $today_for_overdue_check = date('Y-m-d');
    // Gunakan kolom last_overdue_notif_sent untuk email, session untuk notif web
    $notif_key_overdue_web = 'overdue_notif_sent_web_' . $today_for_overdue_check . '_uid' . $current_user_id;

    // Cek untuk notifikasi web (via session)
    if (!isset($_SESSION[$notif_key_overdue_web])) {
        $stmt_overdue_check_web = $conn->prepare(
            "SELECT id, title FROM tasks
             WHERE user_id = ? AND due_date < CURDATE() AND status != 'Completed'"
             // Tidak perlu cek last_overdue_notif_sent untuk notif web, cukup session flag
        );
        // ... (logika notifikasi web Anda yang sudah ada untuk overdue) ...
        if ($stmt_overdue_check_web) {
            $stmt_overdue_check_web->bind_param("i", $current_user_id);
            $stmt_overdue_check_web->execute();
            $result_overdue_tasks_web = $stmt_overdue_check_web->get_result();
            $tasks_overdue_web = [];
            while ($task_overdue_web_item = $result_overdue_tasks_web->fetch_assoc()) {
                $tasks_overdue_web[] = htmlspecialchars($task_overdue_web_item['title']);
            }
            $stmt_overdue_check_web->close();

            if (!empty($tasks_overdue_web)) {
                $task_list_str_overdue_web = implode(", ", array_map(function($title) { return "\"".$title."\""; }, $tasks_overdue_web));
                $message_plural_overdue_web = count($tasks_overdue_web) > 1 ? "Tugas-tugas" : "Tugas";
                $overdue_message_web = "<span class='message-content'><strong>TERLEWAT:</strong> $message_plural_overdue_web $task_list_str_overdue_web telah melewati batas waktu!</span>";
                add_notification($overdue_message_web, "deadline_soon"); // Tipe bisa 'overdue_task'
                $_SESSION[$notif_key_overdue_web] = true;
            }
        }
    }

    // Cek untuk pengiriman email Overdue (gunakan kolom last_overdue_notif_sent)
    $stmt_overdue_email = $conn->prepare(
        "SELECT id, title, description, DATE_FORMAT(due_date, '%W, %e %M %Y') as due_date_formatted_email
         FROM tasks WHERE user_id = ? AND due_date < CURDATE() AND status != 'Completed'
         AND (last_overdue_notif_sent IS NULL OR DATE(last_overdue_notif_sent) < CURDATE())"
    );
    if ($stmt_overdue_email) {
        $stmt_overdue_email->bind_param("i", $current_user_id);
        $stmt_overdue_email->execute();
        $result_overdue_tasks_email = $stmt_overdue_email->get_result();
        $tasks_for_email_overdue = [];
        $task_ids_to_update_overdue_email = [];
        while ($task_email_ov = $result_overdue_tasks_email->fetch_assoc()) {
            $tasks_for_email_overdue[] = $task_email_ov;
            $task_ids_to_update_overdue_email[] = $task_email_ov['id'];
        }
        $stmt_overdue_email->close();

        if (!empty($tasks_for_email_overdue)) {
            $email_subject_overdue = "Pemberitahuan: Tugas Terlewat Batas Waktu di List In";
            $email_body_overdue = get_email_template_header();
            $email_body_overdue .= "<div class='content'>
                                <h2>Halo " . htmlspecialchars($_SESSION['username']) . ",</h2>
                                <p>Kami ingin memberitahukan bahwa tugas-tugas berikut telah melewati batas waktu pengerjaan dan statusnya belum 'Selesai'.</p>
                                <ul class='task-list'>";
            foreach ($tasks_for_email_overdue as $task_item_email_ov) {
                $email_body_overdue .= "<li><strong>" . htmlspecialchars($task_item_email_ov['title']) . "</strong> (Batas Waktu: " . htmlspecialchars($task_item_email_ov['due_date_formatted_email']) . ")<br>" .
                                  "<small>Deskripsi: " . (empty($task_item_email_ov['description']) ? '<em>Tidak ada</em>' : nl2br(htmlspecialchars($task_item_email_ov['description']))) . "</small></li>";
            }
            $email_body_overdue .= "</ul>
                               <p>Mohon segera periksa dan perbarui status tugas-tugas tersebut di aplikasi List In. Keterlambatan dapat mempengaruhi progres Anda.</p>
                               <a href='" . BASE_URL . "/manajemen_tugas.php' class='button'>Kelola Tugas Saya</a>
                               <p>Terima kasih atas perhatiannya.</p>
                               <p>Salam,<br>Tim List In</p>
                               </div>";
            $email_body_overdue .= get_email_template_footer();

            if (send_email($current_user_email, $email_subject_overdue, $email_body_overdue)) {
                if (!empty($task_ids_to_update_overdue_email)) {
                    $ids_placeholder_ov = implode(',', array_fill(0, count($task_ids_to_update_overdue_email), '?'));
                    $stmt_update_overdue_sent = $conn->prepare(
                        "UPDATE tasks SET last_overdue_notif_sent = NOW() WHERE id IN ($ids_placeholder_ov) AND user_id = ?"
                    );
                    if ($stmt_update_overdue_sent) {
                        $types_update_ov = str_repeat('i', count($task_ids_to_update_overdue_email)) . 'i';
                        $params_update_ov = array_merge($task_ids_to_update_overdue_email, [$current_user_id]);
                        $stmt_update_overdue_sent->bind_param($types_update_ov, ...$params_update_ov);
                        $stmt_update_overdue_sent->execute();
                        $stmt_update_overdue_sent->close();
                    }
                }
            }
        }
    }
}


$has_unread_badge_for_icon = $_SESSION['has_unread_notifications_badge'] ?? false;

// ... (sisa kode header.php Anda, seperti format_tanggal_indonesia_header, HTML head, body, header utama)
// Pastikan path CSS dan JS menggunakan BASE_URL jika perlu, atau relatif jika struktur folder tetap.
// Contoh untuk CSS: <link rel="stylesheet" href="<?php echo BASE_URL; ?>/css/style.css?v=<?php echo time(); ?>">
// Namun, jika header.php ada di includes/, maka path relatif seperti ../css/style.css atau css/style.css (jika BASE_URL diatur dengan benar di HTML base tag atau server) mungkin sudah cukup.
// Untuk script FOUC prevent di <head>, itu sudah baik.
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <base href="<?php echo BASE_URL . '/'; ?>"> <!-- Penting untuk path relatif -->
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
    <title><?php echo isset($page_title) ? htmlspecialchars($page_title) . ' - List In' : 'List In'; ?></title>
</head>
<body class="<?php echo in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php']) ? 'auth-page' : ''; ?> <?php if (isset($_SESSION['sidebar_hidden']) && $_SESSION['sidebar_hidden'] === 'true' && !in_array($current_page, ['login.php', 'register.php'])) echo 'sidebar-hidden'; ?>">
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
                <p><?php echo htmlspecialchars(format_tanggal_indonesia_header(date('Y-m-d'))[0] ?? ''); ?></p>
                <span><?php echo htmlspecialchars(format_tanggal_indonesia_header(date('Y-m-d'))[1] ?? ''); ?></span>
            </div>
        </div>
    </header>
    <div class="content">
    <?php endif; ?>
```
*Catatan: Fungsi `format_tanggal_indonesia_header` Anda mengembalikan array. Saya sesuaikan cara pemanggilannya.*

**5.3. `includes/footer.php`**
*   Pastikan `$current_page` terdefinisi jika footer dipanggil dari halaman auth.
*   Logika sidebar toggle sudah ada, pastikan `sidebar_hidden` di-set di session atau localStorage.

```php
<?php
// includes/footer.php
$current_page = $current_page ?? basename($_SERVER['SCRIPT_NAME']); // Pastikan terdefinisi
?>
<?php if (!in_array($current_page, ['login.php', 'register.php', 'forgot_password.php', 'reset_password.php'])): ?>
    </div> <!-- penutup div.content -->

    <div id="notification-popup">
        <!-- ... (konten notifikasi Anda sudah baik) ... -->
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
                        else if($notif_item['type'] == 'deadline_soon' || $notif_item['type'] == 'overdue_task') $message_class_popup = 'notif-deadline_soon';
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

            // ... (fungsi togglePopup Anda) ...
            function togglePopup(popupElement, iconElement, otherPopupElement) {
                if (otherPopupElement && otherPopupElement.classList.contains('show')) {
                    otherPopupElement.classList.remove('show');
                }
                popupElement.classList.toggle('show');

                if (popupElement === notificationPopup && popupElement.classList.contains('show')) {
                    if (bellIcon && bellIcon.classList.contains('has-notif')) {
                        fetch('mark_notifications_viewed.php') // Pastikan path ini benar dari root
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
                        fetch('ajax_clear_all_notifications.php') // Pastikan path ini benar
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
                const calendarContainerPopup = document.getElementById('calendar-container-popup');
                if(calendarContainerPopup){ // Cek elemen ada
                    flatpickr(calendarContainerPopup, { inline: true, dateFormat: "d/m/Y", locale: "id" });
                }
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

            // ... (sisa JavaScript Anda untuk flatpickr, deleteButtons, dll sudah baik) ...
            // Sesuaikan selector jika perlu, misal untuk report date range
            const dateInputs = document.querySelectorAll('input[type="text"][id$="Date"], input[type="text"][id$="DueDate"], input[type="text"][id*="DateRange"]');
            dateInputs.forEach(input => {
                let config = { dateFormat: "d/m/Y", locale: "id", allowInput: true };
                if (input.id === 'taskDueDate' || input.id === 'editTaskDueDate') {
                     config.minDate = "today";
                }
                // Tambahkan untuk date range di laporan.php
                if (input.id === 'filterHistoryDateRange' || input.id === 'filterPerformanceDateRange' || input.id === 'reportDateRange') {
                    config.mode = "range";
                } else if (input.id === 'filterDate') {
                    config.mode = "single";
                }
                flatpickr(input, config);
            });

            // Tema Gelap & Sidebar Toggle (logika Anda sudah baik, pastikan selector id benar)
            const themeToggleCheckbox = document.getElementById('themeToggleCheckbox');
            function updateThemeOnPage(theme) {
                // ... (logika Anda)
                 if (theme === 'dark-theme') {
                    htmlElement.classList.add('dark-theme-active');
                    if (themeToggleCheckbox) themeToggleCheckbox.checked = true;
                } else {
                    htmlElement.classList.remove('dark-theme-active');
                    if (themeToggleCheckbox) themeToggleCheckbox.checked = false;
                }
            }
            // ... (sisa logika tema Anda) ...
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


            const appTitleToggle = document.getElementById('appTitleToggle');
            const sidebarElement = document.getElementById('sidebar');

            if (appTitleToggle && sidebarElement) {
                function setSidebarState(isHidden) {
                    if (isHidden) {
                        bodyElement.classList.add('sidebar-hidden');
                        localStorage.setItem('sidebarHidden', 'true'); // Simpan ke localStorage
                    } else {
                        bodyElement.classList.remove('sidebar-hidden');
                        localStorage.setItem('sidebarHidden', 'false'); // Simpan ke localStorage
                    }
                }

                // Initial state based on localStorage and screen width
                const sidebarHiddenStored = localStorage.getItem('sidebarHidden') === 'true';
                if (window.innerWidth > 768) {
                    setSidebarState(sidebarHiddenStored);
                } else {
                    setSidebarState(false); // Always show on mobile load, but don't store this preference for desktop
                }

                appTitleToggle.addEventListener('click', () => {
                    if (window.innerWidth > 768) { // Only allow toggle on desktop
                        const isNowHidden = bodyElement.classList.toggle('sidebar-hidden');
                        localStorage.setItem('sidebarHidden', isNowHidden);
                    }
                });

                window.addEventListener('resize', () => {
                    if (window.innerWidth <= 768) {
                        // On mobile, ensure sidebar is not in 'hidden' state visually,
                        // but don't overwrite the desktop preference in localStorage.
                        bodyElement.classList.remove('sidebar-hidden');
                    } else {
                        // Re-apply stored state on resize to desktop
                        const sidebarHiddenStoredOnResize = localStorage.getItem('sidebarHidden') === 'true';
                        setSidebarState(sidebarHiddenStoredOnResize);
                    }
                });
            }

            // Klik card tugas (logika Anda sudah baik)
            // ...
             if (document.querySelector('.main-content-manajemen')) {
                const taskListContainerManajemen = document.getElementById('managementTaskList');

                if (taskListContainerManajemen) {
                    taskListContainerManajemen.addEventListener('click', function(event) {
                        // ... (logika Anda) ...
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

                        fetch('ajax_update_task_status.php', { // Pastikan path ini benar
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
<?php endif; ?>
</body>
</html>
<?php
if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
// Hapus output buffer jika ada, untuk menghindari masalah header
if (ob_get_level() > 0) {
    ob_end_flush();
}
?>
```

**5.4. `includes/sidebar.php`**
Tambahkan link ke halaman Laporan.
```php
<?php
// includes/sidebar.php
$current_page = $current_page ?? basename($_SERVER['SCRIPT_NAME']); // Pastikan terdefinisi
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

**5.5. `login.php`**
Tambahkan tombol "Login dengan Google" dan link "Lupa Password".
```php
<?php
// login.php
$page_title = "Masuk"; // Untuk <title> di header.php
require_once 'includes/config.php'; // Untuk GOOGLE_CLIENT_ID dan session
require_once 'includes/db.php';

$errors = [];

if (isset($_SESSION['user_id'])) {
    header("Location: dashboard.php");
    exit();
}

// Google Login URL
$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI);
$client->addScope("email");
$client->addScope("profile");
$google_login_url = $client->createAuthUrl();


if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['email'])) { // Pastikan ini form login biasa
    $email = trim($_POST['email']);
    $password = $_POST['password'];

    if (empty($email)) $errors[] = "Alamat email wajib diisi.";
    if (empty($password)) $errors[] = "Kata sandi wajib diisi.";

    if (empty($errors)) {
        $stmt = $conn->prepare("SELECT id, username, password, profile_image, is_google_verified FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($user = $result->fetch_assoc()) {
            // Jika akun dibuat via Google dan belum set password manual
            if ($user['is_google_verified'] == 1 && empty($user['password'])) {
                $errors[] = "Akun ini terdaftar melalui Google. Silakan masuk menggunakan Google atau atur kata sandi manual jika fitur tersebut tersedia.";
            } elseif (password_verify($password, $user['password'])) {
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

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Selamat Datang Kembali!</h2>
        <p class="subtitle">Masuk untuk melanjutkan dan mengatur tugas Anda.</p>

        <?php if (isset($_SESSION['success_message'])): ?>
            <div style="background-color: #d4edda; color: #155724; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
                <p><?php echo $_SESSION['success_message']; ?></p>
            </div>
            <?php unset($_SESSION['success_message']); ?>
        <?php endif; ?>
        <?php if (isset($_SESSION['error_message'])): ?>
            <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
                <p><?php echo $_SESSION['error_message']; ?></p>
            </div>
            <?php unset($_SESSION['error_message']); ?>
        <?php endif; ?>

        <?php if (!empty($errors)): ?>
            <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
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
            <div class="form-group" style="text-align: right; margin-bottom: 10px;">
                <a href="forgot_password.php" style="font-size: 0.8rem; color: #7e47b8;">Lupa Kata Sandi?</a>
            </div>
            <button type="submit" class="btn-submit">Masuk Akun</button>
        </form>
        <div style="text-align: center; margin: 15px 0; color: #777;">ATAU</div>
        <a href="<?php echo htmlspecialchars($google_login_url); ?>" class="btn-submit" style="background-color: #db4437; display: flex; align-items: center; justify-content: center; text-decoration:none;">
            <i class="fab fa-google" style="margin-right: 8px;"></i> Masuk dengan Google
        </a>
        <p class="auth-link">Belum punya akun? <a href="register.php">Daftar sekarang</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

**5.6. `register.php`**
Sudah cukup baik. Hanya tambahkan `$page_title`.
```php
<?php
// register.php
$page_title = "Daftar Akun";
require_once 'includes/config.php';
require_once 'includes/db.php';
// ... sisa kode register.php Anda ...
?>
<?php require_once 'includes/header.php'; ?>
    <!-- ... konten HTML form registrasi Anda ... -->
<?php require_once 'includes/footer.php'; ?>
```

**5.7. `profil.php`**
Tambahkan opsi untuk mengatur frekuensi laporan.
```php
<?php
// profil.php
$page_title = "Profil Saya";
require_once 'includes/header.php'; // Sudah ada config.php
require_once 'includes/sidebar.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];
$user_profile_data = null;

if (isset($conn) && $conn) {
    $stmt_profile = $conn->prepare("SELECT username, email, profile_image, created_at, report_frequency FROM users WHERE id = ?");
    if ($stmt_profile) {
        $stmt_profile->bind_param("i", $user_id);
        $stmt_profile->execute();
        $result_profile = $stmt_profile->get_result();
        $user_profile_data = $result_profile->fetch_assoc();
        $stmt_profile->close();
    } else {
        add_notification("Gagal mengambil data profil: " . $conn->error, "error");
    }
} else {
    add_notification("Koneksi database tidak tersedia.", "error");
}

if (!$user_profile_data) {
    $user_profile_data = ['username' => 'N/A', 'email' => 'N/A', 'profile_image' => 'images/placeholder-profile.png', 'created_at' => null, 'report_frequency' => 'none'];
}

$profile_image_display_path = (!empty($user_profile_data['profile_image']) && file_exists(__DIR__ . '/../' . $user_profile_data['profile_image']))
                            ? BASE_URL . '/' . $user_profile_data['profile_image']
                            : BASE_URL . '/images/placeholder-profile.png';
$member_since = 'N/A';
if ($user_profile_data['created_at']) {
    try {
        $date_joined = new DateTime($user_profile_data['created_at']);
        $member_since = "Bergabung sejak " . $date_joined->format('d F Y');
    } catch (Exception $e) { /* Biarkan N/A */ }
}

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['save_report_settings'])) {
    $report_frequency = $_POST['report_frequency'] ?? 'none';
    $valid_frequencies = ['none', 'daily', 'weekly', 'monthly'];
    if (in_array($report_frequency, $valid_frequencies)) {
        $stmt_update_freq = $conn->prepare("UPDATE users SET report_frequency = ? WHERE id = ?");
        if ($stmt_update_freq) {
            $stmt_update_freq->bind_param("si", $report_frequency, $user_id);
            if ($stmt_update_freq->execute()) {
                add_notification("Pengaturan laporan berhasil disimpan.", "success");
                $user_profile_data['report_frequency'] = $report_frequency; // Update data lokal
            } else {
                add_notification("Gagal menyimpan pengaturan laporan: " . $stmt_update_freq->error, "error");
            }
            $stmt_update_freq->close();
        }
    } else {
        add_notification("Frekuensi laporan tidak valid.", "error");
    }
}

?>
<?php // require_once 'includes/header.php'; // Sudah dipanggil di atas ?>

        <main class="main">
            <h2 class="page-title">Profil</h2>
            <?php
                // Menampilkan notifikasi dari session
                if (isset($_SESSION['notification_messages']) && !empty($_SESSION['notification_messages'])) {
                    foreach ($_SESSION['notification_messages'] as $key => $notif) {
                        $alert_type = $notif['type'] === 'success' ? 'success' : ($notif['type'] === 'error' ? 'danger' : 'info');
                        echo "<div class='alert alert-{$alert_type}' role='alert' style='margin-bottom:15px; padding:10px; border-radius:5px; background-color:" . ($alert_type == 'success' ? '#d4edda; color:#155724;' : ($alert_type == 'danger' ? '#f8d7da; color:#721c24;' : '#cce5ff; color:#004085;')) . "'>{$notif['message']}</div>";
                    }
                    unset($_SESSION['notification_messages']); // Hapus notifikasi setelah ditampilkan
                    unset($_SESSION['has_unread_notifications_badge']); // Reset badge juga
                }
            ?>

            <div class="profile-page-container">
                <div class="profile-info-card">
                    <br><br>
                    <img src="<?php echo htmlspecialchars($profile_image_display_path); ?>?t=<?php echo time(); ?>" alt="Foto Profil" id="profileImageMain">
                    <h2 id="profileName"><?php echo htmlspecialchars($user_profile_data['username']); ?></h2>
                    <p id="profileEmail"><?php echo htmlspecialchars($user_profile_data['email']); ?></p>
                    <p class="member-since"><?php echo $member_since; ?></p>
                </div>

                <div class="profile-actions-card">
                    <h3 class="card-title">Pengaturan Akun</h3>
                    <div class="profile-action-buttons">
                        <a href="edit_profil.php" class="btn btn-primary">
                            <i class="fas fa-user-edit"></i> Edit Informasi Profil
                        </a>
                        <a href="ubah_password.php" class="btn btn-secondary">
                            <i class="fas fa-key"></i> Ubah Kata Sandi
                        </a>
                    </div>

                    <h3 class="card-title" style="margin-top: 25px;">Preferensi Tampilan</h3>
                    <div class="profile-theme-settings">
                        <div class="theme-toggle-item">
                            <span><i class="fas fa-palette"></i> Tema Gelap</span>
                            <label class="theme-toggle-switch">
                                <input type="checkbox" id="themeToggleCheckbox">
                                <span class="theme-slider"></span>
                            </label>
                        </div>
                    </div>

                    <h3 class="card-title" style="margin-top: 25px;">Pengaturan Laporan Email</h3>
                    <form method="POST" action="profil.php">
                        <div class="form-group">
                            <label for="report_frequency" style="font-weight:normal; display:inline-block; margin-right:10px;">Kirim laporan progres ke email saya:</label>
                            <select name="report_frequency" id="report_frequency" style="padding: 8px; border-radius: 5px; border: 1px solid #ccc;">
                                <option value="none" <?php echo ($user_profile_data['report_frequency'] == 'none') ? 'selected' : ''; ?>>Jangan Kirim</option>
                                <option value="daily" <?php echo ($user_profile_data['report_frequency'] == 'daily') ? 'selected' : ''; ?>>Setiap Hari</option>
                                <option value="weekly" <?php echo ($user_profile_data['report_frequency'] == 'weekly') ? 'selected' : ''; ?>>Setiap Minggu</option>
                                <option value="monthly" <?php echo ($user_profile_data['report_frequency'] == 'monthly') ? 'selected' : ''; ?>>Setiap Bulan</option>
                            </select>
                        </div>
                        <button type="submit" name="save_report_settings" class="btn btn-primary" style="margin-top:10px; font-size:0.9rem; padding:8px 15px;">
                            <i class="fas fa-save"></i> Simpan Frekuensi Laporan
                        </button>
                    </form>
                </div>
            </div>
        </main>
<?php require_once 'includes/footer.php'; ?>
```

**5.8. `ubah_password.php`**
Jika pengguna login via Google dan belum punya password, halaman ini idealnya tidak bisa diakses atau formnya disembunyikan dengan pesan. Namun, untuk menjaga kesederhanaan, kita asumsikan mereka bisa set password jika mau.
```php
<?php
// ubah_password.php
$page_title = "Ubah Kata Sandi";
require_once 'includes/header.php';
require_once 'includes/sidebar.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];

$user_has_password = true; // Asumsi awal
$stmt_check_pass = $conn->prepare("SELECT password FROM users WHERE id = ?");
$stmt_check_pass->bind_param("i", $user_id);
$stmt_check_pass->execute();
$result_pass = $stmt_check_pass->get_result();
$user_pass_data = $result_pass->fetch_assoc();
$stmt_check_pass->close();
if ($user_pass_data && empty($user_pass_data['password'])) {
    $user_has_password = false; // User login via Google dan belum set password
}


if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $currentPassword = $_POST['currentPassword'] ?? ''; // Default ke string kosong jika tidak ada
    $newPassword = $_POST['newPassword'];
    $confirmNewPassword = $_POST['confirmNewPassword'];
    $validation_errors_found = false;

    // Jika user tidak punya password (misal dari Google login), field currentPassword tidak wajib
    if ($user_has_password && empty($currentPassword)) {
        add_notification("Kata sandi saat ini wajib diisi.", "error");
        $validation_errors_found = true;
    }
    if (empty($newPassword) || empty($confirmNewPassword)) {
        add_notification("Kata sandi baru dan konfirmasi wajib diisi.", "error");
        $validation_errors_found = true;
    }

    if (!$validation_errors_found) {
        $can_proceed_update = false;
        if ($user_has_password) {
            if ($user_pass_data && password_verify($currentPassword, $user_pass_data['password'])) {
                $can_proceed_update = true;
            } else {
                add_notification("Kata sandi saat ini salah.", "error");
                $validation_errors_found = true;
            }
        } else { // User tidak punya password sebelumnya, langsung set baru
            $can_proceed_update = true;
        }

        if ($can_proceed_update && !$validation_errors_found) {
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
                    // Jika user sebelumnya tidak punya password, sekarang punya
                    // mungkin perlu logout dan login ulang agar session terupdate (jika ada flag `has_password`)
                    header("Location: profil.php");
                    exit();
                } else {
                    add_notification("Gagal mengubah kata sandi: " . $stmt_update->error, "error");
                }
                $stmt_update->close();
            }
        }
    }
}
?>
<?php require_once 'includes/header.php'; ?>

        <main class="main">
            <h2 class="page-title">Ubah Kata Sandi Akun</h2>
            <?php
                // Menampilkan notifikasi
                if (isset($_SESSION['notification_messages']) && !empty($_SESSION['notification_messages'])) {
                    // ... (kode display notifikasi seperti di profil.php) ...
                     foreach ($_SESSION['notification_messages'] as $key => $notif) {
                        $alert_type = $notif['type'] === 'success' ? 'success' : ($notif['type'] === 'error' ? 'danger' : 'info');
                        echo "<div class='alert alert-{$alert_type}' role='alert' style='margin-bottom:15px; padding:10px; border-radius:5px; background-color:" . ($alert_type == 'success' ? '#d4edda; color:#155724;' : ($alert_type == 'danger' ? '#f8d7da; color:#721c24;' : '#cce5ff; color:#004085;')) . "'>{$notif['message']}</div>";
                    }
                    unset($_SESSION['notification_messages']);
                    unset($_SESSION['has_unread_notifications_badge']);
                }
            ?>
            <div class="form-container">
                <form id="changePasswordForm" method="POST" action="ubah_password.php">
                    <?php if ($user_has_password): ?>
                    <div class="form-group">
                        <label for="currentPassword">Kata Sandi Saat Ini</label>
                        <input type="password" id="currentPassword" name="currentPassword" required placeholder="Masukkan kata sandi Anda saat ini">
                    </div>
                    <?php else: ?>
                    <div class="form-group">
                        <p style="padding: 10px; background-color: #eef1f5; border-radius: 5px; color: #3c4250;">
                            Anda saat ini masuk menggunakan akun Google dan belum mengatur kata sandi manual. Masukkan kata sandi baru di bawah ini untuk mengaktifkan login manual.
                        </p>
                         <input type="hidden" name="currentPassword" value=""> <!-- Kirim kosong jika tidak ada pass -->
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

---
**Langkah 6: File Baru untuk Fitur Tambahan**

**6.1. `google_login.php` (Tidak diperlukan, URL dibuat di `login.php`)**
File ini tidak benar-benar diperlukan karena URL otentikasi Google dibuat langsung di `login.php`.

**6.2. `google_callback.php`**
```php
<?php
// google_callback.php
require_once 'includes/config.php'; // Untuk Client ID/Secret, DB, session
require_once 'includes/db.php';

$client = new Google_Client();
$client->setClientId(GOOGLE_CLIENT_ID);
$client->setClientSecret(GOOGLE_CLIENT_SECRET);
$client->setRedirectUri(GOOGLE_REDIRECT_URI);
$client->addScope("email");
$client->addScope("profile");

if (isset($_GET['code'])) {
    $token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
    if (isset($token['error'])) {
        // Tangani error, redirect ke login dengan pesan error
        $_SESSION['error_message'] = "Gagal mendapatkan token dari Google: " . $token['error_description'];
        header('Location: login.php');
        exit;
    }
    $client->setAccessToken($token['access_token']);

    // Dapatkan info profil pengguna
    $google_oauth = new Google_Service_Oauth2($client);
    $google_account_info = $google_oauth->userinfo->get();
    $email = $google_account_info->email;
    $name = $google_account_info->name;
    $google_id = $google_account_info->id;
    $profile_pic_url = $google_account_info->picture;

    // Cek apakah pengguna sudah ada di database
    $stmt = $conn->prepare("SELECT id, username, profile_image FROM users WHERE email = ? OR google_id = ?");
    $stmt->bind_param("ss", $email, $google_id);
    $stmt->execute();
    $result = $stmt->get_result();
    $user = $result->fetch_assoc();
    $stmt->close();

    if ($user) {
        // Pengguna sudah ada, loginkan
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username']; // Bisa jadi username beda dari Google name

        // Update google_id jika login via email tapi google_id belum ada
        if (empty($user['google_id']) && !empty($google_id)) {
            $stmt_update_gid = $conn->prepare("UPDATE users SET google_id = ?, is_google_verified = 1 WHERE id = ?");
            $stmt_update_gid->bind_param("si", $google_id, $user['id']);
            $stmt_update_gid->execute();
            $stmt_update_gid->close();
        }
        // Update profile picture jika dari google lebih baru atau user belum punya
        // (Ini bisa lebih kompleks, untuk sekarang kita update jika berbeda)
        // Anda mungkin ingin menyimpan gambar google ke server Anda
        // Untuk saat ini, kita akan simpan URLnya saja atau biarkan yang ada.
        // Jika ingin menyimpan gambar:
        /*
        if ($profile_pic_url && $user['profile_image'] !== $profile_pic_url) { // Atau logika lain
            $local_image_path = 'uploads/profile_pictures/google_user_' . $user['id'] . '_' . time() . '.jpg';
            if (copy($profile_pic_url, $local_image_path)) {
                 $stmt_update_pic = $conn->prepare("UPDATE users SET profile_image = ? WHERE id = ?");
                 $stmt_update_pic->bind_param("si", $local_image_path, $user['id']);
                 $stmt_update_pic->execute();
                 $stmt_update_pic->close();
                 $_SESSION['profile_image'] = $local_image_path;
            }
        } else {
            $_SESSION['profile_image'] = $user['profile_image'];
        }
        */
        $_SESSION['profile_image'] = $user['profile_image']; // Biarkan yang ada di DB dulu


    } else {
        // Pengguna baru, daftarkan
        // Untuk password, bisa generate random atau biarkan NULL jika is_google_verified = 1
        // $random_password = bin2hex(random_bytes(8)); // Contoh
        // $hashed_password = password_hash($random_password, PASSWORD_DEFAULT);

        // Simpan gambar profil dari Google
        $new_profile_image_path = 'images/placeholder-profile.png'; // default
        if ($profile_pic_url) {
            $upload_dir = 'uploads/profile_pictures/';
            if (!is_dir($upload_dir)) mkdir($upload_dir, 0775, true);
            
            $image_extension = pathinfo(parse_url($profile_pic_url, PHP_URL_PATH), PATHINFO_EXTENSION) ?: 'jpg';
            $new_filename = 'google_user_' . $google_id . '_' . time() . '.' . $image_extension;
            $destination = $upload_dir . $new_filename;
            
            // Ambil gambar dari URL Google
            $image_content = @file_get_contents($profile_pic_url);
            if ($image_content !== false && @file_put_contents($destination, $image_content) !== false) {
                $new_profile_image_path = $destination;
            }
        }


        $stmt = $conn->prepare("INSERT INTO users (username, email, google_id, profile_image, is_google_verified, password) VALUES (?, ?, ?, ?, 1, NULL)");
        $stmt->bind_param("ssss", $name, $email, $google_id, $new_profile_image_path);
        if ($stmt->execute()) {
            $_SESSION['user_id'] = $stmt->insert_id;
            $_SESSION['username'] = $name;
            $_SESSION['profile_image'] = $new_profile_image_path;
        } else {
            $_SESSION['error_message'] = "Gagal mendaftarkan pengguna baru: " . $stmt->error;
            header('Location: register.php');
            exit;
        }
        $stmt->close();
    }

    header('Location: dashboard.php');
    exit();

} else {
    header('Location: login.php');
    exit();
}
?>
```

**6.3. `forgot_password.php`**
```php
<?php
// forgot_password.php
$page_title = "Lupa Kata Sandi";
require_once 'includes/config.php';
require_once 'includes/db.php';
require_once 'includes/send_email.php';

$errors = [];
$success_message = '';

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $email = trim($_POST['email']);

    if (empty($email) || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Masukkan alamat email yang valid.";
    } else {
        $stmt = $conn->prepare("SELECT id, username FROM users WHERE email = ?");
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();
        $user = $result->fetch_assoc();
        $stmt->close();

        if ($user) {
            $token = bin2hex(random_bytes(50));
            $expires = new DateTime('NOW');
            $expires->add(new DateInterval('PT1H')); // Token berlaku 1 jam
            $expires_str = $expires->format('Y-m-d H:i:s');

            $stmt_update = $conn->prepare("UPDATE users SET reset_token = ?, reset_token_expires_at = ? WHERE id = ?");
            $stmt_update->bind_param("ssi", $token, $expires_str, $user['id']);

            if ($stmt_update->execute()) {
                $reset_link = BASE_URL . "/reset_password.php?token=" . $token;
                $subject = "Reset Kata Sandi Akun List In Anda";
                $email_body = get_email_template_header();
                $email_body .= "<div class='content'>
                                <h2>Halo " . htmlspecialchars($user['username']) . ",</h2>
                                <p>Kami menerima permintaan untuk mereset kata sandi akun List In Anda. Jika Anda tidak meminta ini, abaikan email ini.</p>
                                <p>Untuk mereset kata sandi Anda, silakan klik tautan di bawah ini:</p>
                                <a href='" . $reset_link . "' class='button'>Reset Kata Sandi Saya</a>
                                <p>Tautan ini akan kedaluwarsa dalam 1 jam.</p>
                                <p>Jika tombol di atas tidak berfungsi, salin dan tempel URL berikut ke browser Anda:<br>" . $reset_link . "</p>
                                <p>Salam,<br>Tim List In</p>
                               </div>";
                $email_body .= get_email_template_footer();

                if (send_email($email, $subject, $email_body)) {
                    $success_message = "Email instruksi reset kata sandi telah dikirim ke alamat email Anda. Silakan periksa kotak masuk (dan folder spam).";
                } else {
                    $errors[] = "Gagal mengirim email reset. Silakan coba lagi nanti atau hubungi dukungan.";
                }
            } else {
                $errors[] = "Gagal memproses permintaan reset. Database error.";
            }
            $stmt_update->close();
        } else {
            // Untuk keamanan, jangan beritahu apakah email ada atau tidak.
            $success_message = "Jika alamat email Anda terdaftar, Anda akan menerima email instruksi reset kata sandi.";
        }
    }
}
?>
<?php require_once 'includes/header.php'; ?>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Lupa Kata Sandi?</h2>
        <p class="subtitle">Masukkan alamat email Anda di bawah ini untuk menerima tautan reset kata sandi.</p>

        <?php if (!empty($errors)): ?>
            <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
                <?php foreach ($errors as $error): ?><p><?php echo $error; ?></p><?php endforeach; ?>
            </div>
        <?php endif; ?>
        <?php if ($success_message): ?>
            <div style="background-color: #d4edda; color: #155724; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
                <p><?php echo $success_message; ?></p>
            </div>
        <?php endif; ?>

        <?php if (!$success_message) : // Sembunyikan form jika sukses ?>
        <form id="forgotPasswordForm" method="POST" action="forgot_password.php">
            <div class="form-group">
                <label for="email">Alamat Email</label>
                <input type="email" id="email" name="email" required placeholder="cth: pengguna@email.com">
            </div>
            <button type="submit" class="btn-submit">Kirim Tautan Reset</button>
        </form>
        <?php endif; ?>
        <p class="auth-link">Ingat kata sandi Anda? <a href="login.php">Masuk di sini</a></p>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

**6.4. `reset_password.php`**
```php
<?php
// reset_password.php
$page_title = "Reset Kata Sandi";
require_once 'includes/config.php';
require_once 'includes/db.php';

$errors = [];
$success_message = '';
$token_valid = false;
$user_id_for_reset = null;

if (isset($_GET['token'])) {
    $token = $_GET['token'];
    $stmt = $conn->prepare("SELECT id, reset_token_expires_at FROM users WHERE reset_token = ?");
    $stmt->bind_param("s", $token);
    $stmt->execute();
    $result = $stmt->get_result();
    $user = $result->fetch_assoc();
    $stmt->close();

    if ($user) {
        $now = new DateTime();
        $expires = new DateTime($user['reset_token_expires_at']);
        if ($now < $expires) {
            $token_valid = true;
            $user_id_for_reset = $user['id'];
        } else {
            $errors[] = "Tautan reset kata sandi telah kedaluwarsa.";
        }
    } else {
        $errors[] = "Tautan reset kata sandi tidak valid.";
    }
} else {
    $errors[] = "Tautan reset tidak ditemukan.";
}

if ($_SERVER["REQUEST_METHOD"] == "POST" && $token_valid && $user_id_for_reset) {
    $password = $_POST['password'];
    $confirmPassword = $_POST['confirmPassword'];

    if (empty($password) || strlen($password) < 6) {
        $errors[] = "Kata sandi baru minimal 6 karakter.";
    } elseif ($password !== $confirmPassword) {
        $errors[] = "Kata sandi baru dan konfirmasi tidak cocok.";
    }

    if (empty($errors)) {
        $hashed_password = password_hash($password, PASSWORD_DEFAULT);
        // Hapus token setelah digunakan
        $stmt_update = $conn->prepare("UPDATE users SET password = ?, reset_token = NULL, reset_token_expires_at = NULL WHERE id = ?");
        $stmt_update->bind_param("si", $hashed_password, $user_id_for_reset);

        if ($stmt_update->execute()) {
            $success_message = "Kata sandi Anda berhasil direset. Silakan masuk dengan kata sandi baru Anda.";
            $_SESSION['success_message'] = $success_message; // Untuk ditampilkan di halaman login
            header("Location: login.php");
            exit();
        } else {
            $errors[] = "Gagal mereset kata sandi. Database error.";
        }
        $stmt_update->close();
    }
}
?>
<?php require_once 'includes/header.php'; ?>

    <div class="auth-container">
        <div class="logo-container">
            <h1>List In</h1>
        </div>
        <h2>Reset Kata Sandi Anda</h2>

        <?php if (!empty($errors)): ?>
            <div style="background-color: #f8d7da; color: #721c24; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
                <?php foreach ($errors as $error): ?><p><?php echo $error; ?></p><?php endforeach; ?>
            </div>
        <?php endif; ?>
        <?php if ($success_message): ?>
            <div style="background-color: #d4edda; color: #155724; padding: 10px; border-radius: 5px; margin-bottom: 15px;">
                <p><?php echo $success_message; ?></p>
                <p class="auth-link" style="margin-top: 10px;"><a href="login.php">Masuk Sekarang</a></p>
            </div>
        <?php endif; ?>

        <?php if ($token_valid && !$success_message): ?>
        <p class="subtitle">Masukkan kata sandi baru Anda di bawah ini.</p>
        <form id="resetPasswordForm" method="POST" action="reset_password.php?token=<?php echo htmlspecialchars($token); ?>">
            <div class="form-group">
                <label for="password">Kata Sandi Baru</label>
                <input type="password" id="password" name="password" minlength="6" required placeholder="Minimal 6 karakter">
            </div>
            <div class="form-group">
                <label for="confirmPassword">Konfirmasi Kata Sandi Baru</label>
                <input type="password" id="confirmPassword" name="confirmPassword" minlength="6" required placeholder="Ulangi kata sandi baru">
            </div>
            <button type="submit" class="btn-submit">Reset Kata Sandi</button>
        </form>
        <?php elseif(!$success_message): ?>
             <p class="auth-link" style="margin-top: 10px;"><a href="forgot_password.php">Minta tautan baru</a> atau <a href="login.php">Kembali ke Login</a></p>
        <?php endif; ?>
    </div>

<?php require_once 'includes/footer.php'; ?>
```

**6.5. `laporan.php` (Halaman Laporan)**
Ini akan menjadi halaman yang kompleks.
```php
<?php
// laporan.php
$page_title = "Laporan Produktivitas";
require_once 'includes/header.php';
require_once 'includes/sidebar.php';
require_once 'includes/task_helper.php'; // Untuk render_task_card

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
$user_id = $_SESSION['user_id'];

// --- Data untuk Chart dan Status (Mirip Dashboard) ---
$total_tasks_report = 0; $completed_report = 0; $in_progress_report = 0; $not_started_report = 0;
$report_start_date_val = date('Y-m-d', strtotime('-6 days')); // Default 7 hari terakhir
$report_end_date_val = date('Y-m-d');
$filter_date_range_str_report = isset($_GET['reportDateRange']) ? trim($_GET['reportDateRange']) : (date('d/m/Y', strtotime($report_start_date_val)) . ' - ' . date('d/m/Y', strtotime($report_end_date_val)));

if (isset($_GET['reportDateRange']) && !empty(trim($_GET['reportDateRange']))) {
    $range_parts = explode(' - ', $_GET['reportDateRange']);
    if (count($range_parts) >= 1) {
        $date_start_tmp = DateTime::createFromFormat('d/m/Y', trim($range_parts[0]));
        if ($date_start_tmp) $report_start_date_val = $date_start_tmp->format('Y-m-d');

        if (count($range_parts) == 2) {
            $date_end_tmp = DateTime::createFromFormat('d/m/Y', trim($range_parts[1]));
            if ($date_end_tmp) $report_end_date_val = $date_end_tmp->format('Y-m-d');
        } else {
             $report_end_date_val = $report_start_date_val; // Jika hanya 1 tanggal
        }
    }
}
// Pastikan start date tidak lebih besar dari end date
if (strtotime($report_start_date_val) > strtotime($report_end_date_val)) {
    list($report_start_date_val, $report_end_date_val) = [$report_end_date_val, $report_start_date_val];
}


// Query untuk statistik berdasarkan rentang tanggal (misal, tugas yang *jatuh tempo* atau *diselesaikan* dalam rentang ini)
// Untuk 'Status Tugas Keseluruhan', kita mungkin tetap ingin semua tugas user, atau tugas yang relevan dengan rentang waktu.
// Mari kita ambil tugas yang 'updated_at' nya dalam rentang waktu untuk status.
$stmt_stats_report = $conn->prepare(
    "SELECT status, COUNT(*) as count FROM tasks
     WHERE user_id = ? AND DATE(updated_at) BETWEEN ? AND ?
     GROUP BY status"
);
if ($stmt_stats_report) {
    $stmt_stats_report->bind_param("iss", $user_id, $report_start_date_val, $report_end_date_val);
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
    'Selesai'    => ['count' => $completed_report,   'percent' => ($total_tasks_report > 0) ? round(($completed_report / $total_tasks_report) * 100) : 0, 'color' => '#4caf50'],
    'Dikerjakan' => ['count' => $in_progress_report, 'percent' => ($total_tasks_report > 0) ? round(($in_progress_report / $total_tasks_report) * 100) : 0, 'color' => '#2196f3'],
    'Belum Mulai'=> ['count' => $not_started_report, 'percent' => ($total_tasks_report > 0) ? round(($not_started_report / $total_tasks_report) * 100) : 0, 'color' => '#f44336'],
];


// Data untuk Diagram Performa (Tugas Selesai per Hari dalam Rentang)
$line_chart_labels_report = []; $line_chart_data_completed_report = [];
try {
    $current_date_loop_obj = new DateTime($report_start_date_val);
    $end_date_loop_obj = new DateTime($report_end_date_val);
    $interval_one_day = new DateInterval('P1D');
    $loop_count = 0;

    while ($current_date_loop_obj <= $end_date_loop_obj) {
        $date_str_loop = $current_date_loop_obj->format('Y-m-d');
        $line_chart_labels_report[] = $current_date_loop_obj->format('d M'); // Format untuk label chart

        $stmt_completed_on_date = $conn->prepare("SELECT COUNT(*) as count FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) = ?");
        $tasks_completed_on_day = 0;
        if($stmt_completed_on_date) {
            $stmt_completed_on_date->bind_param("is", $user_id, $date_str_loop);
            $stmt_completed_on_date->execute();
            $result_completed_on_date = $stmt_completed_on_date->get_result();
            if($result_completed_on_date && $row_completed = $result_completed_on_date->fetch_assoc()) {
                $tasks_completed_on_day = (int)$row_completed['count'];
            }
            $stmt_completed_on_date->close();
        }
        $line_chart_data_completed_report[] = $tasks_completed_on_day;
        $current_date_loop_obj->add($interval_one_day);
        $loop_count++;
        if ($loop_count > 90 && $current_date_loop_obj <= $end_date_loop_obj) { // Batasi jumlah titik data untuk performa
             $line_chart_labels_report[] = "..."; $line_chart_data_completed_report[] = null; break;
        }
    }
} catch (Exception $e) { error_log("Laporan (Performa): Exception: " . $e->getMessage()); }

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

// Data untuk Kalender Tugas
$tasks_for_calendar = [];
$stmt_calendar_tasks = $conn->prepare("SELECT id, title, status, due_date FROM tasks WHERE user_id = ? AND due_date IS NOT NULL AND ( (status != 'Completed' AND due_date >= ?) OR (status = 'Completed' AND DATE(updated_at) >= ? ) )");
$min_date_for_calendar_query = date('Y-m-d', strtotime('-3 months')); // Ambil tugas 3 bulan ke belakang dan masa depan
if ($stmt_calendar_tasks) {
    $stmt_calendar_tasks->bind_param("iss", $user_id, $min_date_for_calendar_query, $min_date_for_calendar_query);
    $stmt_calendar_tasks->execute();
    $result_calendar = $stmt_calendar_tasks->get_result();
    while ($task_cal = $result_calendar->fetch_assoc()) {
        $date_key = $task_cal['due_date'];
        if ($task_cal['status'] === 'Completed' && !empty($task_cal['updated_at'])) {
            $date_key = date('Y-m-d', strtotime($task_cal['updated_at'])); // Gunakan tanggal selesai jika sudah completed
        }
        if (!isset($tasks_for_calendar[$date_key])) {
            $tasks_for_calendar[$date_key] = [];
        }
        $tasks_for_calendar[$date_key][] = $task_cal;
    }
    $stmt_calendar_tasks->close();
}

?>
<?php //require_once 'includes/header.php'; // sudah di atas ?>

<main class="main">
    <h2 class="page-title">Laporan Produktivitas</h2>
    <?php
        if (isset($_SESSION['notification_messages']) && !empty($_SESSION['notification_messages'])) {
            // ... (kode display notifikasi seperti di profil.php) ...
            foreach ($_SESSION['notification_messages'] as $key => $notif) {
                $alert_type = $notif['type'] === 'success' ? 'success' : ($notif['type'] === 'error' ? 'danger' : 'info');
                echo "<div class='alert alert-{$alert_type}' role='alert' style='margin-bottom:15px; padding:10px; border-radius:5px; background-color:" . ($alert_type == 'success' ? '#d4edda; color:#155724;' : ($alert_type == 'danger' ? '#f8d7da; color:#721c24;' : '#cce5ff; color:#004085;')) . "'>{$notif['message']}</div>";
            }
            unset($_SESSION['notification_messages']);
            unset($_SESSION['has_unread_notifications_badge']);
        }
    ?>
    <div class="filters-container" style="margin-bottom: 20px;">
        <form method="GET" action="laporan.php" style="display: flex; gap: 15px; align-items: flex-end; width:100%;">
            <div class="form-group" style="flex-grow: 1;">
                <label for="reportDateRange">Pilih Rentang Tanggal Laporan</label>
                <input type="text" id="reportDateRange" name="reportDateRange" placeholder="Pilih Rentang Tanggal"
                       value="<?php echo htmlspecialchars($filter_date_range_str_report); ?>">
            </div>
            <button type="submit" class="btn btn-primary"><i class="fas fa-filter"></i> Terapkan Filter</button>
            <a href="generate_report_pdf.php?start_date=<?php echo urlencode($report_start_date_val); ?>&end_date=<?php echo urlencode($report_end_date_val); ?>"
               class="btn btn-secondary" target="_blank"><i class="fas fa-file-pdf"></i> Unduh PDF</a>
        </form>
    </div>

    <div class="dashboard-grid" style="height: auto;"> <!-- Re-use dashboard grid for layout -->
        <div class="dashboard-left-column">
            <section class="widget">
                <h3>Status Tugas (Rentang: <?php echo date('d M Y', strtotime($report_start_date_val)) . " - " . date('d M Y', strtotime($report_end_date_val)); ?>)</h3>
                <div class="widget-content-area status-progress-container" style="padding-top:15px;">
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
                        <p class="no-tasks-message" style="text-align:center; width:100%;">Tidak ada data tugas untuk status pada rentang ini.</p>
                    <?php endif; ?>
                </div>
            </section>

            <section class="widget">
                <h3>Performa Pengerjaan (Rentang: <?php echo date('d M Y', strtotime($report_start_date_val)) . " - " . date('d M Y', strtotime($report_end_date_val)); ?>)</h3>
                <div class="widget-content-area chart-container">
                     <?php
                     $has_valid_line_chart_data_report = false;
                     if (isset($performance_chart_data_report['datasets'][0]['data']) && is_array($performance_chart_data_report['datasets'][0]['data'])) {
                         $filtered_data_report = array_filter($performance_chart_data_report['datasets'][0]['data'], function($x) { return $x !== null && $x >=0; });
                         $has_valid_line_chart_data_report = !empty($filtered_data_report);
                     }
                     if ($has_valid_line_chart_data_report):
                     ?>
                    <canvas id="reportPerformanceLineChart"></canvas>
                    <?php else: ?>
                         <p class="no-tasks-message" style="text-align:center; padding-top:20px;">Tidak ada data tugas selesai untuk chart performa pada rentang ini.</p>
                    <?php endif; ?>
                </div>
            </section>
        </div>

        <div class="dashboard-right-column">
            <section class="widget" style="height:100%;">
                <h3>Kalender Tugas & Aktivitas</h3>
                <div style="display: flex; gap: 15px; height: calc(100% - 40px); /* Adjust for h3 */">
                    <div id="reportCalendarContainer" style="flex: 1; min-width: 280px;">
                        <!-- Kalender akan di-render di sini oleh Flatpickr -->
                    </div>
                    <div id="reportTasksOnSelectedDate" style="flex: 1; overflow-y: auto; border-left: 1px solid #eee; padding-left: 15px;">
                        <h4 id="selectedDateTitle" style="margin-top:0; margin-bottom:10px; color: #34495e; border-bottom: 1px solid #f0f0f0; padding-bottom:5px;">Pilih tanggal pada kalender</h4>
                        <div id="tasksListForDate">
                            <p class="no-tasks-message">Tidak ada tugas untuk tanggal yang dipilih.</p>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </div>
</main>

<script>
document.addEventListener('DOMContentLoaded', () => {
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

    // Kalender Tugas di Halaman Laporan
    const tasksForCalendarJS = <?php echo json_encode($tasks_for_calendar); ?>;
    const reportCalendarContainer = document.getElementById('reportCalendarContainer');
    const tasksListForDateDiv = document.getElementById('tasksListForDate');
    const selectedDateTitleH4 = document.getElementById('selectedDateTitle');

    if (reportCalendarContainer && tasksListForDateDiv && selectedDateTitleH4) {
        flatpickr(reportCalendarContainer, {
            inline: true,
            dateFormat: "Y-m-d", // Sesuai format key di tasksForCalendarJS
            locale: "id",
            // defaultDate: "today", // Bisa set default ke hari ini
            onChange: function(selectedDates, dateStr, instance) {
                if (selectedDates.length > 0) {
                    const selectedDate = dateStr; // dateStr sudah dalam format Y-m-d
                    selectedDateTitleH4.textContent = "Tugas pada " + instance.formatDate(selectedDates[0], "d M Y");
                    tasksListForDateDiv.innerHTML = ''; // Kosongkan list

                    if (tasksForCalendarJS[selectedDate] && tasksForCalendarJS[selectedDate].length > 0) {
                        tasksForCalendarJS[selectedDate].forEach(task => {
                            // Buat card task sederhana (bisa pakai fungsi render_task_card jika mau lebih kompleks)
                            const taskDiv = document.createElement('div');
                            taskDiv.className = 'task-item-card status-' + task.status.toLowerCase().replace(' ', '-');
                            taskDiv.style.fontSize = '0.8rem';
                            taskDiv.style.padding = '8px';

                            let statusText = task.status;
                            if (task.status === 'Not Started') statusText = 'Belum Mulai';
                            else if (task.status === 'In Progress') statusText = 'Dikerjakan';

                            taskDiv.innerHTML = `
                                <div class="task-details" style="margin-right:0;">
                                    <strong>${task.title}</strong>
                                    <p class="meta-info" style="font-size:0.7rem;">Status: ${statusText}</p>
                                </div>`;
                            tasksListForDateDiv.appendChild(taskDiv);
                        });
                    } else {
                        tasksListForDateDiv.innerHTML = '<p class="no-tasks-message">Tidak ada tugas untuk tanggal ini.</p>';
                    }
                }
            },
            onDayCreate: function(dObj, dStr, fp, dayElem){
                // Tambahkan dot jika ada tugas pada hari itu
                const dateKey = fp.formatDate(dayElem.dateObj, "Y-m-d");
                if (tasksForCalendarJS[dateKey] && tasksForCalendarJS[dateKey].length > 0) {
                    let dot = document.createElement('span');
                    dot.className = 'event-dot';
                    dot.style.cssText = `
                        height: 6px; width: 6px; background-color: #7e47b8; border-radius: 50%;
                        display: block; margin: 1px auto 0;`;
                    dayElem.appendChild(dot);
                }
            }
        });
    }
});
</script>
<?php require_once 'includes/footer.php'; ?>
```
Tambahkan CSS untuk event dot di `style.css`:
```css
/* style.css - tambahkan di akhir */
.flatpickr-day .event-dot {
    /* Styling sudah inline di JS, tapi bisa juga di sini */
}
html.dark-theme-active .flatpickr-day .event-dot {
    background-color: #bb86fc; /* Warna dot untuk tema gelap */
}
```

**6.6. `generate_report_pdf.php` (Untuk Unduh PDF)**
```php
<?php
// generate_report_pdf.php
require_once 'includes/config.php'; // Untuk autoload TCPDF
require_once 'includes/db.php';

if (!isset($_SESSION['user_id'])) {
    // Sebaiknya ada cara untuk memvalidasi akses ke laporan ini,
    // mungkin token atau session check yang lebih ketat.
    // Untuk sekarang, asumsikan hanya user login yang bisa akses via link.
    die("Akses ditolak. Silakan login terlebih dahulu.");
}
$user_id = $_SESSION['user_id'];

$start_date_str = $_GET['start_date'] ?? date('Y-m-d', strtotime('-6 days'));
$end_date_str = $_GET['end_date'] ?? date('Y-m-d');

// Validasi tanggal
$start_date_obj = DateTime::createFromFormat('Y-m-d', $start_date_str);
$end_date_obj = DateTime::createFromFormat('Y-m-d', $end_date_str);

if (!$start_date_obj || !$end_date_obj || $start_date_obj->format('Y-m-d') !== $start_date_str || $end_date_obj->format('Y-m-d') !== $end_date_str) {
    die("Format tanggal tidak valid.");
}
if ($start_date_obj > $end_date_obj) {
    list($start_date_obj, $end_date_obj) = [$end_date_obj, $start_date_obj]; // Swap
    $start_date_str = $start_date_obj->format('Y-m-d');
    $end_date_str = $end_date_obj->format('Y-m-d');
}

$user_info_stmt = $conn->prepare("SELECT username, email FROM users WHERE id = ?");
$user_info_stmt->bind_param("i", $user_id);
$user_info_stmt->execute();
$user_result = $user_info_stmt->get_result();
$user_data = $user_result->fetch_assoc();
$user_info_stmt->close();

if (!$user_data) die("User tidak ditemukan.");


// Ambil data tugas dalam rentang
$tasks_completed = [];
$tasks_in_progress = [];
$tasks_not_started = [];
$tasks_overdue = [];

// Selesai dalam rentang
$stmt_completed = $conn->prepare("SELECT title, DATE_FORMAT(updated_at, '%d %M %Y') as completed_date, priority FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ? ORDER BY updated_at DESC");
$stmt_completed->bind_param("iss", $user_id, $start_date_str, $end_date_str);
$stmt_completed->execute();
$res_completed = $stmt_completed->get_result();
while($row = $res_completed->fetch_assoc()) $tasks_completed[] = $row;
$stmt_completed->close();

// Masih dikerjakan (due date dalam rentang atau sebelum rentang tapi belum selesai)
$stmt_in_progress = $conn->prepare("SELECT title, DATE_FORMAT(due_date, '%d %M %Y') as due_date_f, priority FROM tasks WHERE user_id = ? AND status = 'In Progress' AND due_date <= ? ORDER BY due_date ASC");
$stmt_in_progress->bind_param("is", $user_id, $end_date_str);
$stmt_in_progress->execute();
$res_in_progress = $stmt_in_progress->get_result();
while($row = $res_in_progress->fetch_assoc()) $tasks_in_progress[] = $row;
$stmt_in_progress->close();

// Belum dimulai (due date dalam rentang)
$stmt_not_started = $conn->prepare("SELECT title, DATE_FORMAT(due_date, '%d %M %Y') as due_date_f, priority FROM tasks WHERE user_id = ? AND status = 'Not Started' AND due_date BETWEEN ? AND ? ORDER BY due_date ASC");
$stmt_not_started->bind_param("iss", $user_id, $start_date_str, $end_date_str);
$stmt_not_started->execute();
$res_not_started = $stmt_not_started->get_result();
while($row = $res_not_started->fetch_assoc()) $tasks_not_started[] = $row;
$stmt_not_started->close();

// Terlewat (due date sebelum akhir rentang dan belum selesai)
$stmt_overdue = $conn->prepare("SELECT title, DATE_FORMAT(due_date, '%d %M %Y') as due_date_f, priority FROM tasks WHERE user_id = ? AND status != 'Completed' AND due_date < ? AND due_date < CURDATE() ORDER BY due_date ASC");
$stmt_overdue->bind_param("is", $user_id, $end_date_str); // Ambil yang terlewat hingga akhir rentang
$stmt_overdue->execute();
$res_overdue = $stmt_overdue->get_result();
while($row = $res_overdue->fetch_assoc()) $tasks_overdue[] = $row;
$stmt_overdue->close();


// Buat PDF
class MyTCPDF extends TCPDF {
    public function Header() {
        $this->SetFont('helvetica', 'B', 16);
        $this->SetTextColor(126, 71, 184); // Warna ungu ListIn
        $this->Cell(0, 15, 'List In - Laporan Produktivitas', 0, false, 'C', 0, '', 0, false, 'M', 'M');
        $this->Ln(5);
        $this->SetFont('helvetica', '', 10);
        $this->SetTextColor(0,0,0);
        $this->Cell(0, 10, 'Tanggal Cetak: ' . date('d M Y H:i'), 0, false, 'R');
        $this->Line(15, 25, $this->getPageWidth()-15, 25); // Garis bawah header
    }

    public function Footer() {
        $this->SetY(-15);
        $this->SetFont('helvetica', 'I', 8);
        $this->Cell(0, 10, 'Halaman ' . $this->getAliasNumPage() . '/' . $this->getAliasNbPages() . ' | Generated by List In', 0, false, 'C', 0, '', 0, false, 'T', 'M');
    }
}

$pdf = new MyTCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);

$pdf->SetCreator(PDF_CREATOR);
$pdf->SetAuthor('List In Application');
$pdf->SetTitle('Laporan Produktivitas List In');
$pdf->SetSubject('Laporan Produktivitas Pengguna');

$pdf->SetHeaderData("", 0, "", "", array(0,0,0), array(255,255,255)); // Hapus header default
$pdf->setFooterData(array(0,0,0), array(255,255,255));

$pdf->SetMargins(15, 28, 15); // Left, Top (setelah header), Right
$pdf->SetHeaderMargin(10);
$pdf->SetFooterMargin(PDF_MARGIN_FOOTER);

$pdf->SetAutoPageBreak(TRUE, PDF_MARGIN_BOTTOM);
$pdf->setImageScale(PDF_IMAGE_SCALE_RATIO);
$pdf->SetFont('helvetica', '', 10);
$pdf->AddPage();

// Konten Laporan
$html = '<style>
            h1 { font-size: 18px; color: #7e47b8; text-align:center; margin-bottom: 5px;}
            h2 { font-size: 14px; color: #333; margin-top: 15px; margin-bottom: 8px; border-bottom: 1px solid #ccc; padding-bottom: 3px;}
            p { margin-bottom: 5px; line-height: 1.4;}
            table { width: 100%; border-collapse: collapse; margin-bottom:15px; }
            th { background-color: #f0f3f7; border: 1px solid #ddd; padding: 6px; font-weight: bold; text-align: left;}
            td { border: 1px solid #ddd; padding: 6px; }
            .no-data { color: #777; font-style: italic; }
         </style>';

$html .= '<h1>Laporan Produktivitas Pengguna</h1>';
$html .= '<p><strong>Pengguna:</strong> ' . htmlspecialchars($user_data['username']) . ' (' . htmlspecialchars($user_data['email']) . ')</p>';
$html .= '<p><strong>Periode Laporan:</strong> ' . date('d M Y', strtotime($start_date_str)) . ' - ' . date('d M Y', strtotime($end_date_str)) . '</p>';

$html .= '<h2>Ringkasan Umum</h2>';
$html .= '<p>Total Tugas Selesai dalam periode: <strong>' . count($tasks_completed) . '</strong></p>';
$html .= '<p>Tugas Masih Dikerjakan (batas waktu s/d ' . date('d M Y', strtotime($end_date_str)) . '): <strong>' . count($tasks_in_progress) . '</strong></p>';
$html .= '<p>Tugas Belum Dimulai (batas waktu dalam periode): <strong>' . count($tasks_not_started) . '</strong></p>';
$html .= '<p>Tugas Terlewat (batas waktu s/d ' . date('d M Y', strtotime($end_date_str)) . '): <strong>' . count($tasks_overdue) . '</strong></p>';


// Fungsi untuk render tabel tugas
function render_task_table_pdf($title, $tasks, $date_column_name, $date_column_header) {
    $table_html = '<h2>' . htmlspecialchars($title) . '</h2>';
    if (empty($tasks)) {
        $table_html .= '<p class="no-data">Tidak ada tugas dalam kategori ini.</p>';
    } else {
        $table_html .= '<table><thead><tr><th>No.</th><th>Judul Tugas</th><th>Prioritas</th><th>' . htmlspecialchars($date_column_header) . '</th></tr></thead><tbody>';
        $no = 1;
        foreach ($tasks as $task) {
            $table_html .= '<tr>
                                <td style="width:10%; text-align:center;">' . $no++ . '</td>
                                <td style="width:50%;">' . htmlspecialchars($task['title']) . '</td>
                                <td style="width:20%;">' . htmlspecialchars($task['priority']) . '</td>
                                <td style="width:20%;">' . htmlspecialchars($task[$date_column_name]) . '</td>
                            </tr>';
        }
        $table_html .= '</tbody></table>';
    }
    return $table_html;
}

$html .= render_task_table_pdf('Tugas yang Diselesaikan', $tasks_completed, 'completed_date', 'Tanggal Selesai');
$html .= render_task_table_pdf('Tugas Masih Dikerjakan', $tasks_in_progress, 'due_date_f', 'Batas Waktu');
$html .= render_task_table_pdf('Tugas Belum Dimulai', $tasks_not_started, 'due_date_f', 'Batas Waktu');
$html .= render_task_table_pdf('Tugas Terlewat Batas Waktu', $tasks_overdue, 'due_date_f', 'Batas Waktu');

$pdf->writeHTML($html, true, false, true, false, '');
$pdf_filename = 'Laporan_ListIn_' . str_replace(' ', '_', $user_data['username']) . '_' . $start_date_str . '_sd_' . $end_date_str . '.pdf';
$pdf->Output($pdf_filename, 'I'); // 'I' untuk inline (tampil di browser), 'D' untuk download

exit;
?>
```

**6.7. `cron_send_reports.php` (Untuk Cron Job Laporan Email Otomatis)**
```php
<?php
// cron_send_reports.php
// Script ini dijalankan oleh cron job server

// Tingkatkan batas waktu eksekusi jika perlu untuk banyak user/laporan
set_time_limit(0);
ignore_user_abort(true);

require_once __DIR__ . '/includes/config.php';
require_once __DIR__ . '/includes/db.php';
require_once __DIR__ . '/includes/send_email.php';
// Kita perlu fungsi generate_report_pdf atau logika serupa di sini
// Untuk sederhana, kita akan duplikasi sebagian logika PDF generation.
// Idealnya, ini jadi fungsi terpisah yang bisa dipanggil.

echo "Memulai proses pengiriman laporan otomatis...\n";

$today = date('Y-m-d');
$now_timestamp = time();

// Ambil pengguna yang mengaktifkan laporan dan waktunya mengirim
$stmt_users = $conn->prepare(
    "SELECT id, username, email, report_frequency, last_report_sent_at
     FROM users
     WHERE report_frequency != 'none'"
);
$stmt_users->execute();
$result_users = $stmt_users->get_result();

while ($user = $result_users->fetch_assoc()) {
    echo "Memproses user: " . $user['username'] . " (" . $user['email'] . ") - Frekuensi: " . $user['report_frequency'] . "\n";
    $user_id_cron = $user['id'];
    $send_now = false;
    $report_start_date_cron = '';
    $report_end_date_cron = $today; // Laporan selalu hingga hari ini

    switch ($user['report_frequency']) {
        case 'daily':
            // Kirim jika belum pernah atau last_sent kemarin atau lebih lama
            if (empty($user['last_report_sent_at']) || date('Y-m-d', strtotime($user['last_report_sent_at'])) < $today) {
                $send_now = true;
                $report_start_date_cron = $today; // Laporan harian untuk hari ini
            }
            break;
        case 'weekly':
            // Kirim jika hari ini Senin dan (belum pernah atau last_sent > 7 hari lalu)
            if (date('N') == 1) { // 1 = Senin
                if (empty($user['last_report_sent_at']) || strtotime($user['last_report_sent_at']) < strtotime('-6 days', $now_timestamp)) {
                    $send_now = true;
                    $report_start_date_cron = date('Y-m-d', strtotime('-6 days', $now_timestamp)); // 7 hari terakhir
                }
            }
            break;
        case 'monthly':
            // Kirim jika hari ini tanggal 1 dan (belum pernah atau last_sent > 27 hari lalu)
            if (date('j') == 1) { // j = tanggal tanpa leading zero
                 if (empty($user['last_report_sent_at']) || strtotime($user['last_report_sent_at']) < strtotime('-27 days', $now_timestamp)) {
                    $send_now = true;
                    $report_start_date_cron = date('Y-m-d', strtotime('first day of last month', $now_timestamp));
                    $report_end_date_cron = date('Y-m-d', strtotime('last day of last month', $now_timestamp));
                }
            }
            break;
    }

    if ($send_now) {
        echo "  Waktunya mengirim laporan untuk " . $user['username'] . "\n";
        // --- Logika Generate Konten Laporan (mirip generate_report_pdf.php) ---
        // Ambil data tugas dalam rentang $report_start_date_cron dan $report_end_date_cron
        $tasks_completed_cron = []; $tasks_in_progress_cron = []; $tasks_not_started_cron = []; $tasks_overdue_cron = [];
        
        // Query data (contoh untuk completed)
        $stmt_c = $conn->prepare("SELECT title FROM tasks WHERE user_id = ? AND status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?");
        $stmt_c->bind_param("iss", $user_id_cron, $report_start_date_cron, $report_end_date_cron);
        $stmt_c->execute();
        $res_c = $stmt_c->get_result();
        while($row = $res_c->fetch_assoc()) $tasks_completed_cron[] = $row['title'];
        $stmt_c->close();
        // ... (query serupa untuk kategori tugas lainnya) ...
        // Buat query lengkap untuk semua kategori yang relevan

        $email_subject_report = "Laporan Produktivitas List In Anda (" . date('d M Y', strtotime($report_start_date_cron)) . " - " . date('d M Y', strtotime($report_end_date_cron)) . ")";
        $email_body_report = get_email_template_header();
        $email_body_report .= "<div class='content'>
                                <h2>Halo " . htmlspecialchars($user['username']) . ",</h2>
                                <p>Berikut adalah laporan produktivitas Anda dari aplikasi List In untuk periode <strong>" . date('d M Y', strtotime($report_start_date_cron)) . " hingga " . date('d M Y', strtotime($report_end_date_cron)) . "</strong>.</p>
                                
                                <h3>Tugas Selesai: " . count($tasks_completed_cron) . "</h3>";
        if (!empty($tasks_completed_cron)) {
            $email_body_report .= "<ul class='task-list'>";
            foreach (array_slice($tasks_completed_cron, 0, 5) as $task_title) $email_body_report .= "<li>" . htmlspecialchars($task_title) . "</li>"; // Tampilkan maks 5
            if(count($tasks_completed_cron) > 5) $email_body_report .= "<li>Dan lainnya...</li>";
            $email_body_report .= "</ul>";
        } else { $email_body_report .= "<p>Tidak ada tugas yang diselesaikan pada periode ini.</p>"; }

        // ... (tambahkan ringkasan untuk kategori tugas lain) ...

        $email_body_report .= "<p>Untuk detail lebih lanjut atau mengelola tugas Anda, silakan kunjungi aplikasi List In:</p>
                               <a href='" . BASE_URL . "/laporan.php' class='button'>Lihat Laporan Lengkap</a>
                               <p>Anda dapat mengubah frekuensi laporan ini di halaman profil Anda.</p>
                               <p>Salam,<br>Tim List In</p>
                               </div>";
        $email_body_report .= get_email_template_footer();

        // Opsional: Buat PDF dan lampirkan
        // $pdf_content_for_email = generate_pdf_content_function($user_id_cron, $report_start_date_cron, $report_end_date_cron, $conn);
        // $temp_pdf_path = sys_get_temp_dir() . '/report_' . $user_id_cron . '_' . time() . '.pdf';
        // file_put_contents($temp_pdf_path, $pdf_content_for_email);
        // $attachments = [['path' => $temp_pdf_path, 'name' => 'Laporan_ListIn.pdf']];
        // if (send_email($user['email'], $email_subject_report, $email_body_report, '', $attachments)) {
        //    unlink($temp_pdf_path); // Hapus file temp

        if (send_email($user['email'], $email_subject_report, $email_body_report)) {
             echo "  Laporan berhasil dikirim ke " . $user['email'] . "\n";
            // Update last_report_sent_at
            $stmt_update_sent = $conn->prepare("UPDATE users SET last_report_sent_at = NOW() WHERE id = ?");
            $stmt_update_sent->bind_param("i", $user_id_cron);
            $stmt_update_sent->execute();
            $stmt_update_sent->close();
        } else {
            echo "  GAGAL mengirim laporan ke " . $user['email'] . "\n";
        }
    } else {
         echo "  Belum waktunya mengirim laporan untuk " . $user['username'] . "\n";
    }
     echo "--------------------------------------------------\n";
}
$stmt_users->close();

echo "Proses pengiriman laporan otomatis selesai.\n";

// Fungsi untuk generate PDF content (jika ingin dilampirkan)
// Ini akan sangat mirip dengan generate_report_pdf.php, tapi return content PDF bukan output langsung
/*
function generate_pdf_content_function($user_id, $start_date, $end_date, $db_conn_local) {
    // ... (logika query data seperti di generate_report_pdf.php) ...
    // ... (logika pembuatan objek TCPDF dan HTML) ...
    // return $pdf->Output('dummy_filename.pdf', 'S'); // 'S' untuk return as string
}
*/
?>
```

---
**Langkah 7: CSS Tambahan (Opsional)**

Anda mungkin perlu menambahkan beberapa style minor untuk tombol Google atau elemen baru di halaman laporan. Kebanyakan bisa mengikuti gaya yang sudah ada.

Misalnya, di `css/auth.css` untuk tombol Google:
```css
/* css/auth.css - tambahkan atau sesuaikan */
.btn-google {
    background-color: #db4437;
    color: white;
    /* ... style lain seperti btn-submit ... */
}
.btn-google:hover {
    background-color: #c23321;
}
.btn-google i {
    margin-right: 8px;
}
```
Di `css/style.css` untuk halaman laporan:
```css
/* css/style.css - tambahkan */
#reportCalendarContainer .flatpickr-calendar {
    width: 100% !important; /* Agar flatpickr inline mengisi kontainer */
    box-shadow: none;
}
#tasksListForDate .task-item-card {
    font-size: 0.85rem;
    padding: 10px;
    margin-bottom: 8px;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-calendar {
    background: #373737; /* Sesuaikan dengan warna widget dark mode */
    border: 1px solid #555;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-day {
    color: #e0e0e0;
}
html.dark-theme-active #reportCalendarContainer .flatpickr-day:hover {
    background: #4f4f4f;
}
/* ... dan seterusnya untuk styling dark mode kalender ... */
```

---
**Tutorial Langkah Eksternal:**

1.  **Google Cloud Console (Untuk Login Google OAuth 2.0):**
    *   Kunjungi [Google Cloud Console](https://console.cloud.google.com/).
    *   **Buat Proyek Baru** (atau pilih yang sudah ada).
    *   Pergi ke **APIs & Services > Credentials**.
    *   Klik **+ CREATE CREDENTIALS > OAuth client ID**.
    *   Jika diminta, konfigurasikan **OAuth consent screen** terlebih dahulu:
        *   **User Type:** External (atau Internal jika GSuite).
        *   **App name:** List In (atau nama aplikasi Anda).
        *   **User support email:** Email Anda.
        *   **Developer contact information:** Email Anda.
        *   Klik **SAVE AND CONTINUE**.
        *   Di bagian **Scopes**, klik **ADD OR REMOVE SCOPES**. Cari dan tambahkan:
            *   `.../auth/userinfo.email` (untuk email)
            *   `.../auth/userinfo.profile` (untuk nama dan foto profil)
            *   `openid`
        *   Klik **UPDATE**, lalu **SAVE AND CONTINUE**.
        *   Tambahkan **Test users** (email Anda sendiri untuk tahap development). Klik **SAVE AND CONTINUE**.
    *   Kembali ke **Credentials**, klik **+ CREATE CREDENTIALS > OAuth client ID**.
    *   **Application type:** Web application.
    *   **Name:** List In Web Client (atau nama lain).
    *   **Authorized JavaScript origins:** Biarkan kosong atau isi `http://localhost` (jika perlu).
    *   **Authorized redirect URIs:** Klik **+ ADD URI**. Masukkan URI callback Anda, misalnya `http://localhost/nama_folder_proyek_anda/google_callback.php`. Ini *harus* sama persis dengan yang Anda definisikan di `GOOGLE_REDIRECT_URI` dalam `config.php`.
    *   Klik **CREATE**.
    *   Anda akan mendapatkan **Client ID** dan **Client secret**. Salin nilai-nilai ini ke `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET` di `includes/config.php`.
    *   Pastikan **Google People API** dan **OAuth2 API** diaktifkan untuk proyek Anda di **APIs & Services > Library**.

2.  **Pengaturan Email (PHPMailer dengan Gmail):**
    *   Email dan password yang Anda berikan (`listinproject@gmail.com` dan `listin12312`) akan digunakan di `includes/config.php` untuk `SMTP_USERNAME` dan `SMTP_PASSWORD`.
    *   **Jika Akun Gmail Menggunakan 2-Step Verification (Sangat Direkomendasikan):**
        *   Anda *tidak bisa* menggunakan password login biasa. Anda harus membuat **App Password**.
        *   Login ke akun Google Anda.
        *   Pergi ke [Keamanan Akun Google](https://myaccount.google.com/security).
        *   Di bawah "Cara Anda login ke Google", klik **App passwords** (mungkin perlu klik "2-Step Verification" dulu jika App Passwords tidak langsung terlihat).
        *   Anda mungkin diminta login lagi.
        *   Di halaman App passwords, pilih **Select app > Mail** dan **Select device > Other (Custom name)**. Beri nama (misal, "ListIn App PHP").
        *   Klik **GENERATE**.
        *   Anda akan mendapatkan 16 karakter password. **Salin password ini**. Ini yang akan Anda gunakan untuk `SMTP_PASSWORD` di `config.php`. Simpan di tempat aman karena Anda tidak bisa melihatnya lagi.
    *   **Jika Akun Gmail TIDAK Menggunakan 2-Step Verification (Kurang Aman):**
        *   Anda mungkin perlu mengaktifkan "[Less secure app access](https://myaccount.google.com/lesssecureapps)" di pengaturan keamanan akun Google Anda. **Ini tidak direkomendasikan karena kurang aman.** Google sedang menonaktifkan opsi ini. Sebaiknya aktifkan 2FA dan gunakan App Password.
    *   Pastikan `SMTP_HOST`, `SMTP_PORT`, dan `SMTP_SECURE` di `config.php` sudah benar untuk Gmail (`smtp.gmail.com`, port `587`, secure `tls` atau port `465`, secure `ssl`).

3.  **Cron Jobs (Untuk Laporan Email Otomatis):**
    *   Cron job adalah fitur penjadwalan tugas di server berbasis Linux/Unix. Cara konfigurasinya bervariasi tergantung penyedia hosting Anda.
    *   Anda perlu mengatur cron job untuk menjalankan script `cron_send_reports.php` secara berkala (misalnya, sekali sehari).
    *   **Jika Anda menggunakan cPanel:**
        *   Cari ikon "Cron Jobs" di cPanel.
        *   Di bagian "Add New Cron Job", pilih interval (misal, "Once Per Day" di "Common Settings").
        *   Di kolom "Command", Anda perlu memasukkan path ke interpreter PHP dan path absolut ke script Anda. Contoh:
            ```bash
            /usr/local/bin/php /home/username_cpanel/public_html/nama_folder_proyek_anda/cron_send_reports.php > /dev/null 2>&1
            ```
            *   `/usr/local/bin/php`: Path ke PHP CLI (bisa berbeda, cek ke hosting).
            *   `/home/username_cpanel/public_html/nama_folder_proyek_anda/cron_send_reports.php`: Path absolut ke script Anda. Ganti `username_cpanel` dan `nama_folder_proyek_anda`.
            *   `> /dev/null 2>&1`: Untuk menekan output email dari cron job (kecuali jika Anda ingin log).
    *   **Jika Anda menggunakan VPS atau server sendiri (Linux):**
        *   Buka terminal dan ketik `crontab -e`.
        *   Tambahkan baris baru untuk menjadwalkan script. Misalnya, untuk chạy setiap hari jam 3 pagi:
            ```cron
            0 3 * * * /usr/bin/php /var/www/html/nama_folder_proyek_anda/cron_send_reports.php > /dev/null 2>&1
            ```
            *   Path ke PHP CLI dan script mungkin berbeda.
    *   **Penting:** Script `cron_send_reports.php` harus dapat diakses dan dieksekusi oleh server. Pastikan pathnya benar. Anda bisa mengujinya dulu dari command line server: `php /path/to/your/cron_send_reports.php`.

---

Ini adalah kerangka besar. Setiap bagian memerlukan perhatian detail, pengujian, dan kemungkinan debugging.
**Fokus utama Anda selanjutnya:**
1.  Siapkan environment (Composer, folder writable).
2.  Konfigurasi Google API dan email di `config.php`.
3.  Implementasikan fitur login Google & Lupa Password dulu, karena ini mengubah alur autentikasi.
4.  Kemudian, integrasikan pengiriman email untuk notifikasi.
5.  Terakhir, kerjakan halaman Laporan dan sistem cron jobnya.

Semoga berhasil! Ini proyek yang ambisius.
