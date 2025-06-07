Tentu, saya akan membuatkan file PHP mandiri untuk chatbot sesuai permintaan Anda. File ini akan berisi HTML, CSS, dan JavaScript yang dibutuhkan untuk menampilkan ikon chatbot, jendela obrolan, dan menangani interaksi pengguna.

File ini dirancang untuk "disuntikkan" ke dalam halaman Anda, bukan melalui `include` atau `require` di PHP. Anda akan menambahkannya menggunakan satu baris kode di file footer Anda.

### **Langkah 1: Siapkan Aset**

Pastikan Anda memiliki gambar ikon untuk chatbot. Berdasarkan kode yang Anda berikan, Anda memerlukan:
*   `assets/aicoach.png`: Logo yang muncul di header jendela chat.
*   `assets/HabitHub icon.png`: Ikon yang akan menjadi favicon (meskipun ini tidak berhubungan langsung dengan chatbot, ada baiknya disiapkan).
*   `assets/robot-icon.png`: **(REKOMENDASI)** Siapkan ikon robot kecil (misalnya ukuran 64x64px) untuk tombol mengambang di pojok kanan bawah. Saya akan menamakannya `robot-icon.png` dalam kode.

Letakkan semua file gambar ini di dalam folder `assets/` di root proyek Anda.

### **Langkah 2: Buat File CSS Khusus untuk Chatbot**

Buat file baru bernama **`css/chatbot_style.css`**. Ini akan berisi semua gaya visual untuk chatbot agar tidak tercampur dengan CSS utama Anda.

