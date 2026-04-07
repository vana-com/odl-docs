---
title: API Reference Overview
excerpt: Overview of the current public Open Data Labs API surface.
hidden: true
---

# API Reference Overview

The production Open Data Labs API currently focuses on:

- source discovery
- hosted Connect session creation

## Base URL

```text
https://api.opendatalabs.com/api/v1
```

## Authentication

All authenticated requests use a Bearer token:

```text
Authorization: Bearer YOUR_OPENDATALABS_API_KEY
```

Use your API key from [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com) from trusted server-side code only.

## Encryption

All apps must have a data encryption secret configured before creating Connect sessions. Generate one in Dashboard → App Settings → Data Encryption Secret, then store it as `OPENDATALABS_ENCRYPTION_SECRET` in your server environment. The SDK uses it to decrypt connection results; the secret never leaves your server.

```bash
OPENDATALABS_API_KEY=YOUR_OPENDATALABS_API_KEY
OPENDATALABS_ENCRYPTION_SECRET=YOUR_OPENDATALABS_ENCRYPTION_SECRET
```

## Main endpoints

### `GET /sources`

Returns the source catalog available to the authenticated account.

### `GET /sources/{source}/scopes`

Returns the scopes for a given source.

### `POST /connect/sessions`

Creates a hosted Connect session for an approved domain.

Example:

```bash
curl -X POST https://api.opendatalabs.com/api/v1/connect/sessions \
  -H "Authorization: Bearer YOUR_OPENDATALABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "instagram",
    "scopes": ["read:profile", "read:following", "read:ads"],
    "origin": "https://yourapp.com"
  }'
```

## Support

If anything in this overview conflicts with the live API Reference, use the generated endpoint reference as the source of truth.
