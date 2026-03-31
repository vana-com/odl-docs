---
title: Integrating the Connect Flow
excerpt: Implement the current hosted Connect session flow in your app.
---

# Integrating the Connect Flow

The Connect flow is the core integration surface for Open Data Labs. Your backend creates a short-lived session, and your frontend embeds the hosted Connect URL.

## Recommended architecture

Use this split:

- **server**
  - stores `OPENDATALABS_API_KEY`
  - creates Connect sessions for a specific app
- **frontend**
  - uses the public app ID for the integration surface
  - requests a session from your backend
  - opens the returned `connectUrl`
  - handles `postMessage` events from the hosted flow

In the dashboard, create an app for each integration surface where you embed Connect. Domains are approved per app, not globally for the whole account.

## SDK option

For React apps, you can use `@opendatalabs/connect-js` and keep your session-creation logic on your backend:

```tsx
import { OpenDataLabsProvider, useConnect } from "@opendatalabs/connect-js/react";
```

## Create a Connect session

```ts
export async function createConnectSession() {
  const response = await fetch("https://api.opendatalabs.com/api/v1/connect/sessions", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.OPENDATALABS_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      appId: "odl_app_123",
      source: "instagram",
      scopes: ["read:user_profile", "read:posts", "read:engagement"],
      origin: "https://yourapp.com",
    }),
  });

  const data = await response.json();
  if (!response.ok) {
    throw new Error(data.message || "Failed to create Connect session");
  }

  return data;
}
```

## Request fields

| Field | Required | Description |
| --- | --- | --- |
| `source` | Yes | Source identifier like `instagram` or `icloud_notes` |
| `scopes` | No | Scopes to request for the source |
| `appId` | No | Public app ID like `odl_app_...`; if omitted, Open Data Labs uses the default app for the account |
| `origin` | Yes | Exact embedding origin, for example `https://yourapp.com` |
| `redirectUrl` | No | Optional redirect URL for legacy flows |

## Open the flow in a modal

```ts
const session = await fetch("/api/connect-session").then((res) => res.json());

const iframe = document.createElement("iframe");
iframe.src = session.connectUrl;
iframe.style.width = "100%";
iframe.style.height = "720px";
iframe.style.border = "0";

modalBody.appendChild(iframe);
modal.open();
```

## Handle lifecycle events

The hosted flow sends events to the embedding window.

```ts
window.addEventListener("message", (event) => {
  if (event.origin !== "https://dashboard.opendatalabs.com") {
    return;
  }

  switch (event.data?.type) {
    case "ready":
      console.log("Connect ready");
      break;
    case "success":
      console.log("Connection completed", event.data.connectionId);
      break;
    case "exit":
      console.log("User exited Connect");
      break;
  }
});
```

## Domain approval errors

If the origin is not approved for the selected app, session creation will fail with `origin_not_allowed`.

Fix this by adding the exact embedding origin to the matching app in the Open Data Labs dashboard.

Examples:

- `https://app.example.com`
- `https://staging.example.com`
- `http://localhost:3000`

Do not include paths, query strings, or fragments.

## Current best practices

1. Keep your API key on the server.
2. Treat the hosted Connect session as short-lived.
3. Validate your frontend event origin before acting on messages.
4. Use the live source catalog from `/api/v1/sources` instead of hard-coding roadmap sources.
