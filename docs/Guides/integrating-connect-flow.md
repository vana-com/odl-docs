---
title: Integrating the Connect Flow
excerpt: Implement the authentication and connection flow for your users
---

# Integrating the Connect Flow

The connect flow is how users authorize your application to access their data from external sources. This guide walks you through implementation.

## Create a Connect URL

When a user wants to connect a data source, create a connect URL with the necessary parameters:

```javascript
const connectUrl = client.createConnectUrl({
  userId: 'user_123',           // Your internal user ID
  source: 'spotify',             // Data source identifier
  scopes: ['read:user_profile', 'read:playlists'], // Requested permissions
  redirectUrl: 'https://yourapp.com/auth/callback', // Where user returns
});
```

### Connect URL Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `userId` | string | Yes | Your internal user identifier. Used to associate the connection with a user account in your system |
| `source` | string | Yes | The data source to connect: `spotify`, `github`, `netflix`, `google_calendar`, etc. |
| `scopes` | array | Yes | List of scopes to request. Only the user can access data within these scopes |
| `redirectUrl` | string | Yes | URL where the user will be sent after authentication. Must be registered in your Context Gateway dashboard |
| `state` | string | No | Optional state parameter for additional security. Context Gateway will return this unchanged |
| `existingConnectionId` | string | No | If requesting additional scopes on an existing connection, pass the connection ID |
| `connectionName` | string | No | Optional friendly name for this connection (e.g., "Primary Spotify Account") |

## Redirect User to Connect

Send the user to the connect URL. This can be done via a button click:

```javascript
// Express.js example
app.get('/connect', (req, res) => {
  const source = req.query.source || 'spotify';

  const connectUrl = client.createConnectUrl({
    userId: req.user.id,
    source: source,
    scopes: SCOPES_BY_SOURCE[source],
    redirectUrl: `${process.env.BASE_URL}/auth/callback`,
    state: req.sessionID, // Optional but recommended
  });

  res.redirect(connectUrl);
});
```

In your frontend, create a button that links to this endpoint:

```html
<a href="/connect?source=spotify" class="btn btn-primary">
  Connect Spotify
</a>
```

## Handle the Callback

Context Gateway will redirect the user back to your callback URL with either a `connectionId` or an error. Handle both cases:

```javascript
app.get('/auth/callback', async (req, res) => {
  const { connectionId, state, error, error_description } = req.query;

  // Validate state parameter if you used one
  if (state !== req.sessionID) {
    console.error('State mismatch: potential CSRF attack');
    return res.status(400).json({ error: 'Invalid state' });
  }

  // Handle errors
  if (error) {
    return handleConnectError(error, error_description, res);
  }

  // Success: store the connection
  try {
    await db.users.update(req.user.id, {
      contextGatewayConnectionId: connectionId,
      connectedAt: new Date(),
      connectionStatus: 'active',
    });

    // Log the connection for audit purposes
    console.log(`User ${req.user.id} connected: ${connectionId}`);

    // Redirect to success page
    res.redirect('/dashboard?connected=true');
  } catch (err) {
    console.error('Failed to save connection:', err);
    res.status(500).json({ error: 'Failed to save connection' });
  }
});
```

## Error Handling

Context Gateway returns error codes when something goes wrong during the connect flow. Handle each case appropriately:

### Connect Flow Error Codes

| Error Code | Description | User Action | App Response |
|-----------|-------------|------------|--------------|
| `user_denied` | User clicked "Deny" on the permissions dialog | None (intentional) | Allow user to try again; don't show error message |
| `source_error` | The source (Spotify, GitHub, etc.) rejected the request | Retry | Show message: "Temporarily unavailable. Please try again." |
| `provisioning_failed` | Personal Server could not be created | Contact support | Show message: "A system error occurred. Please try again or contact support." |
| `timeout` | The authentication took too long | Retry | Show message: "Authentication timed out. Please try again." |
| `invalid_scope` | A requested scope is not available for this source | Check scope list | Verify scopes are valid for the source |
| `redirect_uri_mismatch` | Callback URL doesn't match registered URL | Check settings | Verify callback URL in dashboard |

### Example Error Handler

```javascript
function handleConnectError(error, description, res) {
  const errorMessages = {
    user_denied: {
      userMessage: 'You declined to connect. No data was accessed.',
      logError: false,
    },
    source_error: {
      userMessage: 'The service is temporarily unavailable. Please try again.',
      logError: true,
    },
    provisioning_failed: {
      userMessage: 'A system error occurred. Please try again or contact support.',
      logError: true,
    },
    timeout: {
      userMessage: 'Authentication timed out. Please try again.',
      logError: false,
    },
  };

  const errorConfig = errorMessages[error] || {
    userMessage: 'An unexpected error occurred.',
    logError: true,
  };

  if (errorConfig.logError) {
    console.error(`Connect error: ${error} - ${description}`);
  }

  res.redirect(`/connect-error?error=${error}&message=${errorConfig.userMessage}`);
}
```

## Implementing UI Feedback

Provide clear feedback to users during the connect flow:

```javascript
app.get('/connect-flow', (req, res) => {
  const sources = {
    spotify: {
      name: 'Spotify',
      icon: '🎵',
      description: 'Access your music, playlists, and listening history',
    },
    github: {
      name: 'GitHub',
      icon: '🐙',
      description: 'Access your repositories and code',
    },
  };

  res.render('connect', {
    sources,
    currentConnections: req.user.connections,
  });
});
```

## Updating an Existing Connection

If a user wants to reconnect or grant additional scopes:

```javascript
app.get('/reconnect', async (req, res) => {
  const existingConnection = await db.connections.find({
    userId: req.user.id,
    source: 'spotify',
  });

  const connectUrl = client.createConnectUrl({
    userId: req.user.id,
    source: 'spotify',
    scopes: ['read:user_profile', 'read:playlists', 'read:playback_history'],
    redirectUrl: `${process.env.BASE_URL}/auth/callback`,
    existingConnectionId: existingConnection?.id,
  });

  res.redirect(connectUrl);
});
```

## Security Best Practices

1. **Always validate the state parameter**: Prevents CSRF attacks
2. **Use HTTPS**: Protect the callback URL and connection ID in transit
3. **Store connection IDs securely**: Treat them like access tokens
4. **Don't expose connection IDs in URLs**: Pass them via POST or secure session storage
5. **Log connections**: Maintain an audit trail for security reviews
6. **Implement rate limiting**: Prevent abuse of the connect flow
7. **Validate redirect URLs**: Only allow registered callback URLs