**File Baru: `css/chatbot_style.css`**
```css
/* File: css/chatbot_style.css */

/* ==========================================================================
   Chatbot Floating Icon
   ========================================================================== */
#chatbot-fab {
    position: fixed;
    bottom: 25px;
    right: 25px;
    width: 60px;
    height: 60px;
    background-color: #7e47b8; /* Warna primer ListIn */
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    box-shadow: 0 4px 12px rgba(0,0,0,0.25);
    z-index: 1010;
    transition: transform 0.2s ease-in-out, background-color 0.2s, opacity 0.3s, visibility 0.3s;
    border: none;
    padding: 0;
}
#chatbot-fab img {
    width: 32px;
    height: 32px;
    filter: brightness(0) invert(1); /* Membuat ikon PNG jadi putih */
}
#chatbot-fab:hover {
    transform: scale(1.1);
    background-color: #6a3aa2; /* Warna primer hover */
}
#chatbot-fab.hidden {
    transform: scale(0.5);
    opacity: 0;
    visibility: hidden;
}
html.dark-theme-active #chatbot-fab {
    background-color: #bb86fc; /* Warna primer dark mode */
}
html.dark-theme-active #chatbot-fab:hover {
    background-color: #a06fec;
}

/* ==========================================================================
   Chatbot Container
   ========================================================================== */
#chatbot-window {
    position: fixed;
    bottom: 25px;
    right: 25px;
    width: 370px;
    max-width: calc(100vw - 40px);
    height: 75vh;
    max-height: 550px;
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 8px 25px rgba(0,0,0,0.2);
    display: flex; /* Menggunakan flex untuk layout */
    flex-direction: column;
    overflow: hidden;
    z-index: 1009;
    border: 1px solid #ddd;
    transform: scale(0.95) translateY(10px);
    opacity: 0;
    visibility: hidden;
    transition: transform 0.3s ease-out, opacity 0.3s ease-out, visibility 0s 0.3s;
}
#chatbot-window.show {
    transform: scale(1) translateY(0);
    opacity: 1;
    visibility: visible;
    transition: transform 0.3s ease-out, opacity 0.3s ease-out, visibility 0s 0s;
}
html.dark-theme-active #chatbot-window {
    background-color: #2c2c2c;
    border-color: #444;
    box-shadow: 0 8px 30px rgba(0,0,0,0.4);
}


/* ==========================================================================
   Chatbot Header
   ========================================================================== */
#chatbot-header {
    background-color: #7e47b8;
    color: white;
    padding: 12px 15px;
    font-weight: 600;
    font-size: 1.1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-shrink: 0; /* Mencegah header menyusut */
}
html.dark-theme-active #chatbot-header {
    background-color: #bb86fc;
    color: #121212;
}
#chatbot-header .title-group {
    display: flex;
    align-items: center;
    gap: 10px;
}
#chatbot-header .title-group img {
    height: 24px;
    width: auto;
}
#chatbot-close-btn {
    background: none;
    border: none;
    color: inherit;
    font-size: 1.8rem;
    cursor: pointer;
    line-height: 1;
    padding: 0 5px;
    opacity: 0.8;
    transition: opacity 0.2s;
}
#chatbot-close-btn:hover {
    opacity: 1;
}

/* ==========================================================================
   Chatbot Messages Area
   ========================================================================== */
#chatbot-messages {
    flex-grow: 1; /* Mengisi sisa ruang */
    padding: 15px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 12px;
}
#chatbot-messages::-webkit-scrollbar { width: 6px; }
#chatbot-messages::-webkit-scrollbar-track { background: #f1f1f1; }
#chatbot-messages::-webkit-scrollbar-thumb { background: #ccc; border-radius: 3px; }
#chatbot-messages::-webkit-scrollbar-thumb:hover { background: #aaa; }
html.dark-theme-active #chatbot-messages::-webkit-scrollbar-track { background: #3a3a3a; }
html.dark-theme-active #chatbot-messages::-webkit-scrollbar-thumb { background: #555; }


.message-wrapper {
    display: flex;
    flex-direction: column;
    max-width: 85%;
}
.message-wrapper.sender-user {
    align-self: flex-end;
    align-items: flex-end;
}
.message-wrapper.sender-ai {
    align-self: flex-start;
    align-items: flex-start;
}
.sender-name {
    font-size: 0.75rem;
    color: #666;
    margin-bottom: 4px;
    padding: 0 8px;
}
html.dark-theme-active .sender-name {
    color: #aaa;
}

.message {
    padding: 10px 14px;
    border-radius: 18px;
    line-height: 1.5;
    font-size: 0.9rem;
    word-wrap: break-word;
}
.message.user {
    background-color: #7e47b8; /* Warna primer */
    color: white;
    border-bottom-right-radius: 4px;
}
.message.ai {
    background-color: #f1f0f0;
    color: #333;
    border-bottom-left-radius: 4px;
}
html.dark-theme-active .message.user {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active .message.ai {
    background-color: #3a3a3a;
    color: #e0e0e0;
}

/* Typing Indicator */
.typing-indicator { display: flex; align-items: center; padding: 5px 0; }
.typing-indicator span {
    height: 8px; width: 8px;
    margin: 0 2px; background-color: #999;
    border-radius: 50%; display: inline-block;
    animation: typing-bounce 1.4s infinite ease-in-out both;
}
.typing-indicator span:nth-of-type(1) { animation-delay: -0.32s; }
.typing-indicator span:nth-of-type(2) { animation-delay: -0.16s; }
@keyframes typing-bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1.0); }
}
html.dark-theme-active .typing-indicator span { background-color: #777; }

/* ==========================================================================
   Chatbot Confirmation Cards
   ========================================================================== */
.multi-confirmation-container {
    margin-top: 10px;
    display: flex;
    flex-direction: column;
    gap: 10px;
}
.confirmation-card {
    background-color: #fff;
    border: 1px solid #e7eaec;
    border-radius: 8px;
    padding: 12px;
    font-size: 0.85rem;
    transition: opacity 0.3s;
}
html.dark-theme-active .confirmation-card {
    background-color: #373737;
    border-color: #555;
}
.confirmation-card p { margin: 0 0 8px 0; }
.confirmation-card .habit-details {
    padding-left: 10px;
    border-left: 2px solid #7e47b8;
    margin-bottom: 12px;
}
html.dark-theme-active .confirmation-card .habit-details {
    border-left-color: #bb86fc;
}
.confirmation-card .actions {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
}
.confirmation-card .actions button {
    border-radius: 5px;
    padding: 6px 12px;
    font-size: 0.8rem;
    font-weight: 500;
    cursor: pointer;
    border: 1px solid;
    transition: background-color 0.2s, color 0.2s;
}
.confirmation-card .actions button.confirm {
    background-color: #7e47b8;
    color: white;
    border-color: #7e47b8;
}
.confirmation-card .actions button.cancel {
    background-color: transparent;
    color: #555;
    border-color: #ccc;
}
.confirmation-card .actions button:disabled {
    cursor: not-allowed;
    opacity: 0.6;
}
.status-label {
    text-align: right;
    font-style: italic;
    font-size: 0.8rem;
    margin-top: 8px !important;
    color: #666;
}
html.dark-theme-active .status-label { color: #aaa; }

/* ==========================================================================
   Chatbot Input Area
   ========================================================================== */
#chatbot-input-area {
    display: flex;
    padding: 12px;
    border-top: 1px solid #eee;
    background-color: #f9f9f9;
    flex-shrink: 0;
}
html.dark-theme-active #chatbot-input-area {
    border-top-color: #444;
    background-color: #333;
}
#chatbot-input {
    flex-grow: 1;
    padding: 10px 15px;
    border: 1px solid #ccc;
    border-radius: 20px;
    margin-right: 10px;
    font-size: 0.9rem;
    outline: none;
    transition: border-color 0.2s;
}
#chatbot-input:focus {
    border-color: #7e47b8;
}
html.dark-theme-active #chatbot-input {
    background-color: #252525;
    border-color: #555;
    color: #e0e0e0;
}
html.dark-theme-active #chatbot-input:focus {
    border-color: #bb86fc;
}
#chatbot-send-btn {
    background-color: #7e47b8;
    color: white;
    border: none;
    border-radius: 50%;
    width: 42px;
    height: 42px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 1.2rem;
    transition: background-color 0.2s;
}
#chatbot-send-btn:hover {
    background-color: #6a3aa2;
}
html.dark-theme-active #chatbot-send-btn {
    background-color: #bb86fc;
    color: #121212;
}
html.dark-theme-active #chatbot-send-btn:hover {
    background-color: #a06fec;
}
```

