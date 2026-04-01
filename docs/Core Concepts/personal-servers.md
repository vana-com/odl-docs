---
title: Personal Servers
excerpt: Learn about per-user encrypted data stores that power Context Gateway
deprecated: true
hidden: true
---

# Personal Servers

A Personal Server is the foundation of Context Gateway's privacy-preserving architecture. It's an encrypted, per-user data store that holds synced data from external sources without exposing credentials or raw data access to your application.

## What is a Personal Server?

When a user connects a data source through Context Gateway, the system automatically provisions a Personal Server: a secure, encrypted environment where data from that source is stored and managed. The Personal Server acts as an intermediary between the source and your application.

Instead of your app accessing data directly from Spotify, GitHub, or Netflix, it queries the user's Personal Server. This provides several critical benefits:

- **No credential storage**: User passwords and API keys never leave the source system
- **No data residency in your app**: Your application never has direct access to raw data
- **User-controlled access**: Users can revoke access to their data immediately
- **Cross-app reuse**: The same Personal Server can be reused across multiple applications the user authorizes

## Key Properties

### User-Controlled with Passkeys

Each Personal Server is secured with the user's passkey. The user can view, manage, and revoke access to their Personal Server at any time. Context Gateway cannot access a user's Personal Server without their authorization.

### Cross-App Reuse

Once a user connects a source (e.g., Spotify), the Personal Server for that connection can be reused across multiple applications. The user grants each app specific scopes and can revoke access per application independently.

```
┌─────────────────┐
│  Personal       │
│  Server         │
│  (Spotify)      │
└────────┬────────┘
         │
    ┌────┴────┬──────────┬──────────┐
    │          │          │          │
    ▼          ▼          ▼          ▼
 Your App   Music Rec   Playlist   Gaming
            App         Maker      App
```

### Automatic Sync

Context Gateway automatically syncs data from the source into the Personal Server on a regular schedule. Changes in the source (e.g., a new Spotify playlist) are reflected in the Personal Server within minutes.

### Source-Agnostic Storage

Personal Servers use a unified, source-agnostic schema. Whether the source is Spotify, GitHub, or Netflix, data is stored consistently, making it easy for your application to work with data from multiple sources.

## Provisioning is Automatic

You don't need to manually provision Personal Servers. When a user completes the authentication flow in the connect flow:

1. Context Gateway receives the authorization from the source
2. A Personal Server is automatically created and associated with the user
3. Initial data sync begins
4. Your application receives a `connectionId` to query that Personal Server

From that point on, the Personal Server is managed automatically. Syncs happen in the background, and your application simply queries it via the API.

## Data Security

Personal Servers are encrypted at rest using AES-256. Access is controlled by:

- **User authentication**: The user's passkey controls access
- **Scope restrictions**: Data is filtered by the scopes the user granted
- **Application context**: Your app can only access data from connections it authorized

Even Context Gateway administrators cannot read the contents of a user's Personal Server without the user's passkey.