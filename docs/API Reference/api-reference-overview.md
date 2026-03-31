---
title: API Reference Overview
excerpt: Overview of the current public Open Data Labs API surface.
---

# API Reference Overview

The production Open Data Labs API currently focuses on:

- account credentials
- approved domains
- source discovery
- hosted Connect session creation

## Base URL

```text
https://api.opendatalabs.com/api/v1
```

## Authentication

All authenticated requests use a Bearer token:

```text
Authorization: Bearer <<apiKey>>
```

Use your API key from [dashboard.opendatalabs.com](https://dashboard.opendatalabs.com) from trusted server-side code only.

## Main endpoints

### `GET /health`

Simple health check.

### `GET /sources`

Returns the source catalog available to the authenticated account.

### `GET /sources/{source}/scopes`

Returns the scopes for a given source.

### `POST /connect/sessions`

Creates a hosted Connect session for an approved domain.

Example:

```bash
curl -X POST https://api.opendatalabs.com/api/v1/connect/sessions \
  -H "Authorization: Bearer <<apiKey>>" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "instagram",
    "scopes": ["read:user_profile", "read:posts", "read:engagement"],
    "origin": "https://yourapp.com"
  }'
```

### `GET /account/credentials`

Returns the current API key plus plan and usage metadata for the authenticated account.

### `GET /account/embed-origins`

Lists the approved domains configured for your account.

### `POST /account/embed-origins`

Adds a new approved domain.

### `DELETE /account/embed-origins/{id}`

Removes an approved domain.

## Current product notes

- `POST /connect` still exists as a legacy alias, but `POST /connect/sessions` is the clearer public shape.
- `/query` is not part of the stable public API yet.
- durable connection-management endpoints are not yet the primary public model.

## Reference source of truth

The generated OpenAPI definition should be treated as the source of truth for request and response shapes. This page is intended as orientation, not as a hand-maintained endpoint-by-endpoint contract.
