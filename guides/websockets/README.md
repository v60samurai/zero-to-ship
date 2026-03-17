# WebSockets and Real-Time Communication: Zero to Ship

> A phone call between your browser and a server, where either side can talk at any time, instead of the usual "ask a question, wait for an answer" routine.

## Why Should You Care?

Normal web communication is like sending letters. Your browser sends a request ("give me the latest messages"), the server sends a response, and the connection closes. If something changes on the server a second later, your browser has no idea until it sends another request. This works fine for loading a blog post. It falls apart for anything that needs to be live: chat, notifications, stock tickers, collaborative editing, multiplayer games, live dashboards, LLM streaming responses.

Real-time means the server can push data to the client the moment something happens, without the client asking first. Your chat message appears on the other person's screen instantly. The notification badge updates the second something changes. The dashboard graph moves as new data arrives. This isn't a nice-to-have anymore. Users expect it. Slack, Figma, Notion, Google Docs, every AI chatbot with streaming responses, all of them rely on real-time communication.

If you're building with AI tools, you'll encounter real-time patterns constantly. Every LLM chat interface streams tokens as they're generated (that's Server-Sent Events). Every collaborative tool uses WebSockets for live sync. Cursor and Claude Code generate WebSocket and SSE code readily, but the generated code almost always misses reconnection logic, authentication, and scaling concerns. Understanding the fundamentals lets you catch what AI tools leave out.

## The 30-Second Version

HTTP is one-way: the client asks, the server answers, the connection closes. For real-time features, you need the server to push data without being asked. Three approaches exist. Polling is the simplest: the client asks repeatedly on a timer ("anything new? anything new? anything new?"). Server-Sent Events (SSE) opens a one-way stream where the server can push data to the client. WebSockets open a persistent, two-way connection where both sides can send messages at any time. Use polling for simple, low-frequency updates. Use SSE when only the server needs to push (notifications, LLM streaming). Use WebSockets when both sides need to talk freely (chat, collaboration, games).

## The Real Explanation (ELI5)

Imagine you're waiting for exam results. The results will be posted on a notice board at school.

**Polling** is driving to school every 10 minutes, checking the board, and driving home. Wasteful. Most trips, there's nothing new. But it's simple and it works.

**Long polling** is driving to school and sitting in the parking lot. You walk in and ask "are the results up?" The receptionist says "not yet" but instead of sending you home, she says "wait here, I'll tell you the moment they're posted." You wait. When the results are posted, she tells you immediately. You read them, then start waiting again. Less wasteful than regular polling, but you're still making a new trip (connection) each time.

**Server-Sent Events (SSE)** is like subscribing to the school's announcement system. They call you the moment results are posted. You just listen. You can't talk back through the announcement system, but you don't need to. You just need to receive the update. One-way communication, server to client.

**WebSockets** is calling the school and keeping the phone line open. Either side can talk at any time. The receptionist can tell you results are posted. You can ask follow-up questions. Your friend on another line can hear updates too. The call stays open until one side hangs up. Two-way communication, always connected.

Now replace "school" with "server," "you" with "browser," "notice board" with "database," and "phone call" with "WebSocket connection." That's real-time communication.

The analogy is precise about one important thing: each approach has a real cost. Polling wastes fuel (bandwidth and server load). SSE ties up a phone line (connection) but only in one direction. WebSockets tie up a full phone line in both directions and need infrastructure to keep thousands of calls active simultaneously. The right choice depends on how often data changes and whether the client needs to talk back.

## How It Actually Works

### The Problem with HTTP

HTTP follows a strict pattern: request, response, done.

```
Client                          Server
  │                                │
  │──── GET /messages ────────────>│
  │                                │
  │<──── [messages 1-50] ─────────│
  │                                │
  Connection closed.

  Server receives new message...
  Server has no way to tell the client.
  Client doesn't know until it asks again.
```

Every interaction requires the client to initiate. The server can never say "hey, something happened" on its own. For a chat app, this means either the client polls constantly (wasteful) or you need a different protocol.

### Polling

The simplest approach. The client asks for updates on a timer.

```
Client                          Server
  │                                │
  │──── GET /messages?after=50 ──>│
  │<──── [] (no new messages) ────│
  │                                │
  ... 2 seconds later ...
  │                                │
  │──── GET /messages?after=50 ──>│
  │<──── [] (no new messages) ────│
  │                                │
  ... 2 seconds later ...
  │                                │
  │──── GET /messages?after=50 ──>│
  │<──── [message 51] ───────────│
  │                                │
```

```javascript
// Client-side polling
setInterval(async () => {
  const res = await fetch(`/api/messages?after=${lastId}`);
  const newMessages = await res.json();
  if (newMessages.length > 0) {
    addMessages(newMessages);
    lastId = newMessages.at(-1).id;
  }
}, 2000); // Every 2 seconds
```

**Pros:** Dead simple. Works with any HTTP server. No special infrastructure.
**Cons:** Wastes bandwidth when nothing changes. Latency equals the polling interval (2s interval means up to 2s delay). Doesn't scale well with many clients (every client hits the server every N seconds, whether or not there's new data).

**When to use it:** Low-frequency updates where a few seconds of delay is acceptable. Checking a job status. Refreshing a dashboard every 30 seconds. Under 100 concurrent users.

### Long Polling

An improvement on polling. The server holds the request open until there's something to send.

