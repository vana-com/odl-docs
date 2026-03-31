---
title: Quickstart
excerpt: Launch your first hosted Connect session with Open Data Labs.
---

# Quickstart

This guide gets you to a working embedded Connect flow using the current production API.

## Prerequisites

Before you start, make sure you have:

- an API key from [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com)
- an approved domain in the dashboard for the app where you will launch Connect
- a server route in your app where you can safely call the Open Data Labs API

## Step 1: Store your API key on the server

Keep your API key in a server-only environment variable.

```bash
OPENDATALABS_API_KEY=<<apiKey>>
```

## Step 2: Create a Connect session on your server

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

The `origin` must match one of your approved domains exactly.

## Step 3: Open the hosted Connect URL in your frontend

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

## Step 4: Listen for success events

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

## Step 5: Retrieve sources and scopes dynamically

You can fetch the current source catalog from the API instead of hard-coding it.

```bash
curl https://api.opendatalabs.com/api/v1/sources \
  -H "Authorization: Bearer <<apiKey>>"
```

## Current production sources

Today, the available production sources are:

- Instagram
- iCloud Notes

## Next steps

- Read [How It Works](/docs/how-it-works) for the product model
- Read [Integrating the Connect Flow](/docs/integrating-connect-flow) for implementation details
- Review the [API Reference Overview](/docs/api-reference-overview)
