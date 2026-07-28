---
title: JavaScript SDK
fullscreen: false
hidden: false
---
Install the `@opendatalabs/connect-js` package.

```bash
npm install @opendatalabs/connect-js
```

## Server

Import server code from `@opendatalabs/connect-js/server`. Keep your API key on the server.

### `createConnectController(options)`

```ts
import { createConnectController } from "@opendatalabs/connect-js/server";

const odl = createConnectController({
  apiBaseUrl: process.env.ODL_API_BASE_URL!,
  apiKey: process.env.ODL_API_KEY!,
  appId: process.env.ODL_APP_ID!,
  defaultOrigin: process.env.APP_URL!,
  source: "instagram",
  scopes: ["instagram.profile", "instagram.posts"],
});
```

| Option | Required | Description |
| --- | --- | --- |
| `apiBaseUrl` | Yes | Base URL for the API. |
| `apiKey` | Yes | API key for your server. |
| `appId` | No | Public app ID from the dashboard. |
| `defaultOrigin` | No | Exact approved origin for your app. |
| `source` | Yes | Source identifier, such as `instagram`. |
| `scopes` | Yes | Scopes to request from the source. |

### `odl.createConnectSession(input)`

Creates a session and a Vana Data Connection Request.

```ts
const session = await odl.createConnectSession({
  redirectUrl: "https://app.example.com",
});
```

Return the session to your frontend. Open `session.connectUrl` from the user click.

### `odl.getStatus(connectionId)`

Returns the current session status and approved scopes.

```ts
const status = await odl.getStatus(connectionId);
```

### `odl.readAllWhenReady(connectionId)`

Waits for approval and reads every requested scope.

```ts
const result = await odl.readAllWhenReady(connectionId);
// result.results contains data keyed by scope
```

## React

Import React helpers from `@opendatalabs/connect-js/react`.

### `useTwoTabConnect(options)`

Use this hook to open Vana from a click handler. It checks the session status and reads the result through your server routes.

```tsx
import { useTwoTabConnect } from "@opendatalabs/connect-js/react";

const connect = useTwoTabConnect({
  openingUrl: process.env.NEXT_PUBLIC_VANA_CONNECT_OPENING_URL,
  createSession: async () => {
    const response = await fetch("/api/connect/session", { method: "POST" });
    if (!response.ok) throw new Error("Failed to create Connect session");
    return response.json();
  },
  getStatus: async (connectionId) => {
    const response = await fetch(`/api/connect/status?connectionId=${encodeURIComponent(connectionId)}`);
    if (!response.ok) throw new Error("Failed to fetch Connect status");
    return response.json();
  },
  readResult: async (connectionId) => {
    const response = await fetch(`/api/connect/data?connectionId=${encodeURIComponent(connectionId)}`);
    if (!response.ok) throw new Error("Failed to read Connect data");
    return response.json();
  },
});
```

Call `connect.start()` from the user click. Keep the original tab open until the read completes.

Questions, feature requests, or support: hello@opendatalabs.com
