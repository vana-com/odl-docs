---
title: Quickstart
excerpt: Get up and running with Context Gateway in a few steps.
---
Get up and running with Context Gateway in a few steps.

## Prerequisites

- A web application built with Node.js/Express or your preferred framework
- Basic familiarity with REST APIs and OAuth-style flows

## Step 1: Get your API key

Create an account at [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com). Your API key is available immediately after sign-up under Settings. Store it as an environment variable. It should never appear in client-side code.

```bash
CONTEXT_GATEWAY_API_KEY=your_api_key_here
```

## Step 2: Install the SDK

```bash
npm install @opendatalabs/context-gateway
```

## Step 3: Initialise the client

```javascript
import { ContextGatewayClient } from '@opendatalabs/context-gateway';

const client = new ContextGatewayClient({
  apiKey: process.env.CONTEXT_GATEWAY_API_KEY,
});
```

## Step 4: Create a Connect URL

When you want a user to connect a data source, create a Connect URL and redirect them. The example below uses Instagram, which is available in the production API.

```javascript
const connectUrl = client.createConnectUrl({
  userId: 'user_123',
  source: 'instagram',
  scopes: ['read:user_profile', 'read:posts'],
  redirectUrl: 'https://yourapp.com/auth/callback',
});

res.redirect(connectUrl);
```

## Step 5: Handle the callback

After the user authenticates, they are redirected to your callback URL with a `connectionId`.

```javascript
app.get('/auth/callback', async (req, res) => {
  const { connectionId, error, error_description } = req.query;

  if (error) {
    console.error(\`Connect failed: \${error} - \${error_description}\`);
    return res.redirect('/connect?error=true');
  }

  await db.users.update(req.user.id, {
    contextGatewayConnectionId: connectionId,
  });

  res.redirect('/dashboard?connected=true');
});
```

## Step 6: Query user data

```javascript
const data = await client.query({
  connectionId: 'conn_abc123def456',
  query: 'SELECT name, username, bio, follower_count FROM user_profile',
});
```

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

Questions, feature requests, or support: hello@opendatalabs.com