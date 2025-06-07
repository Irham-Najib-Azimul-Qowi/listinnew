Tentu, saya akan membantu Anda memodifikasi website tersebut. Saya akan menghapus fitur chatbot dan menambahkan fungsionalitas untuk mengunduh laporan PDF yang dapat dikustomisasi, sambil memastikan semua fitur yang ada tetap berfungsi.

Berikut adalah panduan langkah demi langkah beserta kode lengkap untuk file yang dimodifikasi dan file baru yang diperlukan.

---

### **Tutorial Implementasi**

#### **Langkah 1: Instalasi Library PDF (TCPDF)**

Kita akan menggunakan library `TCPDF` untuk membuat file PDF di sisi server. Cara termudah untuk menginstalnya adalah melalui Composer. Buka terminal atau command prompt di direktori root proyek Anda dan jalankan perintah berikut:

```bash
composer require tecnickcom/tcpdf
```

Ini akan membuat folder `vendor` di proyek Anda, yang berisi library TCPDF dan file `autoload.php` yang akan kita gunakan.

#### **Langkah 2: Siapkan Aset (Logo)**

Pastikan Anda memiliki file logo untuk laporan PDF. Simpan file logo Anda (misalnya `logo.png`) di dalam folder `images/`. Dalam contoh ini, saya akan menggunakan path `images/logo.png`.

#### **Langkah 3: Hapus Fitur Chatbot**

Ikuti langkah-langkah ini untuk menghapus semua komponen chatbot dari aplikasi.

1.  **Hapus File Handler:**
    *   Hapus file `ajax_chatbot_handler.php` dari direktori root proyek Anda.

2.  **Modifikasi `config.php`:**
    *   Hapus baris yang mendefinisikan API Key chatbot.

3.  **Modifikasi `css/style.css`:**
    *   Hapus semua blok CSS yang berhubungan dengan chatbot.

4.  **Modifikasi `includes/footer.php`:**
    *   Hapus elemen HTML dan blok JavaScript yang terkait dengan chatbot.

#### **Langkah 4: Modifikasi dan Buat File untuk Fitur Laporan PDF**

Kita akan memodifikasi `laporan.php` untuk menambahkan tombol dan modal, `css/style.css` untuk styling modal, dan membuat file baru `generate_report.php` untuk memproses dan membuat PDF.

---

### **Kode Lengkap File yang Dimodifikasi dan Baru**

Berikut adalah kode lengkap untuk setiap file yang perlu diubah atau dibuat. Cukup ganti seluruh isi file yang ada dengan kode di bawah ini.

#### **1. `config.php` (Dimodifikasi)**
*   Menghapus `CHATBOT_API_KEY`.

```php
<?php
// File: config.php

// Pengaturan Aplikasi
define('APP_URL', 'http://localhost/nama_proyek'); // GANTI DENGAN URL PROYEK ANDA

// Konfigurasi Database
define('DB_HOST', '127.0.0.1');
define('DB_USER', 'root');
define('DB_PASS', ''); // Ganti jika password root MySQL Anda berbeda
define('DB_NAME', 'db_listin');

// Pengaturan PHPMailer (untuk Lupa Password)
define('SMTP_HOST', 'smtp.gmail.com');
define('SMTP_USERNAME', 'irhamnajibazimulqowi@gmail.com');
define('SMTP_PASSWORD', 'gsra fstt yfrt gpso'); // INI ADALAH APP PASSWORD ANDA
define('SMTP_PORT', 587);
define('SMTP_SECURE', 'tls');
define('EMAIL_FROM_ADDRESS', 'irhamnajibazimulqowi@gmail.com'); 
define('EMAIL_FROM_NAME', 'List In App');

// Pengaturan Error Reporting
error_reporting(E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED); 
ini_set('display_errors', 1); 

date_default_timezone_set('Asia/Jakarta');

if (session_status() == PHP_SESSION_NONE) {
    session_start();
}
?>
```

#### **2. `css/style.css` (Dimodifikasi)**
*   Menghapus semua style chatbot.
*   Menambahkan style untuk modal laporan PDF.

