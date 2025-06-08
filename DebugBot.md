Tentu saja! Anda benar, untuk debugging murni, kita **tidak memerlukan database sama sekali**. Ini akan membuat prosesnya jauh lebih sederhana.

Berikut adalah paket uji coba Gemini yang **100% mandiri tanpa database**.

### **Paket Uji Coba Tanpa Database**

Anda hanya perlu membuat 3 file ini dan menguploadnya ke direktori `htdocs` di InfinityFree Anda.

#### **1. `index.php` (Antarmuka Utama & Debugger)**

File ini akan menjadi satu-satunya halaman yang Anda akses. Ia berisi HTML, CSS, dan JavaScript untuk antarmuka chatbot, ditambah sebuah area khusus untuk menampilkan pesan error.

```php
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Uji Coba Koneksi Gemini</title>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap" rel="stylesheet" />
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons+Outlined" rel="stylesheet" />
    <style>
        /* Tampilan dari kode referensi Anda */
        :root { --primary-color: #5d5fef; --background-color: #1a1a2e; --surface-color: #16213e; --text-color: #e0fbfc; --accent-color: #0f3460; }
        body { font-family: 'Poppins', sans-serif; margin: 0; background-color: var(--background-color); color: var(--text-color); display: flex; justify-content: center; align-items: center; min-height: 100vh; padding: 10px; }
        .background-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(45deg, rgba(22, 33, 62, 0.8), rgba(15, 52, 96, 0.8)); z-index: -1; }
        .test-wrapper { width: 100%; max-width: 800px; display: flex; gap: 20px; }
        .chat-container { width: 100%; max-width: 400px; height: 80vh; max-height: 600px; background-color: var(--surface-color); border-radius: 15px; box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3); display: flex; flex-direction: column; overflow: hidden; border: 1px solid var(--accent-color); }
        .chat-header { padding: 15px; background-color: var(--accent-color); text-align: center; }
        .chat-header img { height: 40px; }
        .chat-messages { flex-grow: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 15px; }
        .message-wrapper { display: flex; flex-direction: column; max-width: 85%; }
        .message-wrapper.sender-ai { align-self: flex-start; }
        .message-wrapper.sender-user { align-self: flex-end; }
        .sender-name { font-size: 0.75rem; color: #9a9a9a; margin-bottom: 4px; padding: 0 10px; }
        .sender-user .sender-name { text-align: right; }
        .message { padding: 10px 15px; border-radius: 20px; line-height: 1.5; }
        .message.ai { background-color: var(--accent-color); color: var(--text-color); border-top-left-radius: 5px; }
        .message.user { background-color: var(--primary-color); color: white; border-top-right-radius: 5px; }
        .typing-indicator span { display: inline-block; width: 8px; height: 8px; border-radius: 50%; background-color: #9a9a9a; margin: 0 2px; animation: bounce 1.4s infinite ease-in-out both; }
        .typing-indicator span:nth-child(1) { animation-delay: -0.32s; }
        .typing-indicator span:nth-child(2) { animation-delay: -0.16s; }
        @keyframes bounce { 0%, 80%, 100% { transform: scale(0); } 40% { transform: scale(1); } }
        .chat-input-area { display: flex; padding: 10px; border-top: 1px solid var(--accent-color); }
        #userInput { flex-grow: 1; background-color: var(--background-color); border: 1px solid var(--accent-color); border-radius: 20px; padding: 10px 15px; color: var(--text-color); font-size: 1rem; }
        #sendButton { background: var(--primary-color); border: none; border-radius: 50%; width: 45px; height: 45px; margin-left: 10px; cursor: pointer; display: flex; justify-content: center; align-items: center; color: white; }
        /* Debug Area */
        .debug-container { flex-grow: 1; background-color: #111; color: #ddd; border-radius: 10px; padding: 20px; font-family: 'Courier New', Courier, monospace; overflow-y: auto; height: 80vh; max-height: 600px;}
        .debug-container h3 { margin-top: 0; color: #0f0; }
        .debug-container pre { white-space: pre-wrap; word-wrap: break-word; background: #222; padding: 10px; border-radius: 5px; font-size: 0.8rem; }
        .debug-container p.error { color: #f00; font-weight: bold; }
        .debug-container p.success { color: #0f0; }
    </style>
</head>
<body>
    <div class="background-overlay"></div>
    
    <div class="test-wrapper">
        <div class="chat-container">
            <div class="chat-header">
                <!-- Anda bisa ganti src ke logo Anda jika sudah diupload -->
                <img src="https://i.imgur.com/w2g4P2B.png" alt="Test Logo" />
            </div>
            <div class="chat-messages" id="chatMessages"></div>
            <div class="chat-input-area">
                <input type="text" id="userInput" placeholder="Ketik pesan Anda..." autocomplete="off" />
                <button id="sendButton">
                    <span class="material-icons-outlined">send</span>
                </button>
            </div>
        </div>

        <div class="debug-container" id="debugContainer">
            <h3>Debug Log</h3>
            <p>Kirim pesan untuk memulai debugging...</p>
        </div>
    </div>

    <script>
        const chatMessages = document.getElementById("chatMessages");
        const userInput = document.getElementById("userInput");
        const sendButton = document.getElementById("sendButton");
        const debugContainer = document.getElementById("debugContainer");

        let conversationHistory = [];

        function logToDebug(title, content, isError = false) {
            const titleEl = document.createElement("h4");
            titleEl.textContent = title;
            if(isError) titleEl.style.color = "#ff8a80";
            
            const contentEl = document.createElement("pre");
            contentEl.textContent = typeof content === 'object' ? JSON.stringify(content, null, 2) : content;

            debugContainer.appendChild(titleEl);
            debugContainer.appendChild(contentEl);
            debugContainer.scrollTop = debugContainer.scrollHeight;
        }

        function addMessageToChat(text, sender, isHTML = false) {
            const messageWrapper = document.createElement("div");
            messageWrapper.classList.add("message-wrapper", `sender-${sender}`);
            const senderName = sender === 'ai' ? 'AI Coach' : 'Anda';

            messageWrapper.innerHTML = `
                <div class="sender-name">${senderName}</div>
                <div class="message ${sender}"></div>
            `;
            
            const messageDiv = messageWrapper.querySelector('.message');
            if (isHTML) {
                messageDiv.innerHTML = text;
            } else {
                messageDiv.textContent = text;
            }
            chatMessages.appendChild(messageWrapper);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        function showTypingIndicator() {
            addMessageToChat('<div class="typing-indicator"><span></span><span></span><span></span></div>', 'ai', true);
            return chatMessages.lastChild;
        }

        async function sendMessage() {
            const userText = userInput.value.trim();
            if (!userText) return;

            debugContainer.innerHTML = '<h3>Debug Log</h3>'; // Bersihkan log lama
            logToDebug("1. Pesan Dikirim dari UI", userText);

            addMessageToChat(userText, "user");
            userInput.value = "";

            conversationHistory.push({ role: "user", parts: [{ text: userText }] });
            logToDebug("2. Riwayat yang Akan Dikirim", conversationHistory);

            const typingIndicator = showTypingIndicator();

            try {
                const response = await fetch("gemini_handler.php", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ history: conversationHistory }),
                });

                logToDebug("3. Status Respons HTTP", `${response.status} ${response.statusText}`);

                const responseText = await response.text();
                logToDebug("4. Teks Respons Mentah dari Server", responseText);
                
                // Hapus indikator loading setelah mendapat respons
                typingIndicator.remove();

                if (!response.ok) {
                    throw new Error(`Server merespons dengan error: ${response.status}. Isi: ${responseText}`);
                }

                const data = JSON.parse(responseText);
                logToDebug("5. Data JSON yang Berhasil Diparsing", data);
                
                if (data.success) {
                    addMessageToChat(data.ai_message, "ai");
                    conversationHistory.push({ role: "model", parts: [{ text: data.ai_message }] });
                    logToDebug("6. Hasil Akhir", "Sukses!", false);
                } else {
                    addMessageToChat("Error: " + data.message, "ai");
                    logToDebug("6. Hasil Akhir", "Gagal (dari server). Pesan: " + data.message, true);
                }

            } catch (error) {
                if (typingIndicator) typingIndicator.remove();
                addMessageToChat("Koneksi gagal. Cek log debug untuk detail.", "ai");
                logToDebug("6. Hasil Akhir", "Gagal (catch di JavaScript). Pesan: " + error.message, true);
            }
        }

        sendButton.addEventListener("click", sendMessage);
        userInput.addEventListener("keypress", (e) => {
            if (e.key === "Enter") sendMessage();
        });

        // Pesan selamat datang awal
        addMessageToChat("Halo! Ini adalah antarmuka uji coba Gemini. Silakan kirim pesan.", "ai");
    </script>
</body>
</html>
```

