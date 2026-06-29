# Technical Guide: WebSockets (`socket`)

Skalfa API provides real-time, bi-directional communication using Socket.IO via the `socket` utility.

---

## 1. Initializing the Socket Server

The socket server is started alongside the main HTTP server in `app/app.ts`:

```typescript
// app/app.ts
import { Elysia } from 'elysia'
import { socket } from '@utils'

const app = new Elysia()
// ... HTTP setup

// Start socket server on port 3001
socket.start(app, 3001)
```

---

## 2. Registering Socket Events

Socket events are registered in `app/jobs/sockets/` and can be grouped by namespace.

```typescript
// app/jobs/sockets/chat.socket.ts
import { socket } from '@utils'

// Public namespace
socket.on("connection", (io, client) => {
  console.log(`🔌 Client connected: ${client.id}`);

  // Listen to custom event
  client.on("send-message", (payload) => {
    // Broadcast message to all connected clients
    io.emit("message-received", {
      sender:    client.id,
      message:   payload.message,
      timestamp: new Date()
    });
  });

  client.on("disconnect", () => {
    console.log(`❌ Client disconnected: ${client.id}`);
  });
})
```

---

## 3. Authenticating Socket Connections

You can protect socket connections using middleware during connection:

```typescript
socket.use((client, next) => {
  const token = client.handshake.auth.token;
  if (!token) {
    return next(new Error("Authentication error: Token missing"));
  }
  
  // Verify token logic...
  next();
})
```

---

## 4. Emitting Events from Controllers or Services

You can emit real-time events from anywhere in your backend (e.g., from a controller or queue worker):

```typescript
import { socket } from '@utils'

// Emit to all clients in the "orders" room
socket.to("orders").emit("order-status-updated", {
  orderId: 123,
  status:  "shipped"
});
```