```css
/* ==========================================================================
   Reset dan Base Styles
   ========================================================================== */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    background: #eef1f5;
    font-family: 'Poppins', sans-serif;
    color: #3c4250;
    overflow-x: hidden;
    line-height: 1.55;
}

/* ==========================================================================
   Header
   ========================================================================== */
.header {
    background: #ffffff;
    width: 100%;
    padding: 10px 25px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
    position: sticky;
    top: 0;
    z-index: 1000;
    height: 55px;
}

.header-left {
    display: flex;
    align-items: center;
}

.header-profile-pic {
    width: 30px;
    height: 30px;
    border-radius: 50%;
    margin-right: 12px;
    cursor: pointer;
    border: 1.5px solid #7e47b8;
    object-fit: cover;
}
.header-profile-pic:hover {
    opacity: 0.85;
}

.header-left h2 {
    color: #7e47b8;
    font-size: 1.7rem;
    margin: 0;
}

.search-bar {
    flex-grow: 1;
    max-width: 450px;
    margin: 0 25px;
    display: flex;
    align-items: center;
    background: #f0f3f7;
    padding: 7px 15px;
    border-radius: 25px;
    border: 1px solid #dfe3e8;
}
.search-bar input {
    border: none;
    outline: none;
    width: 100%;
    background: transparent;
    font-size: 0.9rem;
    color: #555;
}
.search-bar input::placeholder {
    color: #888;
}
.search-bar i {
    color: #7e47b8;
    margin-left: 8px;
    font-size: 1rem;
}
.search-bar button i.fa-search {
    color: #7e47b8;
    font-size: 1rem;
}

.header-right {
    display: flex;
    align-items: center;
}
.header-right i.fas {
    color: #5f6368;
    padding: 8px;
    font-size: 1.2rem;
    cursor: pointer;
    transition: color 0.2s, background-color 0.2s;
    margin-left: 10px;
    border-radius: 50%;
}
.header-right i.fas:hover {
    color: #7e47b8;
    background-color: #f0e9f7;
}
.header-right i.fas.fa-bell.has-notif {
    position: relative;
}
.header-right i.fas.fa-bell.has-notif::after {
    content: '';
    position: absolute;
    top: 6px;
    right: 6px;
    width: 8px;
    height: 8px;
    background-color: #ef4444;
    border-radius: 50%;
    border: 1px solid white;
}

.date-container {
    text-align: right;
    margin-left: 15px;
}
.date-container p {
    margin: 0;
    font-size: 0.8rem;
    color: #555;
}
.date-container span {
    color: #7e47b8;
    font-weight: 500;
    font-size: 0.8rem;
}

/* ==========================================================================
   Content Area (Sidebar + Main Content)
   ========================================================================== */
.content {
    display: flex;
    flex: 1;
    padding: 15px;
    gap: 15px;
    max-height: calc(100vh - 55px);
    overflow: hidden;
}

/* ==========================================================================
   Sidebar
   ========================================================================== */
.sidebar {
    background: #2c3e50;
    color: #ecf0f1;
    padding: 15px;
    display: flex;
    flex-direction: column;
    border-radius: 8px;
    height: calc(100vh - 55px - 30px);
    position: sticky;
    top: calc(55px + 15px);
    flex-shrink: 0;
    overflow-y: auto;
}
.sidebar::-webkit-scrollbar {
    display: none;
}

.menu {
    width: 100%;
}
.menu a {
    display: flex;
    align-items: center;
    padding: 10px 12px;
    text-decoration: none;
    color: #bdc3c7;
    transition: background-color 0.2s, color 0.2s;
    margin-bottom: 6px;
    border-radius: 6px;
    font-size: 0.9rem;
}
.menu a:hover {
    background: #34495e;
    color: #fff;
}
.menu a.active {
    background: #7e47b8;
    color: #fff;
    font-weight: 500;
}
.menu a i {
    margin-right: 12px;
    width: 18px;
    text-align: center;
    font-size: 1rem;
}

.logout {
    margin-top: auto;
    color: #bdc3c7;
    text-decoration: none;
    display: flex;
    align-items: center;
    width: 100%;
    padding: 10px 12px;
    border-radius: 6px;
    font-size: 0.9rem;
    transition: background-color 0.2s, color 0.2s;
}
.logout i { margin-right: 12px; }
.logout:hover {
    background-color: #c0392b;
    color: #fff;
}

/* ==========================================================================
   Main Content Area
   ========================================================================== */
.main {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: auto;
    min-width: 0;
    padding: 0 15px 15px 0;
}
.main::-webkit-scrollbar  {
    display: none;
}

.main h2.page-title {
    font-size: 1.5rem;
    color: #2c3e50;
    margin-bottom: 15px;
    padding-bottom: 8px;
    border-bottom: 1px solid #dfe3e8;
    flex-shrink: 0;
}

/* ==========================================================================
   Dashboard Layout (Grid)
   ========================================================================== */
.main-dashboard {
    display: grid;
    grid-template-columns: 2fr 1fr;
    grid-template-rows: auto 1fr;
    gap: 15px;
    flex-grow: 1;
    overflow: hidden;
}
.main-dashboard .page-title { grid-column: 1 / -1; flex-shrink: 0; }
.main-dashboard .todo { grid-row: 2 / span 2; }
.main-dashboard .status { grid-row: 2 / span 1; }
.main-dashboard .completed { grid-row: 3 / span 1; }

.widget, .form-container, .profile-card, .table-container, .filters-container {
    background: #fff;
    border-radius: 8px;
    padding: 18px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
    margin-bottom: 15px;
}
.widget:last-child, .form-container:last-child, .profile-card:last-child, .table-container:last-child, .filters-container:last-child {
    margin-bottom: 0;
    height: auto;
}

.widget h3 {
    font-size: 1.15rem;
    margin-bottom: 12px;
    color: #34495e;
    padding-bottom: 8px;
    border-bottom: 1px solid #f0f0f0;
}
.widget h3 span { font-size: 0.85rem; color: #777; }

#todo-list, #completed-list, #managementTaskList, #historyTaskList {
    padding-right: 2px;
}
.widget.scrollable-list > div[id$="-list"] {
    max-height: 300px;
    overflow-y: auto;
}
.widget.scrollable-list > div[id$="-list"]::-webkit-scrollbar {
    display: none;
}

.task-item-card {
    background-color: #f9fafb;
    border: 1px solid #e7eaec;
    border-radius: 6px;
    padding: 12px;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: box-shadow 0.2s ease, border-color 0.2s ease, border-left-color 0.2s ease, background-color 0.2s ease;
    font-size: 0.85rem;
}
.task-item-card:hover {
    border-color: #d1d5db;
    box-shadow: 0 2px 4px rgba(0,0,0,0.06);
}
.task-item-card .task-details { flex-grow: 1; margin-right: 10px;}
.task-item-card .task-details strong {
    font-size: 0.95rem;
    color: #1f2937;
    display: block;
    margin-bottom: 2px;
}
.task-details a.task-title-link {
    text-decoration: none;
    color: inherit;
}
.task-details a.task-title-link:hover strong {
    color: #7e47b8;
}
.task-item-card .task-details .description {
    font-size: 0.8rem;
    color: #4b5563;
    margin-bottom: 4px;
    word-break: break-word;
}
.task-item-card .task-details .meta-info {
    font-size: 0.75rem;
    color: #6b7280;
}
.task-item-card.completed .task-details strong {
    color: #868e99;
    font-weight: normal;
}
.task-item-card.completed .task-details .description,
.task-item-card.completed .task-details .meta-info {
    color: #9ca3af;
}
.task-item-card .task-actions { display: flex; align-items: center; flex-shrink: 0; }
.task-item-card .task-actions select,
.task-item-card .task-actions select.task-status-select {
    padding: 6px 8px;
    border-radius: 5px;
    border: 1px solid #d1d5db;
    font-size: 0.8rem;
    background-color: #fff;
    margin-right: 8px;
    min-width: 100px;
    cursor: pointer;
}
.task-item-card .task-actions button, .task-item-card .task-actions a {
    background: transparent;
    color: #6b7280;
    border: none;
    padding: 5px;
    border-radius: 50%;
    cursor: pointer;
    transition: color 0.2s, background-color 0.2s;
    font-size: 0.9rem;
    margin-left: 4px;
    width: 28px; height: 28px;
    display: inline-flex; align-items: center; justify-content: center;
    text-decoration: none;
}
.task-item-card .task-actions button:hover, .task-item-card .task-actions a:hover {
    background-color: #e5e7eb;
}
.task-item-card .task-actions .edit-btn:hover, .task-item-card .task-actions .reopen-btn:hover { color: #2563eb; }
.task-item-card .task-actions .delete-btn:hover { color: #dc2626; }

.progress-container { display: flex; justify-content: space-around; margin-top: 15px; }
.progress { text-align: center; }
.progress-circle {
    width: 65px; height: 65px;
    background: conic-gradient(#4caf50 0% var(--progress), #e7eaec var(--progress) 100%);
    border-radius: 50%; display: flex; align-items: center; justify-content: center; position: relative; margin: 0 auto 8px;
}
.progress-circle.in-progress { background: conic-gradient(#2196f3 0% var(--progress), #e7eaec var(--progress) 100%); }
.progress-circle.not-started { background: conic-gradient(#f44336 0% var(--progress), #e7eaec var(--progress) 100%); }
.progress-circle::before {
    content: attr(data-progress) '%';
    position: absolute; width: 48px; height: 48px;
    background: #fff; border-radius: 50%; display: flex; align-items: center; justify-content: center;
    font-size: 0.85rem; font-weight: 600; color: #333;
}
.progress p { font-size: 0.8rem; color: #4b5563; }

/* ==========================================================================
   Pop-up Notifikasi & Kalender
   ========================================================================== */
#notification-popup, #calendar-popup {
    position: fixed;
    top: 60px;
    background: #fff; border: 1px solid #ddd; border-radius: 8px; padding: 15px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1); z-index: 1001;
    opacity: 0; transform: translateY(-10px) scale(0.95);
    transition: opacity 0.2s ease, transform 0.2s ease, visibility 0s 0.2s;
    display: none;
}
#notification-popup { right: 20px; width: 300px; }
#calendar-popup { right: 65px; }

#notification-popup.show, #calendar-popup.show {
    opacity: 1; transform: translateY(0) scale(1);
    display: block;
    transition: opacity 0.2s ease, transform 0.2s ease, visibility 0s 0s;
}

#notification-popup h4 {
    margin-top: 0; font-size: 1rem; border-bottom: 1px solid #eee;
    padding-bottom: 8px; margin-bottom: 10px; color: #333;
}
#notification-list { list-style: none; padding: 0; max-height: 250px; overflow-y: auto; }
#notification-list::-webkit-scrollbar { display: none; }
#notification-list li {
    padding: 8px 5px; border-bottom: 1px solid #f0f0f0; font-size: 0.85rem; line-height: 1.4;
    display: flex; justify-content: space-between; align-items: center; text-align: left;
}
#notification-list li:last-child { border-bottom: none; }
.notif-time { font-size: 0.75rem; color: #777; white-space: nowrap; margin-left: 10px; }

/* ==========================================================================
   General Form Styles
   ========================================================================== */
.form-group { margin-bottom: 15px; }
.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: 500;
    font-size: 0.85rem;
}
.form-group input[type="text"],
.form-group input[type="email"],
.form-group input[type="password"],
.form-group input[type="date"],
.form-group textarea,
.form-group select {
    width: 100%;
    padding: 9px 12px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 0.9rem;
    background-color: #f9fafb;
    color: #1f2937;
}
.form-group input:focus, .form-group textarea:focus, .form-group select:focus {
    border-color: #7e47b8;
    box-shadow: 0 0 0 2.5px rgba(126, 71, 184, 0.2);
    outline: none;
    background-color: #fff;
}
.form-group textarea { min-height: 90px; resize: vertical; }

/* ==========================================================================
   Tombol (Buttons)
   ========================================================================== */
.btn {
    padding: 9px 18px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.9rem;
    font-weight: 500;
    transition: background-color 0.2s ease, box-shadow 0.2s ease, transform 0.1s ease;
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
}
.btn:active { transform: translateY(1px); }

.btn-primary { background-color: #7e47b8; color: white; }
.btn-primary:hover { background-color: #6a3aa2; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.btn-secondary { background-color: #6c757d; color: white; }
.btn-secondary:hover { background-color: #545b62; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.btn-danger { background-color: #e74c3c; color: white; }
.btn-danger:hover { background-color: #c0392b; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.btn-full-width { display: block; width: 100%; }

.form-actions {
    margin-top: 20px;
    display: flex;
    gap: 10px;
    justify-content: center;
}

/* ==========================================================================
   Filters Container
   ========================================================================== */
.filters-container {
    background: #fff;
    border-radius: 8px;
    padding: 12px 15px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
    margin-bottom: 15px;
    display: flex;
    gap: 10px;
    align-items: flex-end;
    flex-wrap: wrap;
    flex-shrink: 0;
}
.filters-container .form-group {
    margin-bottom: 0;
    flex-grow: 1;
    min-width: 150px;
}
.filters-container label {
    font-size: 0.8rem;
    margin-bottom: 3px;
}
.filters-container select,
.filters-container input[type="text"] {
    padding: 7px 10px;
    font-size: 0.85rem;
    height: 34px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    width: 100%;
}
.filters-container .btn-filter-group {
    display: flex;
    gap: 8px;
    align-items: flex-end;
    margin-left: auto;
}
.filters-container .btn {
    padding: 7px 15px;
    font-size: 0.85rem;
    height: 34px;
    min-width: 100px;
    white-space: nowrap;
}

/* ==========================================================================
   Pesan "Tidak Ada Tugas"
   ========================================================================== */
.no-tasks-message {
    text-align:center; padding: 25px; color: #6b7280; font-style: italic;
    background-color: #f9fafb; border-radius: 6px; border: 1px dashed #e5e7eb;
}

/* ==========================================================================
   Halaman Profil
   ========================================================================== */
.main .profile-card {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(5px);
    padding: 30px 35px;
    border-radius: 10px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
    width: 100%;
    max-width: 380px;
    text-align: center;
    position: relative;
    z-index: 2;
    margin: 20px auto;
}
.profile-card img#profileImageMain {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    margin-bottom: 15px;
    border: 3px solid #7e47b8;
    object-fit: cover;
    display: block;
    margin-left: auto;
    margin-right: auto;
}
.profile-card h2#profileName { color: #333; margin-bottom: 5px; font-size: 1.6rem; }
.profile-card p#profileEmail { color: #666; margin-bottom: 25px; font-size: 1rem; }
.profile-actions {
    display: flex;
    justify-content: center;
    gap: 15px;
    margin-top: 15px;
}

.main .profile-page-container {
    display: flex;
    flex-direction: row;
    gap: 20px;
    padding: 10px;
}

.main .profile-card-display {
    background: #ffffff;
    border-radius: 12px;
    padding: 25px 30px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
    text-align: center;
    flex-basis: 350px;
    flex-shrink: 0;
    height: auto;
}
.profile-card-display img#profileImageMain {
    width: 130px;
    height: 130px;
    border-radius: 50%;
    margin-bottom: 18px;
    border: 4px solid #7e47b8;
    object-fit: cover;
    display: block;
    margin-left: auto;
    margin-right: auto;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}
.profile-card-display h2#profileName {
    color: #2c3e50;
    margin-bottom: 6px;
    font-size: 1.7rem;
    font-weight: 600;
}
.profile-card-display p#profileEmail {
    color: #555;
    margin-bottom: 0;
    font-size: 1rem;
}

.main .profile-actions-and-info {
    background: #ffffff;
    border-radius: 12px;
    padding: 25px 30px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
    flex-grow: 1;
}
.profile-actions-and-info h3.section-title {
    font-size: 1.2rem;
    color: #34495e;
    margin-bottom: 20px;
    padding-bottom: 10px;
    border-bottom: 1px solid #f0f0f0;
}

.profile-actions a.btn {
    width: 100%;
    text-align: center;
    padding-top: 10px;
    padding-bottom: 10px;
    font-size: 0.95rem;
}
.profile-actions a.btn i {
    margin-right: 8px;
}

/* ==========================================================================
   Pengaturan Layout Spesifik .main dan Widget di Dashboard
   ========================================================================== */
.main-dashboard .widget {
    background: #fff;
    border-radius: 8px;
    padding: 18px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
    display: flex;
    flex-direction: column;
    overflow: hidden;
}
.main-dashboard .widget h3 {
    font-size: 1.15rem;
    margin-bottom: 12px;
    color: #34495e;
    padding-bottom: 8px;
    border-bottom: 1px solid #f0f0f0;
    flex-shrink: 0;
}
.main-dashboard .widget > div[id$="-list"] {
    flex-grow: 1;
    overflow-y: auto;
    padding-right: 5px;
}
.main-dashboard .widget > div[id$="-list"]::-webkit-scrollbar  {
    display: none;
}
.main-dashboard .widget.status .progress-container{
    flex-grow: 1;
    display: flex;
    align-items: center;
    justify-content: space-around;
    overflow-y: visible;
}

/* ==========================================================================
   Table Container (untuk halaman Manajemen & Riwayat)
   ========================================================================== */
.table-container {
    background: #fff;
    border-radius: 8px;
    padding: 18px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}
.table-container #managementTaskList,
.table-container #historyTaskList {
    flex-grow: 1;
    overflow-y: auto;
    padding-right: 8px;
}

/* Menyembunyikan scrollbar di Webkit (Chrome, Edge, Safari) */
.table-container #managementTaskList::-webkit-scrollbar,
.table-container #historyTaskList::-webkit-scrollbar {
    display: none;
}

/* ==========================================================================
   Dashboard Layout Baru (Grid 4 Kotak / 2 Kolom) dan Styling Terkait
   ========================================================================== */
html {
    font-size: 14px;
}

.dashboard-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: auto 1fr;
    gap: 15px;
    flex-grow: 1;
    overflow: hidden;
    height: calc(100% - 40px - 15px);
    align-items: stretch;
}

.dashboard-grid .page-title-container {
    grid-column: 1 / -1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 0;
    flex-shrink: 0;
    height: 40px;
    border-bottom: 1px solid #dfe3e8;
}
.dashboard-grid .page-title-container .page-title {
    margin-bottom: 0;
    border-bottom: none;
}

.dashboard-left-column,
.dashboard-right-column {
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.dashboard-grid .widget h3 {
    font-size: 1.05rem;
    margin-bottom: 8px;
    color: #34495e;
    padding-bottom: 8px;
    border-bottom: 1px solid #f0f0f0;
    flex-shrink: 0;
}

.dashboard-grid .widget .widget-content-area {
    flex-grow: 1;
    overflow-y: auto;
    position: relative;
    padding: 5px;
    padding-block-end: 10px;
}

.dashboard-grid .widget .widget-content-area::-webkit-scrollbar {
    display: none;
}

.dashboard-grid .widget .chart-container canvas {
    max-height: 180px;
    width: 100% !important;
    height: auto !important;
}

.dashboard-right-column .widget {
    height: 100%;
}
.dashboard-right-column #dashboard-active-tasks-list {
    overflow-y: auto;
    padding-right: 8px;
    flex-grow: 1;
    height: 68vh;
}

.dashboard-right-column #dashboard-active-tasks-list::-webkit-scrollbar {
    display: none;
}

.status-progress-container {
    display: flex;
    justify-content: space-around;
}
.progress-item {
    text-align: center;
    flex: 1;
}
.progress-circle-display,
.progress-circle {
    width: 70px;
    height: 70px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    margin: 0 auto 8px;
    background: conic-gradient(
        var(--progress-color, #e7eaec) 0% var(--progress-percent, 0%),
        #e7eaec var(--progress-percent, 0%) 100%
    );
}
.progress-circle-display::before,
.progress-circle::before {
    content: attr(data-progress) '%';
    position: absolute;
    width: calc(100% - 16px);
    height: calc(100% - 16px);
    background: #fff;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.85rem;
    font-weight: 600;
    color: #333;
    z-index: 1;
}
.progress-item p {
    font-size: 0.8rem;
    color: #4b5563;
    margin-top: 5px;
}

/* ==========================================================================
   Filter Performa Dashboard & Filter Umum (Versi Minimalis)
   ========================================================================== */
.performance-filter-container,
.filters-container
{
    background: #fff;
    border-radius: 8px;
    padding: 10px 15px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.07);
    margin-bottom: 15px;
    display: flex;
    gap: 10px;
    align-items: flex-end;
    flex-wrap: wrap;
    flex-shrink: 0;
    align-items: center;
}
.performance-filter-container {
    margin-bottom: 0;
    padding: 0;
    box-shadow: none;
    background: transparent;
}

.performance-filter-container .form-group,
.filters-container .form-group {
    margin-bottom: 0;
    flex-grow: 0;
    min-width: 150px;
}
.performance-filter-container label,
.filters-container label {
    font-size: 0.8rem;
    margin-bottom: 4px;
    display: block;
}
.performance-filter-container select,
.performance-filter-container input[type="text"],
.filters-container select,
.filters-container input[type="text"] {
    padding: 7px 10px;
    font-size: 0.85rem;
    height: 36px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    width: 100%;
}

.filter-preset-buttons {
    display: flex;
    gap: 5px;
    align-items: flex-end;
    margin-bottom: 0;
}
.filter-preset-buttons .btn {
    padding: 7px 10px;
    font-size: 0.8rem;
    height: 36px;
    background-color: #f0f3f7;
    color: #555;
    border: 1px solid #dfe3e8;
}
.filter-preset-buttons .btn.active,
.filter-preset-buttons .btn:hover {
    background-color: #7e47b8;
    color: white;
    border-color: #7e47b8;
}

.filter-action-buttons {
    display: flex;
    gap: 8px;
    align-items: flex-end;
    margin-left: auto;
}
.filter-action-buttons .btn {
    padding: 7px 15px;
    font-size: 0.85rem;
    height: 36px;
    min-width: 80px;
}

.performance-filter-form {
    display: flex;
    gap: 8px;
    align-items: center;
    margin-bottom: 10px;
    flex-wrap: nowrap;
    padding: 5px 0;
}
.performance-filter-form label {
    font-size: 0.85rem;
    white-space: nowrap;
    margin-right: 5px;
    color: #555;
}
.performance-filter-form .filter-preset-buttons {
    display: flex;
    gap: 5px;
}
.performance-filter-form .filter-preset-buttons .btn {
    padding: 5px 10px;
    font-size: 0.75rem;
    height: 30px;
    background-color: #f0f3f7;
    color: #555;
    border: 1px solid #dfe3e8;
}
.performance-filter-form .filter-preset-buttons .btn.active,
.performance-filter-form .filter-preset-buttons .btn:hover {
    background-color: #7e47b8;
    color: white;
    border-color: #7e47b8;
}
.performance-filter-form input[type="text"] {
    padding: 5px 8px;
    font-size: 0.8rem;
    height: 30px;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    width: 150px;
    margin-left: 5px;
}
.performance-filter-form .btn-apply-perf {
    padding: 5px 12px;
    font-size: 0.8rem;
    height: 30px;
    margin-left: 5px;
}

/* ==========================================================================
   Responsive Design
   ========================================================================== */
@media (max-width: 992px) {
    .main-dashboard {
        grid-template-columns: 1fr;
        grid-template-rows: auto auto auto auto;
    }
    .main-dashboard .todo { grid-row: 2; }
    .main-dashboard .status { grid-row: 3; }
    .main-dashboard .completed { grid-row: 4; }

    .dashboard-grid {
        grid-template-columns: 1fr;
        height: auto;
    }
    .performance-filter-container {
        flex-direction: column;
        align-items: stretch;
    }
    .performance-filter-container .form-group,
    .performance-filter-container .filter-preset-buttons,
    .performance-filter-container .filter-action-buttons {
        width: 100%;
        margin-left: 0;
    }

    .sidebar {
    }
    .search-bar { max-width: 300px; margin: 0 15px; }
}

@media (max-width: 768px) {
    html { font-size: 13.5px; }

    .header {
        height: auto;
        padding: 8px 15px;
        flex-wrap: wrap;
    }
    .header-left { order: 1; width: auto; margin-bottom: 8px; }
    .header-right { order: 2; margin-left: auto; margin-bottom: 8px; }
    .search-bar { order: 3; width: 100%; margin: 0 0 5px 0; max-width: none; }

    .content {
        flex-direction: column;
        padding: 10px;
        gap: 10px;
        max-height: none;
        overflow: visible;
        height: auto;
    }

    .sidebar {
        width: auto;
        position: static;
        height: auto;
        flex-direction: row;
        justify-content: space-between;
        padding: 10px;
        overflow-x: hidden;
        white-space: nowrap;
        border-radius: 8px;
        margin-bottom:10px;
    }
    .sidebar .menu { display: flex; gap: 5px; width: auto; justify-content: center;}
    .sidebar .menu a { margin-bottom: 0; padding: 8px 10px; font-size: 0.85rem; }
    .sidebar .logout { display: flex; width: 7vh;}
    .sidebar .menu a span { display: none;}
    .sidebar a span { display: none;}

    .main {
        padding: 0;
        overflow-y: visible;
    }
    .main h2.page-title { font-size: 1.3rem; margin-bottom: 10px; }

    .widget.scrollable-list > div[id$="-list"],
    #todo-list, #completed-list, #managementTaskList, #historyTaskList {
        max-height: 250px;
    }

    .filters-container {
        flex-direction: column;
        align-items: stretch;
    }
    .filters-container .form-group,
    .filters-container .filter-preset-buttons,
    .filters-container .filter-action-buttons {
        width: 100%;
        margin-left: 0;
    }
    .filter-action-buttons {
        flex-direction: column;
    }
    .filter-action-buttons .btn {
        width: 100%;
    }

    .main .profile-page-container {
        flex-direction: column;
    }
    .profile-actions {
        flex-direction: column;
        gap: 10px;
    }
    .profile-actions a.btn {
        width: 100%;
    }
}

/* ==========================================================================
   Popup Notifikasi Styling Lanjutan
   ========================================================================== */
#notification-popup {
    max-height: 400px;
}

#notification-popup .notification-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 15px;
    border-bottom: 1px solid #eee;
    position: sticky;
    top: 0;
    background-color: white;
    z-index: 1;
}

#notification-popup .notification-header h4 {
    margin: 0;
    font-size: 1.1em;
}

.btn-clear-notif {
    background-color: #e74c3c;
    color: white;
    border: none;
    padding: 6px 10px;
    font-size: 0.8em;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.btn-clear-notif:hover {
    background-color: #c0392b;
}

#notification-popup ul#notification-list {
    list-style-type: none;
    padding: 0;
    margin: 0;
    overflow-y: auto;
    height: 50vh;
}
#notification-popup ul#notification-list::-webkit-scrollbar  {
    display: none;
}

#notification-popup ul#notification-list li {
    padding: 10px 15px;
    border-bottom: 1px solid #f0f0f0;
    font-size: 0.9em;
    display: flex;
    flex-direction: column;
}
#notification-popup ul#notification-list li:last-child {
    border-bottom: none;
}

#notification-popup ul#notification-list li.no-notifications {
    text-align: center;
    color: #777;
    padding: 20px 15px;
}

#notification-popup .notif-time {
    font-size: 0.8em;
    color: #888;
    margin-top: 4px;
}

#notification-popup .notif-success {
    border-left: 3px solid #2ecc71;
    background-color: #f2fdf5;
}

#notification-popup .notif-error {
    border-left: 3px solid #e74c3c;
    background-color: #fef6f5;
}
#notification-popup .notif-info {
    border-left: 3px solid #3498db;
    background-color: #f5faff;
}

.header-right .fa-bell.has-notif {
    color: #e74c3c;
    position: relative;
}
.header-right .fa-bell.has-notif::after {
    content: '';
    position: absolute;
    top: 0px;
    right: 0px;
    width: 8px;
    height: 8px;
    background-color: red;
    border-radius: 50%;
    border: 1px solid white;
}

/* ==========================================================================
   MODIFIKASI TAMBAHAN - REVISI PROFIL, TEMA, NOTIF, STATUS, SIDEBAR
   ========================================================================== */

/* ==========================================================================
   Tema Gelap (Dark Theme)
   ========================================================================== */
html.dark-theme-active body {
    background-color: #121212;
    color: #e0e0e0;
    overflow:auto;
}
html.dark-theme-active .header {
    background-color: #1e1e1e;
    box-shadow: 0 1px 3px rgba(255, 255, 255, 0.06);
}
html.dark-theme-active .header-left h2,
html.dark-theme-active .header-left h2.app-title-toggle,
html.dark-theme-active .search-bar i,
html.dark-theme-active .search-bar button i.fa-search {
    color: #bb86fc;
}
html.dark-theme-active .header-right i.fas { color: #bdbdbd; }
html.dark-theme-active .header-right i.fas:hover { color: #bb86fc; background-color: rgba(187, 134, 252, 0.1); }
html.dark-theme-active .date-container p { color: #bdbdbd; }
html.dark-theme-active .date-container span { color: #bb86fc; }

html.dark-theme-active .search-bar { background: #2c2c2c; border-color: #444; }
html.dark-theme-active .search-bar input { color: #e0e0e0; }
html.dark-theme-active .search-bar input::placeholder { color: #aaa; }

html.dark-theme-active .sidebar { background: #1e1e1e; color: #e0e0e0; }
html.dark-theme-active .menu a { color: #bdbdbd; }
html.dark-theme-active .menu a:hover { background: #333; color: #fff; }
html.dark-theme-active .menu a.active { background: #bb86fc; color: #121212; }
html.dark-theme-active .logout { color: #bdbdbd; }
html.dark-theme-active .logout:hover { background-color: #cf6679; color: #121212; }

html.dark-theme-active .main h2.page-title { color: #f5f5f5; border-bottom-color: #444; }

html.dark-theme-active .widget,
html.dark-theme-active .form-container,
html.dark-theme-active .profile-info-card,
html.dark-theme-active .profile-actions-card,
html.dark-theme-active .table-container,
html.dark-theme-active .filters-container {
    background: #2c2c2c;
    box-shadow: 0 1px 3px rgba(0,0,0,0.3);
    border: 1px solid #444;
}
html.dark-theme-active .widget h3,
html.dark-theme-active .profile-actions-card h3.card-title,
html.dark-theme-active .dashboard-grid .widget h3 {
    color: #f5f5f5;
    border-bottom-color: #444;
}
html.dark-theme-active .widget.status .progress-container p,
html.dark-theme-active .progress-item p { color: #bdbdbd; }
html.dark-theme-active .progress-circle-display::before,
html.dark-theme-active .progress-circle::before { background: #2c2c2c; color: #e0e0e0; }


html.dark-theme-active .task-item-card { background-color: #373737; border-color: #555; }
html.dark-theme-active .task-item-card:hover { border-color: #777; box-shadow: 0 2px 4px rgba(0,0,0,0.2); }
html.dark-theme-active .task-item-card .task-details strong,
html.dark-theme-active .task-details a.task-title-link strong { color: #f5f5f5; }
html.dark-theme-active .task-details a.task-title-link:hover strong { color: #bb86fc; }
html.dark-theme-active .task-item-card .task-details .description { color: #ccc; }
html.dark-theme-active .task-item-card .task-details .meta-info { color: #bbb; }
html.dark-theme-active .task-item-card .task-actions select,
html.dark-theme-active .task-item-card .task-actions select.task-status-select { background-color: #373737; border-color: #555; color: #e0e0e0; }
html.dark-theme-active .task-item-card .task-actions button,
html.dark-theme-active .task-item-card .task-actions a { color: #bdbdbd; }
html.dark-theme-active .task-item-card .task-actions button:hover,
html.dark-theme-active .task-item-card .task-actions a:hover { background-color: #4f4f4f; }
html.dark-theme-active .task-item-card .task-actions .edit-btn:hover,
html.dark-theme-active .task-item-card .task-actions .reopen-btn:hover { color: #90caf9; }
html.dark-theme-active .task-item-card .task-actions .delete-btn:hover { color: #ef9a9a; }

/* ==========================================================================
   Warna Status Card - Light Mode
   ========================================================================== */
.task-item-card.status-not-started {
    background-color: #f9fafb;
    border-left: 4px solid #b0bec5;
}
.task-item-card.status-in-progress {
    background-color: #e3f2fd;
    border-left: 4px solid #2196f3;
}
.task-item-card.status-completed-history {
    background-color: #e8f5e9;
    border-left: 4px solid #4caf50;
}
.task-item-card.status-overdue-uncompleted-history {
    background-color: #fff8e1;
    border-left: 4px solid #ffb300;
}

/* ==========================================================================
   Warna Status Card - Dark Mode
   ========================================================================== */
html.dark-theme-active .task-item-card.status-not-started {
    background-color: #373737;
    border-left-color: #78909c;
}
html.dark-theme-active .task-item-card.status-in-progress {
    background-color: #2a3b4d;
    border-left-color: #64b5f6;
}
html.dark-theme-active .task-item-card.status-completed-history {
    background-color: #2e4b35;
    border-left-color: #81c784;
}
html.dark-theme-active .task-item-card.status-overdue-uncompleted-history {
    background-color: #4d4021;
    border-left-color: #ffca28;
}

.task-item-card.status-completed-history .task-details strong,
html.dark-theme-active .task-item-card.status-completed-history .task-details strong {
}

.main-content-manajemen .task-item-card[data-task-id] { cursor: pointer; }
.main-content-manajemen .task-item-card[data-task-id]:hover { box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
html.dark-theme-active .main-content-manajemen .task-item-card[data-task-id]:hover { box-shadow: 0 4px 10px rgba(255,255,255,0.1); }


html.dark-theme-active .form-group label { color: #ccc; }
html.dark-theme-active .form-group input[type="text"],
html.dark-theme-active .form-group input[type="email"],
html.dark-theme-active .form-group input[type="password"],
html.dark-theme-active .form-group input[type="date"],
html.dark-theme-active .form-group textarea,
html.dark-theme-active .form-group select {
    background-color: #373737; border-color: #555; color: #e0e0e0;
}
html.dark-theme-active .form-group input:focus,
html.dark-theme-active .form-group textarea:focus,
html.dark-theme-active .form-group select:focus {
    border-color: #bb86fc; box-shadow: 0 0 0 2.5px rgba(187, 134, 252, 0.3); background-color: #3c3c3c;
}

html.dark-theme-active .btn-primary { background-color: #bb86fc; color: #121212; }
html.dark-theme-active .btn-primary:hover { background-color: #a06fec; }
html.dark-theme-active .btn-secondary { background-color: #555; color: #e0e0e0; }
html.dark-theme-active .btn-secondary:hover { background-color: #666; }
html.dark-theme-active .btn-danger { background-color: #cf6679; color: #121212; }
html.dark-theme-active .btn-danger:hover { background-color: #b04c5f; }

html.dark-theme-active .no-tasks-message { color: #aaa; background-color: #373737; border-color: #555; }

/* ==========================================================================
   Dark Theme Popups (Notifikasi & Kalender)
   ========================================================================== */
html.dark-theme-active #notification-popup,
html.dark-theme-active #calendar-popup {
    background: #2c2c2c;
    border: 1px solid #444;
    color: #e0e0e0;
}
html.dark-theme-active #notification-popup .notification-header,
html.dark-theme-active #calendar-popup .calendar-header { /* Assuming .calendar-header for consistency */
    background-color: #2c2c2c;
    border-bottom-color: #444;
}
html.dark-theme-active #notification-popup .notification-header h4,
html.dark-theme-active #calendar-popup .calendar-header h4 { /* Assuming h4 in calendar popup header */
    color: #f5f5f5;
}
html.dark-theme-active #notification-popup ul#notification-list li {
    border-bottom-color: #444;
    /* Text color inherited from #notification-popup */
}
html.dark-theme-active #notification-popup .notif-time,
html.dark-theme-active #calendar-popup .notif-time { /* If calendar uses .notif-time */
    color: #aaa;
}
html.dark-theme-active .btn-clear-notif {
    background-color: #cf6679;
    color: #121212;
}
html.dark-theme-active .btn-clear-notif:hover {
    background-color: #b04c5f;
}

/* ==========================================================================
   Dark Theme Jenis Notifikasi (Revisi Warna untuk Kejelasan)
   ========================================================================== */
html.dark-theme-active #notification-popup .notif-success {
    border-left-color: #66bb6a; /* Material Green 400 */
    background-color: rgba(102, 187, 106, 0.15);
}
html.dark-theme-active #notification-popup .notif-error {
    border-left-color: #ef5350; /* Material Red 400 */
    background-color: rgba(239, 83, 80, 0.15);
}
html.dark-theme-active #notification-popup .notif-info {
    border-left-color: #42a5f5; /* Material Blue 400 */
    background-color: rgba(66, 165, 245, 0.15);
}
html.dark-theme-active #notification-popup .notif-deadline_soon {
    border-left-color: #ffc107; /* Amber 500 (lebih terang dari #ffa000) */
    background-color: rgba(255, 193, 7, 0.25); /* Latar belakang kuning lembut transparan */
}
html.dark-theme-active #notification-popup .notif-deadline_soon .message-content strong {
    color: #fff59d; /* Kuning sangat terang untuk kontras */
}
html.dark-theme-active #notification-popup .notif-deadline_soon .notif-time {
    color: #ffe082; /* Kuning terang untuk waktu */
}
html.dark-theme-active #notification-popup ul#notification-list li.no-notifications {
    color: #aaa; /* Warna teks untuk pesan 'tidak ada notifikasi' */
}


/* ==========================================================================
   Dark Theme Kalender (Contoh Styling Internal)
   ========================================================================== */
html.dark-theme-active #calendar-popup .calendar-day-names span { /* Misal: Sen, Sel, Rab */
    color: #bb86fc; /* Warna aksen */
    font-weight: 500;
}
html.dark-theme-active #calendar-popup .calendar-date { /* Untuk setiap tanggal */
    color: #e0e0e0;
    cursor: pointer;
}
html.dark-theme-active #calendar-popup .calendar-date:hover {
    background-color: #3a3a3a;
    border-radius: 4px;
}
html.dark-theme-active #calendar-popup .calendar-date.today {
    background-color: #bb86fc;
    color: #121212;
    border-radius: 50%;
    font-weight: bold;
}
html.dark-theme-active #calendar-popup .calendar-date.selected {
    background-color: #7e47b8; /* Warna primer lebih gelap untuk 'selected' */
    color: #ffffff;
    border-radius: 50%;
}
html.dark-theme-active #calendar-popup .calendar-date.other-month { /* Tanggal dari bulan lain */
    color: #757575; /* Redup */
    cursor: default;
}
html.dark-theme-active #calendar-popup .calendar-navigation button { /* Tombol navigasi bulan (prev/next) */
    color: #bb86fc;
    background: transparent;
    border: none;
}
html.dark-theme-active #calendar-popup .calendar-navigation button:hover {
    background-color: rgba(187, 134, 252, 0.1);
}


/* ==========================================================================
   Styling Halaman Profil (Dark Theme sudah diintegrasikan di atas)
   ========================================================================== */
.profile-page-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    align-items: flex-start;
}

.profile-info-card {
    flex: 1 1 320px;
    min-width: 280px;
    background: #fff;
    border-radius: 10px;
    padding: 25px 30px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
    text-align: center;
    height: 75vh;
}
.profile-info-card img#profileImageMain {
    width: 120px; height: 120px; border-radius: 50%;
    margin: 0 auto 20px auto;
    border: 4px solid #7e47b8;
    object-fit: cover;
    display: block;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
html.dark-theme-active .profile-info-card img#profileImageMain {
    border-color: #bb86fc;
    box-shadow: 0 2px 8px rgba(255,255,255,0.1);
}
.profile-info-card h2#profileName {
    color: #2c3e50; margin-bottom: 8px; font-size: 1.8rem; font-weight: 600;
}
html.dark-theme-active .profile-info-card h2#profileName { color: #f5f5f5; }
.profile-info-card p#profileEmail {
    color: #555; margin-bottom: 12px; font-size: 1rem;
}
html.dark-theme-active .profile-info-card p#profileEmail { color: #ccc; }
.profile-info-card .member-since {
    font-size: 0.8rem; color: #777; margin-top: 10px;
}
html.dark-theme-active .profile-info-card .member-since { color: #aaa; }

.profile-actions-card {
    flex: 1 1 380px;
    min-width: 300px;
    background: #fff;
    border-radius: 10px;
    padding: 20px 25px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
    height: 75vh;
}
.profile-actions-card .card-title {
    font-size: 1.25rem;
    color: #34495e;
    margin-top: 0;
    margin-bottom: 20px;
    padding-bottom: 10px;
    border-bottom: 1px solid #e7e7e7;
}
html.dark-theme-active .profile-actions-card .card-title {
    border-bottom-color: #444;
}

.profile-action-buttons .btn,
.profile-theme-settings .theme-toggle-item {
    display: flex;
    align-items: center;
    width: 100%;
    padding: 12px 15px;
    margin-bottom: 10px;
    border-radius: 8px;
    text-align: left;
    font-size: 0.95rem;
}
.profile-action-buttons .btn i,
.profile-theme-settings .theme-toggle-item i {
    margin-right: 12px;
    width: 20px;
    text-align: center;
    font-size: 1.1em;
}

.profile-theme-settings .theme-toggle-item {
    justify-content: space-between;
    background-color: #f8f9fa;
    border: 1px solid #e9ecef;
    color: #495057;
}
html.dark-theme-active .profile-theme-settings .theme-toggle-item {
    background-color: #3a3a3a;
    border-color: #555;
    color: #e0e0e0;
}

/* ==========================================================================
   Styling Toggle Switch Tema
   ========================================================================== */
.theme-toggle-switch { position: relative; display: inline-block; width: 50px; height: 26px; }
.theme-toggle-switch input { opacity: 0; width: 0; height: 0; }
.theme-slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #ccc; transition: .4s; border-radius: 26px; }
.theme-slider:before { position: absolute; content: ""; height: 20px; width: 20px; left: 3px; bottom: 3px; background-color: white; transition: .4s; border-radius: 50%; }
input:checked + .theme-slider { background-color: #7e47b8; }
html.dark-theme-active input:checked + .theme-slider { background-color: #bb86fc; }
input:checked + .theme-slider:before { transform: translateX(24px); }

/* ==========================================================================
   Sidebar Toggle
   ========================================================================== */
.sidebar {
    transition: margin-left 0.3s ease-in-out;
    flex-shrink: 0;
}
body.sidebar-hidden .sidebar {
    margin-left: -200px;
}

.header-left h2.app-title-toggle {
    cursor: pointer;
    user-select: none;
}

/* ==========================================================================
   Penyesuaian Responsive Profil & Sidebar
   ========================================================================== */
@media (max-width: 768px) {
    .profile-page-container {
        flex-direction: column;
    }
    .profile-info-card, .profile-actions-card {
        flex-basis: 100%;
        width: 100%;
    }
    body.sidebar-hidden .sidebar {
      margin-left: 0;
    }
}

/* ==========================================================================
   Filter Riwayat
   ========================================================================== */
.filters-container .form-group label[for="filterHistoryType"] {
}
.filters-container select#filterHistoryType {
    min-width: 180px;
}

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
#reportTasksList::-webkit-scrollbar  {
    display: none;
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

/* Laporan Page - Calendar Markers */
.calendar-day-report {
    position: relative; /* Diperlukan untuk positioning absolut marker */
}

.calendar-markers-container {
    position: absolute;
    bottom: 2px; /* Sesuaikan posisi vertikal */
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 2px; /* Jarak antar marker jika ada lebih dari satu */
}

.calendar-marker {
    display: inline-block;
    width: 6px; /* Ukuran marker */
    height: 6px; /* Ukuran marker */
    border-radius: 50%;
}

.marker-active { background-color: #2196f3; /* Biru untuk aktif */ }
.marker-completed { background-color: #4caf50; /* Hijau untuk selesai */ }
.marker-overdue { background-color: #f44336; /* Merah untuk terlewat */ }

html.dark-theme-active .marker-active { background-color: #64b5f6; }
html.dark-theme-active .marker-completed { background-color: #81c784; }
html.dark-theme-active .marker-overdue { background-color: #ef5350; }

/* Penyesuaian agar marker tidak mengganggu teks tanggal */
.calendar-day-report span.day-number { /* Jika Anda membungkus nomor tanggal dalam span */
    position: relative;
    z-index: 1;
}

/* MODAL PDF REPORT STYLES - BARU */
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2000;
    opacity: 0;
    visibility: hidden;
    transition: opacity 0.3s ease, visibility 0s 0.3s;
}
.modal-overlay.show {
    opacity: 1;
    visibility: visible;
    transition: opacity 0.3s ease, visibility 0s 0s;
}
.modal-container {
    background: #fff;
    padding: 25px 30px;
    border-radius: 10px;
    box-shadow: 0 5px 20px rgba(0,0,0,0.2);
    width: 100%;
    max-width: 500px;
    transform: scale(0.95);
    transition: transform 0.3s ease;
}
.modal-overlay.show .modal-container {
    transform: scale(1);
}
.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #e9ecef;
    padding-bottom: 15px;
    margin-bottom: 20px;
}
.modal-header h3 {
    margin: 0;
    font-size: 1.25rem;
    color: #343a40;
}
.modal-close-btn {
    background: none;
    border: none;
    font-size: 1.6rem;
    color: #6c757d;
    cursor: pointer;
}
.modal-body .form-group {
    margin-bottom: 20px;
}
.modal-body .checkbox-group {
    display: flex;
    flex-wrap: wrap;
    gap: 15px;
}
.modal-body .checkbox-item {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.9rem;
}
.modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    margin-top: 25px;
    padding-top: 15px;
    border-top: 1px solid #e9ecef;
}

/* DARK THEME FOR MODAL */
html.dark-theme-active .modal-container {
    background: #2c2c2c;
    border: 1px solid #444;
}
html.dark-theme-active .modal-header {
    border-bottom-color: #444;
}
html.dark-theme-active .modal-header h3 {
    color: #f5f5f5;
}
html.dark-theme-active .modal-close-btn {
    color: #bdbdbd;
}
html.dark-theme-active .modal-body .checkbox-item {
    color: #e0e0e0;
}
html.dark-theme-active .modal-footer {
    border-top-color: #444;
}

/* PAGE TITLE CONTAINER */
.page-title-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    margin-bottom: 15px; /* Sesuaikan dengan .main h2.page-title margin-bottom */
}
.page-title-container .page-title {
    margin-bottom: 0; /* Hapus margin bawah karena sudah diatur oleh container */
    border-bottom: none; /* Hapus border karena sudah diatur oleh container */
}
```