### **Langkah 3: Buat File PHP Mandiri untuk Chatbot**

Buat file baru bernama **`chatbot.php`** di direktori root proyek Anda. File ini akan berisi semua HTML, CSS, dan JavaScript yang dibutuhkan.

**File Baru: `chatbot.php`**
```php
<?php
// File: chatbot.php (Mandiri)
// Tujuan: Menyediakan HTML, CSS, dan JS untuk chatbot
// yang akan disuntikkan ke dalam halaman utama.
// File ini TIDAK BOLEH diakses langsung oleh pengguna.

// Memulai session untuk mendapatkan user_id jika ada
if (session_status() == PHP_SESSION_NONE) {
    session_start();
}

// Hanya jalankan jika pengguna sudah login
if (!isset($_SESSION['user_id'])) {
    // Keluar secara diam-diam jika tidak ada sesi.
    // Ini mencegah file ini dirender di halaman login/register.
    exit(); 
}

$user_id = $_SESSION['user_id'];
$username = $_SESSION['username'] ?? 'Pengguna';

// Buat kunci unik untuk Local Storage berdasarkan user_id
// Ini memastikan riwayat chat setiap pengguna terpisah.
$localStorageKey = "listin_chat_history_" . $user_id;

?>

<!-- 1. LINK KE CSS KHUSUS CHATBOT -->
<link rel="stylesheet" href="css/chatbot_style.css?v=<?php echo time(); ?>">

<!-- 2. HTML UNTUK IKON DAN JENDELA CHAT -->

<!-- Ikon Mengambang (Floating Action Button) -->
<button id="chatbot-fab" title="Tanya ListIn Bot">
    <img src="assets/robot-icon.png" alt="Chat Bot">
</button>

<!-- Jendela Chat (Awalnya Tersembunyi) -->
<div id="chatbot-window">
    <div id="chatbot-header">
        <div class="title-group">
            <img src="assets/aicoach.png" alt="ListIn Bot">
            <span>ListIn Bot</span>
        </div>
        <button id="chatbot-close-btn" title="Bersihkan & Tutup Chat">&times;</button>
    </div>
    <div id="chatbot-messages">
        <!-- Pesan akan dimuat di sini oleh JS -->
    </div>
    <div id="chatbot-input-area">
        <input type="text" id="chatbot-input" placeholder="Ketik pesan Anda..." autocomplete="off">
        <button id="chatbot-send-btn" title="Kirim Pesan">
            <i class="fas fa-paper-plane"></i>
        </button>
    </div>
</div>


<!-- 3. JAVASCRIPT UNTUK SEMUA LOGIKA CHATBOT -->
<script>
document.addEventListener('DOMContentLoaded', () => {
    // Seleksi elemen-elemen dari DOM
    const fab = document.getElementById('chatbot-fab');
    const window = document.getElementById('chatbot-window');
    const closeBtn = document.getElementById('chatbot-close-btn');
    const messagesDiv = document.getElementById('chatbot-messages');
    const input = document.getElementById('chatbot-input');
    const sendBtn = document.getElementById('chatbot-send-btn');

    // Ambil variabel dari PHP
    const chatHistoryKey = '<?php echo $localStorageKey; ?>';
    const currentUsername = '<?php echo $username; ?>';

    // Inisialisasi riwayat dari Local Storage atau buat baru
    let conversationHistory = JSON.parse(localStorage.getItem(chatHistoryKey)) || [];

    // Fungsi untuk menambah pesan ke tampilan chat
    const addMessageToChat = (text, sender, senderName, isHTML = false) => {
        const messageWrapper = document.createElement('div');
        messageWrapper.classList.add('message-wrapper', `sender-${sender}`);

        const senderNameDiv = document.createElement('div');
        senderNameDiv.classList.add('sender-name');
        senderNameDiv.textContent = senderName;

        const messageDiv = document.createElement('div');
        messageDiv.classList.add('message', sender);
        
        if (isHTML) {
            messageDiv.innerHTML = text;
        } else {
            messageDiv.textContent = text;
        }

        messageWrapper.appendChild(senderNameDiv);
        messageWrapper.appendChild(messageDiv);
        messagesDiv.appendChild(messageWrapper);
        messagesDiv.scrollTop = messagesDiv.scrollHeight; // Auto-scroll ke bawah
        return messageDiv;
    };

    // Fungsi untuk menyimpan seluruh riwayat ke Local Storage
    const saveHistory = () => {
        // Batasi riwayat agar tidak terlalu besar (misal 50 item terakhir)
        if (conversationHistory.length > 50) {
            conversationHistory = conversationHistory.slice(conversationHistory.length - 50);
        }
        localStorage.setItem(chatHistoryKey, JSON.stringify(conversationHistory));
    };

    // Fungsi untuk memuat dan menampilkan riwayat dari `conversationHistory`
    const loadAndDisplayHistory = () => {
        messagesDiv.innerHTML = '';
        if (conversationHistory.length === 0) {
            // Jika tidak ada riwayat, tampilkan pesan selamat datang
            const welcomeMsg = "Halo! Saya ListIn Bot. Ada yang bisa saya bantu terkait tugas Anda?";
            addMessageToChat(welcomeMsg, 'ai', 'ListIn Bot');
            // Tambahkan ke riwayat untuk dikirim ke API nanti
            conversationHistory.push({ role: "model", parts: [{ text: welcomeMsg }] });
        } else {
            // Jika ada riwayat, tampilkan semuanya
            conversationHistory.forEach(msg => {
                if (msg.role === 'user') {
                    addMessageToChat(msg.parts[0].text, 'user', currentUsername);
                } else if (msg.role === 'model') {
                    // Cek jika pesan model berisi kartu konfirmasi
                    if (msg.parts[0].text.includes('confirmation-card')) {
                         addMessageToChat(msg.parts[0].text, 'ai', 'ListIn Bot', true);
                    } else {
                         addMessageToChat(msg.parts[0].text, 'ai', 'ListIn Bot');
                    }
                }
            });
        }
    };
    
    // Fungsi untuk menampilkan indikator "mengetik..."
    const showTypingIndicator = () => {
        const typingIndicator = document.createElement('div');
        typingIndicator.id = 'typing-indicator';
        typingIndicator.classList.add('message-wrapper', 'sender-ai');
        typingIndicator.innerHTML = `
            <div class="sender-name">ListIn Bot</div>
            <div class="message ai typing-indicator">
                <span></span><span></span><span></span>
            </div>
        `;
        messagesDiv.appendChild(typingIndicator);
        messagesDiv.scrollTop = messagesDiv.scrollHeight;
        return typingIndicator;
    };

    // Fungsi untuk menghapus indikator "mengetik..."
    const removeTypingIndicator = () => {
        const indicator = document.getElementById('typing-indicator');
        if (indicator) {
            indicator.remove();
        }
    };
    
    // Fungsi untuk menampilkan kartu konfirmasi
    const displayConfirmationCard = (aiMessageText, suggestionsArray) => {
        let baseMessage = aiMessageText.replace(/\[SUGGESTION_START\].*\[SUGGESTION_END\]/s, '').trim();
        let confirmationCardsHTML = "";
        
        suggestionsArray.forEach((suggestion, index) => {
            let detailsHTML = "";
            let operationType = "";

            if (suggestion.intent === "create_task_suggestion") {
                operationType = "Buat Tugas Baru";
                detailsHTML = `<p><strong>Judul:</strong> ${suggestion.task_details.title || "N/A"}</p>`;
            } else if (suggestion.intent === "update_task_suggestion") {
                operationType = "Perbarui Tugas";
                detailsHTML = `<p><strong>Tugas Lama:</strong> ${suggestion.old_title || "N/A"}</p>`;
            } else if (suggestion.intent === "delete_task_suggestion") {
                operationType = "Hapus Tugas";
                detailsHTML = `<p><strong>Judul:</strong> ${suggestion.title || "N/A"}</p>`;
            } else if (suggestion.intent === "misc_action_suggestion") {
                 operationType = "Aksi Lain";
                 detailsHTML = `<p><strong>Aksi:</strong> ${suggestion.action_type.replace(/_/g, ' ')}</p>`;
            }
            
            if (operationType) {
                 confirmationCardsHTML += `
                    <div class="confirmation-card" data-suggestion-index="${index}">
                        <p><strong>${operationType}</strong></p>
                        <div class="habit-details">${detailsHTML}</div>
                        <div class="actions">
                            <button class="cancel" data-index="${index}">Batal</button>
                            <button class="confirm" data-index="${index}">Lanjutkan</button>
                        </div>
                    </div>`;
            }
        });
        
        const fullHTML = `${baseMessage.replace(/\n/g, '<br>')}<div class="multi-confirmation-container">${confirmationCardsHTML}</div>`;
        const messageDiv = addMessageToChat(fullHTML, "ai", "ListIn Bot", true);

        // Tambahkan event listener ke tombol yang baru dibuat
        messageDiv.querySelectorAll('button.confirm, button.cancel').forEach(btn => {
            btn.addEventListener('click', () => {
                const index = parseInt(btn.dataset.index);
                const isConfirmed = btn.classList.contains('confirm');
                handleSingleConfirmation(index, isConfirmed, suggestionsArray[index], messageDiv);
            });
        });

        // Simpan pesan AI yang berisi HTML ke riwayat
        conversationHistory.push({ role: 'model', parts: [{ text: fullHTML }] });
        saveHistory();
    };

    // Fungsi untuk menangani klik pada tombol konfirmasi
    const handleSingleConfirmation = async (index, isConfirmed, suggestion, originalMessageDiv) => {
        const cardElement = originalMessageDiv.querySelector(`.confirmation-card[data-suggestion-index="${index}"]`);
        if (cardElement) {
            cardElement.querySelectorAll(".actions button").forEach(btn => btn.disabled = true);
            cardElement.style.opacity = "0.7";
            cardElement.insertAdjacentHTML('beforeend', `<p class="status-label">${isConfirmed ? "Akan diproses..." : "Dibatalkan."}</p>`);
        }

        const userResponseText = isConfirmed ? `Ya, saya setuju.` : `Tidak, batalkan.`;
        addMessageToChat(userResponseText, 'user', currentUsername);
        conversationHistory.push({ role: 'user', parts: [{ text: userResponseText }] });
        saveHistory();

        if (isConfirmed) {
            const typingIndicator = showTypingIndicator();
            try {
                const response = await fetch("ajax_chatbot_handler.php", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ action: "confirm_suggested_action", suggestion }),
                });
                const data = await response.json();
                addMessageToChat(data.message, 'ai', 'ListIn Bot');
                conversationHistory.push({ role: 'model', parts: [{ text: data.message }] });
                saveHistory();
            } catch (error) {
                addMessageToChat("Maaf, ada masalah koneksi saat mengkonfirmasi.", 'ai', 'ListIn Bot');
            } finally {
                removeTypingIndicator(typingIndicator);
            }
        } else {
            addMessageToChat("Baik, aksi dibatalkan.", 'ai', 'ListIn Bot');
            conversationHistory.push({ role: 'model', parts: [{ text: "Baik, aksi dibatalkan." }] });
            saveHistory();
        }
    };

    // Fungsi utama untuk mengirim pesan
    const sendMessage = async () => {
        const messageText = input.value.trim();
        if (!messageText) return;

        addMessageToChat(messageText, 'user', currentUsername);
        conversationHistory.push({ role: "user", parts: [{ text: messageText }] });
        saveHistory(); // Simpan pesan pengguna
        input.value = "";

        const typingIndicator = showTypingIndicator();
        input.disabled = true;
        sendBtn.disabled = true;

        try {
            const response = await fetch("ajax_chatbot_handler.php", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({
                    action: "chat_with_ai",
                    history: conversationHistory.slice(0, -1) // Kirim semua riwayat KECUALI pesan terakhir
                }),
            });
            const data = await response.json();
            
            if (data.success) {
                if (data.suggestions_array && data.suggestions_array.length > 0) {
                    displayConfirmationCard(data.ai_message, data.suggestions_array);
                } else {
                    addMessageToChat(data.ai_message, 'ai', 'ListIn Bot');
                    conversationHistory.push({ role: "model", parts: [{ text: data.ai_message }] });
                    saveHistory(); // Simpan balasan AI
                }
            } else {
                addMessageToChat(data.message || "Maaf, terjadi kesalahan.", 'ai', 'ListIn Bot');
            }
        } catch (error) {
            addMessageToChat("Maaf, terjadi kesalahan koneksi.", 'ai', 'ListIn Bot');
        } finally {
            removeTypingIndicator();
            input.disabled = false;
            sendBtn.disabled = false;
            input.focus();
        }
    };
    
    // --- Event Listeners ---
    fab.addEventListener('click', () => {
        window.classList.toggle('show');
        fab.classList.toggle('hidden');
        if (window.classList.contains('show')) {
            loadAndDisplayHistory();
            input.focus();
        }
    });

    closeBtn.addEventListener('click', () => {
        if (confirm("Anda yakin ingin menutup dan membersihkan riwayat chat ini?")) {
            window.classList.remove('show');
            fab.classList.remove('hidden');
            // Hapus riwayat dari memori dan Local Storage
            conversationHistory = [];
            localStorage.removeItem(chatHistoryKey);
            // Kosongkan tampilan
            messagesDiv.innerHTML = '';
        }
    });
    
    sendBtn.addEventListener('click', sendMessage);
    input.addEventListener('keypress', (event) => {
        if (event.key === 'Enter') {
            sendMessage();
        }
    });

});
</script>
```

