---
title: Consent and Access
excerpt: Manage user consent, scopes, and data access control
---

# Consent and Access

Context Gateway's consent model ensures users maintain fine-grained control over their data. This document explains how consent works, how it's enforced, and how to handle revocation in your application.

## How Consent Works

Consent in Context Gateway happens in two phases:

### Phase 1: Initial Authorization

When a user connects a data source, they authenticate and explicitly grant scopes:

```
User → "Connect Spotify" button in your app
User → Redirected to Context Gateway
User → Authenticates with Spotify
User → Sees permission dialog: "This app wants to read your playlists and profile"
User → Clicks "Allow"
User → Redirected back to your app with connectionId
```

At this point, your app has a `connectionId` for that source and can query data within the granted scopes.

### Phase 2: Ongoing Access

Your app can query the user's data as long as:

- The `connectionId` is valid
- The user has not revoked consent
- Your query only requests data within the granted scopes

If any of these conditions is violated, the query fails.

## Scoping by Data Type

Context Gateway uses **scopes** to limit what data your app can access. Scopes are source-specific but follow common patterns.

### Example Scopes for Spotify

- `read:user_profile` - Access to user's name, email, profile image
- `read:playlists` - Access to user's playlists
- `read:playback_history` - Access to user's listening history
- `read:top_tracks` - Access to user's top 100 tracks
- `read:library` - Access to user's saved tracks and albums

### Example Scopes for GitHub

- `read:user_profile` - Access to public profile information
- `read:repositories` - Access to list of repositories
- `read:code` - Access to repository contents
- `read:issues` - Access to issues and pull requests

When you create a connect URL, you specify exactly which scopes to request. The user grants or denies each scope group.

## Scoping by Application

Users can grant different scopes to different applications. A single Personal Server can serve multiple apps, each with independent access:

```
Personal Server (Spotify)
├── Your Music App: read:user_profile, read:playlists
├── Music Recommender: read:playlists, read:playback_history
└── Gaming App: read:user_profile
```

Each application only sees data within its granted scopes.

## Revocation

Users can revoke access at any time via the Context Gateway dashboard or within your application (if you implement a revocation UI).

### What Happens on Revocation

- The `connectionId` remains valid, but queries fail
- Your app receives a `consent_revoked` error
- The user's data remains in their Personal Server
- The user can re-grant access later

### Example: Checking Connection Status

Before querying, check if the connection is still valid:

```javascript
const connection = await client.getConnection(connectionId);

if (!connection.isValid) {
  console.log('User has revoked access');
  // Redirect to reconnect flow or show UI prompt
  return res.redirect('/reconnect');
}

// Safe to query
const data = await client.query({
  connectionId,
  query: 'SELECT * FROM user_profile',
});
```

### Handling Revocation Gracefully

Design your app to handle revocation without surprises:

```javascript
try {
  const data = await client.query({
    connectionId: user.contextGatewayConnectionId,
    query: 'SELECT name, email FROM user_profile',
  });
  return res.json(data);
} catch (error) {
  if (error.code === 'consent_revoked') {
    // User has revoked access
    return res.status(403).json({
      error: 'Access revoked',
      message: 'Please reconnect your Spotify account to continue',
      reconnectUrl: '/connect?source=spotify',
    });
  }
  throw error;
}
```

## Cross-App Consent

When a user connects a source, they can use the same Personal Server across multiple applications. However, consent is managed per-application.

### How It Works

1. **User connects Spotify** in App A and grants `read:playlists`
2. **User tries to use App B** and clicks "Connect Spotify"
3. **Context Gateway detects** existing Personal Server for this Spotify account
4. **User grants different scopes** (e.g., `read:user_profile`) in App B
5. **Two separate connections** now exist, each with their own scopes

### Revoking Access to One App

If the user revokes access in App A:

- App A can no longer query Spotify data
- App B continues to work (separate connection)
- The Personal Server and data remain intact

Each application's access is independent.

## Requesting Additional Scopes

If your app needs additional permissions after the initial connection, you can request them:

```javascript
const newConnectUrl = client.createConnectUrl({
  userId: 'user_123',
  source: 'spotify',
  scopes: ['read:user_profile', 'read:playlists', 'read:playback_history'], // New scope
  existingConnectionId: 'conn_abc123def456', // Re-authenticate for new scopes
  redirectUrl: 'https://yourapp.com/auth/callback',
});

res.redirect(newConnectUrl);
```

The user will be asked to authorize the new scopes. On callback, you'll receive the same or updated `connectionId` with the expanded scopes.

## Best Practices

1. **Request minimal scopes**: Only ask for data you actually use
2. **Check before querying**: Always verify the connection is valid
3. **Handle revocation**: Never assume access will persist
4. **Be transparent**: Tell users what data you're accessing
5. **Cache wisely**: Don't cache user data indefinitely
6. **Respect timing**: Revocation happens immediately; update your UI accordingly
