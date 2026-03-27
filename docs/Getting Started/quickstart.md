---
title: Quickstart
excerpt: Get started with Context Gateway in 5 minutes
---

# Quickstart

Get up and running with Context Gateway in just a few steps.

> **Early Access**: Context Gateway is currently in early access. Sign up for access at [contextgateway.com](https://contextgateway.com) before you begin.

## Prerequisites

Before you start, make sure you have:

- An API key from Context Gateway (available in your [dashboard](https://dashboard.contextgateway.com))
- A web application built with Node.js/Express or your preferred framework
- Basic familiarity with REST APIs and OAuth-style flows

## Step 1: Install the SDK

Install the Context Gateway JavaScript SDK via npm:

```bash
npm install @opendatalabs/context-gateway
```

## Step 2: Initialize the Client

Initialize the SDK in your application:

```javascript
import { ContextGatewayClient } from '@opendatalabs/context-gateway';

const client = new ContextGatewayClient({
  apiKey: process.env.CONTEXT_GATEWAY_API_KEY,
});
```

## Step 3: Create a Connect URL

When you want to let a user connect a data source, create a connect URL and redirect them:

```javascript
const connectUrl = client.createConnectUrl({
  userId: 'user_123', // Your internal user identifier
  source: 'spotify', // The data source (spotify, github, netflix, etc.)
  scopes: ['read:user_profile', 'read:playlists'], // Data types to access
  redirectUrl: 'https://yourapp.com/auth/callback', // Where to send user after auth
});

// Redirect user to the connect URL
res.redirect(connectUrl);
```

## Step 4: Handle the Callback

After the user authenticates, they'll be redirected back to your callback URL with a `connectionId`. Handle this in your server:

```javascript
app.get('/auth/callback', async (req, res) => {
  const { connectionId, error, error_description } = req.query;

  if (error) {
    console.error(`Connect failed: ${error} - ${error_description}`);
    return res.redirect('/connect?error=true');
  }

  // Store the connectionId in your database, associated with the user
  await db.users.update(req.user.id, {
    contextGatewayConnectionId: connectionId,
  });

  res.redirect('/dashboard?connected=true');
});
```

## Step 5: Query User Data

Now you can query the user's data via the Context Gateway API:

```javascript
const data = await client.query({
  connectionId: 'conn_abc123def456', // From the callback
  query: 'SELECT name, email, profile_image FROM user_profile',
});

console.log(data);
// Output:
// {
//   name: "John Doe",
//   email: "john@example.com",
//   profile_image: "https://..."
// }
```

## Next Steps

- Learn about [Personal Servers](/docs/personal-servers) and how data is stored
- Understand [data ownership](/docs/data-ownership) and trust boundaries
- Read the full [API reference](/docs/api-reference-overview)
- Explore [integrating the connect flow](/docs/integrating-connect-flow) with advanced options
- Check out [querying user data](/docs/querying-user-data) for complex queries

## Common First Steps

**Troubleshooting connection issues?**
See [Handling Consent](/docs/handling-consent) for debugging connection status.

**Need to request additional scopes?**
Users can grant additional permissions at any time. See [Requesting Additional Scopes](/docs/handling-consent) for implementation details.

**Want to support revocation?**
Learn how to [handle consent revocation](/docs/handling-consent) gracefully in your app.
