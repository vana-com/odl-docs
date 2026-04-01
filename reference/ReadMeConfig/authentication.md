---
title: Authentication
excerpt: >-
  Use your Open Data Labs API key on the server and keep browser clients on app
  IDs plus hosted sessions.
api_config: authentication
hidden: false
icon: icon-key1
---
Use your Open Data Labs API key for server-to-server API calls. Do not expose it in client-side code.

For frontend integrations, use your public `appId` in the client and create Connect sessions from your server. 

## Server-side bearer token

```text
Authorization: Bearer <<keys:id>>
```

## Recommended environment variable

```bash
export OPENDATALABS_API_KEY=<<keys:id>>
```

## Account state

Plan: `<<plan>>`

Usage: `<<usageSummary>>`

Use the rest of the API Reference for endpoint details and request examples.
