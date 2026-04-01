---
title: Handling Consent
excerpt: Check connection status, handle revocation, and request additional scopes
---

# Handling Consent

User consent is central to Context Gateway. This guide covers checking connection status, handling revocation, and requesting additional permissions.

## Checking Connection Status

Before querying user data, always verify that the connection is still valid. Users can revoke access at any time:

```javascript
const connection = await client.getConnection(connectionId);

console.log(connection);
// {
//   id: 'conn_abc123def456',
//   userId: 'user_123',
//   source: 'spotify',
//   isValid: true,
//   scopes: ['read:user_profile', 'read:playlists'],
//   connectedAt: '2026-03-27T10:30:00Z',
//   lastUsed: '2026-03-27T14:45:00Z',
//   lastSyncAt: '2026-03-27T15:00:00Z',
// }
```

### Connection Status Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | The connection ID |
| `userId` | string | Associated user ID |
| `source` | string | Data source (spotify, github, etc.) |
| `isValid` | boolean | Whether the connection is currently valid |
| `scopes` | array | Granted scopes for this connection |
| `connectedAt` | timestamp | When the connection was established |
| `lastUsed` | timestamp | When this connection was last queried |
| `lastSyncAt` | timestamp | When data was last synced from the source |
| `expiresAt` | timestamp | When the connection will expire (if applicable) |

## Handling Revocation

When a user revokes access, queries immediately fail with `consent_revoked`. Always handle this error gracefully:

```javascript
app.get('/user/playlists', async (req, res) => {
  const connectionId = req.user.contextGatewayConnectionId;

  if (!connectionId) {
    return res.status(400).json({
      error: 'Not connected',
      message: 'Please connect your Spotify account first',
      action: '/connect?source=spotify',
    });
  }

  try {
    const data = await client.query({
      connectionId,
      query: 'SELECT id, name, tracks_count FROM playlists',
    });
    return res.json(data.data);
  } catch (error) {
    if (error.code === 'consent_revoked') {
      // User revoked access
      // Clear the connection from your database
      await db.users.update(req.user.id, {
        contextGatewayConnectionId: null,
        connectionRevokedAt: new Date(),
      });

      return res.status(403).json({
        error: 'Access revoked',
        message: 'Your connection has been revoked. Please reconnect to continue.',
        action: '/connect?source=spotify',
      });
    }

    // Handle other errors
    console.error('Query error:', error);
    return res.status(500).json({ error: 'Failed to fetch playlists' });
  }
});
```

## Proactive Revocation Detection

Check connection status periodically to catch revoked access before queries fail:

```javascript
// Middleware to check connection health before each request
const checkConnectionMiddleware = async (req, res, next) => {
  if (!req.user?.contextGatewayConnectionId) {
    return next();
  }

  try {
    const connection = await client.getConnection(
      req.user.contextGatewayConnectionId
    );

    if (!connection.isValid) {
      // Connection is revoked but we detected it proactively
      await db.users.update(req.user.id, {
        contextGatewayConnectionId: null,
      });

      return res.status(403).json({
        error: 'Connection revoked',
        message: 'Your data access has been revoked',
      });
    }

    // Attach connection info to request
    req.connection = connection;
    next();
  } catch (error) {
    console.error('Failed to check connection:', error);
    next(); // Allow request to continue; let query handle the error
  }
};

app.use(checkConnectionMiddleware);
```

## Requesting Additional Scopes

If your application needs additional permissions, you can request them at any time:

```javascript
app.post('/request-additional-scopes', async (req, res) => {
  const newScopes = req.body.scopes; // e.g., ['read:playback_history']

  const connectUrl = client.createConnectUrl({
    userId: req.user.id,
    source: 'spotify',
    scopes: newScopes,
    existingConnectionId: req.user.contextGatewayConnectionId,
      redirectUrl: `${process.env.BASE_URL}/connect/return`,
  });

  res.json({ connectUrl });
});
```

When you request additional scopes with an `existingConnectionId`:

1. The user is redirected to Context Gateway
2. Context Gateway recognizes the existing connection
3. The user is shown a permission dialog for the new scopes only (not re-authenticating)
4. After approval, the same `connectionId` is returned with expanded scopes

## Handling Connection Expiration

Some sources require periodic re-authentication. If a connection expires:

```javascript
try {
  const data = await client.query({
    connectionId,
    query: 'SELECT * FROM user_profile',
  });
} catch (error) {
  if (error.code === 'connection_expired') {
    // Connection needs re-authentication
    const connectUrl = client.createConnectUrl({
      userId: req.user.id,
      source: 'spotify',
      scopes: ['read:user_profile', 'read:playlists'],
      existingConnectionId: connectionId,
      redirectUrl: `${process.env.BASE_URL}/connect/return`,
    });

    return res.json({
      error: 'Connection expired',
      message: 'Please re-authenticate to continue',
      reconnectUrl: connectUrl,
    });
  }
}
```

## Cross-App Consent Management

If you manage multiple applications using Context Gateway, users can control consent independently:

```javascript
// Get all connections for a user across all apps
const allConnections = await client.listConnections(userId);

console.log(allConnections);
// [
//   {
//     id: 'conn_abc123',
//     source: 'spotify',
//     app: 'music-app',
//     isValid: true,
//     scopes: ['read:user_profile', 'read:playlists']
//   },
//   {
//     id: 'conn_def456',
//     source: 'spotify',
//     app: 'recommendation-engine',
//     isValid: true,
//     scopes: ['read:playback_history']
//   }
// ]
```

Each application has its own connection with independent scopes and revocation.

## UI Best Practices

### Connection Status Display

```javascript
app.get('/settings', async (req, res) => {
  const connection = req.user.contextGatewayConnectionId
    ? await client.getConnection(req.user.contextGatewayConnectionId)
    : null;

  res.render('settings', {
    connection,
    isConnected: connection?.isValid,
    lastSync: connection?.lastSyncAt,
    connectedSource: connection?.source,
  });
});
```

### Template

```html
<div class="connection-status">
  <% if (isConnected) { %>
    <div class="alert alert-success">
      <strong>Connected:</strong> <%= connectedSource %>
      <p><small>Last synced: <%= lastSync %></small></p>
      <a href="/disconnect" class="btn btn-sm btn-danger">Disconnect</a>
    </div>
  <% } else { %>
    <div class="alert alert-warning">
      <strong>Not Connected</strong>
      <p>Connect your account to access your data.</p>
      <a href="/connect" class="btn btn-primary">Connect Now</a>
    </div>
  <% } %>
</div>
```

## Best Practices

1. **Always check before querying**: Never assume a connection is valid
2. **Handle revocation gracefully**: Don't show errors; guide users to reconnect
3. **Request minimal scopes**: Only ask for what you need
4. **Sync connection status**: Periodically verify connections are still valid
5. **Be transparent**: Show users what data you're accessing and when
6. **Respect revocation**: Delete cached data when a connection is revoked
7. **Provide reconnection UI**: Make it easy for users to re-grant access
8. **Log consent changes**: Maintain an audit trail for security and compliance
