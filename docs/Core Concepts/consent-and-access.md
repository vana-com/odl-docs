---
title: Consent and Access
excerpt: >-
  Context Gateway's consent model ensures users maintain control over their
  data.
---
Context Gateway's consent model ensures users maintain control over their data. This page covers how consent works, how it is enforced, and how to handle revocation in your application.

## How consent works

**Phase 1: Initial authorisation**

When a user connects a data source, they authenticate and explicitly grant scopes:

```
User clicks 'Connect Instagram' in your app
User is redirected to Context Gateway
User authenticates with Instagram
User sees: 'This app wants to read your profile and posts'
User clicks Allow
User is redirected back to your app with connectionId
```

**Phase 2: Ongoing access**

Your app can query the user's data as long as the \`connectionId\` is valid, the user has not revoked consent, and your query only requests data within the granted scopes. If any of these conditions is not met, the query fails.

## Scopes

Context Gateway uses scopes to limit what data your application can access. Scopes are source-specific but follow common patterns.

| Source | Scope | Access |
|---|---|---|
| Instagram | read:user_profile | Name, username, bio, follower count, profile image |
| Instagram | read:posts | Post content, captions, timestamps |
| Instagram | read:engagement | Likes, comments, reach per post |
| iCloud Notes | read:notes | Note titles and content |
| iCloud Notes | read:folders | Folder structure and organisation |

## Cross-app consent

A single Personal Server can serve multiple applications. Consent is managed per-application. If a user revokes access to one application, others are unaffected.

## Revocation

Users can revoke access at any time via the Context Gateway dashboard or within your application if you implement a revocation UI.

**What happens on revocation**

- All future queries fail immediately with a \`consent_revoked\` error
- The user's data remains in their Personal Server
- The user can re-grant access at any time

**Checking connection status**

```javascript
const connection = await client.getConnection(connectionId);

if (!connection.isValid) {
  return res.redirect('/reconnect');
}

const data = await client.query({ connectionId, query: '...' });
```

**Handling revocation gracefully**

```javascript
try {
  const data = await client.query({
    connectionId: user.contextGatewayConnectionId,
    query: 'SELECT name, username FROM user_profile',
  });
  return res.json(data);
} catch (error) {
  if (error.code === 'consent_revoked') {
    return res.status(403).json({
      error: 'Access revoked',
      reconnectUrl: '/connect?source=instagram',
    });
  }
  throw error;
}
```

## Best practices

- Request minimal scopes. Only ask for data you actually use.
- Check before querying. Always verify the connection is valid.
- Handle revocation. Never assume access will persist.
- Be transparent. Tell users what data you are accessing.
- Do not cache user data indefinitely.
- Revocation is immediate. Update your UI accordingly.