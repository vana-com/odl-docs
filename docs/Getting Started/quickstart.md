---
title: Quickstart
excerpt: Launch your first Connect flow with Open Data Labs.
---
This guide gets you to a working Connect flow using the current production API.

<Callout icon="📘" theme="info">
  Get your **API key** from the <a href="https://dashboard.opendatalabs.com" target="_blank" rel="noreferrer">OpenDataLabs Dashboard</a>.
</Callout>

<Callout icon="🚧" theme="warn">
  **Need help?** [Book a call](https://calendar.google.com/calendar/u/0/appointments/schedules/AcZssZ2rpuc4WGsHiEugwjHcFVX7dGT4edhjEHIHU05iuHElg05-Goi0lVYGCNMxO4RNnt6E-ii69zcP).
</Callout>

## Supported data sources

Instagram, iCloud Notes, GitHub, and Oura Ring are available today. Spotify and others are coming soon. See [Data Sources](/docs/data-sources) for full schemas and available scopes.

## Prerequisites

Before you start, make sure you have:

* an API key from [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com)
* an app in the dashboard with at least one approved domain where you will launch Connect
* a data encryption secret from your app settings in the dashboard
* a server route in your app where you can safely call the Open Data Labs API

## Step 1: Store your API key and secret on the server

Keep your API key and encryption secret in server-only environment variables.

```bash
OPENDATALABS_API_KEY=YOUR_OPENDATALABS_API_KEY
OPENDATALABS_ENCRYPTION_SECRET=YOUR_OPENDATALABS_ENCRYPTION_SECRET
ODL_APP_ID=odl_app_123
APP_URL=https://yourapp.com
```

## Step 2: Get your public app ID

Each dashboard app has a public app ID, for example `odl_app_...`. Use it in your frontend or include it when your server creates sessions for a specific app.

## Step 3: Install the SDK

If you are building a React frontend, install the SDK and use its React helpers to open Connect cleanly.

```bash
npm install @opendatalabs/connect-js
```

## Step 4: Create Connect routes on your server

Choose the source and scopes on the server.

```ts
// lib/odl.ts
import { createConnectController } from "@opendatalabs/connect-js/server";

export const odl = createConnectController({
  apiBaseUrl: "https://api.opendatalabs.com/api/v1",
  apiKey: process.env.OPENDATALABS_API_KEY!,
  appId: process.env.ODL_APP_ID!,
  defaultOrigin: process.env.APP_URL!,
  source: "instagram",
  scopes: ["read:profile", "read:posts"],
});
```

```ts
// app/api/connect/session/route.ts
import { odl } from "@/lib/odl";

export async function POST() {
  return Response.json(
    await odl.createConnectSession({
      redirectUrl: `${process.env.APP_URL!}/connect/return`,
    })
  );
}
```

```ts
// app/api/connect/status/route.ts
import { odl } from "@/lib/odl";

export async function GET(request: Request) {
  const connectionId = new URL(request.url).searchParams.get("connectionId");
  if (!connectionId) {
    return Response.json({ error: "Missing connectionId" }, { status: 400 });
  }
  return Response.json(await odl.getStatus(connectionId));
}
```

```ts
// app/api/connect/data/route.ts
import { odl } from "@/lib/odl";

export async function GET(request: Request) {
  const connectionId = new URL(request.url).searchParams.get("connectionId");
  if (!connectionId) {
    return Response.json({ error: "Missing connectionId" }, { status: 400 });
  }
  return Response.json(await odl.readAllWhenReady(connectionId));
}
```

## Step 5: Open Connect in a second tab

Connect runs in Vana. Keep your app open while the user approves the request.

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

<button disabled={connect.state.type !== "idle"} onClick={() => connect.start()}>
  Connect Instagram
</button>;
```

For development or staging, set `openingUrl` to the matching Vana app, for
example `https://app-dev.vana.org/connect/opening`.

## Step 6: Retrieve connection data

`readAllWhenReady()` returns `result.results`, keyed by approved scope.

```ts
const result = await odl.readAllWhenReady(connectionId);
// result.results contains data keyed by scope
```

## Step 7: Retrieve sources and scopes dynamically

You can fetch the current source catalog from the API instead of hard-coding it.

```bash
curl https://api.opendatalabs.com/api/v1/sources \
  -H "Authorization: Bearer $OPENDATALABS_API_KEY"
```

## Current production sources

Today, the available production sources are:

* Instagram
* iCloud Notes
* GitHub
* Oura Ring

## Next steps

* Read [How It Works](/docs/how-it-works) for the product model
* Read [Integrating the Connect Flow](/docs/integrating-connect-flow) for implementation details
* Review the [API Reference Overview](/docs/api-reference-overview)

<br />

<br />

**Questions? Get in touch!** [hello@opendatalabs.com](mailto:hello@opendatalabs.com)
