---
title: Quickstart
excerpt: Add a complete Connect flow to a Next.js app.
---
This guide adds a complete Connect flow to a Next.js app. Your app opens Vana in a second tab, then displays the approved data in the original tab.

## Before you start

You need an Open Data Labs account and a Next.js app. In the dashboard, create an app. Add `http://localhost:3000` to its Embed Origins. Copy its API key and public app ID.

## Step 1: Create the app

Create a Next.js app, then install the Connect SDK.

```bash
npx create-next-app@latest my-odl-app --ts --app --no-src-dir --import-alias "@/*" --use-npm --yes
cd my-odl-app
npm install @opendatalabs/connect-js
```

## Step 2: Add environment variables

Create `.env.local` in the project root.

```bash
ODL_API_BASE_URL=https://api.opendatalabs.com/api/v1
ODL_API_KEY=your_api_key
ODL_APP_ID=odl_app_123
APP_URL=http://localhost:3000
NEXT_PUBLIC_VANA_CONNECT_OPENING_URL=https://app.vana.org/connect/opening
```

Keep `ODL_API_KEY` on the server. Do not add it to a `NEXT_PUBLIC_` variable.

## Step 3: Create the Connect controller

Create `lib/odl.ts`.

```ts
import { createConnectController } from "@opendatalabs/connect-js/server";

export const odl = createConnectController({
  apiBaseUrl: process.env.ODL_API_BASE_URL!,
  apiKey: process.env.ODL_API_KEY!,
  appId: process.env.ODL_APP_ID!,
  defaultOrigin: process.env.APP_URL!,
  source: "instagram",
  scopes: ["read:profile", "read:posts"],
});
```

This example requests Instagram profile and post data. Choose the source and scopes on your server.

## Step 4: Create the server routes

Create these routes in your app.

```ts
// app/api/connect/session/route.ts
import { odl } from "@/lib/odl";

export async function POST() {
  return Response.json(
    await odl.createConnectSession({
      redirectUrl: process.env.APP_URL!,
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

## Step 5: Add the Connect page

Replace `app/page.tsx` with this client component.

```tsx
"use client";

import { useState } from "react";
import { useTwoTabConnect } from "@opendatalabs/connect-js/react";

async function getJson(url: string, init?: RequestInit) {
  const response = await fetch(url, init);
  if (!response.ok) {
    throw new Error(await response.text());
  }
  return response.json();
}

export default function Home() {
  const [result, setResult] = useState<unknown>(null);
  const [error, setError] = useState<string | null>(null);
  const connect = useTwoTabConnect({
    openingUrl: process.env.NEXT_PUBLIC_VANA_CONNECT_OPENING_URL,
    createSession: () => getJson("/api/connect/session", { method: "POST" }),
    getStatus: (connectionId) =>
      getJson(`/api/connect/status?connectionId=${encodeURIComponent(connectionId)}`),
    readResult: (connectionId) =>
      getJson(`/api/connect/data?connectionId=${encodeURIComponent(connectionId)}`),
  });

  async function start() {
    setError(null);
    try {
      setResult(await connect.start());
    } catch (cause) {
      setError(cause instanceof Error ? cause.message : "Connect failed.");
    }
  }

  return (
    <main>
      <button
        disabled={connect.state.type !== "idle"}
        onClick={start}
        type="button"
      >
        {connect.state.type === "idle" ? "Connect Instagram" : "Connecting..."}
      </button>
      {error ? <p>{error}</p> : null}
      {result ? <pre>{JSON.stringify(result, null, 2)}</pre> : null}
    </main>
  );
}
```

## Step 6: Run the flow

Start the app.

```bash
npm run dev
```

Open `http://localhost:3000` and select **Connect Instagram**. Vana opens in a second tab. Complete the request there. Return to the original tab.

You are done when the original tab displays JSON for the approved scopes.

Questions, feature requests, or support: hello@opendatalabs.com
