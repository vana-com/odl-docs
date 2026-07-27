---
title: JavaScript SDK
fullscreen: false
hidden: false
---
Start by installing the `@opendatalabs/connect-js` package.

```bash
npm install @opendatalabs/connect-js
```

***

## Server

Import from `@opendatalabs/connect-js/server`. Run server-side only — never expose your API key or encryption secret to the browser.

### `createConnectController(options)`

```ts
import { createConnectController } from "@opendatalabs/connect-js/server";

const odl = createConnectController({
  apiBaseUrl: "https://api.opendatalabs.com/api/v1",
  apiKey: process.env.OPENDATALABS_API_KEY!,
  appId: process.env.ODL_APP_ID!,
  defaultOrigin: process.env.APP_URL!,
  source: "instagram",
  scopes: ["read:profile", "read:posts"],
});
```

| Option          | Required | Description                                |
| --------------- | -------- | ------------------------------------------ |
| `apiBaseUrl`    | Yes      | Base URL for the API                       |
| `apiKey`        | Yes      | Your server API key                        |
| `appId`         | No       | Public app ID from the dashboard           |
| `defaultOrigin` | No       | Exact approved origin for your app         |
| `source`        | Yes      | Source identifier, for example `instagram` |
| `scopes`        | Yes      | Scopes your server may request             |

The controller creates Connect sessions, checks their status, and reads the
scopes approved by the user.

***

### `odl.createConnectSession(input)`

Creates a Connect session and Vana Data Connection Request.

```ts
const session = await odl.createConnectSession({
  redirectUrl: `${process.env.APP_URL!}/connect/return`,
});
```

Open `session.connectUrl` in a new tab.

***

### `odl.getStatus(connectionId)`

Returns the current connection status and approved scopes.

```ts
const status = await odl.getStatus(connectionId);
```

***

### `odl.readAllWhenReady(connectionId)`

Waits for approval and reads each approved scope from the user's Personal
Server.

```ts
const result = await odl.readAllWhenReady(connectionId);
// result.results contains data keyed by scope
```

***

## React

Import from `@opendatalabs/connect-js/react`. Run client-side.

### `useTwoTabConnect(options)`

Use this hook to open Vana from a click handler, check status, and read data
after approval.

```tsx
import { useTwoTabConnect } from "@opendatalabs/connect-js/react";

const connect = useTwoTabConnect({
  createSession: async () => {
    const response = await fetch("/api/connect/session", { method: "POST" });
    if (!response.ok) throw new Error("Failed to create Connect session");
    return response.json();
  },
  getStatus: async (connectionId) => {
    const response = await fetch(`/api/connect/status?connectionId=${connectionId}`);
    if (!response.ok) throw new Error("Failed to fetch Connect status");
    return response.json();
  },
  readResult: async (connectionId) => {
    const response = await fetch(`/api/connect/data?connectionId=${connectionId}`);
    if (!response.ok) throw new Error("Failed to read Connect data");
    return response.json();
  },
});
```

`connect.start()` opens the Vana flow in a new tab. Keep the app open until
the read completes.

<br />

<br />

**Questions? Get in touch!** [hello@opendatalabs.com](mailto:hello@opendatalabs.com)
