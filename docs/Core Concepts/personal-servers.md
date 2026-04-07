---
title: Personal Servers
excerpt: >-
  A Personal Server is the per-user data store that sits at the centre of the
  Context Gateway architecture.
deprecated: true
hidden: true
---
A Personal Server is the per-user data store that sits at the centre of the Context Gateway architecture. It holds synced data from external sources without exposing credentials or raw data access to your application.

## What is a Personal Server?

When a user connects a data source through Context Gateway, the system automatically provisions a Personal Server: an encrypted environment where data from that source is stored and managed. The Personal Server acts as an intermediary between the source and your application.

Your application queries the Personal Server rather than the source directly. This means:

- No credential storage. User passwords and API keys never leave the source system.
- No data residency in your app. Your application never has direct access to raw data.
- User-controlled access. Users can revoke at any time.
- Cross-app reuse. The same Personal Server can serve multiple applications the user authorises.

## Portability and the open source foundation

Personal Servers are built on Vana's open source data portability infrastructure. Vana provides the tools that allow individuals to reclaim their data from platforms and port it wherever they choose.

Context Gateway adds the hosted availability layer on top. Where Vana is stateless and decentralised by design, Context Gateway maintains data uptime and availability so the context your users share is standardised and ready to query whenever your application needs it.

This separation matters for developers. You get the compliance posture that comes with user-initiated, consent-documented data portability. You do not have to manage the underlying infrastructure that makes it work.

## Key properties

**User-controlled with passkeys**
Each Personal Server is secured with the user's passkey. The user can view, manage, and revoke access at any time. Context Gateway cannot access a user's Personal Server without their authorisation.

**Cross-app reuse**
Once a user connects a source, the Personal Server for that connection can be reused across multiple applications. Each application is granted specific scopes and the user can revoke access per application independently.

```
Personal Server (Instagram)
    |
    +-- Your App: read:user_profile, read:posts
    +-- Analytics App: read:user_profile
    +-- Social Dashboard: read:posts, read:engagement
```

**Automatic sync**
Context Gateway automatically syncs data from the source into the Personal Server on a regular schedule. Your application queries a current, maintained snapshot.

**Standardised schema**
Personal Servers use a unified, source-agnostic schema. Whether the source is Instagram or iCloud Notes, data is stored consistently so your application can work with data from multiple sources without source-specific parsing logic.

## Provisioning

You do not provision Personal Servers manually. When a user completes the hosted Connect flow:

- Context Gateway receives authorisation from the source
- A Personal Server is automatically created and associated with the user
- Initial data sync begins
- Your application receives a \`connectionId\` to query that Personal Server

## Security

Personal Servers are encrypted at rest using AES-256. Access is controlled by the user's passkey, scope restrictions granted by the user, and application context. Even Context Gateway administrators cannot read the contents of a user's Personal Server without the user's passkey.