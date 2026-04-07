---
title: JavaScript SDK
excerpt: Reference for @opendatalabs/connect-js — server and React exports.
---

# JavaScript SDK

```bash
npm install @opendatalabs/connect-js
```

---

## Server

Import from `@opendatalabs/connect-js/server`. Run server-side only — never expose your API key or encryption secret to the browser.

### `createClient(options)`

```ts
import { createClient } from "@opendatalabs/connect-js/server";

const odl = createClient({
  apiBaseUrl: "https://api.opendatalabs.com/api/v1",
  apiKey: process.env.OPENDATALABS_API_KEY!,
  secret: process.env.OPENDATALABS_ENCRYPTION_SECRET!,
});
```

| Option | Required | Description |
| --- | --- | --- |
| `apiBaseUrl` | Yes | Base URL for the API |
| `apiKey` | Yes | Your server API key |
| `secret` | Yes | Your encryption secret. Comma-separate multiple values to support key rotation |

Returns an object with `createConnectSession` and `fetchConnectionResult`.

---

### `odl.createConnectSession(input)`

Creates a hosted Connect session.

```ts
const session = await odl.createConnectSession({
  appId: "odl_app_123",
  source: "instagram",
  scopes: ["read:user_profile", "read:posts"],
  origin: "https://yourapp.com",
});
// session.connectUrl — open this in an iframe
// session.connectionId — use this to fetch results after success
```

| Field | Required | Description |
| --- | --- | --- |
| `source` | Yes | Source identifier, e.g. `instagram` |
| `origin` | Yes | Exact embedding origin, e.g. `https://yourapp.com` |
| `appId` | No | Public app ID; defaults to the account's default app |
| `scopes` | No | Scopes to request |
| `redirectUrl` | No | Return URL for native or webview flows |

---

### `odl.fetchConnectionResult(connectionId)`

Fetches connection data after a successful flow. Decrypts the response automatically using the configured secret.

```ts
const result = await odl.fetchConnectionResult(connectionId);
// result.data — the user's connected data
```

---

## React

Import from `@opendatalabs/connect-js/react`. Run client-side.

### `OpenDataLabsProvider`

Wrap your app (or the subtree that needs Connect) with `OpenDataLabsProvider`. Pass it a `createSession` function that calls your backend — this keeps your API key and encryption secret off the client.

```tsx
import { OpenDataLabsProvider } from "@opendatalabs/connect-js/react";

<OpenDataLabsProvider
  appId="odl_app_123"
  createSession={async (input) => {
    const res = await fetch("/api/connect-session", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(input),
    });
    return res.json(); // { connectionId, connectToken, connectUrl }
  }}
>
  {children}
</OpenDataLabsProvider>
```

| Prop | Required | Description |
| --- | --- | --- |
| `createSession` | Yes | Async function that calls your backend and returns `{ connectionId, connectToken, connectUrl }` |
| `appId` | No | Passed through to `createSession` |

---

### `useConnect()`

Returns a function that creates a Connect session via the provider's `createSession`.

```ts
const connect = useConnect();

const session = await connect({
  source: "instagram",
  scopes: ["read:user_profile"],
});
// session.connectUrl, session.connectionId
```

Must be used inside `OpenDataLabsProvider`.

---

### `createConnectIframe(session)`

Creates an `<iframe>` pointed at `session.connectUrl`.

```ts
import { createConnectIframe } from "@opendatalabs/connect-js/react";

const iframe = createConnectIframe(session);
iframe.style.height = "720px";
document.getElementById("modal-body")?.appendChild(iframe);
```

---

### `listenForConnectMessages(connectUrl, events)`

Listens for lifecycle events posted by the hosted Connect flow. Returns a cleanup function.

```ts
import { listenForConnectMessages } from "@opendatalabs/connect-js/react";

const unlisten = listenForConnectMessages(session.connectUrl, {
  onReady: () => setLoading(false),
  onSuccess: ({ connectionId }) => handleSuccess(connectionId),
  onExit: () => closeModal(),
});

// Call unlisten() when done
```

| Event | Payload | Description |
| --- | --- | --- |
| `onReady` | — | The hosted flow has loaded |
| `onSuccess` | `{ connectionId }` | The user completed the flow |
| `onExit` | — | The user closed the flow without completing |

<br />

<br />

**Questions? Get in touch!** [hello@opendatalabs.com](mailto:hello@opendatalabs.com)
