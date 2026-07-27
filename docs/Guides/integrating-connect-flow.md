---
title: Integrating the Connect Flow
excerpt: Implement the current Connect flow in your app.
---
# Integrating the Connect Flow

The Connect flow is the core integration surface for Open Data Labs. Your
backend creates a short-lived session, and your frontend opens Vana in a second
tab while the user approves the request.

## Recommended architecture

Use this split:

- **server**
  - stores `OPENDATALABS_API_KEY` and `OPENDATALABS_ENCRYPTION_SECRET`
  - creates sessions for a specific app
  - checks status and reads approved data
- **frontend**
  - requests a session from your backend
  - opens Vana in a second tab
  - displays the data returned by your backend

In the dashboard, create an app for each integration surface. Domains are
approved per app, not globally for the whole account.

## Create a Connect controller

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

## Create a Connect session

```ts
const session = await odl.createConnectSession({
  redirectUrl: `${process.env.APP_URL!}/connect/return`,
});
```

Return the session to your frontend. Keep the API key on the server.

## Open Vana from the frontend

For React apps, use `useTwoTabConnect`. It opens Vana from the user's click,
then checks the connection status and reads the result through your backend.

```tsx
import { useTwoTabConnect } from "@opendatalabs/connect-js/react";

const connect = useTwoTabConnect({
  createSession: () => fetch("/api/connect/session", { method: "POST" }).then((res) => res.json()),
  getStatus: (connectionId) =>
    fetch(`/api/connect/status?connectionId=${connectionId}`).then((res) => res.json()),
  readResult: (connectionId) =>
    fetch(`/api/connect/data?connectionId=${connectionId}`).then((res) => res.json()),
});
```

Keep the app open while Vana is open.

## Retrieve connection data

```ts
const result = await odl.readAllWhenReady(connectionId);
// result.results contains data keyed by approved scope
```

## Domain approval errors

If the origin is not approved for the selected app, session creation will fail with `origin_not_allowed`.

Fix this by adding the exact embedding origin to the matching app in the Open Data Labs dashboard.

Examples:

* `https://app.example.com`
* `https://staging.example.com`
* `http://localhost:3000`

Do not include paths, query strings, or fragments.

## Current best practices

1. Keep your API key on the server.
2. Choose sources and scopes on the server.
3. Keep the app open until the read completes.
4. Use the live source catalog from `/api/v1/sources` instead of hard-coding roadmap sources.

<br />

<br />

**Questions? Get in touch!** [hello@opendatalabs.com](mailto:hello@opendatalabs.com)
