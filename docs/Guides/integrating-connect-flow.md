---
title: Integrating the Connect Flow
excerpt: Implement the Connect flow in your app.
---
The Connect flow has a server part and a frontend part. Your server creates the request and reads approved data. Your frontend opens Vana in a second tab.

## Recommended architecture

| Layer | Responsibility |
| --- | --- |
| Server | Stores the API key. Chooses the source and scopes. Creates sessions and reads data. |
| Frontend | Opens Vana from a user click. Shows the data that your server returns. |

Create an app for each integration surface in the dashboard. Add the exact origin for that app, such as `https://app.example.com` or `http://localhost:3000`.

## Create a session

Use `createConnectController` on your server. It creates a Vana Data Connection Request when you call `createConnectSession`.

```ts
const session = await odl.createConnectSession({
  redirectUrl: "https://app.example.com",
});
```

Return the session from your server route. Keep the API key on the server.

## Open Vana from the frontend

For React apps, use `useTwoTabConnect`. It opens Vana from the user click, checks the session status, and requests the approved data through your server routes.

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

Keep the app open until the read completes. The user can finish consent in Vana while the original tab waits for the result.

## Read approved data

Call `readAllWhenReady` from your server route. It waits for the connection to become ready, reads each requested scope, and confirms that the request completed.

```ts
const result = await odl.readAllWhenReady(connectionId);
// result.results contains data keyed by scope
```

## Domain approval errors

If the origin is not approved for the selected app, session creation returns `origin_not_allowed`.

Add the exact origin in the dashboard. Do not include a path, query string, or fragment.

Questions, feature requests, or support: hello@opendatalabs.com
