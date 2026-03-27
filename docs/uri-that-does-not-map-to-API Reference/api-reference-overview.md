---
title: API Reference Overview
excerpt: Complete reference for the Context Gateway REST API
---

# API Reference Overview

Context Gateway provides a REST API for managing connections and querying user data. This reference covers authentication, endpoints, rate limits, and error handling.

> **Early Access**: Context Gateway is currently in early access. The API is stable, but endpoint URLs and response formats may change before general availability.

## Base URL

All API requests should be made to:

```
https://api.contextgateway.com/v1
```

## Authentication

All requests to the Context Gateway API must include a Bearer token in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

Your API key is available in your [Context Gateway dashboard](https://dashboard.contextgateway.com). Never expose your API key in client-side code; always make API requests from your server.

```javascript
// Example request with authentication
const response = await fetch('https://api.contextgateway.com/v1/connections/conn_123', {
  headers: {
    'Authorization': `Bearer ${process.env.CONTEXT_GATEWAY_API_KEY}`,
    'Content-Type': 'application/json',
  },
});
```

## Endpoints

### Create a Connect URL

**POST** `/connect`

Creates a URL that users visit to authenticate and grant permissions.

```bash
curl -X POST https://api.contextgateway.com/v1/connect \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user_123",
    "source": "spotify",
    "scopes": ["read:user_profile", "read:playlists"],
    "redirectUrl": "https://yourapp.com/auth/callback"
  }'
```

**Response:**

```json
{
  "connectUrl": "https://gateway.contextgateway.com/connect/abc123def456"
}
```

### Get Connection Details

**GET** `/connections/{id}`

Retrieve information about a specific connection.

```bash
curl -X GET https://api.contextgateway.com/v1/connections/conn_abc123def456 \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:**

```json
{
  "id": "conn_abc123def456",
  "userId": "user_123",
  "source": "spotify",
  "isValid": true,
  "scopes": ["read:user_profile", "read:playlists"],
  "connectedAt": "2026-03-27T10:30:00Z",
  "lastUsed": "2026-03-27T14:45:00Z",
  "lastSyncAt": "2026-03-27T15:00:00Z"
}
```

### Revoke Connection

**DELETE** `/connections/{id}`

Revoke a user's connection, immediately stopping all queries.

```bash
curl -X DELETE https://api.contextgateway.com/v1/connections/conn_abc123def456 \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:**

```json
{
  "status": "revoked"
}
```

### Query User Data

**POST** `/query`

Execute a query against a user's Personal Server.

```bash
curl -X POST https://api.contextgateway.com/v1/query \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "connectionId": "conn_abc123def456",
    "query": "SELECT name, email FROM user_profile",
    "format": "json",
    "timeout": 30
  }'
```

**Response:**

```json
{
  "status": "success",
  "data": [
    {
      "name": "John Doe",
      "email": "john@example.com"
    }
  ],
  "metadata": {
    "rowCount": 1,
    "executionTime": 125,
    "source": "spotify"
  }
}
```

### List Available Sources

**GET** `/sources`

Get a list of supported data sources.

```bash
curl -X GET https://api.contextgateway.com/v1/sources \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:**

```json
{
  "sources": [
    {
      "id": "spotify",
      "name": "Spotify",
      "description": "Music streaming service",
      "status": "available",
      "scopes": ["read:user_profile", "read:playlists", "read:playback_history"]
    },
    {
      "id": "github",
      "name": "GitHub",
      "description": "Developer platform",
      "status": "available",
      "scopes": ["read:user_profile", "read:repositories", "read:code"]
    }
  ]
}
```

### Get Source Scopes

**GET** `/sources/{source}/scopes`

List available scopes for a specific source.

```bash
curl -X GET https://api.contextgateway.com/v1/sources/spotify/scopes \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:**

```json
{
  "source": "spotify",
  "scopes": [
    {
      "id": "read:user_profile",
      "description": "Access user profile information",
      "category": "Profile"
    },
    {
      "id": "read:playlists",
      "description": "Access user's playlists",
      "category": "Playlists"
    },
    {
      "id": "read:playback_history",
      "description": "Access user's listening history",
      "category": "Playback"
    }
  ]
}
```

## Rate Limiting

Context Gateway enforces rate limits based on your subscription tier. Rate limit information is returned in response headers.

### Rate Limit Tiers

| Tier | Requests/Minute | Concurrent Queries | Monthly Limit | Price |
|------|-----------------|-------------------|---------------|-------|
| **Free** | 60 | 5 | 10,000 | Free |
| **Pro** | 600 | 25 | 500,000 | $99/month |
| **Enterprise** | Unlimited | Unlimited | Custom | Custom |

### Rate Limit Headers

Each response includes rate limit information:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1648392000
```

When rate limited, you'll receive a 429 response:

```json
{
  "error": "rate_limited",
  "message": "Too many requests. Please try again later.",
  "retryAfter": 60
}
```

## Error Format

When an error occurs, Context Gateway returns a JSON error object:

```json
{
  "status": "error",
  "error": "query_error",
  "message": "Invalid query syntax",
  "code": "INVALID_QUERY",
  "timestamp": "2026-03-27T15:30:00Z",
  "requestId": "req_abc123def456"
}
```

### Error Fields

| Field | Description |
|-------|-------------|
| `status` | Always "error" for error responses |
| `error` | Machine-readable error code |
| `message` | Human-readable error message |
| `code` | Additional error code for debugging |
| `timestamp` | When the error occurred (ISO 8601) |
| `requestId` | Unique request ID for support inquiries |

### Common Error Codes

| Code | HTTP Status | Description |
|------|------------|-------------|
| `unauthorized` | 401 | Invalid API key or missing authentication |
| `forbidden` | 403 | Access denied (consent_revoked, etc.) |
| `not_found` | 404 | Connection or resource not found |
| `invalid_query` | 400 | Malformed query or invalid parameters |
| `rate_limited` | 429 | Rate limit exceeded |
| `consent_revoked` | 403 | User revoked access to this data |
| `connection_expired` | 401 | Connection needs re-authentication |
| `source_error` | 503 | External source is unavailable |
| `provisioning_failed` | 500 | Personal Server creation failed |
| `timeout` | 504 | Query execution timed out |

## SDKs

### JavaScript

The official JavaScript SDK is available on npm:

```bash
npm install @opendatalabs/context-gateway
```

```javascript
import { ContextGatewayClient } from '@opendatalabs/context-gateway';

const client = new ContextGatewayClient({
  apiKey: process.env.CONTEXT_GATEWAY_API_KEY,
});

const data = await client.query({
  connectionId: 'conn_123',
  query: 'SELECT * FROM user_profile',
});
```

[View JavaScript SDK documentation →](/docs/javascript-sdk)

### Python

Python SDK support is coming soon. In the meantime, use the REST API directly:

```python
import requests

response = requests.post(
    'https://api.contextgateway.com/v1/query',
    headers={'Authorization': f'Bearer {API_KEY}'},
    json={
        'connectionId': 'conn_123',
        'query': 'SELECT * FROM user_profile'
    }
)
```

### Go

Go SDK support is coming soon. Use the REST API directly or submit a feature request.

## Webhook Events

Context Gateway can send webhook events to your application for connection and sync events.

Supported events:
- `connection.created` - User completed authentication
- `connection.revoked` - User revoked access
- `connection.expired` - Connection needs re-authentication
- `sync.completed` - Data sync from source completed

Configure webhooks in your [dashboard](https://dashboard.contextgateway.com).

## Support

For API support and questions:

- **Documentation**: [contextgateway.com/docs](https://contextgateway.com/docs)
- **GitHub Issues**: [github.com/opendatalabs/context-gateway](https://github.com/opendatalabs/context-gateway)
- **Email Support**: [support@opendatalabs.com](mailto:support@opendatalabs.com)
- **Status Page**: [status.contextgateway.com](https://status.contextgateway.com)