#### **2. `config.php`**

File ini hanya berisi API Key Anda.

```php
<?php
// File: config.php

// GANTI DENGAN API KEY GEMINI ANDA
define('GEMINI_API_KEY', 'AIzaSyXXXXXXXXXXXXXXXXXXXXXXX');
?>
```

#### **3. `gemini_handler.php`**

Ini adalah backend yang bersih, tanpa dependensi ke database atau session.

```php
<?php
// File: gemini_handler.php (Versi Tanpa Database/Session)

require_once 'config.php';

header('Content-Type: application/json');

// Ambil riwayat dari permintaan
$input = json_decode(file_get_contents('php://input'), true);
$history = $input['history'] ?? null;

if (!$history) {
    http_response_code(400);
    echo json_encode(['success' => false, 'message' => 'Riwayat (history) tidak ditemukan dalam permintaan.']);
    exit;
}

// Konfigurasi cURL untuk menghubungi API Gemini
$apiUrl = "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=" . GEMINI_API_KEY;
$payload = ['contents' => $history];

$ch = curl_init($apiUrl);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
curl_setopt($ch, CURLOPT_TIMEOUT, 30); // 30 detik timeout

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
$curlError = curl_error($ch);
curl_close($ch);

// Periksa hasil cURL
if ($curlError) {
    http_response_code(500);
    echo json_encode(['success' => false, 'message' => 'cURL Error: ' . $curlError]);
    exit;
}

// Kirim kembali respons apa adanya, baik sukses maupun gagal dari Gemini
http_response_code($httpCode);

// Coba parse respons, jika sukses, tambahkan flag 'success'
try {
    $responseData = json_decode($response, true, 512, JSON_THROW_ON_ERROR);
    
    if (isset($responseData['candidates'][0]['content']['parts'][0]['text'])) {
        $ai_text = $responseData['candidates'][0]['content']['parts'][0]['text'];
        echo json_encode(['success' => true, 'ai_message' => $ai_text]);
    } else {
        // Jika formatnya tidak dikenali tapi koneksi berhasil
        echo json_encode(['success' => false, 'message' => 'Format respons dari Gemini tidak dikenali.', 'details' => $responseData]);
    }
} catch (JsonException $e) {
    // Jika respons bukan JSON yang valid
    echo json_encode(['success' => false, 'message' => 'Gagal mem-parsing respons JSON dari Gemini.', 'raw_response' => $response]);
}
?>
```

