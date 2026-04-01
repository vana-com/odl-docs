---
title: Data Ownership
excerpt: Understand the ownership model and trust boundaries in Context Gateway
deprecated: true
hidden: true
---

# Data Ownership

Context Gateway is built on the principle that users own their data. This document explains what that means in practice and how it affects your application.

## Ownership Principles

### User is Sole Data Owner

The user has complete ownership of their data. They control:

- **What sources connect**: Only the user can authorize a source connection
- **What data is accessed**: Only the user can grant specific scopes
- **Who can access their data**: Each application must be individually authorized
- **When access is revoked**: The user can revoke access at any time

Your application is a temporary custodian of the user's data, not an owner.

### No Data Residency in Your App

Context Gateway ensures that your application never stores user data directly. Data is queried on-demand from the Personal Server and returned to your app, but your app should not:

- Store raw user data in its database
- Cache queries for extended periods
- Export user data without explicit consent
- Build secondary indexes on user data

If you need persistent access to data, you should sync it with the user's knowledge and maintain an audit trail.

### No Credential Sharing

User credentials never leave the source system. Your application:

- Never receives the user's password
- Never stores API keys or tokens
- Cannot impersonate the user to the source
- Cannot access the data without the Personal Server

## Trust Boundaries

Context Gateway operates across four trust boundaries. Understanding these helps you design secure integrations.

| Boundary | From | To | Data Flow | Trust Model |
|----------|------|-----|-----------|------------|
| **User ↔ Source** | User | Data Source (Spotify, GitHub, etc.) | User credentials, authorization | User trusts source with credentials |
| **User ↔ Personal Server** | User | Personal Server | Ownership, access grants, revocation | User controls via passkey |
| **App ↔ Context Gateway** | Your App | Context Gateway API | API key, connection requests, queries | Mutual authentication via API key |
| **Context Gateway ↔ Personal Server** | Context Gateway | Personal Server | Requests, query results | Authenticated but limited access |

## What This Means for Your App

### No Compliance Burden for Data Residency

You don't need to:

- Encrypt user data at rest
- Maintain backups of user data
- Implement access logs for user data
- Comply with data residency regulations

Context Gateway handles these concerns. Your app can focus on application logic.

### Revocation = Immediate Loss of Access

When a user revokes access to their data:

- All future queries fail immediately with `consent_revoked`
- You receive no data
- The user's data remains in their Personal Server
- The user can re-grant access at any time

Your application cannot:

- Continue using cached data after revocation
- Request a "grace period" to migrate data
- Appeal the revocation decision

Design your application to handle revocation gracefully. Never rely on persistent access.

### Audit Trail Responsibility

If you need to track what data a user accessed:

- You must log queries and results yourself
- You cannot rely on Context Gateway's audit logs
- Be transparent with users about what you log

### User Can Revoke Without Notice

Users can revoke access at any time, including:

- After logging out of your app
- Without visiting your app
- Without notification to you

Always check connection status before querying, even for users with active sessions.

## Data Ownership in Practice

Here's how data ownership flows through a typical integration:

```
1. User authenticates with Spotify
   → User owns Spotify data and Personal Server

2. User grants your app read:playlists scope
   → Your app can query playlists from the Personal Server

3. Your app stores user name in session memory
   → You're temporarily using the data

4. User logs out of your app
   → You should delete the cached data

5. User revokes access in Context Gateway dashboard
   → Your app can no longer query the Personal Server

6. User re-grants access after 6 months
   → Your app can query again; no data reconciliation needed
```

The user is in control at every step.