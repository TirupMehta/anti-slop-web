# Real-Time WebSockets & Event Listener Baseline

Real-time features (WebSockets, Server-Sent Events, Socket.io, WebRTC) introduce memory leaks, stale authorization states, and unvalidated frame processing vulnerabilities.

---

## 1. Handshake Authentication & Re-Authentication

* **Initial Connection Handshake**: Authenticate WebSocket connections during the initial HTTP upgrade handshake using session tokens or short-lived auth tickets. Reject unauthorized connection requests before upgrading the socket.
* **Token Expiration on Reconnect**: When sockets auto-reconnect after a network drop, re-verify session validity. Never allow a client to maintain a socket connection if their underlying session has expired or been revoked.

---

## 2. Room & Topic Authorization Boundaries

* **Subscribing to Channels**: Before adding a socket connection to a channel/room (e.g., `room:workspace_123`), verify that the authenticated user has explicit read permissions for that workspace.
* **Never Trust Broadcast Payloads**: Incoming socket messages must be validated using Zod schemas on the server before broadcasting them to other room participants.

---

## 3. Client-Side Cleanup & Listener Memory Leaks

Client components subscribing to real-time events must clean up subscriptions, intervals, and event listeners when unmounting.

```typescript
// ❌ SLOP: Listener leak on component unmount
export function ChatComponent({ roomId }: { roomId: string }) {
  useEffect(() => {
    socket.subscribe(`room:${roomId}`);
    socket.on("message", (msg) => setMessages((prev) => [...prev, msg]));
    // Missing cleanup return function!
  }, [roomId]);
}

// ✅ PRODUCTION-GRADE: Explicit cleanup on unmount
export function ChatComponent({ roomId }: { roomId: string }) {
  useEffect(() => {
    const channel = socket.subscribe(`room:${roomId}`);
    const handleMessage = (msg: unknown) => {
      const parsed = messageSchema.safeParse(msg);
      if (parsed.success) {
        setMessages((prev) => [...prev, parsed.data]);
      }
    };

    channel.on("message", handleMessage);

    return () => {
      channel.off("message", handleMessage);
      socket.unsubscribe(`room:${roomId}`);
    };
  }, [roomId]);
}
```
