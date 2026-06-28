# Panduan Teknis: WebSocket Realtime (Socket)

Skalfa API menyediakan layanan WebSocket berbasis Socket.IO untuk mengirimkan update data, status, atau notifikasi secara real-time ke client tanpa menggunakan polling.

---

## 1. Menjalankan Socket Server

Socket server berjalan pada port terpisah dari HTTP server utama. Port dikonfigurasi dan dinyalakan menggunakan `socket.start`.

```typescript
// src/jobs/sockets/workers.ts
import { socket } from '@utils'

// Menjalankan socket server pada port 4001
socket.start(4001)
```

---

## 2. Registrasi Event Socket

Event didaftarkan menggunakan `socket.event.on`. Anda dapat mendaftarkan event publik (bebas akses) atau event privat yang membutuhkan verifikasi token masuk (`auth()`).

```typescript
import { socket } from '@utils'

// 1. Event Publik (Tanpa Autentikasi)
socket.event.on('ping', (client, data) => {
  socket.send(client, 'pong', { time: Date.now() })
})

// 2. Event Terproteksi (Wajib Login)
// Di bawah kap, auth() memverifikasi JWT token sebelum memproses handler
socket.event.auth().on('subscribe-chat', (client, data) => {
  const user = client.data.user; // Mengakses data user yang login
  socket.join(client, `chat-room:${data.room_id}`);
})
```

---

## 3. Mengirimkan Event (Emit)

Skalfa menyediakan helper terpadu untuk menyebarkan (*broadcast*) event ke client:

```typescript
// A. Kirim ke satu client tertentu
socket.send(client, 'message:new', { text: "Halo!" })

// B. Kirim ke semua client yang tergabung di room tertentu
socket.room('chat-room:102').emit('message:new', { text: "Halo semua!" })

// C. Kirim ke seluruh client yang terhubung secara global
socket.emit('announcement', { text: "Server akan maintenance" })
```

---

## 4. Integrasi dengan Lapisan Lain (API / Queue)
Pola arsitektur terbaik di Skalfa adalah menggunakan **Queue Worker** untuk memproses transaksi berat, lalu memicu pengiriman notifikasi visual ke layar user menggunakan **Socket**:

```typescript
// Di dalam Queue Worker
queue.worker('payment-process', async (payload) => {
  // ... proses pembayaran ...
  
  // Kirim notifikasi realtime ke layar user
  socket.room(`user:${payload.user_id}`).emit('payment:success', {
    booking_id: payload.booking_id
  });
})
```
*Catatan untuk Agen: Anggap socket hanya sebagai channel event trigger (misal: memberitahu client bahwa ada data baru). Hindari mengirim data besar atau melakukan logika bisnis berat di dalam socket handler.*