#### **3. `includes/footer.php` (Dimodifikasi)**
*   Menghapus semua elemen dan script yang berhubungan dengan chatbot.

```php
<?php
// $current_page dari header.php
$no_standard_footer_pages = [
    'login.php', 'register.php', 'forgot_password.php', 'reset_password.php', 'google_auth_callback.php',
];
$is_special_page_footer = in_array($current_page, $no_standard_footer_pages);

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
            ?>
            <?php if (!empty($current_session_notifications_for_popup)): ?>
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
                    <li class="<?php echo htmlspecialchars($message_class_popup); ?>">
                        <?php echo $notif_item['message']; ?> 
                        <small class="notif-time"><?php echo htmlspecialchars($time_ago_popup); ?></small>
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

            const dateInputs = document.querySelectorAll('input[type="text"][id$="Date"], input[type="text"][id$="DueDate"], input[type="text"][id$="DateRange"], input[type="text"][id="filterHistoryDateRange"], input[type="text"][id="filterReportDateRange"], input[type="text"][id="reportPdfDateRange"]');
            dateInputs.forEach(input => {
                let config = { dateFormat: "d/m/Y", locale: "id", allowInput: true };
                if (input.id === 'taskDueDate' || input.id === 'editTaskDueDate') {
                     config.minDate = "today";
                }
                if (input.id === 'filterHistoryDateRange' || input.id === 'filterPerformanceDateRange' || input.id === 'filterReportDateRange' || input.id === 'reportPdfDateRange') { 
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
            if (themeToggleCheckbox) { 
                if (initialThemeIsDark) {
                    themeToggleCheckbox.checked = true;
                } else {
                    themeToggleCheckbox.checked = false;
                }
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
                        if (currentStatus === 'Not Started') {
                            nextStatus = 'In Progress';
                        } else if (currentStatus === 'In Progress') {
                            nextStatus = 'Completed';
                        } else {
                            return; 
                        }
                        
                        fetch('ajax_update_task_status.php', {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/x-www-form-urlencoded', },
                            body: `task_id=${taskId}&new_status=${nextStatus}`
                        })
                        .then(response => response.json())
                        .then(data => {
                            if (data.success) {
                                const statusTextElement = card.querySelector('.meta-info .task-status-text');
                                if (statusTextElement && data.new_status_text) {
                                    statusTextElement.textContent = data.new_status_text;
                                }
                                card.classList.remove('status-not-started', 'status-in-progress');
                                if(data.new_status_text === 'Dikerjakan') card.classList.add('status-in-progress');
                                else if(data.new_status_text === 'Selesai') card.classList.add('status-completed-history');
                                card.dataset.currentStatus = nextStatus;

                                if (data.is_completed) {
                                    card.style.transition = 'opacity 0.4s ease-out, transform 0.4s ease-out, max-height 0.5s ease-in-out, padding 0.5s ease-in-out, margin 0.5s ease-in-out';
                                    card.style.opacity = '0'; card.style.transform = 'scale(0.9)';
                                    card.style.maxHeight = '0px'; card.style.paddingTop = '0px';
                                    card.style.paddingBottom = '0px'; card.style.marginBottom = '0px';
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
                                alert('Gagal memperbarui status: ' + (data.message || 'Error tidak diketahui.'));
                            }
                        })
                        .catch(error => { 
                            console.error('Error AJAX:', error);
                            alert('Terjadi kesalahan koneksi saat memperbarui status.');
                        });
                    });
                }
            }

        });
    </script>
<?php else: ?>
<?php endif; ?>
</body>
</html>
<?php
if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
// Hapus state tambah tugas dari session server jika halaman direfresh dan bukan POST
if (isset($_SESSION['chatbot_add_task_step']) && $_SERVER["REQUEST_METHOD"] !== "POST") {
    unset($_SESSION['chatbot_add_task_step']);
    unset($_SESSION['chatbot_pending_task_data']);
}
// Hapus juga state konfirmasi umum jika halaman direfresh
if (isset($_SESSION['action_confirmation_pending']) && $_SERVER["REQUEST_METHOD"] !== "POST") {
    unset($_SESSION['action_confirmation_pending']);
    unset($_SESSION['pending_action_details']);
}
?>
```

