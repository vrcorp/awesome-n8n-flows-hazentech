# 🤖 Asisten Virtual Toko Kelontong Ahtong (Telegram Chatbot)

*Dibuat menggunakan n8n*

Ini adalah *workflow* n8n yang berfungsi sebagai **Asisten Virtual** ("Bu Ahtong") untuk Toko Kelontong Ahtong, sebuah toko kebutuhan harian. Chatbot ini berjalan di **Telegram**, menggunakan **Google Gemini** (melalui node AI Agent) untuk memproses permintaan pelanggan, dan mencari jawaban dari data **FAQ di Google Sheets**.

## 📌 Fitur Utama

* **Pemicu Telegram (Telegram Trigger):** Menerima pesan masuk dari pelanggan dan mendapatkan ID pengguna untuk membalas.
* **AI Agent (Bu Ahtong):** Menggunakan model bahasa (Google Gemini Chat Model) dan alat pencarian (FAQ\_SHEET\_CONTENT) untuk memproses pertanyaan pelanggan.
* **Akses FAQ:** Menggunakan **Google Sheets Tool** (dinamai `FAQ_SHEET_CONTENT`) untuk mencari jawaban terkait harga, stok, atau informasi toko dari data FAQ yang telah disiapkan.
* **Memory Chat (Simple Memory):** Mempertahankan konteks percakapan dengan `sessionIdType: customKey` berdasarkan ID pengguna Telegram, dengan batas 10 pesan terakhir (`contextWindowLength: 10`).
* **Respons Terstruktur:** AI Agent diinstruksikan untuk selalu mengeluarkan respons dalam format **JSON** spesifik (`pesan` dan `sumber`).
* **Pembersihan Output (Code in JavaScript):** Memproses *output* dari AI Agent untuk membersihkan *markup* (`\`\`\`json`), mem-*parse* JSON, dan menyediakan *fallback* jika terjadi kesalahan *parsing*.

## ⚙️ Alur Kerja (Workflow)

Secara garis besar, alur kerja ini menerima pesan Telegram, mengirimkannya ke AI Agent, memproses respons, dan mengirimkannya kembali ke pelanggan.

1.  **Telegram Trigger**
    $\downarrow$
2.  **AI Agent** (Terkoneksi ke Google Gemini Chat Model, Simple Memory, dan FAQ\_SHEET\_CONTENT)
    $\downarrow$
3.  **Code in JavaScript**
    $\downarrow$
4.  **Send a text message**

### Detail Langkah-Langkah

| Node Name | Tipe Node | Fungsi Utama |
| :--- | :--- | :--- |
| **Telegram Trigger** | `telegramTrigger` | Memicu *workflow* saat ada pesan baru, menangkap `message.text` dan `message.from.id`. |
| **AI Agent** | `agent` | Memproses pesan, menerapkan persona "Bu Ahtong", dan mencari jawaban dari FAQ sheet. |
| **FAQ\_SHEET\_CONTENT** | `googleSheetsTool` | Alat yang diakses AI Agent, menunjuk ke Google Sheet ID `1FuCw69NnkXh3epmAdHpmcCNLRuGgVLpvaePXxMdupdM` dan Sheet Name `FAQ_Toko_Kelontong_Ahtong_Lengkap`. |
| **Code in JavaScript** | `code` | Membersihkan dan memvalidasi *output* JSON dari AI Agent, membuat *fallback* jika ada kegagalan *parsing*. |
| **Send a text message** | `telegram` | Mengirimkan balasan (`$json.pesan`) ke `chatId` pengguna dan me-*reply* pesan aslinya (`reply_to_message_id`). |

## 🛠️ Konfigurasi Kredensial

Untuk menjalankan *workflow* ini, Anda memerlukan kredensial berikut yang harus dikaitkan di setiap node yang relevan:

* **Telegram API:** Digunakan oleh `Telegram Trigger` dan `Send a text message` (Nama Credential: `BotAhtong2`).
* **Google Palm API / Gemini:** Digunakan oleh `Google Gemini Chat Model` (Nama Credential: `Gemini Ahtong2`).
* **Google Sheets OAuth2 API:** Digunakan oleh `FAQ_SHEET_CONTENT` (Nama Credential: `Gsheet Ahtong`).

## 💡 Prompt dan Logika AI Agent

AI Agent dikonfigurasi dengan `systemMessage` yang sangat detail untuk mendefinisikan persona dan *output* yang diharapkan:

* **Persona:** "Bu Ahtong", asisten virtual dari Toko Kelontong Ahtong—ramah, jujur, sabar, sopan, dan *helpful*.
* **Instruksi:** Carikan jawaban paling relevan dari `Tools FAQ_SHEET_CONTENT`. Jika tidak ditemukan, berikan jawaban alami dan ramah.
* **Format Wajib:** *Output* **selalu dalam format JSON** spesifik:
    ```json
    {
      "pesan": "isi jawaban singkat dan ramah, 1-2 kalimat saja",
      "sumber": "FAQ" atau "Manual" 
    }
    ```