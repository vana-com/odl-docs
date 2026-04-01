---
title: Authentication
excerpt: >-
  Use your Open Data Labs API key on the server and keep browser clients on app
  IDs plus hosted sessions.
api:
  file: opendatalabs-api.json
  operationId: listSources
api_config: authentication
hidden: false
icon: icon-key1
---
Use the API key shown above from secure server-side code.

For frontend integrations, use your public `appId` in the client and create Connect sessions from your server.

## Server-side bearer token

```text
Authorization: Bearer YOUR_OPENDATALABS_API_KEY
```

## Recommended environment variable

```bash
export OPENDATALABS_API_KEY=YOUR_OPENDATALABS_API_KEY
```

## Account state

Plan: `<<plan>>`

Usage: `<<usageSummary>>`

Use the rest of the API Reference for endpoint details and request examples.