### **Langkah 4: Implementasi ke Proyek Anda**

Anda tidak perlu `include 'chatbot.php';` di mana pun. Sebagai gantinya, Anda akan menambahkan satu baris kode di file **`includes/footer.php`** untuk "menyuntikkan" konten dari `chatbot.php` ke dalam halaman.

Buka file **`includes/footer.php`** dan tambahkan kode PHP berikut **tepat sebelum** tag `</body>`.

**File yang Diubah: `includes/footer.php`**
```php
<?php
// ... (semua kode footer.php yang sudah ada) ...
?>
<?php else: ?>
<?php endif; ?>

<?php
// ==========================================================
// ==========       SUNTIKKAN CHATBOT DI SINI        ==========
// ==========================================================
// Tentukan halaman mana saja yang TIDAK akan menampilkan chatbot
$no_chatbot_pages = [
    'profil.php', 'edit_profil.php', 'ubah_password.php',
    'login.php', 'register.php', 'forgot_password.php', 'reset_password.php'
];

$current_page_for_chatbot = basename($_SERVER['SCRIPT_NAME']);

// Jika halaman saat ini TIDAK ADA di dalam daftar $no_chatbot_pages,
// maka tampilkan konten dari chatbot.php
if (!in_array($current_page_for_chatbot, $no_chatbot_pages)) {
    // readfile() akan membaca isi file dan langsung menampilkannya (output)
    // Ini lebih efisien daripada include/require untuk tujuan ini.
    @readfile('chatbot.php');
}
// ==========================================================
// ==========          AKHIR DARI SUNTIKAN         ==========
// ==========================================================
?>

</body>
</html>
<?php
// ... (sisa kode footer.php) ...
?>
```