#### **4. `laporan.php` (Dimodifikasi)**
*   Menambahkan tombol "Unduh Laporan" dan HTML untuk modal kustomisasi.
*   Menambahkan script untuk membuka/menutup modal, menangkap gambar chart, dan submit form ke `generate_report.php`.

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
    <div class="page-title-container">
        <h2 class="page-title">Laporan Produktivitas</h2>
        <button id="downloadReportBtn" class="btn btn-primary">
            <i class="fas fa-download"></i> Unduh Laporan
        </button>
    </div>

    <section class="widget report-chart-widget">
        <h3>Performa Pengerjaan Tugas</h3>
        <form method="GET" action="laporan.php" class="performance-filter-form report-performance-filter">
            <label>Rentang:</label>
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
        <h3>Detail Tugas Berdasarkan Tanggal Deadline</h3>
        <div class="report-interactive-area">
            <div class="report-task-list-container">
                <p id="selectedDateText" class="selected-date-indicator">Pilih tanggal di kalender untuk melihat detail tugas.</p>
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

<!-- MODAL UNTUK KUSTOMISASI LAPORAN PDF -->
<div id="reportModal" class="modal-overlay">
    <div class="modal-container">
        <div class="modal-header">
            <h3>Kustomisasi Laporan PDF</h3>
            <button id="closeReportModalBtn" class="modal-close-btn">&times;</button>
        </div>
        <form id="reportPdfForm" action="generate_report.php" method="POST" target="_blank">
            <div class="modal-body">
                <div class="form-group">
                    <label for="reportPdfDateRange">Rentang Tanggal Laporan</label>
                    <input type="text" id="reportPdfDateRange" name="report_date_range" placeholder="Pilih rentang tanggal..." required>
                </div>
                <div class="form-group">
                    <label>Sertakan Status Tugas:</label>
                    <div class="checkbox-group">
                        <div class="checkbox-item">
                            <input type="checkbox" id="status_all" name="status_all" checked>
                            <label for="status_all">Pilih Semua</label>
                        </div>
                        <div class="checkbox-item">
                            <input type="checkbox" class="status-filter" id="status_completed" name="statuses[]" value="Completed" checked>
                            <label for="status_completed">Selesai</label>
                        </div>
                        <div class="checkbox-item">
                            <input type="checkbox" class="status-filter" id="status_in_progress" name="statuses[]" value="In Progress" checked>
                            <label for="status_in_progress">Dikerjakan</label>
                        </div>
                        <div class="checkbox-item">
                            <input type="checkbox" class="status-filter" id="status_not_started" name="statuses[]" value="Not Started" checked>
                            <label for="status_not_started">Belum Mulai</label>
                        </div>
                         <div class="checkbox-item">
                            <input type="checkbox" class="status-filter" id="status_overdue" name="statuses[]" value="Overdue" checked>
                            <label for="status_overdue">Terlewat</label>
                        </div>
                    </div>
                </div>
                <!-- Hidden input untuk mengirim gambar chart -->
                <input type="hidden" name="chart_image_base64" id="chartImageBase64">
            </div>
            <div class="modal-footer">
                <button type="button" id="cancelReportBtn" class="btn btn-secondary">Batal</button>
                <button type="submit" id="generateReportPdfBtn" class="btn btn-primary">Buat & Unduh PDF</button>
            </div>
        </form>
    </div>
