Baik, ini adalah permintaan pengembangan yang cukup komprehensif. Saya akan mencoba memberikan solusi selengkap mungkin. Mengingat kompleksitasnya, terutama untuk fitur unduh laporan PDF dan integrasi AI Chatbot yang lebih detail, Anda mungkin perlu melakukan beberapa penyesuaian dan debugging.

**Struktur Perubahan:**

1.  **Halaman Laporan (`laporan.php`):**
    *   Modifikasi AJAX untuk mengambil semua tugas pada tanggal terpilih (aktif, selesai, terlewat).
    *   Modifikasi `render_task_card` untuk menangani styling card yang berbeda di halaman laporan.
    *   AJAX baru untuk mendapatkan tanggal-tanggal yang memiliki tugas (untuk pewarnaan kalender).
    *   JavaScript untuk mewarnai tanggal di kalender.
2.  **Fitur Unduh Laporan PDF:**
    *   Tombol unduh di `laporan.php`.
    *   Modal (popup) untuk kustomisasi laporan.
    *   File PHP baru (`generate_report_pdf.php`) untuk membuat PDF menggunakan library seperti FPDF atau TCPDF (saya akan menggunakan FPDF karena lebih ringan untuk contoh dasar).
3.  **AI Chatbot (Google AI Studio / Gemini API):**
    *   Penjelasan dan contoh kode untuk `ajax_chatbot_handler.php`.
4.  **CSS:** Penambahan style untuk kalender di laporan, modal unduh laporan.
5.  **File yang Dimodifikasi:** `laporan.php`, `includes/task_helper.php`, `ajax_get_tasks_for_date.php` (berganti nama menjadi `ajax_get_report_data_for_date.php`), `css/style.css`.
6.  **File Baru:** `ajax_get_task_dates_for_calendar.php`, `generate_report_pdf.php`.

