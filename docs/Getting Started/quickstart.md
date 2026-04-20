---
title: Quickstart
excerpt: Get up and running with Context Gateway in a few steps.
---
Get up and running with Context Gateway in a few steps.

## Prerequisites

- A web application built with Node.js/Express or your preferred framework
- Basic familiarity with REST APIs and OAuth-style flows

## Step 1: Get your API key

Create an account at [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com). Generate an API key under Settings and store it as an environment variable. To fetch connection results later, also generate a data encryption secret and store it as `OPENDATALABS_ENCRYPTION_SECRET`. Keep both values out of client-side code.

```bash
OPENDATALABS_API_KEY=your_api_key_here
OPENDATALABS_ENCRYPTION_SECRET=your_encryption_secret_here
```

## Step 2: Install the SDK

```bash
npm install @opendatalabs/connect-js
```

## Step 3: Initialise the client

```javascript
import { createClient } from '@opendatalabs/connect-js/server';

const client = createClient({
  apiBaseUrl: 'https://api.opendatalabs.com/api/v1',
  apiKey: process.env.OPENDATALABS_API_KEY,
  secret: process.env.OPENDATALABS_ENCRYPTION_SECRET,
});
```

## Step 4: Create a Connect session

When you want a user to connect a data source, create a hosted Connect session and redirect them to the returned `connectUrl`. The example below uses Instagram, which is available in the production API.

```javascript
const session = await client.createConnectSession({
  appId: 'odl_app_123',
  source: 'instagram',
  scopes: ['read:user_profile', 'read:posts'],
  origin: 'https://yourapp.com',
  redirectUrl: 'https://yourapp.com/auth/callback',
});

res.redirect(session.connectUrl);
```

## Step 5: Handle the callback

After the user authenticates, they are redirected to your callback URL with a `connectionId`.

```javascript
app.get('/auth/callback', async (req, res) => {
  const { connectionId, error, error_description } = req.query;

  if (error) {
    console.error(`Connect failed: ${error} - ${error_description}`);
    return res.redirect('/connect?error=true');
  }

  await db.users.update(req.user.id, {
    contextGatewayConnectionId: connectionId,
  });

  res.redirect('/dashboard?connected=true');
});
```

## Step 6: Fetch the connection result

```javascript
const result = await client.fetchConnectionResult(connectionId);
const data = result.data;
```

Connection data is single-use. Persist `result.data` if you need to access it again later.

## Example response

```json
{
  "source": "instagram",
  "retrieved_at": "2026-04-05T09:15:00Z",
  "data": {
    "name": "Jane Smith",
    "username": "janesmith",
    "bio": "Designer based in London.",
    "follower_count": 3820,
    "following_count": 412,
    "post_count": 94,
    "profile_image": "https://cdn.example.com/profile/janesmith.jpg",
    "is_verified": false,
    "account_type": "personal"
  }
}
```

Full schema documentation per source — including all available fields, types, and nullability — is in the API Reference at [dev.opendatalabs.com/reference](https://dev.opendatalabs.com/reference)

## Next steps

- Read [Personal Servers](/docs/personal-servers) to understand how data is stored
- Read [Consent and Access](/docs/consent-and-access) for scope handling and revocation
- Read the full API Reference at [dev.opendatalabs.com/reference](https://dev.opendatalabs.com/reference)
- Read [Integrating the Connect Flow](/docs/integrating-the-connect-flow) for advanced options

## Get started with AI

Copy this prompt into your AI coding assistant if you want help scaffolding an integration:

```
Help me add Open Data Labs Connect to my app.

Use these docs as the source of truth:
- https://dev.opendatalabs.com/docs/quickstart
- https://dev.opendatalabs.com/docs/javascript-sdk

Implement:
1. npm install @opendatalabs/connect-js
2. A server-side createClient(...) using OPENDATALABS_API_KEY and OPENDATALABS_ENCRYPTION_SECRET
3. A backend route that calls createConnectSession(...)
4. A callback handler that stores the returned connectionId
5. A server-side call to fetchConnectionResult(connectionId)

Keep the API key and encryption secret off the client.
```

Questions, feature requests, or support: hello@opendatalabs.com
