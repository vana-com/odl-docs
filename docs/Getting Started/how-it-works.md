---
title: How It Works
excerpt: >-
  Context Gateway uses a hosted Connect model. Your backend creates short-lived
  Connect sessions. Your frontend opens the returned hosted URL in an embedded
  flow.
---
Context Gateway uses a hosted Connect model. Your backend creates short-lived Connect sessions. Your frontend opens the returned hosted URL in an embedded flow. Context Gateway handles everything in between.

## Architecture

```
┌──────────────┐         ┌──────────────────┐         ┌──────────┐         ┌──────────┐
│   YOUR APP   │ ──────> │ CONTEXT GATEWAY  │ ──────> │   USER   │ ──────> │  SOURCE  │
│creates session│        │  hosts the flow  │         │authenticates│       │Instagram │
└──────────────┘         └──────────────────┘         └──────────┘         └──────────┘
1. Backend creates               │                  3. User grants        4. Source data
   Connect session               ▼                     scopes                retrieved
                        ┌──────────────────┐
                        │ PERSONAL SERVER  │
                        │encrypted, per-user│
                        └──────────────────┘
                                 │
                                 ▼
┌──────────────┐         ┌──────────────────┐
│   YOUR APP   │ <────── │ CONTEXT GATEWAY  │
│receives context│       │  returns result  │
└──────────────┘         └──────────────────┘
6. Context ready         5. App queries
   to use                   Personal Server
```

## The flow in detail

**Step 1: Your server creates a Connect session**
Your backend calls the API with your server API key, the source to connect, the scopes you want to request, and the exact origin where the flow will run.

**Step 2: Context Gateway validates the request**
Before returning a Connect URL, Context Gateway checks that the source is available, the request is authenticated, and the requested origin is approved for your account.

**Step 3: The hosted Connect flow runs**
The user completes the connection flow in a Context Gateway-hosted interface. Your application does not build or maintain the source-specific auth and automation flow.

**Step 4: The source data is retrieved**
Context Gateway retrieves the authorised data from the source and stores it in the user's Personal Server. Initial sync begins immediately.

**Step 5: Your app queries the Personal Server**
Your application queries the user's context via the API. Context Gateway authenticates the request, checks scopes, and returns structured data from the Personal Server.

**Step 6: Your app receives context**
Structured, standardised user context is returned to your application. The same schema regardless of source.

## Trust model

| Component | Responsibility |
|---|---|
| Your server | Holds the API key and creates Connect sessions |
| Your frontend | Opens the hosted Connect flow |
| Context Gateway | Hosts the Connect UI and executes the source-specific flow |
| The user | Authenticates and authorises access |
| Personal Server | Encrypted per-user store, synced from the source |

## Approved domains

Context Gateway enforces approved domains as part of Connect session creation.

- The browser origin must match an approved domain exactly
- Local development domains can be added separately in the dashboard
- Unsupported or mismatched origins are rejected before the session is created

Questions or integration support: hello@opendatalabs.com