### **Penjelasan Cara Kerjanya**

1.  **Isolasi Kode**: Semua yang berhubungan dengan chatbot (HTML, CSS, JS) sekarang ada di `chatbot.php` dan `css/chatbot_style.css`. Ini menjaga file-file utama Anda (seperti `footer.php`) tetap bersih.
2.  **Penyimpanan Per Pengguna**: Di `chatbot.php`, kunci untuk Local Storage dibuat secara dinamis menggunakan ID pengguna (`$localStorageKey = "listin_chat_history_" . $user_id;`). Ini secara otomatis memastikan riwayat obrolan setiap pengguna disimpan secara terpisah di browser mereka sendiri.
3.  **Logika Tampilan Ikon yang Persisten**:
    *   Ikon dan jendela chat sekarang ada di file `chatbot.php`.
    *   Logika di `footer.php` memutuskan apakah akan memuat `chatbot.php` atau tidak berdasarkan halaman yang sedang dibuka.
    *   Di dalam JavaScript `chatbot.php`, visibilitas ikon (`#chatbot-fab`) dan jendela (`#chatbot-window`) dikelola secara independen. Saat jendela chat terbuka, ikon disembunyikan. Saat jendela ditutup, ikon ditampilkan kembali.
    *   Karena logika ini terisolasi dan tidak bergantung pada status popup lain (notifikasi/kalender), mengklik di luar jendela chat tidak akan menghilangkan ikonnya. Ikon hanya akan hilang sementara jika jendelanya terbuka.
4.  **Implementasi Non-invasif**: Dengan menggunakan `readfile('chatbot.php');`, kita hanya "mencetak" isi dari file tersebut ke bagian bawah halaman. Ini adalah cara yang bersih untuk menambahkan blok fungsionalitas tanpa harus mengubah struktur file `footer.php` secara drastis. Tanda `@` di depan `readfile` digunakan untuk menekan error jika `chatbot.php` tidak ditemukan, meskipun seharusnya selalu ada.

Setelah menerapkan perubahan ini, chatbot Anda akan berfungsi sebagai komponen mandiri yang muncul di halaman yang tepat, dengan riwayat yang persisten per pengguna, dan visibilitas ikon yang lebih andal.
