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
    "scopes": ["read:user_profile", "read:posts", "read:engagement"],
    "origin": "https://yourapp.com"
  }'
```

## Support

If anything in this overview conflicts with the live API Reference, use the generated endpoint reference as the source of truth.
