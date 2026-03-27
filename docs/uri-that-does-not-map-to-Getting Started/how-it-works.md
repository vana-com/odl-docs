---
title: How It Works
excerpt: Understand the four-step flow and architecture of Context Gateway
---

# How Context Gateway Works

Context Gateway enables your application to access user data from any source without storing credentials, building integrations, or managing data compliance. Here's how the system works.

## The Four-Step Flow

### Step 1: App Initiates Connect

Your application redirects the user to Context Gateway with a connect request containing:
- Your API key
- The data source (e.g., Spotify, GitHub, Netflix)
- The specific data scopes needed (e.g., read:playlists, read:user_profile)
- A callback URL where the user should return after authentication

### Step 2: User Authenticates with Source

The user is directed to the source's authentication flow (e.g., Spotify's login). Context Gateway never sees the user's credentials—the source handles authentication directly.

### Step 3: Personal Server Provisioned

Upon successful authentication, Context Gateway automatically provisions a Personal Server: an encrypted, per-user data store. The user's identity is tied to the source account, and Context Gateway syncs data from the source into this Personal Server.

### Step 4: App Queries Data via API

Your application can now query the user's data via the Context Gateway API using a connection ID. All queries hit the Personal Server, not the source directly. The user can revoke access at any time, immediately stopping all queries.

## System Architecture

```
┌────────────┐
│  Your App  │
└──────┬─────┘
       │ 1. Initiates connect
       │ 4. Queries data via API
       ▼
┌──────────────────────────────┐
│  Context Gateway API         │
│  (Authentication, Routing)   │
└──────┬───────────────┬───────┘
       │ 2. Auth flow  │ 3. Provisions & syncs
       ▼               ▼
   ┌────────┐    ┌──────────────┐
   │ Source │    │ Personal     │
   │        │    │ Server       │
   │        │    │ (Encrypted)  │
   └────────┘    └──────────────┘
```

## Trust Model

Context Gateway's security model is built on clear separation of responsibilities. Here's who can access what:

| Component | Can Access User Credentials? | Can Access Raw User Data? | Responsibility |
|-----------|------------------------------|--------------------------|-----------------|
| **Your App** | No | No (via queries only) | Request data, handle results, implement application logic |
| **Context Gateway** | No | No | Route requests, manage Personal Servers, enforce consent |
| **Personal Server** | No (never stored) | Yes (encrypted storage) | Store synced data, respond to queries, enforce scopes |
| **User** | Yes (with source) | Yes (owns data) | Authenticate, grant/revoke consent, control data |

### Key Trust Principles

- **User is sole data owner**: Only the user can grant or revoke access to their data
- **Credentials never cross boundaries**: User credentials are only known to the source
- **Personal Server is per-user**: Even Context Gateway cannot access another user's Personal Server
- **Scopes limit access**: Your app can only query data within the scopes the user granted
- **Revocation is immediate**: When a user revokes access, all queries fail immediately