**PENTING: LIBRARY PDF**
Untuk membuat PDF, kita akan menggunakan FPDF. Jika Anda belum memilikinya, Anda bisa mengunduhnya dari [http://www.fpdf.org/](http://www.fpdf.org/) dan letakkan folder `fpdf` di dalam direktori `includes` atau direktori library lain yang Anda inginkan. Atau, jika Anda menggunakan Composer, Anda bisa mencari paket FPDF. Untuk contoh ini, saya akan berasumsi Anda meletakkan FPDF secara manual.

---

**LANGKAH 1: Modifikasi `includes/task_helper.php`**

Kita perlu menyesuaikan `render_task_card` agar bisa menangani styling yang berbeda di halaman laporan.

**includes/task_helper.php (Dimodifikasi)**
```php
<?php
// File: includes/task_helper.php

if (!function_exists('render_task_card')) {
    /**
     * Merender kartu tugas HTML.
     *
     * @param array $task Data tugas (harus memiliki 'id', 'title', 'description', 'priority', 'status', 'due_date_formatted', 'due_date' (YYYY-MM-DD)).
     * @param string $page_type Konteks halaman ('dashboardTodo', 'management', 'history', 'reportActive', 'reportCompleted', 'reportOverdue').
     * @param string $current_page_for_redirect_url URL lengkap halaman saat ini untuk parameter 'from'.
     * @param string $history_card_type Khusus untuk $page_type = 'history'.
     * @return string HTML string dari kartu tugas.
     */
    function render_task_card($task, $page_type = 'dashboardTodo', $current_page_for_redirect_url = 'dashboard.php', $history_card_type = '') {

        $status_options_display = [
            'Not Started' => 'Belum Mulai',
            'In Progress' => 'Dikerjakan',
            'Completed' => 'Selesai'
        ];
        $current_status_key = $task['status'] ?? 'Not Started';
        $current_status_text = $status_options_display[$current_status_key] ?? $current_status_key;

        $card_status_class = '';
        $is_overdue = false;
        if (isset($task['due_date']) && $task['status'] !== 'Completed' && strtotime($task['due_date']) < strtotime(date('Y-m-d'))) {
            $is_overdue = true;
        }

        // Penentuan class berdasarkan page_type dan status
        switch ($page_type) {
            case 'management':
            case 'dashboardTodo': // Tugas aktif di dashboard
            case 'reportActive':  // Tugas aktif di laporan
                if ($is_overdue) {
                    $card_status_class = 'status-overdue-uncompleted-history'; // Sama seperti di riwayat untuk styling
                } elseif ($current_status_key == 'Not Started') {
                    $card_status_class = 'status-not-started';
                } elseif ($current_status_key == 'In Progress') {
                    $card_status_class = 'status-in-progress';
                }
                break;
            case 'reportCompleted': // Tugas selesai di laporan
            case 'history': // Juga mencakup tugas selesai di riwayat
                if ($history_card_type === 'completed' || $page_type === 'reportCompleted') {
                    $card_status_class = 'status-completed-history';
                } elseif ($history_card_type === 'overdue_uncompleted' || ($is_overdue && $page_type === 'history')) {
                     // Khusus untuk 'history' jika tipenya overdue_uncompleted
                    $card_status_class = 'status-overdue-uncompleted-history';
                }
                break;
            case 'reportOverdue': // Tugas terlewat di laporan (yang belum selesai)
                $card_status_class = 'status-overdue-uncompleted-history';
                break;
            default:
                // Fallback atau class default jika diperlukan
                if ($current_status_key == 'Not Started') $card_status_class = 'status-not-started';
                elseif ($current_status_key == 'In Progress') $card_status_class = 'status-in-progress';
                break;
        }


        $task_title_html = '<strong>' . htmlspecialchars($task['title'] ?? 'Tanpa Judul') . '</strong>';
        if ($page_type === 'management' || $page_type === 'dashboardTodo' || $page_type === 'reportActive') {
             // Tambahkan link ke edit_tugas jika tugas masih aktif dan bukan dari history
             $edit_action_url = 'edit_tugas.php?id=' . ($task['id'] ?? 0);
             $edit_action_url .= '&from=' . urlencode($current_page_for_redirect_url); // Untuk kembali ke halaman yang benar
             $task_title_html = '<a href="' . htmlspecialchars($edit_action_url) . '" class="task-title-link">' . $task_title_html . '</a>';
        }


        ob_start();
        ?>
        <div class="task-item-card <?php echo htmlspecialchars($card_status_class); ?>" 
             data-task-id="<?php echo htmlspecialchars($task['id'] ?? 0); ?>" 
             data-current-status="<?php echo htmlspecialchars($current_status_key); ?>">
            <div class="task-details">
                <?php echo $task_title_html; // Sekarang bisa berupa link atau teks biasa ?>
                <p class="description"><?php echo nl2br(htmlspecialchars(($task['description'] ?? '') ?: 'Tidak ada deskripsi.')); ?></p>
                <p class="meta-info">
                    Prioritas: <span class="priority-<?php echo strtolower(htmlspecialchars($task['priority'] ?? 'Medium')); ?>"><?php echo htmlspecialchars($task['priority'] ?? 'Medium'); ?></span> | 
                    Status: <span class="task-status-text"><?php echo htmlspecialchars($current_status_text); ?></span> | 
                    Deadline: <?php echo htmlspecialchars(($task['due_date_formatted'] ?? '') ?: 'N/A'); ?>
                    <?php if ($is_overdue && $page_type !== 'reportOverdue' && $page_type !== 'history'): ?>
                        <span class="overdue-indicator">(Terlewat)</span>
                    <?php endif; ?>
                </p>
            </div>

            <div class="task-actions">
                <?php if ($page_type === 'management'): ?>
                    <?php
                    // URL edit sudah dihandle di $task_title_html
                    $delete_action_url = $current_page_for_redirect_url; 
                    $query_separator_delete = (strpos($delete_action_url, '?') === false) ? '?' : '&';
                    $delete_action_url .= $query_separator_delete . 'action=delete&task_id=' . ($task['id'] ?? 0);
                    ?>
                    <!-- Tombol edit bisa dihilangkan dari sini jika judul sudah link -->
                    <!-- <a href="<?php //echo htmlspecialchars($edit_action_url_management); ?>" class="edit-btn" title="Edit Tugas"><i class="fas fa-edit"></i></a> -->
                    <a href="<?php echo htmlspecialchars($delete_action_url); ?>" class="delete-btn" title="Hapus Tugas"><i class="fas fa-trash"></i></a>
                
                <?php elseif ($page_type === 'history'): ?>
                    <?php 
                    $action_handler_page = 'manajemen_tugas.php'; 
                    $reopen_action_url = $action_handler_page . "?task_id=" . ($task['id'] ?? 0) . "&action=reopen&from=" . urlencode($current_page_for_redirect_url);
                    $delete_history_action_url = $action_handler_page . "?task_id=" . ($task['id'] ?? 0) . "&action=delete&from=" . urlencode($current_page_for_redirect_url);
                    ?>
                    <a href="<?php echo htmlspecialchars($reopen_action_url); ?>" class="reopen-btn" title="Buka Kembali Tugas"><i class="fas fa-undo"></i></a>
                    <a href="<?php echo htmlspecialchars($delete_history_action_url); ?>" class="delete-btn" title="Hapus Permanen"><i class="fas fa-trash"></i></a>
                
                <?php elseif (in_array($page_type, ['reportActive', 'reportCompleted', 'reportOverdue'])): ?>
                    <?php // Tidak ada aksi default untuk laporan, tapi bisa ditambahkan jika perlu ?>
                    <!-- Contoh: Jika ingin tombol edit di tugas aktif laporan -->
                    <?php if ($page_type === 'reportActive' && !$is_overdue && $current_status_key !== 'Completed'):
                        $edit_report_action_url = 'edit_tugas.php?id=' . ($task['id'] ?? 0) . '&from=' . urlencode($current_page_for_redirect_url . '?selected_date=' . ($task['due_date'] ?? ''));
                    ?>
                        <!-- <a href="<?php //echo htmlspecialchars($edit_report_action_url); ?>" class="edit-btn" title="Edit Tugas"><i class="fas fa-edit"></i></a> -->
                    <?php endif; ?>
                <?php endif; ?>
            </div>
        </div>
        <?php
        return ob_get_clean();
    }
}
?>
```
**Perubahan Utama:**
*   Parameter `$page_type` sekarang bisa menerima `'reportActive'`, `'reportCompleted'`, `'reportOverdue'`.
*   Logika penentuan `$card_status_class` disesuaikan untuk halaman laporan agar tugas yang terlewat atau selesai memiliki styling yang sama dengan di halaman riwayat.
*   Judul tugas di halaman `management`, `dashboardTodo`, dan `reportActive` sekarang menjadi link ke halaman edit tugas.
*   Indikator `(Terlewat)` ditambahkan jika tugas terlewat dan bukan di halaman riwayat atau `reportOverdue`.

---

**LANGKAH 2: AJAX Baru untuk Kalender Laporan**

Buat file baru `ajax_get_task_dates_for_calendar.php` di root proyek.

**ajax_get_task_dates_for_calendar.php (Baru)**
```php
<?php
// File: ajax_get_task_dates_for_calendar.php
require_once 'includes/db.php';

header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'dates' => []]);
    exit();
}
$user_id = $_SESSION['user_id'];

$task_dates = [
    'active' => [],    // Tugas yang masih aktif (Not Started, In Progress) dan belum lewat deadline
    'completed' => [], // Tugas yang sudah Completed
    'overdue' => []    // Tugas yang sudah lewat deadline dan belum Completed
];

// Ambil tanggal untuk tugas aktif (belum selesai dan belum overdue)
$sql_active = "SELECT DISTINCT DATE_FORMAT(due_date, '%Y-%m-%d') as task_date 
               FROM tasks 
               WHERE user_id = ? AND status != 'Completed' AND due_date >= CURDATE()";
$stmt_active = $conn->prepare($sql_active);
if ($stmt_active) {
    $stmt_active->bind_param("i", $user_id);
    $stmt_active->execute();
    $result_active = $stmt_active->get_result();
    while ($row = $result_active->fetch_assoc()) {
        $task_dates['active'][] = $row['task_date'];
    }
    $stmt_active->close();
}

// Ambil tanggal untuk tugas selesai (berdasarkan updated_at atau due_date jika lebih relevan)
// Untuk kesederhanaan, kita gunakan due_date untuk pewarnaan kalender
$sql_completed = "SELECT DISTINCT DATE_FORMAT(due_date, '%Y-%m-%d') as task_date 
                  FROM tasks 
                  WHERE user_id = ? AND status = 'Completed'";
$stmt_completed = $conn->prepare($sql_completed);
if ($stmt_completed) {
    $stmt_completed->bind_param("i", $user_id);
    $stmt_completed->execute();
    $result_completed = $stmt_completed->get_result();
    while ($row = $result_completed->fetch_assoc()) {
        $task_dates['completed'][] = $row['task_date'];
    }
    $stmt_completed->close();
}

// Ambil tanggal untuk tugas overdue (belum selesai dan sudah lewat deadline)
$sql_overdue = "SELECT DISTINCT DATE_FORMAT(due_date, '%Y-%m-%d') as task_date 
                FROM tasks 
                WHERE user_id = ? AND status != 'Completed' AND due_date < CURDATE()";
$stmt_overdue = $conn->prepare($sql_overdue);
if ($stmt_overdue) {
    $stmt_overdue->bind_param("i", $user_id);
    $stmt_overdue->execute();
    $result_overdue = $stmt_overdue->get_result();
    while ($row = $result_overdue->fetch_assoc()) {
        $task_dates['overdue'][] = $row['task_date'];
    }
    $stmt_overdue->close();
}

// Hapus duplikasi jika sebuah tanggal masuk beberapa kategori (misal, ada tugas aktif dan selesai di tanggal yang sama)
// Prioritas pewarnaan bisa diatur di JavaScript client-side
$task_dates['active'] = array_unique($task_dates['active']);
$task_dates['completed'] = array_unique($task_dates['completed']);
$task_dates['overdue'] = array_unique($task_dates['overdue']);

echo json_encode(['success' => true, 'dates' => $task_dates]);

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```

---

**LANGKAH 3: Modifikasi AJAX Handler untuk Daftar Tugas Laporan**

Ganti nama file `ajax_get_tasks_for_date.php` menjadi `ajax_get_report_data_for_date.php` (atau buat file baru dengan nama ini dan hapus yang lama). File ini sekarang akan mengambil semua jenis tugas untuk tanggal yang dipilih.

**ajax_get_report_data_for_date.php (Dimodifikasi/Baru)**
```php
<?php
// File: ajax_get_report_data_for_date.php
require_once 'includes/db.php';
require_once 'includes/task_helper.php';

header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'User not authenticated.', 'tasks_html' => []]);
    exit();
}
$user_id = $_SESSION['user_id'];
$selected_date_str = $_GET['date'] ?? null; // Format YYYY-MM-DD

if (!$selected_date_str) {
    echo json_encode(['success' => false, 'message' => 'Tanggal tidak disediakan.', 'tasks_html' => []]);
    exit();
}

$date_parts = explode('-', $selected_date_str);
if (count($date_parts) !== 3 || !checkdate((int)$date_parts[1], (int)$date_parts[2], (int)$date_parts[0])) {
    echo json_encode(['success' => false, 'message' => 'Format tanggal tidak valid.', 'tasks_html' => []]);
    exit();
}

$tasks_html_output = [];
// Query untuk mengambil SEMUA tugas yang relevan dengan tanggal tersebut (baik due_date atau tanggal selesai)
// Kita akan memprioritaskan due_date untuk pemilihan, tapi status akan menentukan styling
$sql = "SELECT id, title, description, priority, status, 
               DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted, 
               due_date, 
               updated_at 
        FROM tasks
        WHERE user_id = ? AND 
              ( (status != 'Completed' AND due_date = ?) OR 
                (status = 'Completed' AND DATE(updated_at) = ?) )
        ORDER BY CASE status 
                    WHEN 'In Progress' THEN 1 
                    WHEN 'Not Started' THEN 2 
                    WHEN 'Completed' THEN 3 
                    ELSE 4 
                 END, 
                 due_date ASC, 
                 CASE priority WHEN 'High' THEN 1 WHEN 'Medium' THEN 2 WHEN 'Low' THEN 3 ELSE 4 END";

$stmt = $conn->prepare($sql);
if ($stmt) {
    $stmt->bind_param("iss", $user_id, $selected_date_str, $selected_date_str);
    $stmt->execute();
    $result = $stmt->get_result();
    
    $current_page_for_redirect = 'laporan.php';

    while ($task = $result->fetch_assoc()) {
        $page_type_for_card = 'reportActive'; // Default
        $is_overdue_card = false;
        if (isset($task['due_date']) && $task['status'] !== 'Completed' && strtotime($task['due_date']) < strtotime(date('Y-m-d'))) {
            $is_overdue_card = true;
        }

        if ($task['status'] === 'Completed') {
            $page_type_for_card = 'reportCompleted';
        } elseif ($is_overdue_card) {
            $page_type_for_card = 'reportOverdue';
        }
        // Jika tidak completed dan tidak overdue, tetap 'reportActive'

        $tasks_html_output[] = render_task_card(
            $task, 
            $page_type_for_card, 
            $current_page_for_redirect . '?selected_date=' . urlencode($selected_date_str)
        );
    }
    $stmt->close();
    echo json_encode(['success' => true, 'tasks_html' => $tasks_html_output]);
} else {
    echo json_encode(['success' => false, 'message' => 'Database query error: ' . $conn->error, 'tasks_html' => []]);
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```
**Perubahan Utama:**
*   Query SQL sekarang mengambil tugas yang `due_date`-nya adalah tanggal terpilih (jika belum selesai) ATAU tugas yang `status = 'Completed'` dan `updated_at` (tanggal selesainya) adalah tanggal terpilih. Ini memastikan semua jenis tugas yang relevan dengan tanggal tersebut muncul.
*   Logika penentuan `$page_type_for_card` untuk `render_task_card` disesuaikan.

---

**LANGKAH 4: Modifikasi `laporan.php`**

Ini akan melibatkan perubahan pada JavaScript untuk kalender dan penambahan tombol/modal untuk unduh laporan.

**laporan.php (Dimodifikasi)**
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

// --- DATA UNTUK DIAGRAM BATANG (Performa Pengerjaan - sama seperti sebelumnya) ---
// ... (Salin logika chart performa dari kode laporan.php Anda sebelumnya) ...
$today_for_default_perf = new DateTimeImmutable();
$performance_start_date_val = $today_for_default_perf->modify('-6 days')->format('Y-m-d');
$performance_end_date_val = $today_for_default_perf->format('Y-m-d');
$active_preset_perf_val = 'last7days';
// ... (logika filter tanggal untuk diagram batang) ...
// ... (logika query data chart) ...
$report_bar_chart_data_php_final = [ /* ... data chart Anda ... */ ];

?>
<title>Laporan Tugas - List In</title>

<main class="main">
    <div class="report-page-header">
        <h2 class="page-title">Laporan Produktivitas</h2>
        <button id="downloadReportBtn" class="btn btn-primary"><i class="fas fa-download"></i> Unduh Laporan</button>
    </div>

    <section class="widget report-chart-widget">
        <h3>Performa Pengerjaan Tugas</h3>
        <form method="GET" action="laporan.php" class="performance-filter-form report-performance-filter">
            <!-- ... (Form filter chart sama seperti sebelumnya) ... -->
        </form>
        <div class="widget-content-area chart-container" style="height: 250px; margin-top: 15px;">
            <canvas id="reportPerformanceBarChart"></canvas>
        </div>
    </section>

    <section class="report-tasks-by-date-section widget">
        <h3>Detail Tugas Harian</h3>
        <div class="report-interactive-area">
            <div class="report-task-list-container">
                <p id="selectedDateText" class="selected-date-indicator">Pilih tanggal di kalender untuk melihat detail tugas.</p>
                <div id="reportTasksList" class="widget-content-area scrollable-list">
                    <p class="no-tasks-message">Pilih tanggal pada kalender.</p>
                </div>
            </div>
            <div class="report-calendar-container">
                <div id="reportCalendar"></div>
            </div>
        </div>
    </section>

    <!-- Modal untuk Kustomisasi Laporan PDF -->
    <div id="reportDownloadModal" class="modal">
        <div class="modal-content">
            <span class="close-modal-btn">&times;</span>
            <h3>Kustomisasi Laporan PDF</h3>
            <form id="reportDownloadForm">
                <div class="form-group">
                    <label for="reportPdfDateRange">Rentang Tanggal Laporan:</label>
                    <input type="text" id="reportPdfDateRange" name="reportPdfDateRange" placeholder="Pilih rentang tanggal..." required>
                </div>
                <div class="form-group">
                    <label>Sertakan dalam Laporan:</label>
                    <div>
                        <input type="checkbox" id="includePdfChart" name="includePdfChart" value="1" checked>
                        <label for="includePdfChart" class="checkbox-label">Diagram Performa Pengerjaan</label>
                    </div>
                    <div>
                        <input type="checkbox" id="includePdfTaskDetails" name="includePdfTaskDetails" value="1" checked>
                        <label for="includePdfTaskDetails" class="checkbox-label">Detail Daftar Tugas (sesuai rentang)</label>
                    </div>
                </div>
                <div class="form-group">
                    <label for="reportTitle">Judul Laporan (Opsional):</label>
                    <input type="text" id="reportTitle" name="reportTitle" placeholder="cth: Laporan Produktivitas Mingguan">
                </div>
                 <div class="form-group">
                    <label for="reportNotes">Catatan Tambahan (Opsional):</label>
                    <textarea id="reportNotes" name="reportNotes" placeholder="Catatan atau kesimpulan singkat..."></textarea>
                </div>
                <button type="submit" class="btn btn-primary btn-full-width"><i class="fas fa-file-pdf"></i> Buat & Unduh PDF</button>
            </form>
        </div>
    </div>

</main>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // --- Script untuk Diagram Batang Laporan (sama seperti sebelumnya) ---
    // ... (Salin script Chart.js dari laporan.php Anda sebelumnya) ...


    // --- Script untuk Kalender Interaktif dan Daftar Tugas ---
    const calendarContainer = document.getElementById('reportCalendar');
    const tasksListContainer = document.getElementById('reportTasksList');
    const selectedDateText = document.getElementById('selectedDateText');
    let currentYear, currentMonth;
    let taskDatesData = { active: [], completed: [], overdue: [] }; // Untuk menyimpan tanggal dari AJAX

    function fetchTaskDatesAndRenderCalendar(year, month) {
        fetch('ajax_get_task_dates_for_calendar.php')
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    taskDatesData = data.dates;
                } else {
                    console.error("Gagal mengambil data tanggal tugas:", data.message);
                    taskDatesData = { active: [], completed: [], overdue: [] }; // Reset jika gagal
                }
                renderCalendar(year, month); // Render kalender setelah data diterima (atau jika gagal)
            })
            .catch(error => {
                console.error('Error fetching task dates:', error);
                taskDatesData = { active: [], completed: [], overdue: [] };
                renderCalendar(year, month); // Tetap render kalender dasar
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
            <button id="prevMonthBtnReportCal">&lt;</button>
            <span>${monthNames[month]} ${year}</span>
            <button id="nextMonthBtnReportCal">&gt;</button>
        `;
        calendarContainer.appendChild(header);

        const daysGrid = document.createElement('div');
        daysGrid.classList.add('calendar-days-grid-report');
        dayNames.forEach(day => { /* ... (render day names) ... */ }); // Sama

        const firstDayOfMonth = new Date(year, month, 1).getDay();
        const daysInMonth = new Date(year, month + 1, 0).getDate();

        for (let i = 0; i < firstDayOfMonth; i++) { /* ... (render empty cells) ... */ } // Sama

        for (let day = 1; day <= daysInMonth; day++) {
            const dayCell = document.createElement('div');
            dayCell.classList.add('calendar-day-report');
            dayCell.textContent = day;
            const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            dayCell.dataset.date = dateStr;
            
            // Pewarnaan tanggal
            if (taskDatesData.overdue.includes(dateStr)) {
                dayCell.classList.add('has-overdue-tasks');
            } else if (taskDatesData.active.includes(dateStr)) {
                dayCell.classList.add('has-active-tasks');
            } else if (taskDatesData.completed.includes(dateStr)) {
                dayCell.classList.add('has-completed-tasks');
            }
            // Prioritas: overdue > active > completed untuk pewarnaan tunggal.
            // Anda bisa membuat class lebih spesifik jika ingin kombinasi warna.

            const today = new Date();
            if (year === today.getFullYear() && month === today.getMonth() && day === today.getDate()) {
                dayCell.classList.add('today');
            }

            dayCell.addEventListener('click', function() { /* ... (event listener sama) ... */ }); // Sama
            daysGrid.appendChild(dayCell);
        }
        calendarContainer.appendChild(daysGrid);

        document.getElementById('prevMonthBtnReportCal').addEventListener('click', () => {
            month--; if (month < 0) { month = 11; year--; }
            fetchTaskDatesAndRenderCalendar(year, month); // Panggil fungsi yang fetch data juga
        });
        document.getElementById('nextMonthBtnReportCal').addEventListener('click', () => {
            month++; if (month > 11) { month = 0; year++; }
            fetchTaskDatesAndRenderCalendar(year, month); // Panggil fungsi yang fetch data juga
        });
    }

    function loadTasksForDate(dateStr) {
        // ... (fungsi loadTasksForDate sama, TAPI pastikan memanggil ajax_get_report_data_for_date.php) ...
        // Ganti fetch URL di dalam fungsi ini menjadi:
        // fetch(`ajax_get_report_data_for_date.php?date=${dateStr}`)
        // ... dan pastikan 'tasks_html' diakses dari data.tasks_html
    }

    const today = new Date();
    fetchTaskDatesAndRenderCalendar(today.getFullYear(), today.getMonth()); // Panggil fungsi awal

    // --- Script untuk Modal Unduh Laporan PDF ---
    const downloadReportBtn = document.getElementById('downloadReportBtn');
    const reportModal = document.getElementById('reportDownloadModal');
    const closeModalBtn = reportModal.querySelector('.close-modal-btn');
    const reportDownloadForm = document.getElementById('reportDownloadForm');

    if (downloadReportBtn) {
        downloadReportBtn.addEventListener('click', () => {
            reportModal.style.display = 'flex';
            // Inisialisasi Flatpickr untuk rentang tanggal PDF
            flatpickr(document.getElementById('reportPdfDateRange'), {
                mode: "range",
                dateFormat: "d/m/Y",
                locale: "id",
                defaultDate: ["today", new Date().fp_incr(6)], // Contoh default 7 hari
                allowInput: true
            });
        });
    }
    if (closeModalBtn) {
        closeModalBtn.addEventListener('click', () => {
            reportModal.style.display = 'none';
        });
    }
    window.addEventListener('click', (event) => {
        if (event.target == reportModal) {
            reportModal.style.display = 'none';
        }
    });

    if (reportDownloadForm) {
        reportDownloadForm.addEventListener('submit', function(event) {
            event.preventDefault();
            const formData = new FormData(this);
            const params = new URLSearchParams(formData).toString();
            
            // Buka tab baru untuk mengunduh/menampilkan PDF
            // Parameter akan dikirim via GET ke skrip PHP pembuat PDF
            window.open(`generate_report_pdf.php?${params}`, '_blank');
            
            reportModal.style.display = 'none'; // Tutup modal setelah submit
        });
    }

});
</script>
```
**Perubahan Utama JavaScript di `laporan.php`:**
*   `fetchTaskDatesAndRenderCalendar`: Fungsi baru untuk mengambil data tanggal tugas dari `ajax_get_task_dates_for_calendar.php` sebelum me-render kalender.
*   `renderCalendar`: Sekarang menggunakan `taskDatesData` untuk menambahkan kelas CSS (`has-active-tasks`, `has-completed-tasks`, `has-overdue-tasks`) ke sel tanggal.
*   Fungsi `loadTasksForDate` harus dimodifikasi untuk memanggil `ajax_get_report_data_for_date.php` dan menggunakan `data.tasks_html`.
*   Logika untuk menampilkan dan menangani modal unduh laporan PDF ditambahkan.

---

**LANGKAH 5: File PHP untuk Generate PDF**

Buat file baru `generate_report_pdf.php` di root proyek. **Anda perlu mengunduh FPDF dan meletakkannya di `includes/fpdf/fpdf.php` atau sesuaikan path require.**

**generate_report_pdf.php (Baru - Contoh Dasar dengan FPDF)**
```php
<?php
require_once 'includes/db.php'; // Untuk koneksi dan session
require_once 'includes/fpdf/fpdf.php'; // SESUAIKAN PATH KE FPDF ANDA

if (!isset($_SESSION['user_id'])) {
    die("Akses tidak sah.");
}
$user_id = $_SESSION['user_id'];

// Ambil parameter dari GET request
$date_range_str = $_GET['reportPdfDateRange'] ?? '';
$include_chart = isset($_GET['includePdfChart']) && $_GET['includePdfChart'] == '1';
$include_tasks = isset($_GET['includePdfTaskDetails']) && $_GET['includePdfTaskDetails'] == '1';
$report_title_custom = trim($_GET['reportTitle'] ?? '');
$report_notes = trim($_GET['reportNotes'] ?? '');

// Proses rentang tanggal
$start_date_mysql = null;
$end_date_mysql = null;
$date_range_display = "Semua Tanggal";

if (!empty($date_range_str)) {
    $dates = explode(' - ', $date_range_str);
    if (count($dates) >= 1 && !empty(trim($dates[0]))) {
        $date_parts_start = explode('/', trim($dates[0]));
        if (count($date_parts_start) == 3 && checkdate((int)$date_parts_start[1], (int)$date_parts_start[0], (int)$date_parts_start[2])) {
            $start_date_mysql = $date_parts_start[2] . '-' . $date_parts_start[1] . '-' . $date_parts_start[0];
        }
    }
    if (count($dates) == 2 && !empty(trim($dates[1]))) {
        $date_parts_end = explode('/', trim($dates[1]));
         if (count($date_parts_end) == 3 && checkdate((int)$date_parts_end[1], (int)$date_parts_end[0], (int)$date_parts_end[2])) {
            $end_date_mysql = $date_parts_end[2] . '-' . $date_parts_end[1] . '-' . $date_parts_end[0];
        }
    } elseif (count($dates) == 1 && $start_date_mysql) {
        $end_date_mysql = $start_date_mysql; // Jika hanya satu tanggal, rentangnya adalah tanggal itu sendiri
    }

    if ($start_date_mysql && $end_date_mysql) {
        if (strtotime($start_date_mysql) > strtotime($end_date_mysql)) {
            list($start_date_mysql, $end_date_mysql) = [$end_date_mysql, $start_date_mysql];
        }
        $date_range_display = date("d M Y", strtotime($start_date_mysql)) . " - " . date("d M Y", strtotime($end_date_mysql));
    } elseif ($start_date_mysql) {
        $date_range_display = date("d M Y", strtotime($start_date_mysql));
    }
}


// --- KELAS PDF KUSTOM (untuk Header & Footer) ---
class PDF extends FPDF {
    private $reportTitleMain;
    private $dateRangeDisplayMain;

    function __construct($orientation='P', $unit='mm', $size='A4', $reportTitle = 'Laporan Produktivitas', $dateRange = '') {
        parent::__construct($orientation, $unit, $size);
        $this->reportTitleMain = $reportTitle ?: 'Laporan Produktivitas Tugas';
        $this->dateRangeDisplayMain = $dateRange;
    }

    // Page header
    function Header() {
        // Logo (opsional)
        // $this->Image('path/to/logo.png',10,6,30);
        $this->SetFont('Arial','B',15);
        $this->Cell(80); // Pindah ke tengah
        $this->Cell(30,10,'ListIn',0,0,'C'); // Nama Aplikasi
        $this->Ln(10);

        $this->SetFont('Arial','B',12);
        $this->Cell(0,10, $this->reportTitleMain,0,1,'C');
        if (!empty($this->dateRangeDisplayMain)) {
            $this->SetFont('Arial','',10);
            $this->Cell(0,7,'Periode: ' . $this->dateRangeDisplayMain,0,1,'C');
        }
        $this->Ln(5); // Jarak setelah header
        // Garis pemisah header
        $this->SetDrawColor(180,180,180);
        $this->Line($this->GetX(), $this->GetY(), $this->GetX() + $this->GetPageWidth() - $this->lMargin - $this->rMargin, $this->GetY());
        $this->Ln(5);
    }

    // Page footer
    function Footer() {
        $this->SetY(-15);
        $this->SetFont('Arial','I',8);
        $this->SetTextColor(128);
        // Garis pemisah footer
        $this->Line($this->GetX(), $this->GetY() - 2, $this->GetX() + $this->GetPageWidth() - $this->lMargin - $this->rMargin, $this->GetY() - 2);
        $this->Cell(0,10,'Halaman '.$this->PageNo().'/{nb}',0,0,'C');
        $this->SetX($this->lMargin);
        $this->Cell(0,10,'Laporan dihasilkan pada: ' . date('d M Y, H:i'),0,0,'L');
    }

    // Fungsi untuk seksi dengan judul dan garis
    function SectionTitle($title) {
        $this->SetFont('Arial','B',11);
        $this->SetFillColor(230,230,230);
        $this->Cell(0,8,$title,0,1,'L', true);
        $this->Ln(2);
        $this->SetFont('Arial','',10);
    }
     // Fungsi untuk item tugas
    function TaskItem($title, $status, $priority, $dueDate, $description = '') {
        $this->SetFont('Arial','B',10);
        $this->MultiCell(0,6, iconv('UTF-8', 'ISO-8859-1//TRANSLIT', "Judul: " . $title)); // iconv untuk karakter non-latin
        $this->SetFont('Arial','',9);
        $this->Cell(0,5, iconv('UTF-8', 'ISO-8859-1//TRANSLIT', "Status: $status | Prioritas: $priority | Deadline: $dueDate"),0,1);
        if (!empty($description)) {
            $this->SetFont('Arial','I',9);
            $this->SetTextColor(80,80,80);
            $this->MultiCell(0,5, iconv('UTF-8', 'ISO-8859-1//TRANSLIT', "Deskripsi: " . $description));
            $this->SetTextColor(0,0,0); // Kembalikan warna teks
        }
        $this->Ln(3); // Jarak antar tugas
    }
}
// --- AKHIR KELAS PDF KUSTOM ---


// Buat instance PDF
$pdf_title_to_use = !empty($report_title_custom) ? iconv('UTF-8', 'ISO-8859-1//TRANSLIT', $report_title_custom) : 'Laporan Produktivitas Tugas';
$pdf = new PDF('P', 'mm', 'A4', $pdf_title_to_use, $date_range_display);
$pdf->AliasNbPages(); // Untuk nomor halaman total {nb}
$pdf->AddPage();
$pdf->SetFont('Arial','',10);
$pdf->SetTextColor(0);


// Bagian Catatan Tambahan (jika ada)
if ($include_tasks && !empty($report_notes)) { // Tampilkan catatan jika detail tugas juga disertakan
    $pdf->SectionTitle('Catatan Tambahan');
    $pdf->MultiCell(0,6, iconv('UTF-8', 'ISO-8859-1//TRANSLIT', $report_notes));
    $pdf->Ln(5);
}


// Bagian Diagram Performa (Placeholder - Implementasi Chart di PDF Lebih Kompleks)
if ($include_chart) {
    $pdf->SectionTitle('Diagram Performa Pengerjaan');
    // Untuk memasukkan chart ke PDF dengan FPDF secara dinamis itu kompleks.
    // Opsi:
    // 1. Buat chart sebagai gambar di sisi server (misal pakai pChart, JpGraph, atau library chart PHP)
    //    lalu $pdf->Image('path/to/chart_image.png', ...);
    // 2. Sederhanakan menjadi tabel data performa.
    // 3. Gunakan library PDF yang lebih canggih yang mendukung charting (misal, mPDF, Dompdf jika dari HTML).
    // Untuk contoh ini, kita tampilkan placeholder atau data tabel sederhana.
    $pdf->Cell(0,8,'[Visualisasi diagram performa pengerjaan akan ditampilkan di sini jika diimplementasikan.]');
    $pdf->Ln();
    $pdf->Cell(0,6,'Fitur ini memerlukan pembuatan gambar chart di server atau penggunaan library PDF yang lebih advance.');
    $pdf->Ln(8);

    // Contoh data tabel performa (jika Anda ingin)
    // $pdf->SetFont('Arial','B',10);
    // $pdf->Cell(40,7,'Tanggal',1);
    // $pdf->Cell(40,7,'Tugas Selesai',1);
    // $pdf->Ln();
    // $pdf->SetFont('Arial','',10);
    // Loop data chart Anda dan tampilkan di sini
    // foreach ($performance_chart_data['labels'] as $index => $label) {
    //    $pdf->Cell(40,6,$label,1);
    //    $pdf->Cell(40,6,$performance_chart_data['datasets'][0]['data'][$index],1);
    //    $pdf->Ln();
    // }
    // $pdf->Ln(5);
}


// Bagian Detail Daftar Tugas
if ($include_tasks) {
    $pdf->SectionTitle('Detail Daftar Tugas');
    
    $sql_tasks = "SELECT title, description, priority, status, DATE_FORMAT(due_date, '%d/%m/%Y') as due_date_formatted, due_date 
                  FROM tasks 
                  WHERE user_id = ?";
    $params_tasks = [$user_id];
    $types_tasks = "i";

    if ($start_date_mysql && $end_date_mysql) {
        $sql_tasks .= " AND ((status != 'Completed' AND due_date BETWEEN ? AND ?) OR (status = 'Completed' AND DATE(updated_at) BETWEEN ? AND ?))";
        array_push($params_tasks, $start_date_mysql, $end_date_mysql, $start_date_mysql, $end_date_mysql);
        $types_tasks .= "ssss";
    } elseif ($start_date_mysql) { // Hanya satu tanggal (rentang satu hari)
         $sql_tasks .= " AND ((status != 'Completed' AND due_date = ?) OR (status = 'Completed' AND DATE(updated_at) = ?))";
        array_push($params_tasks, $start_date_mysql, $start_date_mysql);
        $types_tasks .= "ss";
    }
    // Tambahkan ORDER BY jika perlu
    $sql_tasks .= " ORDER BY due_date ASC, CASE status WHEN 'In Progress' THEN 1 ELSE 2 END";


    $stmt_tasks_pdf = $conn->prepare($sql_tasks);
    if ($stmt_tasks_pdf) {
        $stmt_tasks_pdf->bind_param($types_tasks, ...$params_tasks);
        $stmt_tasks_pdf->execute();
        $result_tasks_pdf = $stmt_tasks_pdf->get_result();

        if ($result_tasks_pdf->num_rows > 0) {
            while ($task = $result_tasks_pdf->fetch_assoc()) {
                $status_display = $task['status'];
                if ($task['status'] !== 'Completed' && !empty($task['due_date']) && strtotime($task['due_date']) < strtotime(date('Y-m-d'))) {
                    $status_display .= " (Terlewat)";
                }
                $pdf->TaskItem(
                    $task['title'],
                    $status_display,
                    $task['priority'],
                    $task['due_date_formatted'] ?: 'N/A',
                    $task['description']
                );
            }
        } else {
            $pdf->Cell(0,10,'Tidak ada data tugas untuk periode yang dipilih.',0,1);
        }
        $stmt_tasks_pdf->close();
    } else {
        $pdf->Cell(0,10,'Gagal mengambil data tugas: ' . $conn->error,0,1);
    }
}

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}

// Output PDF
$filename = "Laporan_ListIn_" . date('Ymd_His') . ".pdf";
$pdf->Output('I', $filename); // 'I' untuk inline (tampil di browser), 'D' untuk download
exit;
?>
```
**PENTING untuk `generate_report_pdf.php`:**
*   **FPDF:** Pastikan Anda sudah mengunduh FPDF dan `require_once 'includes/fpdf/fpdf.php';` menunjuk ke lokasi yang benar.
*   **Karakter Non-Latin:** FPDF secara default tidak menangani UTF-8 dengan baik. Saya menggunakan `iconv('UTF-8', 'ISO-8859-1//TRANSLIT', $text)` untuk mencoba mengkonversi teks. Jika Anda memiliki banyak karakter non-standar, Anda mungkin perlu menggunakan FPDF dengan ekstensi UTF-8 (seperti tFPDF atau mPDF/Dompdf yang mendukung UTF-8 secara native).
*   **Visualisasi Chart di PDF:** Membuat chart dinamis di PDF dengan FPDF itu rumit. Contoh di atas hanya memberikan placeholder. Untuk chart sungguhan, Anda perlu:
    1.  Membuat gambar chart di sisi server menggunakan library PHP seperti JpGraph atau pChart.
    2.  Menyimpan gambar tersebut sementara.
    3.  Menggunakan `$pdf->Image('path/to/chart.png', ...)` untuk menyisipkannya ke PDF.
    Atau gunakan library PDF yang lebih canggih seperti mPDF atau Dompdf yang bisa merender HTML (termasuk JavaScript untuk Chart.js) ke PDF, meskipun ini lebih berat.

---

**LANGKAH 6: CSS Baru dan Modifikasi `css/style.css`**

Tambahkan CSS berikut ke akhir file `css/style.css` Anda.

**Tambahan untuk `css/style.css`:**
```css
/* ... (Semua CSS Anda sebelumnya) ... */

/* ==========================================================================
   Laporan Page - Kalender & Download Modal
   ========================================================================== */
.report-page-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px; /* Sesuaikan dengan .page-title-container jika ada */
}
.report-page-header .page-title {
    margin-bottom: 0;
    border-bottom: none;
}

/* Style untuk tanggal di kalender laporan yang memiliki tugas */
.calendar-day-report.has-active-tasks {
    /* background-color: #e3f2fd; Warna biru muda untuk tugas aktif */
    /* border: 1px solid #90caf9; */
    font-weight: bold;
    position: relative;
}
.calendar-day-report.has-active-tasks::after {
    content: '';
    position: absolute;
    bottom: 2px;
    left: 50%;
    transform: translateX(-50%);
    width: 5px;
    height: 5px;
    background-color: #2196f3; /* Biru untuk aktif */
    border-radius: 50%;
}

.calendar-day-report.has-completed-tasks {
    /* background-color: #e8f5e9; Warna hijau muda untuk tugas selesai */
    /* color: #388e3c; */
    position: relative;
}
.calendar-day-report.has-completed-tasks::after {
    content: '';
    position: absolute;
    bottom: 2px;
    left: 50%;
    transform: translateX(-50%);
    width: 5px;
    height: 5px;
    background-color: #4caf50; /* Hijau untuk selesai */
    border-radius: 50%;
}


.calendar-day-report.has-overdue-tasks {
    /* background-color: #fff8e1; Warna kuning muda untuk tugas terlewat */
    /* color: #ef6c00; */
    /* border: 1px solid #ffb300; */
    font-weight: bold;
    position: relative;
}
.calendar-day-report.has-overdue-tasks::after {
    content: '';
    position: absolute;
    bottom: 2px;
    left: 50%;
    transform: translateX(-50%);
    width: 5px;
    height: 5px;
    background-color: #f44336; /* Merah untuk overdue */
    border-radius: 50%;
}

/* Jika satu tanggal memiliki beberapa jenis, overdue akan menimpa active, active menimpa completed */
.calendar-day-report.has-active-tasks.has-completed-tasks::after { background-color: #2196f3; } /* Prioritaskan aktif */
.calendar-day-report.has-overdue-tasks.has-active-tasks::after { background-color: #f44336; } /* Prioritaskan overdue */
.calendar-day-report.has-overdue-tasks.has-completed-tasks::after { background-color: #f44336; } /* Prioritaskan overdue */
.calendar-day-report.has-overdue-tasks.has-active-tasks.has-completed-tasks::after { background-color: #f44336;} /* Prioritaskan overdue */


/* Modal Styling */
.modal {
    display: none; /* Hidden by default */
    position: fixed; /* Stay in place */
    z-index: 1050; /* Sit on top */
    left: 0;
    top: 0;
    width: 100%; /* Full width */
    height: 100%; /* Full height */
    overflow: auto; /* Enable scroll if needed */
    background-color: rgba(0,0,0,0.5); /* Black w/ opacity */
    align-items: center;
    justify-content: center;
}

.modal-content {
    background-color: #fefefe;
    margin: auto;
    padding: 25px 30px;
    border: 1px solid #888;
    width: 80%;
    max-width: 500px;
    border-radius: 8px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.3);
    position: relative;
    animation: fadeInModal 0.3s ease-out;
}

@keyframes fadeInModal {
    from { opacity: 0; transform: translateY(-20px) scale(0.95); }
    to { opacity: 1; transform: translateY(0) scale(1); }
}


.close-modal-btn {
    color: #aaa;
    float: right; /* Deprecated, better use flexbox/grid on parent */
    position: absolute;
    top: 10px;
    right: 15px;
    font-size: 28px;
    font-weight: bold;
    line-height: 1;
}

.close-modal-btn:hover,
.close-modal-btn:focus {
    color: black;
    text-decoration: none;
    cursor: pointer;
}

.modal-content h3 {
    margin-top: 0;
    margin-bottom: 20px;
    color: #333;
    font-size: 1.3rem;
    border-bottom: 1px solid #eee;
    padding-bottom: 10px;
}

.modal .form-group label.checkbox-label {
    font-weight: normal;
    font-size: 0.9rem;
    margin-left: 5px;
    display: inline; /* Agar sejajar dengan checkbox */
}
.modal .form-group input[type="checkbox"] {
    width: auto; /* Kembalikan ke default */
    margin-right: 5px;
    vertical-align: middle;
}


/* Dark Theme untuk Modal */
html.dark-theme-active .modal-content {
    background-color: #2c2c2c;
    border-color: #555;
    color: #e0e0e0;
}
html.dark-theme-active .modal-content h3 {
    color: #f5f5f5;
    border-bottom-color: #444;
}
html.dark-theme-active .close-modal-btn {
    color: #888;
}
html.dark-theme-active .close-modal-btn:hover,
html.dark-theme-active .close-modal-btn:focus {
    color: #ccc;
}
html.dark-theme-active .modal .form-group label,
html.dark-theme-active .modal .form-group label.checkbox-label {
    color: #ccc;
}
html.dark-theme-active .modal .form-group input[type="text"],
html.dark-theme-active .modal .form-group textarea {
    background-color: #373737;
    border-color: #555;
    color: #e0e0e0;
}
html.dark-theme-active .modal .form-group input[type="text"]:focus,
html.dark-theme-active .modal .form-group textarea:focus {
    border-color: #bb86fc;
    box-shadow: 0 0 0 2.5px rgba(187, 134, 252, 0.3);
    background-color: #3c3c3c;
}
html.dark-theme-active .modal .form-group input[type="checkbox"] {
    /* Styling checkbox di dark mode bisa tricky, mungkin perlu custom */
}

/* Indikator terlewat di card (jika belum ada) */
.task-item-card .meta-info .overdue-indicator {
    color: #e74c3c;
    font-weight: bold;
    margin-left: 5px;
}
html.dark-theme-active .task-item-card .meta-info .overdue-indicator {
    color: #ef9a9a;
}
```

---

**LANGKAH 7: Tutorial Lanjutan AI Chatbot dengan Google AI Studio (Gemini API)**

Mengintegrasikan Gemini API dari Google AI Studio memerlukan beberapa langkah:

**A. Dapatkan API Key dari Google AI Studio:**

1.  **Kunjungi Google AI Studio:** Pergi ke [https://aistudio.google.com/](https://aistudio.google.com/).
2.  **Login dengan Akun Google Anda.**
3.  **Buat API Key:**
    *   Di antarmuka AI Studio, cari opsi untuk mendapatkan API Key. Biasanya ada tombol seperti "**Get API key**" atau di bagian pengaturan proyek/akun Anda.
    *   Klik "**Create API key in new project**" (atau pilih proyek yang sudah ada jika ada).
    *   Salin API key yang dihasilkan. **Simpan API key ini dengan aman.** Ini adalah kredensial Anda untuk mengakses model Gemini.

**B. Instal Google AI Generative Language Client Library untuk PHP (via Composer):**

Pustaka ini akan memudahkan interaksi dengan Gemini API.

1.  Buka terminal Laragon Anda.
2.  Navigasikan ke direktori proyek Anda: `cd C:\laragon\www\nama_proyek`
3.  Jalankan perintah Composer:
    ```bash
    composer require google/generative-ai
    ```

**C. Modifikasi `ajax_chatbot_handler.php`:**

```php
<?php
// File: ajax_chatbot_handler.php
require_once 'config.php'; // Untuk CHATBOT_API_KEY (sekarang Google AI API Key), session_start()
require_once 'includes/db.php'; // Untuk $conn
require_once 'includes/header.php'; // Untuk add_notification (opsional)
require_once __DIR__ . '/vendor/autoload.php'; // Untuk pustaka Google Generative AI

// Gunakan namespace dari pustaka
use Google\AI\Generativelanguage\V1beta\SafetySetting;
use Google\AI\Generativelanguage\V1beta\SafetySetting\HarmBlockThreshold;
use Google\AI\Generativelanguage\V1beta\HarmCategory;
use Google\AI\Generativelanguage\V1beta\Client\GenerativeServiceClient;
use Google\AI\Generativelanguage\V1beta\GenerateContentRequest;
use Google\AI\Generativelanguage\V1beta\Content;
use Google\AI\Generativelanguage\V1beta\Part;

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

$bot_reply = "Maaf, saya tidak mengerti permintaan itu saat ini."; // Default reply
$google_ai_api_key = CHATBOT_API_KEY; // Ambil dari config.php

if (empty($google_ai_api_key) || $google_ai_api_key === 'MASUKKAN_API_KEY_CHATBOT_ANDA_DISINI') {
    echo json_encode(['success' => false, 'reply' => 'API Key untuk AI Chatbot belum dikonfigurasi.']);
    exit();
}

try {
    // Inisialisasi klien Gemini
    $client = new GenerativeServiceClient(['apiEndpoint' => 'generativelanguage.googleapis.com']);
    $model = 'gemini-pro'; // Atau model lain yang tersedia seperti 'gemini-1.5-flash-latest'

    // Buat prompt dengan konteks tugas aplikasi Anda
    // Ini adalah bagian penting untuk membuat chatbot relevan dengan aplikasi Anda
    $system_instruction = "Anda adalah ListIn Bot, asisten virtual untuk aplikasi manajemen tugas bernama ListIn. " .
                          "Tugas Anda adalah membantu pengguna mengelola tugas mereka. " .
                          "Anda bisa memberikan informasi tentang tugas, menghapus notifikasi, atau membersihkan daftar tugas berdasarkan perintah pengguna. " .
                          "Selalu jawab dengan ramah dan profesional. Jika diminta melakukan aksi, konfirmasi sebelum melakukannya jika aksinya destruktif. " .
                          "Anda TIDAK BISA mengubah email, username, password, atau foto profil pengguna. " .
                          "Data tugas pengguna akan diberikan jika relevan dengan pertanyaan mereka. " .
                          "Contoh perintah yang bisa Anda tangani: 'tampilkan tugasku', 'apa saja tugasku hari ini?', 'hapus semua notifikasi', 'bersihkan riwayat tugasku', 'hapus tugas \"Selesaikan laporan\"'. ";
    
    // Tambahkan konteks data tugas pengguna jika diperlukan (contoh sangat sederhana)
    // Untuk implementasi nyata, Anda mungkin perlu query tugas berdasarkan pertanyaan pengguna
    $user_tasks_context = "";
    // if (preg_match('/tugas/i', $user_message)) { // Jika pesan mengandung kata "tugas"
    //     $stmt_ctx = $conn->prepare("SELECT title, status, DATE_FORMAT(due_date, '%d %b') as due_date_f FROM tasks WHERE user_id = ? AND status != 'Completed' ORDER BY due_date ASC LIMIT 3");
    //     if ($stmt_ctx) {
    //         $stmt_ctx->bind_param("i", $user_id);
    //         $stmt_ctx->execute();
    //         $res_ctx = $stmt_ctx->get_result();
    //         if ($res_ctx->num_rows > 0) {
    //             $user_tasks_context .= "\n\nBerikut beberapa tugas aktif pengguna saat ini:\n";
    //             while ($task_ctx = $res_ctx->fetch_assoc()) {
    //                 $user_tasks_context .= "- Judul: " . $task_ctx['title'] . ", Status: " . $task_ctx['status'] . ", Deadline: " . ($task_ctx['due_date_f'] ?: 'N/A') . "\n";
    //             }
    //         } else {
    //             $user_tasks_context .= "\n\nPengguna saat ini tidak memiliki tugas aktif.";
    //         }
    //         $stmt_ctx->close();
    //     }
    // }

    // Gabungkan instruksi sistem, konteks, dan pesan pengguna
    // Untuk Gemini, lebih baik menggunakan format chat dengan peran 'user' dan 'model'
    $chat_history = [];
    if(!empty($system_instruction)){
        // Memberikan instruksi sistem sebagai pesan pertama dari 'user' bisa membantu model memahami perannya
        // Atau, beberapa model mendukung parameter 'system_instruction' terpisah.
        // Untuk Gemini Pro API PHP library saat ini, kita akan masukkannya ke chat history.
         $chat_history[] = (new Content())->setParts([(new Part())->setText($system_instruction)])->setRole('user'); // Instruksi sistem
         $chat_history[] = (new Content())->setParts([(new Part())->setText("Baik, saya mengerti. Saya ListIn Bot.")])->setRole('model'); // Respon model terhadap instruksi
    }
    // if(!empty($user_tasks_context)){ // Jika ada konteks tugas spesifik
    //    $chat_history[] = (new Content())->setParts([(new Part())->setText($user_tasks_context)])->setRole('user');
    //    $chat_history[] = (new Content())->setParts([(new Part())->setText("Data tugas diterima.")])->setRole('model');
    // }
    $chat_history[] = (new Content())->setParts([(new Part())->setText($user_message)])->setRole('user'); // Pesan pengguna


    $request = (new GenerateContentRequest())
        ->setModel($client->modelName($model))
        ->setContents($chat_history)
        ->setGenerationConfig(
            (new \Google\AI\Generativelanguage\V1beta\GenerationConfig())
                ->setTemperature(0.7) // Kreativitas (0.0 - 1.0)
                ->setTopK(40)
                ->setTopP(0.95)
                ->setMaxOutputTokens(200) // Batasi panjang output
        )
        ->setSafetySettings([
            new SafetySetting([
                'category' => HarmCategory::HARM_CATEGORY_HARASSMENT,
                'threshold' => HarmBlockThreshold::BLOCK_ONLY_HIGH // Sesuaikan level keamanan
            ]),
            new SafetySetting([
                'category' => HarmCategory::HARM_CATEGORY_HATE_SPEECH,
                'threshold' => HarmBlockThreshold::BLOCK_ONLY_HIGH
            ]),
            // Tambahkan kategori keamanan lain jika perlu
        ]);

    // Panggil API Gemini
    $response = $client->generateContent($request, ['x-goog-api-key' => $google_ai_api_key]);
    
    if ($response && $response->getCandidates() && count($response->getCandidates()) > 0) {
        $candidate = $response->getCandidates()[0]; // Ambil kandidat pertama
        if ($candidate->getContent() && $candidate->getContent()->getParts() && count($candidate->getContent()->getParts()) > 0) {
            $bot_reply = $candidate->getContent()->getParts()[0]->getText();
            
            // --- DETEKSI INTENT SEDERHANA DAN EKSEKUSI AKSI (CONTOH) ---
            // Ini masih perlu diandalkan pada kemampuan LLM untuk mengikuti instruksi
            // atau Anda bisa membuat sistem deteksi intent yang lebih canggih.

            $lower_bot_reply = strtolower($bot_reply); // Untuk pencocokan case-insensitive
            $lower_user_message = strtolower($user_message);

            // Contoh: Jika bot mengkonfirmasi untuk menghapus notifikasi
            if (strpos($lower_user_message, "hapus semua notifikasi") !== false && 
                (strpos($lower_bot_reply, "notifikasi telah dihapus") !== false || strpos($lower_bot_reply, "menghapus semua notifikasi") !== false )) {
                if (isset($_SESSION['notification_messages'])) unset($_SESSION['notification_messages']);
                $_SESSION['has_unread_notifications_badge'] = false;
                // add_notification("Notifikasi dibersihkan via chatbot.", "info"); // Notif internal
            } 
            // Contoh: Hapus riwayat
            else if (strpos($lower_user_message, "bersihkan riwayat") !== false &&
                     (strpos($lower_bot_reply, "riwayat telah dibersihkan") !== false || strpos($lower_bot_reply, "membersihkan riwayat") !== false )) {
                $stmt_clear_hist = $conn->prepare("DELETE FROM tasks WHERE user_id = ? AND (status = 'Completed' OR (due_date < CURDATE() AND status != 'Completed'))");
                if($stmt_clear_hist){
                    $stmt_clear_hist->bind_param("i", $user_id);
                    $stmt_clear_hist->execute();
                    $stmt_clear_hist->close();
                }
            }
            // Contoh: Hapus tugas tertentu (membutuhkan LLM untuk mengekstrak nama tugas)
            // Ini lebih kompleks karena Anda perlu LLM untuk mengidentifikasi nama tugas dari $user_message
            // dan kemudian Anda mencocokkan konfirmasi dari $bot_reply.
            // Misalnya, jika user_message: "hapus tugasku yang judulnya Selesaikan Laporan Proyek"
            // Dan bot_reply: "Baik, tugas 'Selesaikan Laporan Proyek' akan saya hapus."
            // Anda perlu parsing 'Selesaikan Laporan Proyek' dari bot_reply atau user_message.
            // Ini bisa dilakukan dengan Function Calling di Gemini jika modelnya mendukung, atau regex.

            // Tambahkan kondisi lain di sini

        } else {
            $bot_reply = "Maaf, saya menerima respons kosong dari AI.";
        }
    } else {
        // Cek jika ada blok karena safety settings
        $block_reason = "Tidak diketahui";
        if ($response && $response->getPromptFeedback() && $response->getPromptFeedback()->getBlockReason()) {
            $block_reason = HarmBlockThreshold::name($response->getPromptFeedback()->getBlockReason());
        } elseif ($response && count($response->getCandidates()) > 0 && $response->getCandidates()[0]->getFinishReason()){
            $block_reason = \Google\AI\Generativelanguage\V1beta\Candidate\FinishReason::name($response->getCandidates()[0]->getFinishReason());
        }
         $bot_reply = "Maaf, permintaan Anda tidak dapat diproses saat ini. Kemungkinan terblokir karena alasan: " . $block_reason;
    }

} catch (\Google\ApiCore\ApiException $e) {
    error_log("Google AI API Exception: " . $e->getMessage() . " Status: " . $e->getStatus() . " Details: " . print_r($e->getMetadata(), true));
    $bot_reply = "Terjadi kesalahan saat menghubungi layanan AI: " . $e->getMessage();
    if (strpos($e->getMessage(), 'API key not valid') !== false) {
        $bot_reply = "API Key untuk AI Chatbot tidak valid atau salah. Mohon periksa konfigurasi.";
    }
} catch (Exception $e) {
    error_log("Chatbot Handler Exception: " . $e->getMessage());
    $bot_reply = "Terjadi kesalahan internal pada chatbot.";
} finally {
    if (isset($client)) {
        $client->close();
    }
}

echo json_encode(['success' => true, 'reply' => nl2br(htmlspecialchars($bot_reply))]);

if (isset($conn) && $conn instanceof mysqli) {
    $conn->close();
}
?>
```
**Penjelasan Penting untuk `ajax_chatbot_handler.php` dengan Gemini:**
*   **API Key:** Pastikan `CHATBOT_API_KEY` di `config.php` diisi dengan API Key dari Google AI Studio Anda.
*   **Pustaka:** `require __DIR__ . '/vendor/autoload.php';` dan `use` statement diperlukan.
*   **Model:** `gemini-pro` adalah model teks umum yang baik. Anda bisa mencoba model lain seperti `gemini-1.5-flash-latest` yang lebih cepat jika tersedia untuk API key Anda.
*   **Prompt Engineering (Sangat Penting):**
    *   `$system_instruction`: Ini adalah "peran" yang Anda berikan kepada AI. Jelaskan secara detail apa tugasnya, batasan-batasannya, dan bagaimana ia harus berinteraksi. Semakin baik instruksi sistem Anda, semakin baik pula respons AI.
    *   **Konteks Tugas:** Bagian `$user_tasks_context` adalah contoh bagaimana Anda bisa menyertakan data relevan dari database Anda ke dalam prompt. Untuk pertanyaan spesifik tentang tugas, Anda mungkin perlu query database berdasarkan input pengguna dan menyertakan detail tugas tersebut dalam prompt ke Gemini.
    *   **Format Chat:** Gemini API bekerja paling baik dengan format percakapan (daftar pesan dengan peran `user` dan `model`).
*   **Konfigurasi Generasi (`GenerationConfig`):**
    *   `temperature`: Mengontrol kreativitas. Nilai lebih tinggi lebih kreatif, lebih rendah lebih deterministik.
    *   `maxOutputTokens`: Batasi panjang respons agar tidak terlalu panjang.
*   **Safety Settings:** Konfigurasikan filter keamanan untuk menghindari konten berbahaya.
*   **Deteksi Intent dan Eksekusi Aksi:**
    *   Contoh sederhana untuk `hapus semua notifikasi` dan `bersihkan riwayat` disertakan. Ini mengandalkan LLM untuk memberikan respons yang mengandung frasa konfirmasi tertentu.
    *   Untuk aksi yang lebih kompleks seperti "hapus tugas X", Anda memerlukan strategi yang lebih canggih:
        *   **Prompting yang Lebih Baik:** Instruksikan AI untuk mengekstrak nama tugas dan meminta konfirmasi dalam format tertentu.
        *   **Function Calling (Fitur Lanjutan Gemini):** Ini adalah cara yang lebih andal di mana Anda mendefinisikan fungsi-fungsi yang bisa dipanggil AI (misalnya, `hapusTugasDatabase(namaTugas)`). AI akan mengembalikan permintaan untuk memanggil fungsi tersebut dengan argumen yang diekstrak, dan Anda yang mengeksekusinya di backend. Ini lebih kompleks untuk diimplementasikan.
        *   **Regex atau String Matching pada Respons AI:** Anda bisa mencoba mencocokkan pola pada respons AI untuk mengekstrak informasi atau konfirmasi.
*   **Error Handling:** Tangani `ApiException` dari Google dan exception umum lainnya. Perhatikan pesan error jika API key tidak valid.
*   **Percobaan dan Iterasi:** Mengembangkan chatbot yang andal dengan LLM memerlukan banyak percobaan dengan prompt dan logika untuk menangani berbagai input pengguna.

---

**Sisa File (Tidak ada perubahan signifikan dari permintaan sebelumnya, jadi saya tidak akan menyertakannya lagi secara penuh, kecuali ada catatan):**

*   `css/style.css`: Tambahkan CSS yang telah diberikan di Langkah 6.
*   `includes/footer.php`: Pastikan logika untuk menampilkan/menyembunyikan chatbot sudah benar dan `ajax_chatbot_handler.php` dipanggil dengan benar.
*   File-file lain seperti `login.php`, `register.php`, `dashboard.php`, `manajemen_tugas.php`, `riwayat.php`, `tambah_tugas.php`, `edit_tugas.php`, `profil.php`, `ubah_password.php`, `forgot_password.php`, `reset_password.php`, `includes/sidebar.php`, `includes/header.php` (pastikan fungsi notifikasi disalin kembali), `includes/db.php`, `config.php` (pastikan `CHATBOT_API_KEY` diisi dengan Google AI API Key) harus sudah sesuai dengan versi terakhir yang kita diskusikan (dengan login Google dihilangkan dan SMTP SendGrid/Gmail dikonfigurasi).

---

**Penting Sebelum Menjalankan:**

1.  **Unduh FPDF:** Jika belum, letakkan di `includes/fpdf/`.
2.  **Instal `google/generative-ai`:** Jalankan `composer require google/generative-ai`.
3.  **Isi API Key Google AI Studio:** Masukkan API Key Anda ke `CHATBOT_API_KEY` di `config.php`.
4.  **Sesuaikan Prompt Chatbot:** Luangkan waktu untuk menyempurnakan `$system_instruction` di `ajax_chatbot_handler.php` agar sesuai dengan kemampuan yang Anda inginkan.

Ini adalah pengembangan yang cukup besar. Uji setiap fitur secara terpisah. Jika Anda mengalami masalah spesifik, berikan detail error dan kode yang relevan.