```
Client                          Server
  │                                │
  │──── GET /messages/poll ──────>│
  │          (server holds         │
  │           connection open,     │
  │           waiting...)          │
  │                                │
  │    ... 45 seconds pass ...     │
  │                                │
  │<──── [message 51] ───────────│
  │                                │
  │──── GET /messages/poll ──────>│  (immediately reconnect)
  │          (waiting again...)    │
```

**Pros:** Near-instant delivery. No wasted requests when nothing changes.
**Cons:** More complex server-side (holding connections open). Each reconnect cycle has overhead. Still one-directional (client can't send through the same connection).

**When to use it:** When you need lower latency than polling but can't use SSE or WebSockets (rare today, but some corporate proxies block them).

### Server-Sent Events (SSE)

A browser-native API for receiving a stream of events from the server. One-way: server to client only.

```
Client                          Server
  │                                │
  │──── GET /events ─────────────>│
  │      (Accept: text/event-stream)
  │                                │
  │<──── event: message           │
  │       data: {"text": "hello"} │
  │                                │
  │<──── event: message           │
  │       data: {"text": "world"} │
  │                                │
  │<──── event: notification      │
  │       data: {"count": 3}      │
  │                                │
  │   (connection stays open,      │
  │    server pushes whenever)     │
```

```javascript
// Client
const source = new EventSource("/api/events");

source.addEventListener("message", (e) => {
  const data = JSON.parse(e.data);
  console.log("New message:", data);
});

source.addEventListener("notification", (e) => {
  const data = JSON.parse(e.data);
  updateBadge(data.count);
});

// Built-in reconnection: if the connection drops,
// EventSource automatically reconnects.
```

```javascript
// Server (Express/Node.js)
app.get("/api/events", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // Send an event
  res.write(`event: message\ndata: ${JSON.stringify({ text: "hello" })}\n\n`);

  // Send more events whenever you want
  const interval = setInterval(() => {
    res.write(`event: ping\ndata: ${Date.now()}\n\n`);
  }, 30000);

  // Clean up when client disconnects
  req.on("close", () => {
    clearInterval(interval);
  });
});
```

**Pros:** Browser-native (no library needed). Auto-reconnection built in. Simple protocol (just HTTP with a special content type). Works through most proxies and firewalls.
**Cons:** One-way only (server to client). Limited to text data (no binary). Some browsers limit the number of SSE connections per domain to 6 (HTTP/1.1 limit, not an issue with HTTP/2).

**When to use it:** Notifications, live feeds, activity streams, LLM token streaming, any case where the server pushes and the client just listens.

**Why LLM streaming uses SSE:** When ChatGPT or Claude streams a response, each token is sent as an SSE event. The client renders tokens as they arrive. SSE is perfect for this because the data only flows one direction (server to client), auto-reconnection is built in, and the text/event-stream format is trivially simple. The client sends the prompt via a normal POST request, then opens an SSE connection (or the POST itself returns a stream) to receive the response token by token.

### WebSockets

A persistent, bidirectional connection. Either side can send messages at any time.

```
Client                          Server
  │                                │
  │──── HTTP Upgrade Request ────>│
  │      (Upgrade: websocket)      │
  │                                │
  │<──── 101 Switching Protocols ─│
  │                                │
  │ ═══════ WebSocket Open ══════ │
  │                                │
  │──── "hello from client" ─────>│
  │<──── "hello from server" ─────│
  │                                │
  │<──── "new message: hey!" ─────│
  │──── "typing..." ─────────────>│
  │<──── "user2 is typing" ───────│
  │                                │
  │ ═══ Connection stays open ═══ │
```

**The handshake:**

WebSocket connections start as a normal HTTP request with an "Upgrade" header. The server agrees (responds with 101), and the protocol switches from HTTP to WebSocket. After that, both sides communicate in WebSocket frames, not HTTP.

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

After this handshake, the connection is no longer HTTP. It's a raw TCP connection with a thin framing protocol on top. This is why WebSockets need special server support and why some proxies/load balancers need specific configuration to handle them.

**Native WebSocket API (browser):**

```javascript
const ws = new WebSocket("wss://server.example.com/chat");

ws.onopen = () => {
  console.log("Connected");
  ws.send(JSON.stringify({ type: "join", room: "general" }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log("Received:", data);
};

ws.onclose = (event) => {
  console.log("Disconnected:", event.code, event.reason);
  // You need to handle reconnection yourself
};

ws.onerror = (error) => {
  console.error("WebSocket error:", error);
};
```

**Native WebSocket server (Node.js with `ws`):**

```javascript
import { WebSocketServer } from "ws";

const wss = new WebSocketServer({ port: 8080 });

wss.on("connection", (ws) => {
  console.log("Client connected");

  ws.on("message", (data) => {
    const message = JSON.parse(data);
    console.log("Received:", message);

    // Broadcast to all connected clients
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify(message));
      }
    });
  });

  ws.on("close", () => {
    console.log("Client disconnected");
  });
});
```

### Socket.IO vs Native WebSocket API

Socket.IO is a library built on top of WebSockets that adds features you'd otherwise build yourself.

| Feature | Native WebSocket | Socket.IO |
|---------|-----------------|-----------|
| Bidirectional communication | Yes | Yes |
| Auto-reconnection | No (manual) | Yes (built-in, with backoff) |
| Rooms/channels | No (manual) | Yes (built-in) |
| Fallback transports | No | Yes (falls back to polling if WS fails) |
| Acknowledgements | No | Yes (callbacks on message receipt) |
| Binary support | Yes | Yes |
| Broadcasting | Manual loop | One-line API |
| Middleware | No | Yes (auth, logging) |
| Library required | No (browser-native) | Yes (client + server) |
| Protocol overhead | Minimal | Slight overhead (envelope format) |

**When to use native WebSocket:**
- You're building something simple (one channel, no rooms).
- You need minimal overhead (games, high-frequency data).
- The client isn't a browser (IoT device, another server).
- You want to avoid dependencies.

**When to use Socket.IO:**
- You need rooms, broadcasting, or acknowledgements.
- You need reliable reconnection.
- Your users might be behind restrictive proxies/firewalls (Socket.IO falls back to polling).
- You want to build a feature, not a real-time framework.

**Socket.IO is not a WebSocket wrapper.** It uses its own protocol on top of WebSockets. A plain WebSocket client can't connect to a Socket.IO server, and vice versa. If you use Socket.IO on the server, you must use the Socket.IO client library on the client.

### Rooms and Channels

Real-time apps rarely broadcast everything to everyone. You need to group connections and send messages to specific subsets.

**Rooms in Socket.IO:**

```javascript
// Server
io.on("connection", (socket) => {
  // Join a room
  socket.on("join-room", (roomId) => {
    socket.join(roomId);
    // Notify others in the room
    socket.to(roomId).emit("user-joined", { userId: socket.id });
  });

  // Send a message to a room
  socket.on("chat-message", ({ roomId, text }) => {
    io.to(roomId).emit("chat-message", {
      userId: socket.id,
      text,
      timestamp: Date.now(),
    });
  });

  // Leave a room
  socket.on("leave-room", (roomId) => {
    socket.leave(roomId);
    socket.to(roomId).emit("user-left", { userId: socket.id });
  });
});
```

```javascript
// Client
const socket = io("http://localhost:3000");

// Join a room
socket.emit("join-room", "general");

// Listen for messages in the room
socket.on("chat-message", (msg) => {
  addMessageToUI(msg);
});
```

**Key Socket.IO broadcasting methods:**

```javascript
// Send to everyone (including sender)
io.emit("event", data);

// Send to everyone in a room (including sender)
io.to("room1").emit("event", data);

// Send to everyone in a room EXCEPT the sender
socket.to("room1").emit("event", data);

// Send to a specific socket
io.to(socketId).emit("event", data);

// Send to everyone in multiple rooms
io.to("room1").to("room2").emit("event", data);

// Send to everyone EXCEPT a specific room
io.except("room1").emit("event", data);
```

**Without Socket.IO (native WebSockets):**

You manage rooms yourself with a data structure:

```javascript
const rooms = new Map(); // roomId -> Set of WebSocket connections

function joinRoom(ws, roomId) {
  if (!rooms.has(roomId)) rooms.set(roomId, new Set());
  rooms.get(roomId).add(ws);
}

function broadcastToRoom(roomId, message, excludeWs = null) {
  const room = rooms.get(roomId);
  if (!room) return;
  for (const client of room) {
    if (client !== excludeWs && client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(message));
    }
  }
}
```

It's not complex, but Socket.IO gives you this for free.

### Scaling WebSockets

WebSocket connections are stateful. Each connection lives on a specific server. This creates a problem when you have multiple servers.

```
The problem:

Server A has:  User1, User2, User3
Server B has:  User4, User5, User6

User1 sends a message in "general" room.
Server A broadcasts to User2 and User3. ✓
User4, User5, User6 never get it.    ✗
They're on a different server.
```

**Why this happens:** A load balancer distributes new connections across servers. Each server only knows about its own connections. When User1 sends a message, Server A doesn't know that Server B has users in the same room.

**Solution 1: Sticky sessions**

Force the load balancer to route all requests from the same client to the same server. This doesn't solve cross-server broadcasting, but it ensures a client's connection always hits the same server (important for Socket.IO's polling fallback).

```
# Nginx config for sticky sessions
upstream websocket_servers {
    ip_hash;  # Same client IP always goes to same server
    server server1:3000;
    server server2:3000;
}
```

**Solution 2: Redis pub/sub (the standard approach)**

Use Redis as a message bus between servers. When Server A receives a message, it publishes to Redis. All servers subscribe and re-broadcast to their local connections.

```
User1 sends message
        │
        ▼
Server A receives it
        │
        ├──> Broadcasts to Server A's local clients (User2, User3)
        │
        └──> Publishes to Redis channel "room:general"
                │
                ▼
        Server B subscribes to "room:general"
                │
                └──> Broadcasts to Server B's local clients (User4, User5, User6)
```

With Socket.IO, this is a one-line setup using the Redis adapter:

```javascript
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-adapter";
import { createClient } from "redis";

const io = new Server(3000);

const pubClient = createClient({ url: "redis://localhost:6379" });
const subClient = pubClient.duplicate();

await pubClient.connect();
await subClient.connect();

io.adapter(createAdapter(pubClient, subClient));

// That's it. Rooms, broadcasting, everything works across servers now.
```

**Solution 3: Managed real-time services**

Skip the infrastructure entirely. Services like Supabase Realtime, Ably, Pusher, and Liveblocks handle scaling, reconnection, and presence. You pay per connection/message.

### Reconnection

Connections drop. WiFi switches. Laptops sleep. Mobile networks are unstable. A real-time app that doesn't handle reconnection isn't a real-time app.

**What needs to happen when a connection drops:**

```
1. Detect the disconnect (immediately or via heartbeat timeout)
2. Attempt to reconnect (with exponential backoff)
3. Re-authenticate (the new connection doesn't inherit the old session)
4. Re-join rooms/channels
5. Sync missed messages (what happened while disconnected?)
6. Update UI (show connection status)
```

**Socket.IO handles steps 1-2 automatically:**

```javascript
const socket = io("http://localhost:3000", {
  reconnection: true,
  reconnectionDelay: 1000,      // Start with 1 second
  reconnectionDelayMax: 30000,  // Cap at 30 seconds
  reconnectionAttempts: Infinity,
});

socket.on("connect", () => {
  console.log("Connected");
  statusIndicator.classList.add("online");

  // Step 4: Re-join rooms after reconnection
  socket.emit("join-room", currentRoom);
});

socket.on("disconnect", (reason) => {
  console.log("Disconnected:", reason);
  statusIndicator.classList.remove("online");
});
```

**You still handle steps 3-6 yourself:**

```javascript
// Step 5: Sync missed messages after reconnect
socket.on("connect", () => {
  // Tell the server the last message you received
  socket.emit("sync", { lastMessageId: lastSeenId });
});

// Server sends missed messages
socket.on("sync-response", (missedMessages) => {
  missedMessages.forEach(addMessageToUI);
});
```

**Native WebSocket reconnection (manual):**

```javascript
function createConnection() {
  const ws = new WebSocket("wss://server.example.com/chat");
  let reconnectDelay = 1000;

  ws.onopen = () => {
    reconnectDelay = 1000; // Reset on successful connect
    console.log("Connected");
  };

  ws.onclose = () => {
    console.log(`Reconnecting in ${reconnectDelay}ms...`);
    setTimeout(() => {
      createConnection();
      reconnectDelay = Math.min(reconnectDelay * 2, 30000); // Exponential backoff
    }, reconnectDelay);
  };

  ws.onmessage = (event) => {
    handleMessage(JSON.parse(event.data));
  };

  return ws;
}

let ws = createConnection();
```

## The Mental Model

**Think of WebSockets as a phone call and HTTP as texting.** Texting (HTTP) is great when you need to send a message and get a response. But you hang up between messages. A phone call (WebSocket) keeps the line open: either person can talk at any time, and there's no delay in starting to speak. This means whenever you need instant, back-and-forth communication, think WebSocket. Whenever you need one-off requests with responses, think HTTP.

**Think of SSE as a radio broadcast.** You tune in (subscribe) and the station (server) sends audio (events) whenever it wants. You can't talk back through the radio. You'd need a phone (HTTP request) for that. This means whenever the server needs to push updates but the client doesn't need to send data through the same channel, SSE is simpler than WebSockets. LLM streaming, notifications, live scores, they're all radio broadcasts.

**Think of polling as constantly checking your mailbox.** You walk to the mailbox every 5 minutes. Usually it's empty. Occasionally there's a letter. It works, but you're doing a lot of walking for very little mail. This means whenever you're tempted to poll, ask yourself: "How much of my polling will return empty?" If the answer is "most of it," consider SSE or WebSockets instead.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Real-time | Data appears on the client the moment it's available on the server, without the client asking. | Chat, notifications, live dashboards, collaborative editing. |
| WebSocket | A protocol for persistent, bidirectional communication between client and server. | `ws://` or `wss://` URLs, chat systems, games. |
| SSE (Server-Sent Events) | A browser-native API for receiving a stream of events from the server. One-way only. | LLM streaming, notifications, live feeds. |
| Polling | The client repeatedly asks the server for updates on a timer. | Simple dashboards, job status checks. |
| Long polling | The server holds the request open until it has data to send. | Older real-time systems, fallback transport. |
| Handshake | The initial HTTP request that upgrades the connection from HTTP to WebSocket. | Connection setup, debugging connection failures. |
| Frame | A single unit of data sent over a WebSocket connection. | Low-level WebSocket debugging. |
| Socket.IO | A library that adds rooms, reconnection, and fallbacks on top of WebSockets. | Most real-time web apps. |
| Room | A named group of connections that can receive messages together. | Chat rooms, per-user channels, per-document collaboration. |
| Broadcasting | Sending a message to many connections at once. | Chat messages, live updates. |
| Namespace | A way to separate different types of communication on the same server (Socket.IO-specific). | `/chat`, `/notifications` on one server. |
| Heartbeat/Ping | A periodic message to check if the connection is still alive. | Connection health, detecting dead connections. |
| Backoff | Increasing the delay between reconnection attempts to avoid overwhelming the server. | Reconnection logic. |
| Sticky session | Routing all requests from the same client to the same server. | Load-balanced WebSocket servers. |
| Pub/sub | Publish/subscribe. A pattern where senders publish messages to a channel and receivers subscribe. | Redis adapter for multi-server WebSockets. |
| EventSource | The browser API for SSE. Creates an auto-reconnecting event stream. | `new EventSource("/api/events")`. |
| `wss://` | WebSocket Secure. WebSockets over TLS (encrypted). Like HTTPS for WebSockets. | Production WebSocket URLs. |

## Common Patterns

### Pattern 1: Chat Room

**When to use it:** Users send messages to each other in real time, organized by rooms or channels.

**How it works:** Each user connects via WebSocket, joins a room, and sends/receives messages. The server broadcasts messages to all users in the room.

```javascript
// Server (Socket.IO)
io.on("connection", (socket) => {
  socket.on("join", ({ roomId, username }) => {
    socket.join(roomId);
    socket.data.username = username;
    socket.data.roomId = roomId;
    socket.to(roomId).emit("system", `${username} joined`);
  });

  socket.on("message", ({ text }) => {
    const { roomId, username } = socket.data;
    io.to(roomId).emit("message", {
      id: crypto.randomUUID(),
      username,
      text,
      timestamp: Date.now(),
    });
  });

  socket.on("typing", () => {
    const { roomId, username } = socket.data;
    socket.to(roomId).emit("typing", { username });
  });

  socket.on("disconnect", () => {
    const { roomId, username } = socket.data;
    if (roomId) {
      socket.to(roomId).emit("system", `${username} left`);
    }
  });
});
```

```javascript
// Client
const socket = io();

socket.emit("join", { roomId: "general", username: "Alice" });

socket.on("message", (msg) => addMessageToUI(msg));
socket.on("typing", ({ username }) => showTypingIndicator(username));
socket.on("system", (text) => addSystemMessage(text));

function sendMessage(text) {
  socket.emit("message", { text });
}
```

### Pattern 2: Live Notifications

**When to use it:** Users need to see updates the moment they happen (new follower, order shipped, comment reply).

**How it works:** SSE is usually enough here since notifications flow one way (server to client). Each user opens an SSE connection. The server pushes events when something relevant happens.

```javascript
// Server (Express)
app.get("/api/notifications/stream", authenticate, (req, res) => {
  const userId = req.user.id;

  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // Register this connection
  addUserConnection(userId, res);

  req.on("close", () => {
    removeUserConnection(userId, res);
  });
});

// Somewhere in your app when a notification is created:
function notifyUser(userId, notification) {
  const connections = getUserConnections(userId);
  for (const res of connections) {
    res.write(`event: notification\ndata: ${JSON.stringify(notification)}\n\n`);
  }
}
```

```javascript
// Client
const source = new EventSource("/api/notifications/stream");

source.addEventListener("notification", (e) => {
  const notification = JSON.parse(e.data);
  showToast(notification.message);
  incrementBadge();
});
```

### Pattern 3: Real-Time Dashboard with Live Data

**When to use it:** A dashboard that shows metrics, graphs, or data tables that update as new data arrives.

**How it works:** SSE pushes data updates to the dashboard. If the data changes frequently (multiple times per second), throttle the updates to avoid overwhelming the UI.

```javascript
// Server: push updates at a controlled rate
app.get("/api/dashboard/stream", authenticate, (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // Send initial state
  const stats = getDashboardStats();
  res.write(`event: stats\ndata: ${JSON.stringify(stats)}\n\n`);

  // Send updates every 2 seconds (even if data changes more often)
  const interval = setInterval(() => {
    const stats = getDashboardStats();
    res.write(`event: stats\ndata: ${JSON.stringify(stats)}\n\n`);
  }, 2000);

  req.on("close", () => clearInterval(interval));
});
```

```javascript
// Client
const source = new EventSource("/api/dashboard/stream");

source.addEventListener("stats", (e) => {
  const stats = JSON.parse(e.data);
  updateChart(stats.revenue);
  updateCounter(stats.activeUsers);
  updateTable(stats.recentOrders);
});
```

### Pattern 4: Collaborative Editing (Basic)

**When to use it:** Multiple users edit the same document/board/list and see each other's changes in real time.

**How it works:** Each user joins a document room via WebSocket. Edits are sent as operations (not the full document). The server broadcasts operations to other users, who apply them locally.

```javascript
// Server
io.on("connection", (socket) => {
  socket.on("join-doc", (docId) => {
    socket.join(`doc:${docId}`);
    // Send current document state to the new user
    const doc = getDocument(docId);
    socket.emit("doc-state", doc);
  });

  socket.on("edit", ({ docId, operation }) => {
    // Apply operation to server state
    applyOperation(docId, operation);
    // Broadcast to others editing the same doc
    socket.to(`doc:${docId}`).emit("remote-edit", operation);
  });

  socket.on("cursor-move", ({ docId, position }) => {
    socket.to(`doc:${docId}`).emit("remote-cursor", {
      userId: socket.data.userId,
      position,
    });
  });
});
```

**Honest caveat:** Real collaborative editing (like Google Docs) is genuinely complex. It uses algorithms like OT (Operational Transformation) or CRDTs (Conflict-free Replicated Data Types) to handle simultaneous edits without conflicts. The pattern above works for simple cases (collaborative list, shared whiteboard) but will produce conflicts if two users edit the same text at the same cursor position. For serious collaborative editing, use a library like Yjs or Liveblocks rather than building from scratch.

### Pattern 5: LLM Streaming Responses via SSE

**When to use it:** You're building a chat interface that streams AI responses token by token, like ChatGPT.

**How it works:** The client sends a prompt via POST. The response is an SSE stream. Each event contains a chunk of the generated text. The client renders chunks as they arrive.

```javascript
// Server (Express + Anthropic SDK)
app.post("/api/chat", authenticate, async (req, res) => {
  const { messages } = req.body;

  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  const stream = await anthropic.messages.stream({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    messages,
  });

  for await (const event of stream) {
    if (event.type === "content_block_delta") {
      res.write(`data: ${JSON.stringify({ text: event.delta.text })}\n\n`);
    }
  }

  res.write(`data: ${JSON.stringify({ done: true })}\n\n`);
  res.end();
});
```

```javascript
// Client
async function streamChat(messages) {
  const response = await fetch("/api/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ messages }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = "";

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split("\n\n");
    buffer = lines.pop(); // Keep incomplete chunk in buffer

    for (const line of lines) {
      if (line.startsWith("data: ")) {
        const data = JSON.parse(line.slice(6));
        if (data.done) return;
        appendToResponse(data.text);
      }
    }
  }
}
```

Note: This uses the `fetch` streaming API rather than `EventSource` because the request is a POST with a body. `EventSource` only supports GET requests. The pattern is the same (SSE protocol), but the client-side code differs.

## Mistakes Everyone Makes

### 1. Using WebSockets when SSE or polling would be simpler

**What people do wrong:** They reach for WebSockets (or Socket.IO) for every real-time feature, even when the data only flows one direction.

**Why it seems right:** WebSockets are "the real-time technology." If you need real-time, use WebSockets. Right?

**What actually happens:** You set up a WebSocket server, manage connections, handle reconnection, configure your load balancer for sticky sessions, add a Redis adapter for scaling. For what? Pushing notifications that the user never sends anything back through. SSE would have done the same thing with five lines of code, automatic reconnection, and zero infrastructure changes.

**What to do instead:** Use this decision tree:
- Does the client need to send real-time data back? No → use SSE.
- Is the update frequency less than once every 10 seconds? → polling might be fine.
- Do you need bidirectional, high-frequency communication? → WebSockets.
- Are you unsure? → Start with SSE. Upgrade to WebSockets only when you have a concrete reason.

### 2. Not handling reconnection

**What people do wrong:** They set up a WebSocket connection and assume it stays open forever.

**Why it seems right:** "The connection is persistent. That's the whole point."

**What actually happens:** The user's WiFi blips. Their laptop sleeps and wakes. A proxy times out the idle connection. A deploy restarts the server. The connection drops silently. The user stares at a chat room that looks connected but receives nothing. No error message. No reconnection attempt. Just silence.

**What to do instead:** Always implement reconnection with exponential backoff. With Socket.IO, it's built in. With native WebSockets, wrap the connection in a function that recreates it on close (see the reconnection code in the "How It Actually Works" section). Always show a connection status indicator in the UI so the user knows when they're disconnected.

### 3. No heartbeat/ping to detect dead connections

**What people do wrong:** They rely on the `close` event to know when a connection drops.

**Why it seems right:** "The WebSocket API has an onclose handler. That's how I detect disconnections."

**What actually happens:** Not all disconnections trigger `close` cleanly. If the user's network drops abruptly (airplane mode, cable unplugged), the server may not know the connection is dead for minutes. The TCP keepalive timeout is typically 2 hours by default. During that time, the server thinks the client is connected and sends messages into the void.

**What to do instead:** Implement a heartbeat. The server sends a ping every 30 seconds. If the client doesn't respond with a pong within 10 seconds, consider the connection dead and clean it up.

```javascript
// Server (Socket.IO does this automatically with pingInterval/pingTimeout)
const io = new Server(server, {
  pingInterval: 25000,  // Send ping every 25 seconds
  pingTimeout: 20000,   // Wait 20 seconds for pong before disconnecting
});

// Server (native ws)
const interval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (!ws.isAlive) return ws.terminate();
    ws.isAlive = false;
    ws.ping();
  });
}, 30000);

wss.on("connection", (ws) => {
  ws.isAlive = true;
  ws.on("pong", () => { ws.isAlive = true; });
});
```

### 4. Sending too much data too frequently (flooding the client)

**What people do wrong:** They broadcast every database change, every sensor reading, every keystroke to every connected client, as fast as the data arrives.

**Why it seems right:** "Real-time means real-time. Send everything immediately."

**What actually happens:** The client's browser can't render 60 DOM updates per second. The UI freezes. The browser tab consumes 100% CPU. On mobile, the battery drains in minutes. The user experience is worse than polling every 5 seconds would be, because the client is drowning in data.

**What to do instead:** Throttle or batch updates. For a live dashboard, aggregate data and push every 1-2 seconds instead of on every change. For a live cursor/typing indicator, throttle to 5-10 updates per second. For a data feed, send diffs (what changed) instead of the full state.

```javascript
// Server-side throttling
let pendingUpdate = null;
let sendTimeout = null;

function scheduleUpdate(data) {
  pendingUpdate = data;
  if (!sendTimeout) {
    sendTimeout = setTimeout(() => {
      io.to("dashboard").emit("stats", pendingUpdate);
      pendingUpdate = null;
      sendTimeout = null;
    }, 1000); // Max one update per second
  }
}
```

### 5. Not authenticating WebSocket connections

**What people do wrong:** They authenticate the initial HTTP request but don't verify the user's identity when the WebSocket connection is established.

**Why it seems right:** "The user logged in on the website. The WebSocket is from the same browser. They must be the same user."

**What actually happens:** WebSocket connections don't automatically inherit HTTP session cookies in all cases. Even when they do, the connection might outlive the session (user logs out but the WebSocket stays open). Anyone who can guess or intercept the WebSocket URL can connect and receive data.

**What to do instead:** Authenticate during the WebSocket handshake. With Socket.IO, use middleware:

```javascript
// Socket.IO auth middleware
io.use(async (socket, next) => {
  const token = socket.handshake.auth.token;
  if (!token) return next(new Error("Authentication required"));

  try {
    const user = await verifyToken(token);
    socket.data.user = user;
    next();
  } catch (err) {
    next(new Error("Invalid token"));
  }
});
```

```javascript
// Client: pass the token when connecting
const socket = io("http://localhost:3000", {
  auth: {
    token: localStorage.getItem("accessToken"),
  },
});
```

For native WebSockets, pass the token as a query parameter or in the first message after connection, and reject unauthenticated connections immediately.

### 6. Not cleaning up connections on the server

**What people do wrong:** They track connected users in memory but forget to remove them when they disconnect, or they set up event listeners and intervals that leak.

**Why it seems right:** "I handle the connection event. The disconnect will handle itself."

**What actually happens:** Memory usage grows over time. The server tracks thousands of "connected" users who left hours ago. Broadcast loops iterate over dead connections. Event listeners pile up. Eventually the server runs out of memory or becomes unresponsive.

**What to do instead:** Always clean up in the disconnect handler. Remove the user from every room/map. Clear intervals and timeouts. Dereference the socket.

```javascript
io.on("connection", (socket) => {
  const userId = socket.data.user.id;
  onlineUsers.set(userId, socket.id);

  const heartbeat = setInterval(() => {
    socket.emit("ping");
  }, 30000);

  socket.on("disconnect", () => {
    onlineUsers.delete(userId);
    clearInterval(heartbeat);
  });
});
```

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Pick the simplest technology that solves your problem.** Polling for low-frequency updates. SSE for server-to-client streams. WebSockets only when you need bidirectional communication. Complexity has a cost, so don't pay it when you don't have to.

2. **Always show connection status in the UI.** A small dot, a banner, anything that tells the user whether they're connected, reconnecting, or offline. Silent disconnections destroy trust in real-time features.

3. **Implement reconnection with exponential backoff from day one.** Start at 1 second, double each attempt, cap at 30 seconds. This isn't optional. Connections will drop.

4. **Authenticate WebSocket connections during the handshake.** Don't assume a WebSocket connection is authorized just because the user is logged into the website. Verify tokens before accepting the connection.

5. **Throttle outgoing messages on the server.** Real-time doesn't mean "as fast as possible." It means "fast enough that the user perceives it as instant." For most UIs, one update per second is indistinguishable from 60.

6. **Use heartbeats to detect dead connections.** Ping every 25-30 seconds. If no pong within 10-20 seconds, terminate the connection and let the client reconnect cleanly.

7. **Plan for multi-server from the start.** Even if you run one server now, use patterns that scale. Socket.IO with the Redis adapter, or a managed service like Supabase Realtime. Refactoring from single-server to multi-server later is painful.

8. **Send diffs, not full state.** When data changes, send what changed, not the entire dataset. The client applies the diff locally. This reduces bandwidth, improves performance, and makes it easier to handle partial updates.

## The "Just Tell Me What to Do" Quickstart

You'll build a minimal real-time chat with Socket.IO in under 10 minutes. Multiple browser tabs can chat with each other.

**Prerequisites:** Node.js installed.

**Step 1: Set up the project**

```bash
mkdir realtime-quickstart && cd realtime-quickstart
npm init -y
npm install express socket.io
```

**Step 2: Create the server**

Create `server.js`:

```javascript
const express = require("express");
const { createServer } = require("http");
const { Server } = require("socket.io");

const app = express();
const server = createServer(app);
const io = new Server(server);

app.get("/", (req, res) => {
  res.sendFile(__dirname + "/index.html");
});

io.on("connection", (socket) => {
  console.log("User connected:", socket.id);

  socket.on("chat-message", (msg) => {
    io.emit("chat-message", {
      id: socket.id.slice(0, 6),
      text: msg,
      time: new Date().toLocaleTimeString(),
    });
  });

  socket.on("disconnect", () => {
    console.log("User disconnected:", socket.id);
  });
});

server.listen(3000, () => {
  console.log("Running at http://localhost:3000");
});
```

**Step 3: Create the client**

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Real-Time Chat</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: system-ui; background: #111; color: #eee; height: 100vh; display: flex; flex-direction: column; }
    #messages { flex: 1; overflow-y: auto; padding: 16px; }
    .msg { margin-bottom: 8px; }
    .msg span { color: #888; font-size: 13px; }
    form { display: flex; padding: 16px; gap: 8px; border-top: 1px solid #333; }
    input { flex: 1; padding: 10px; background: #222; border: 1px solid #444; color: #eee; border-radius: 6px; font-size: 15px; }
    button { padding: 10px 20px; background: #E8612D; border: none; color: white; border-radius: 6px; cursor: pointer; font-size: 15px; }
    #status { padding: 8px 16px; font-size: 13px; color: #888; }
  </style>
</head>
<body>
  <div id="status">Connecting...</div>
  <div id="messages"></div>
  <form id="form">
    <input id="input" placeholder="Type a message..." autocomplete="off" />
    <button>Send</button>
  </form>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();
    const messages = document.getElementById("messages");
    const form = document.getElementById("form");
    const input = document.getElementById("input");
    const status = document.getElementById("status");

    socket.on("connect", () => { status.textContent = "Connected"; });
    socket.on("disconnect", () => { status.textContent = "Disconnected. Reconnecting..."; });

    socket.on("chat-message", (msg) => {
      const div = document.createElement("div");
      div.className = "msg";
      div.innerHTML = `<span>${msg.id} ${msg.time}</span> ${msg.text}`;
      messages.appendChild(div);
      messages.scrollTop = messages.scrollHeight;
    });

    form.addEventListener("submit", (e) => {
      e.preventDefault();
      if (input.value.trim()) {
        socket.emit("chat-message", input.value);
        input.value = "";
      }
    });
  </script>
</body>
</html>
```

**Step 4: Run it**

```bash
node server.js
```

Open `http://localhost:3000` in two browser tabs. Type in one, see it appear in both instantly.

**Step 5: Verify reconnection**

1. Open two tabs chatting.
2. Stop the server (Ctrl+C). Both tabs show "Disconnected. Reconnecting..."
3. Restart the server. Both tabs reconnect automatically and show "Connected."

Socket.IO's built-in reconnection is working. That's real-time with resilience in under 50 lines of server code.

## How to Prompt AI Tools About This

### Context to provide

When asking Claude, Cursor, or Copilot about real-time features, always specify:
- Whether data flows one way (server to client) or both ways
- How many concurrent connections you expect
- Your server framework ("Express," "Fastify," "Next.js API routes")
- Whether you need rooms/channels
- Whether the client is a browser, mobile app, or another server

### Example prompts that produce good results

**Choosing the right approach:**
> "I need live notifications in a Next.js app. Notifications are server-to-client only, maybe 1-2 per minute per user. Should I use WebSockets, SSE, or polling? Show me the implementation for whichever you recommend."

**Socket.IO setup:**
> "Set up Socket.IO with Express and TypeScript. I need: rooms that users can join/leave, authentication middleware that verifies a JWT on connection, typing indicators, and reconnection handling that re-joins rooms. Include both server and client code."

**SSE for LLM streaming:**
> "I'm building a chat interface with Next.js that streams responses from the Anthropic API. Show me the API route that accepts a POST with messages and returns an SSE stream of tokens. Include the client-side code that reads the stream and renders tokens as they arrive."

**Scaling:**
> "I have a Socket.IO chat app on one server. I need to scale to 3 servers behind a load balancer. Show me how to add the Redis adapter, configure Nginx for WebSocket support with sticky sessions, and verify that cross-server messaging works."

### What to watch out for in AI-generated real-time code

- **No reconnection logic.** AI-generated WebSocket code almost never includes reconnection. If you see `new WebSocket(url)` without a reconnection wrapper, add one.
- **No authentication.** AI tends to skip WebSocket authentication entirely. Always ask for it explicitly.
- **Missing cleanup.** AI often handles `connection` but not `disconnect`. Check that intervals are cleared, users are removed from maps, and listeners are cleaned up.
- **Using WebSocket for everything.** AI defaults to WebSockets even when SSE would be simpler. Push back if your data only flows one direction.
- **Mixing Socket.IO server with native WebSocket client.** AI sometimes generates a Socket.IO server and a `new WebSocket()` client. These are incompatible. Socket.IO server requires the Socket.IO client library.
- **No error handling on the SSE stream.** AI often forgets `source.onerror` for EventSource or doesn't handle stream interruptions in fetch-based SSE.

### Key terms to use in prompts

Using these terms produces better output: "Socket.IO rooms," "SSE EventSource," "exponential backoff," "Redis adapter," "typing indicator," "connection status," "heartbeat ping/pong," "message acknowledgement," "JWT handshake auth."

## Ship It: Build This

Build a real-time chat application with Socket.IO that has multiple rooms, typing indicators, reconnection handling, and connection status.

**What you're building:** A multi-room chat app where users can pick a username, join different rooms, send messages, and see who's typing. The connection status is always visible. If the connection drops, the app reconnects automatically and re-joins the room. Messages show timestamps and usernames. A sidebar shows which rooms exist and how many users are in each.

**Why this project specifically:** It exercises every important real-time concept: bidirectional WebSocket communication, rooms and broadcasting, presence (who's online), ephemeral state (typing indicators), reconnection with room re-joining, and connection status UI. It's also a project you can extend into something real (add a database for message history, add authentication, add direct messages).

**Rough architecture:**

- Express + Socket.IO server
- Vanilla HTML/CSS/JS client (no framework, to focus on the real-time concepts)
- Multiple rooms that users can join and leave
- Typing indicator (debounced, shows "Alice is typing..." for 2 seconds after last keystroke)
- Connection status indicator (connected/disconnected/reconnecting)
- Username selection on first visit
- Room sidebar showing room names and user counts
- System messages for join/leave events
- Socket.IO built-in reconnection with room re-join on reconnect
- Server-side tracking of which users are in which rooms

**Estimated time:** 3-4 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn CRDTs and Operational Transformation** when you need real collaborative editing (multiple users editing the same document simultaneously). Libraries like Yjs and Automerge handle the hard parts.
- **Learn WebSocket compression (permessage-deflate)** when you're sending large volumes of text data and bandwidth is a concern. It trades CPU for bandwidth.
- **Learn WebTransport** when you need something faster than WebSockets for latency-sensitive applications. It's built on HTTP/3 and QUIC, supports unreliable delivery (useful for games), and is still relatively new.
- **Learn Supabase Realtime or Ably** when you want real-time features without managing WebSocket infrastructure. They handle scaling, reconnection, presence, and authentication as a service.
- **Learn binary WebSocket protocols (Protocol Buffers, MessagePack)** when JSON serialization/deserialization becomes a performance bottleneck, typically at very high message rates or with large payloads.