### **Langkah 4: Cara Hosting dan Debugging**

1.  **Konfigurasi**: Edit `config.php` dan masukkan API Key Gemini Anda.
2.  **Upload**: Upload ketiga file ini (`index.php`, `config.php`, `gemini_handler.php`) ke direktori `htdocs` di InfinityFree.
3.  **Akses**: Buka domain Anda (misal `nama-anda.rf.gd`). Halaman `index.php` akan langsung tampil.
4.  **Uji Coba**:
    *   Ketik pesan di kotak chat, misalnya "halo".
    *   Klik tombol kirim.
    *   **Perhatikan area Debug Log di sebelah kanan.**

### **Cara Membaca Debug Log**

*   **1. Pesan Dikirim dari UI**: Menampilkan teks yang Anda ketik.
*   **2. Riwayat yang Akan Dikirim**: Menampilkan struktur data JSON yang dikirim ke `gemini_handler.php`.
*   **3. Status Respons HTTP**: Ini sangat penting.
    *   `200 OK`: Berarti JavaScript berhasil menghubungi `gemini_handler.php`.
    *   `404 Not Found`: File `gemini_handler.php` tidak ditemukan. Cek nama filenya.
    *   `500 Internal Server Error`: Ada error fatal di dalam `gemini_handler.php`.
*   **4. Teks Respons Mentah dari Server**: Ini adalah output paling jujur dari `gemini_handler.php`. Jika ada error PHP, Anda akan melihatnya di sini.
*   **5. Data JSON yang Berhasil Diparsing**: Jika langkah 4 adalah JSON yang valid, Anda akan melihat strukturnya di sini.
*   **6. Hasil Akhir**: Memberi kesimpulan apakah prosesnya sukses atau gagal dan apa penyebabnya.

Dengan paket ini, Anda memiliki semua alat yang dibutuhkan untuk melihat dengan tepat di mana letak masalahnya saat di-hosting.