</div>


<script>
document.addEventListener('DOMContentLoaded', () => {
    let reportChartInstance; // Variabel global untuk instance Chart

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
            reportChartInstance = new Chart(barCtx, {
                type: 'bar',
                data: reportBarData,
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    animation: false, // Penting: nonaktifkan animasi untuk tangkapan gambar yg konsisten
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                stepSize: 1,
                                precision: 0, 
                                callback: function(value) {if (Number.isInteger(value)) {return value;}}
                            }
                        }
                    },
                    plugins: {
                        legend: { display: false },
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
    // ... Sisa script chart (filter preset, dll) ...
    const reportPresetButtons = document.querySelectorAll('.report-performance-filter .filter-preset-buttons .btn');
    const reportDateRangeInput = document.getElementById('filterReportDateRange');
    reportPresetButtons.forEach(button => {
        button.addEventListener('click', function(e) {
            if (reportDateRangeInput) reportDateRangeInput.value = ''; 
        });
    });
     if (reportDateRangeInput) {
        reportDateRangeInput.addEventListener('input', function() { 
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
    let taskDatesFromServer = { active: [], completed: [], overdue: [] };

    function fetchTaskDatesAndRenderCalendar(year, month) {
        fetch('ajax_get_task_dates_for_calendar.php')
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    taskDatesFromServer = data.dates;
                } else {
                    console.error('Failed to fetch task dates for calendar markers.');
                }
                renderCalendar(year, month);
            })
            .catch(error => {
                console.error('Error fetching task dates:', error);
                renderCalendar(year, month);
            });
    }


    function renderCalendar(year, month) {
        currentYear = year;
        currentMonth = month;
        calendarContainer.innerHTML = ''; 

        const monthNames = ["Januari", "Februari", "Maret", "April", "Mei", "Juni", "Juli", "Agustus", "September", "Oktober", "November", "Desember"];
        const dayNames = ["Min", "Sen", "Sel", "Rab", "Kam", "Jum", "Sab"];

        const header = document.createElement('div');
        header.classList.add('calendar-header-report');
        header.innerHTML = `
            <button id="prevMonthBtn"><</button>
            <span>${monthNames[month]} ${year}</span>
            <button id="nextMonthBtn">></button>
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
            
            const dayNumberSpan = document.createElement('span');
            dayNumberSpan.classList.add('day-number');
            dayNumberSpan.textContent = day;
            dayCell.appendChild(dayNumberSpan);

            const currentDateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            dayCell.dataset.date = currentDateStr;
            
            const today = new Date();
            if (year === today.getFullYear() && month === today.getMonth() && day === today.getDate()) {
                dayCell.classList.add('today');
            }

            const markersContainer = document.createElement('div');
            markersContainer.classList.add('calendar-markers-container');

            if (taskDatesFromServer.active.includes(currentDateStr)) {
                const marker = document.createElement('span');
                marker.classList.add('calendar-marker', 'marker-active');
                marker.title = 'Ada tugas aktif';
                markersContainer.appendChild(marker);
            }
            if (taskDatesFromServer.completed.includes(currentDateStr)) {
                const marker = document.createElement('span');
                marker.classList.add('calendar-marker', 'marker-completed');
                marker.title = 'Ada tugas selesai';
                markersContainer.appendChild(marker);
            }
            if (taskDatesFromServer.overdue.includes(currentDateStr)) {
                const marker = document.createElement('span');
                marker.classList.add('calendar-marker', 'marker-overdue');
                marker.title = 'Ada tugas terlewat';
                markersContainer.appendChild(marker);
            }
            if (markersContainer.hasChildNodes()) {
                dayCell.appendChild(markersContainer);
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
            if (month < 0) { month = 11; year--; }
            fetchTaskDatesAndRenderCalendar(year, month);
        });

        document.getElementById('nextMonthBtn').addEventListener('click', () => {
            month++;
            if (month > 11) { month = 0; year++; }
            fetchTaskDatesAndRenderCalendar(year, month);
        });
    }

    function loadTasksForDate(dateStr) {
        const dateObj = new Date(dateStr);
        dateObj.setHours(0,0,0,0);
        const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', timeZone: 'UTC' };
        selectedDateText.textContent = `Detail Tugas untuk Deadline: ${dateObj.toLocaleDateString('id-ID', options)}`;
        tasksListContainer.innerHTML = '<p class="loading-message">Memuat tugas...</p>';

        fetch(`ajax_get_tasks_for_date.php?date=${dateStr}`)
            .then(response => {
                if (!response.ok) { throw new Error('Network response was not ok: ' + response.statusText); }
                return response.json();
            })
            .then(data => {
                tasksListContainer.innerHTML = '';
                if (data.success && data.tasks.length > 0) {
                    data.tasks.forEach(taskHtml => {
                        tasksListContainer.insertAdjacentHTML('beforeend', taskHtml);
                    });
                } else if (data.success && data.tasks.length === 0) {
                    tasksListContainer.innerHTML = '<p class="no-tasks-message">Tidak ada tugas dengan deadline pada tanggal ini.</p>';
                } else {
                    tasksListContainer.innerHTML = `<p class="no-tasks-message">Gagal memuat tugas: ${data.message || 'Error tidak diketahui'}</p>`;
                }
            })
            .catch(error => {
                console.error('Error fetching tasks:', error);
                tasksListContainer.innerHTML = `<p class="no-tasks-message">Terjadi kesalahan saat memuat tugas. (${error.message})</p>`;
            });
    }
    const todayInitial = new Date();
    fetchTaskDatesAndRenderCalendar(todayInitial.getFullYear(), todayInitial.getMonth());

    // --- Script untuk Modal Laporan PDF ---
    const downloadBtn = document.getElementById('downloadReportBtn');
    const reportModal = document.getElementById('reportModal');
    const closeReportModalBtn = document.getElementById('closeReportModalBtn');
    const cancelReportBtn = document.getElementById('cancelReportBtn');
    const reportPdfForm = document.getElementById('reportPdfForm');
    const chartImageInput = document.getElementById('chartImageBase64');
    const selectAllStatusCheckbox = document.getElementById('status_all');
    const statusCheckboxes = document.querySelectorAll('.status-filter');

    const openModal = () => reportModal.classList.add('show');
    const closeModal = () => reportModal.classList.remove('show');

    if (downloadBtn) downloadBtn.addEventListener('click', openModal);
    if (closeReportModalBtn) closeReportModalBtn.addEventListener('click', closeModal);
    if (cancelReportBtn) cancelReportBtn.addEventListener('click', closeModal);
    
    reportModal.addEventListener('click', (e) => {
        if (e.target === reportModal) closeModal();
    });

    if (selectAllStatusCheckbox) {
        selectAllStatusCheckbox.addEventListener('change', () => {
            statusCheckboxes.forEach(cb => {
                cb.checked = selectAllStatusCheckbox.checked;
            });
        });
    }
    
    statusCheckboxes.forEach(cb => {
        cb.addEventListener('change', () => {
            if (!cb.checked) {
                selectAllStatusCheckbox.checked = false;
            } else {
                const allChecked = Array.from(statusCheckboxes).every(i => i.checked);
                selectAllStatusCheckbox.checked = allChecked;
            }
        });
    });

    if (reportPdfForm) {
        reportPdfForm.addEventListener('submit', (e) => {
            // Tangkap gambar chart sebelum submit
            if (reportChartInstance) {
                chartImageInput.value = reportChartInstance.toBase64Image();
            } else {
                chartImageInput.value = ''; // Kosongkan jika chart tidak ada
            }
            // Validasi: pastikan setidaknya satu status dipilih
            const anyStatusChecked = Array.from(statusCheckboxes).some(i => i.checked);
            if (!anyStatusChecked) {
                e.preventDefault();
                alert('Pilih setidaknya satu status tugas untuk disertakan dalam laporan.');
                return;
            }

            // Setelah submit, modal akan tetap terbuka. Anda bisa menutupnya di sini jika mau.
            setTimeout(() => {
                closeModal();
            }, 500); // Beri jeda sedikit agar form bisa submit
        });
    }

});
</script>
<?php require_once 'includes/footer.php'; ?>
```

#### **5. `generate_report.php` (File Baru)**
*   Buat file baru ini di direktori root proyek Anda.
*   File ini akan menerima data dari modal, mengambil data dari DB, dan membuat PDF menggunakan TCPDF.

```php
<?php
// File: generate_report.php

require_once 'vendor/autoload.php'; // Autoload dari Composer (untuk TCPDF)
require_once 'includes/db.php'; // Koneksi DB dan session_start

if (!isset($_SESSION['user_id'])) {
    die("Akses ditolak. Silakan login terlebih dahulu.");
}

$user_id = $_SESSION['user_id'];
$username = $_SESSION['username'] ?? 'Pengguna';

// --- Ambil dan Validasi Input dari Form ---
$date_range_str = $_POST['report_date_range'] ?? '';
$statuses = $_POST['statuses'] ?? [];
$chart_image_base64 = $_POST['chart_image_base64'] ?? '';

// Validasi Rentang Tanggal
$start_date_mysql = null;
$end_date_mysql = null;
if (!empty($date_range_str)) {
    $dates = explode(' - ', $date_range_str);
    if (count($dates) >= 1) {
        $date_parts_start = explode('/', trim($dates[0]));
        if (count($date_parts_start) == 3 && checkdate($date_parts_start[1], $date_parts_start[0], $date_parts_start[2])) {
            $start_date_mysql = $date_parts_start[2] . '-' . $date_parts_start[1] . '-' . $date_parts_start[0];
        }
    }
    if (count($dates) == 2) {
        $date_parts_end = explode('/', trim($dates[1]));
        if (count($date_parts_end) == 3 && checkdate($date_parts_end[1], $date_parts_end[0], $date_parts_end[2])) {
            $end_date_mysql = $date_parts_end[2] . '-' . $date_parts_end[1] . '-' . $date_parts_end[0];
        }
    } else {
        $end_date_mysql = $start_date_mysql;
    }
}
if (!$start_date_mysql || !$end_date_mysql) {
    die("Rentang tanggal tidak valid.");
}
if (strtotime($start_date_mysql) > strtotime($end_date_mysql)) {
    list($start_date_mysql, $end_date_mysql) = [$end_date_mysql, $start_date_mysql];
}


// --- Query Database Berdasarkan Filter ---
$sql_conditions_array = ["t.user_id = ?"];
$params = [$user_id];
$types = "i";

$status_conditions = [];
$has_overdue_filter = false;
$valid_statuses = ['Completed', 'In Progress', 'Not Started'];

foreach ($statuses as $status) {
    if (in_array($status, $valid_statuses)) {
        $status_conditions[] = "t.status = ?";
        $params[] = $status;
        $types .= "s";
    }
    if ($status === 'Overdue') {
        $has_overdue_filter = true;
    }
}

$status_sql_part = "";
if (!empty($status_conditions)) {
    $status_sql_part = "(" . implode(" OR ", $status_conditions) . ")";
}

if ($has_overdue_filter) {
    $overdue_sql_part = "(t.due_date < CURDATE() AND t.status != 'Completed')";
    if (!empty($status_sql_part)) {
        $sql_conditions_array[] = "($status_sql_part OR $overdue_sql_part)";
    } else {
        $sql_conditions_array[] = $overdue_sql_part;
    }
} elseif (!empty($status_sql_part)) {
    $sql_conditions_array[] = $status_sql_part;
} else {
    die("Tidak ada status yang dipilih untuk laporan.");
}

// Filter berdasarkan rentang tanggal pada due_date
$sql_conditions_array[] = "t.due_date BETWEEN ? AND ?";
$params[] = $start_date_mysql;
$params[] = $end_date_mysql;
$types .= "ss";

$sql_conditions_string = implode(" AND ", $sql_conditions_array);
$sql = "SELECT t.title, t.priority, t.status, DATE_FORMAT(t.due_date, '%d %b %Y') as due_date_formatted, 
               (CASE WHEN t.status = 'Completed' THEN DATE_FORMAT(t.updated_at, '%d %b %Y') ELSE '-' END) as completed_date_formatted,
               (CASE WHEN t.due_date < CURDATE() AND t.status != 'Completed' THEN 'Ya' ELSE 'Tidak' END) as is_overdue
        FROM tasks t
        WHERE $sql_conditions_string
        ORDER BY t.due_date ASC, t.id ASC";

$stmt_tasks = $conn->prepare($sql);
$report_tasks = [];
if ($stmt_tasks) {
    $stmt_tasks->bind_param($types, ...$params);
    $stmt_tasks->execute();
    $result_tasks = $stmt_tasks->get_result();
    while ($row = $result_tasks->fetch_assoc()) {
        // Mapping status untuk tampilan di PDF
        if ($row['is_overdue'] === 'Ya') {
            $row['status_display'] = 'Terlewat';
        } else {
            switch ($row['status']) {
                case 'Completed': $row['status_display'] = 'Selesai'; break;
                case 'In Progress': $row['status_display'] = 'Dikerjakan'; break;
                case 'Not Started': $row['status_display'] = 'Belum Mulai'; break;
                default: $row['status_display'] = $row['status'];
            }
        }
        $report_tasks[] = $row;
    }
    $stmt_tasks->close();
} else {
    die("Gagal mempersiapkan query data laporan: " . $conn->error);
}

// ============================================================
// PEMBUATAN PDF DENGAN TCPDF
// ============================================================

// Extend class TCPDF untuk membuat Header dan Footer kustom
class MYPDF extends TCPDF {
    private $username;

    public function setUsername($username) {
        $this->username = $username;
    }

    //Page header
    public function Header() {
        // Logo
        $image_file = K_PATH_IMAGES.'logo.png'; // Pastikan logo.png ada di folder images/
        // Parameter: file, x, y, width, height, type, link, align, resize, dpi, palign, ismask, imgmask, border, fitbox, hidden, fitonpage
        $this->Image($image_file, 10, 10, 25, 0, 'PNG', '', 'T', false, 300, '', false, false, 0, false, false, false);
        
        // Set font
        $this->SetFont('helvetica', 'B', 14);
        // Judul
        $this->Cell(0, 15, 'Laporan Produktivitas Tugas', 0, false, 'C', 0, '', 0, false, 'M', 'M');
        
        // Informasi Pengguna
        $this->SetFont('helvetica', '', 9);
        $this->SetXY(150, 10); // Posisi kanan atas
        $this->Cell(0, 10, 'Pengguna: ' . $this->username, 0, false, 'R');
        $this.SetXY(150, 15);
        $this->Cell(0, 10, 'Tanggal Cetak: ' . date('d F Y'), 0, false, 'R');

        // Garis bawah header
        $this->Line(10, 25, $this->getPageWidth() - 10, 25);
    }

    // Page footer
    public function Footer() {
        // Position at 15 mm from bottom
        $this->SetY(-15);
        // Set font
        $this->SetFont('helvetica', 'I', 8);
        // Page number
        $this->Cell(0, 10, 'Halaman ' . $this->getAliasNumPage() . ' dari ' . $this->getAliasNbPages(), 0, false, 'C', 0, '', 0, false, 'T', 'M');
    }
}

// Buat dokumen PDF baru
$pdf = new MYPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);
$pdf->setUsername($username);

// Set informasi dokumen
$pdf->SetCreator(PDF_CREATOR);
$pdf->SetAuthor('List In App');
$pdf->SetTitle('Laporan Produktivitas Tugas - ' . $username);
$pdf->SetSubject('Laporan Produktivitas Tugas');

// Set header dan footer
$pdf->setPrintHeader(true);
$pdf->setPrintFooter(true);

// Set margin
$pdf->SetMargins(PDF_MARGIN_LEFT, 28, PDF_MARGIN_RIGHT); // Margin atas lebih besar untuk header
$pdf->SetHeaderMargin(PDF_MARGIN_HEADER);
$pdf->SetFooterMargin(PDF_MARGIN_FOOTER);

// Set auto page breaks
$pdf->SetAutoPageBreak(TRUE, PDF_MARGIN_BOTTOM);

// Tambah halaman
$pdf->AddPage();

// --- KONTEN PDF ---

// Info Rentang Laporan
$pdf->SetFont('helvetica', '', 10);
$pdf->Ln(5); // Spasi setelah header
$pdf->Cell(0, 10, 'Rentang Laporan: ' . date('d M Y', strtotime($start_date_mysql)) . ' - ' . date('d M Y', strtotime($end_date_mysql)), 0, 1, 'L');
$pdf->Ln(5);

// Diagram Performa
$pdf->SetFont('helvetica', 'B', 12);
$pdf->Cell(0, 10, 'Diagram Performa Pengerjaan', 0, 1, 'L');
$pdf->Ln(2);

if (!empty($chart_image_base64)) {
    // @ suppression operator digunakan untuk mencegah warning jika base64 tidak valid
    $imgdata = base64_decode(preg_replace('#^data:image/\w+;base64,#i', '', $chart_image_base64));
    // Parameter: file, x, y, width, height...
    // Menggunakan '' sebagai x dan y agar posisi otomatis. Lebar 180mm.
    $pdf->Image('@'.$imgdata, '', '', 180, 0, 'PNG', '', '', true, 150, '', false, false, 0, false, false, false);
} else {
    $pdf->SetFont('helvetica', 'I', 9);
    $pdf->Cell(0, 10, '[Diagram tidak tersedia]', 0, 1, 'L');
}
$pdf->Ln(8);

// Tabel Detail Tugas
$pdf->SetFont('helvetica', 'B', 12);
$pdf->Cell(0, 10, 'Detail Daftar Tugas', 0, 1, 'L');
$pdf->Ln(2);

// Buat HTML untuk tabel
$html = '<table border="1" cellpadding="5" cellspacing="0" style="width:100%;">
    <thead>
        <tr style="background-color:#EAEAEA; font-weight:bold; text-align:center;">
            <th width="8%">No.</th>
            <th width="37%">Judul Tugas</th>
            <th width="15%">Prioritas</th>
            <th width="15%">Status</th>
            <th width="12%">Deadline</th>
            <th width="13%">Selesai</th>
        </tr>
    </thead>
    <tbody>';

if (empty($report_tasks)) {
    $html .= '<tr><td colspan="6" style="text-align:center; font-style:italic;">Tidak ada data tugas untuk ditampilkan.</td></tr>';
} else {
    $no = 1;
    foreach ($report_tasks as $task) {
        $html .= '<tr>
            <td style="text-align:center;">' . $no++ . '</td>
            <td>' . htmlspecialchars($task['title']) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($task['priority']) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($task['status_display']) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($task['due_date_formatted']) . '</td>
            <td style="text-align:center;">' . htmlspecialchars($task['completed_date_formatted']) . '</td>
        </tr>';
    }
}

$html .= '</tbody></table>';

$pdf->SetFont('helvetica', '', 9);
$pdf->writeHTML($html, true, false, true, false, '');

// Tutup dan output dokumen PDF
$pdf->Output('Laporan_ListIn_' . date('Ymd') . '.pdf', 'I'); // 'I' untuk inline (tampil di browser), 'D' untuk download

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}

```

### **Penjelasan dan Cara Kerja**

1.  **Penghapusan Chatbot**:
    *   File `ajax_chatbot_handler.php` yang menjadi inti logika chatbot telah dihapus.
    *   Konfigurasi `CHATBOT_API_KEY` di `config.php` juga dihapus karena tidak lagi diperlukan.
    *   Semua elemen HTML (`#chatbot-icon`, `#chatbot-container`, dll.) dan script terkait di `footer.php` telah dihilangkan.
    *   Blok CSS untuk chatbot di `style.css` juga sudah bersih.

2.  **Penambahan Fitur Laporan PDF**:
    *   **Tombol & Modal (`laporan.php` & `css/style.css`)**:
        *   Sebuah tombol "Unduh Laporan" ditambahkan di `laporan.php` di sebelah judul halaman.
        *   HTML untuk modal (jendela pop-up) ditambahkan ke `laporan.php`. Modal ini tersembunyi secara default.
        *   CSS baru di `style.css` (`.modal-overlay`, `.modal-container`, dll.) digunakan untuk menata tampilan modal, termasuk tema gelap.
        *   JavaScript di `laporan.php` menangani logika untuk menampilkan dan menyembunyikan modal saat tombol diklik.

    *   **Kustomisasi Laporan (Modal)**:
        *   Di dalam modal, terdapat form (`<form id="reportPdfForm">`) yang berisi:
            *   Input rentang tanggal (`reportPdfDateRange`) menggunakan Flatpickr.
            *   Sekelompok *checkbox* untuk memilih status tugas mana yang ingin dimasukkan ke dalam laporan. Ada juga tombol "Pilih Semua".
            *   Sebuah *hidden input* (`<input type="hidden" id="chartImageBase64">`).

    *   **Pengambilan Gambar Chart (JavaScript di `laporan.php`)**:
        *   Saat pengguna menekan tombol "Buat & Unduh PDF", script JavaScript akan mengambil alih sejenak sebelum form disubmit.
        *   Ia menggunakan metode `toBase64Image()` dari `Chart.js` untuk mengubah diagram batang di kanvas menjadi gambar berformat Base64 (teks panjang).
        *   Teks Base64 ini dimasukkan ke dalam *hidden input* `chartImageBase64`.

    *   **Proses Pembuatan PDF (`generate_report.php`)**:
        *   Form dari modal akan mengirimkan semua datanya (rentang tanggal, status, dan gambar chart) ke `generate_report.php`.
        *   File ini pertama-tama memuat `autoload.php` dari Composer untuk bisa menggunakan kelas `TCPDF`.
        *   Ia memvalidasi semua input yang diterima.
        *   Kemudian, ia membuat query SQL dinamis berdasarkan filter yang dipilih pengguna untuk mengambil data tugas yang relevan dari database.
        *   Setelah data didapat, ia membuat objek `MYPDF` (kelas kustom yang meng-extend `TCPDF`). Kelas ini mendefinisikan `Header()` (untuk logo, judul, info pengguna) dan `Footer()` (untuk nomor halaman) kustom.
        *   PDF diisi dengan konten:
            1.  Informasi rentang laporan.
            2.  Gambar diagram performa yang diterima dari form (Base64 di-decode dan dirender sebagai gambar).
            3.  Sebuah tabel HTML yang berisi data tugas yang telah difilter. `TCPDF` akan merender HTML ini menjadi tabel di dalam PDF.
        *   Terakhir, `$pdf->Output()` mengirimkan file PDF yang sudah jadi ke browser pengguna, yang akan memicu proses unduh atau menampilkannya langsung.

Dengan mengikuti langkah-langkah ini, aplikasi Anda akan memiliki fitur unduh laporan yang fungsional dan terintegrasi dengan baik, serta bersih dari semua kode chatbot sebelumnya.
