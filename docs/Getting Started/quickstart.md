---
title: Quickstart
excerpt: Launch your first hosted Connect session with Open Data Labs.
---
This guide gets you to a working embedded Connect flow using the current production API.

<Callout icon="📘" theme="info">
  **After speed?** Start with the [Quickstart](/docs/quickstart), then get your API key from the <a href="https://dashboard.opendatalabs.com" target="_blank" rel="noreferrer">dashboard</a>.
</Callout>

<Callout icon="🚧" theme="warn">
  **Need help?** [Book a call](https://calendar.google.com/calendar/u/0/appointments/schedules/AcZssZ2rpuc4WGsHiEugwjHcFVX7dGT4edhjEHIHU05iuHElg05-Goi0lVYGCNMxO4RNnt6E-ii69zcP).
</Callout>

## Prerequisites

Before you start, make sure you have:

* an API key from [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com)
* an app in the dashboard with at least one approved domain where you will launch Connect
* a server route in your app where you can safely call the Open Data Labs API

## Step 1: Store your API key on the server

Keep your API key in a server-only environment variable.

```bash
OPENDATALABS_API_KEY=YOUR_OPENDATALABS_API_KEY
```

## Step 2: Get your public app ID

Each dashboard app has a public app ID, for example `odl_app_...`. Use it in your frontend or include it when your server creates sessions for a specific app.

## Step 3: Install the SDK

If you are building a React frontend, install the SDK and use its React helpers to open Connect cleanly.

```bash
npm install @opendatalabs/connect-js
```

## Step 4: Create a Connect session on your server

Call the Open Data Labs API from your backend and create a hosted Connect session for a source.

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

  if (!response.ok) {
    throw new Error(`Failed to create session: ${response.status}`);
  }

  return response.json();
}
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

## Next steps

* Read [How It Works](/docs/how-it-works) for the product model
* Read [Integrating the Connect Flow](/docs/integrating-connect-flow) for implementation details
* Review the [API Reference Overview](/docs/api-reference-overview)

<br />
