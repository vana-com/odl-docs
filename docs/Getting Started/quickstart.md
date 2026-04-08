---
title: Quickstart
excerpt: Launch your first hosted Connect session with Open Data Labs.
---
This guide gets you to a working embedded Connect flow using the current production API.

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
```

## Step 2: Get your public app ID

Each dashboard app has a public app ID, for example `odl_app_...`. Use it in your frontend or include it when your server creates sessions for a specific app.

## Step 3: Install the SDK

If you are building a React frontend, install the SDK and use its React helpers to open Connect cleanly.

```bash
npm install @opendatalabs/connect-js
```

## Step 4: Create a Connect session on your server

Use the SDK's `createClient()` to create a Connect session. The client handles authentication and encryption automatically.

```ts
import { createClient } from "@opendatalabs/connect-js/server";

const odl = createClient({
  apiBaseUrl: "https://api.opendatalabs.com/api/v1",
  apiKey: process.env.OPENDATALABS_API_KEY!,
  secret: process.env.OPENDATALABS_ENCRYPTION_SECRET!,
});

const session = await odl.createConnectSession({
  appId: "odl_app_123",
  source: "instagram",
  scopes: ["read:user_profile", "read:posts", "read:engagement"],
  origin: "https://yourapp.com",
});
```

The `origin` must match one of the approved domains for the selected app exactly.

## Step 5: Open the hosted Connect URL in your frontend

You can handle this yourself, or use the SDK:

```tsx
import { OpenDataLabsProvider } from "@opendatalabs/connect-js/react";
```

Return the session payload from your backend to your frontend and open `connectUrl` in a modal or iframe.

```ts
const session = await createConnectSessionFromYourBackend();

const iframe = document.createElement("iframe");
iframe.src = session.connectUrl;
iframe.style.width = "100%";
iframe.style.height = "720px";
iframe.style.border = "0";

document.getElementById("connect-modal-body")?.appendChild(iframe);
```

## Step 6: Listen for success events

The hosted Connect flow posts lifecycle events back to the parent window.

```ts
window.addEventListener("message", (event) => {
  if (event.origin !== "https://dashboard.opendatalabs.com") {
    return;
  }

  if (event.data?.type === "ready") {
    console.log("Connect is ready");
  }

  if (event.data?.type === "success") {
    console.log("Connection completed", event.data.connectionId);
  }

  if (event.data?.type === "exit") {
    console.log("User closed the flow");
  }
});
```

## Step 7: Retrieve connection data

After a successful connection, fetch the user's data from your server.

```ts
const result = await odl.fetchConnectionResult(connectionId);
// result.data contains the user's connected data
```

## Step 8: Retrieve sources and scopes dynamically

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
