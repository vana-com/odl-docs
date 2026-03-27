---
title: Querying User Data
excerpt: Query user data from Personal Servers using the Context Gateway API
---

# Querying User Data

Once a user has authorized your application and a Personal Server is provisioned, you can query their data using the Context Gateway API.

## Basic Query Example

The simplest way to query data is using SQL-like syntax:

```javascript
const data = await client.query({
  connectionId: 'conn_abc123def456',
  query: 'SELECT name, email, profile_image FROM user_profile',
});

console.log(data);
// Output:
// {
//   name: 'John Doe',
//   email: 'john@example.com',
//   profile_image: 'https://example.com/image.jpg'
// }
```

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connectionId` | string | Yes | The connection ID returned from the connect flow |
| `query` | string | Yes | SQL-like query selecting the data you need |
| `format` | string | No | Response format: `json` (default), `csv`, `xml` |
| `timeout` | number | No | Query timeout in seconds (default: 30, max: 120) |
| `limit` | number | No | Maximum rows to return (default: 1000) |

## Available Data Types by Source

Different sources expose different data types. Here are common examples:

### Spotify

```
user_profile
├── id (string): User's Spotify ID
├── display_name (string): Display name
├── email (string): Email address
├── profile_image (string): Avatar URL
├── followers (number): Follower count
└── is_premium (boolean)

playlists
├── id (string): Playlist ID
├── name (string): Playlist name
├── description (string): Playlist description
├── tracks_count (number): Number of tracks
├── is_collaborative (boolean)
└── public (boolean)

playback_history
├── track_id (string): Track ID
├── track_name (string): Track name
├── artist_id (string): Artist ID
├── artist_name (string): Artist name
├── played_at (timestamp): When the track was played
└── duration_ms (number): Track duration
```

### GitHub

```
user_profile
├── login (string): GitHub username
├── id (number): GitHub user ID
├── bio (string): User bio
├── public_repos (number): Public repository count
├── followers (number): Follower count
└── location (string): User location

repositories
├── id (number): Repository ID
├── name (string): Repository name
├── description (string): Repository description
├── owner (string): Repository owner
├── stars (number): Star count
├── language (string): Primary language
└── is_private (boolean)

issues
├── id (number): Issue ID
├── title (string): Issue title
├── body (string): Issue description
├── state (string): 'open' or 'closed'
├── created_at (timestamp): Creation time
└── assignee (string): Assigned user
```

### Netflix

```
user_profile
├── account_id (string): Account ID
├── email (string): Email address
├── subscription_type (string): 'basic', 'standard', or 'premium'
└── created_at (timestamp): Account creation date

watch_history
├── title_id (string): Content ID
├── title_name (string): Movie or show name
├── watched_at (timestamp): When it was watched
├── duration_seconds (number): How much was watched
└── content_type (string): 'movie' or 'show'

lists
├── id (string): List ID
├── name (string): List name
├── items (number): Number of items
└── created_at (timestamp)
```

## Query Examples

### Get User Profile

```javascript
const profile = await client.query({
  connectionId,
  query: 'SELECT display_name, email, followers FROM user_profile',
});
```

### List User's Playlists

```javascript
const playlists = await client.query({
  connectionId,
  query: 'SELECT id, name, tracks_count FROM playlists ORDER BY created_at DESC LIMIT 10',
});
```

### Get Recent Listen History

```javascript
const history = await client.query({
  connectionId,
  query: 'SELECT track_name, artist_name, played_at FROM playback_history WHERE played_at > NOW() - INTERVAL 7 DAY ORDER BY played_at DESC',
});
```

### Find Repositories by Language

```javascript
const repos = await client.query({
  connectionId,
  query: 'SELECT name, description, language, stars FROM repositories WHERE language = "JavaScript" ORDER BY stars DESC',
});
```

## Response Format

Responses are returned as JSON by default:

```javascript
const response = await client.query({
  connectionId,
  query: 'SELECT name, email FROM user_profile',
});

// Response structure:
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
    "executionTime": 125, // milliseconds
    "source": "spotify"
  }
}
```

### Alternative Formats

Request CSV or XML format if needed:

```javascript
// CSV format
const csv = await client.query({
  connectionId,
  query: 'SELECT name, email FROM user_profile',
  format: 'csv',
});
// Returns: "name,email\nJohn Doe,john@example.com"

// XML format
const xml = await client.query({
  connectionId,
  query: 'SELECT name, email FROM user_profile',
  format: 'xml',
});
```

## Error Handling

Queries can fail for several reasons. Always handle errors appropriately:

```javascript
try {
  const data = await client.query({
    connectionId,
    query: 'SELECT * FROM user_profile',
  });
  return res.json(data);
} catch (error) {
  switch (error.code) {
    case 'consent_revoked':
      // User revoked access
      return res.status(403).json({
        error: 'Access revoked',
        message: 'Please reconnect your account',
      });

    case 'connection_expired':
      // Authentication token expired
      return res.status(401).json({
        error: 'Connection expired',
        message: 'Please reconnect your account',
      });

    case 'query_error':
      // Malformed query
      return res.status(400).json({
        error: 'Invalid query',
        message: error.message,
      });

    case 'rate_limited':
      // Rate limit exceeded
      return res.status(429).json({
        error: 'Rate limited',
        message: 'Please try again in a few moments',
        retryAfter: error.retryAfter,
      });

    case 'source_error':
      // External source is down
      return res.status(503).json({
        error: 'Service unavailable',
        message: 'The service is temporarily unavailable',
      });

    default:
      console.error('Unexpected error:', error);
      return res.status(500).json({ error: 'Internal server error' });
  }
}
```

## Query Optimization

For better performance:

1. **Select only needed fields**: `SELECT name, email FROM user_profile` (not `SELECT *`)
2. **Use limits**: Add `LIMIT` to restrict results
3. **Filter early**: Use `WHERE` clauses to reduce data transfer
4. **Avoid large joins**: Query tables separately if possible
5. **Cache when appropriate**: Store results locally if they don't change frequently

```javascript
// Good: Specific fields, limited results
const recent = await client.query({
  connectionId,
  query: 'SELECT track_name, artist_name FROM playback_history LIMIT 10',
  timeout: 10,
});

// Less efficient: All fields, no limit
const allHistory = await client.query({
  connectionId,
  query: 'SELECT * FROM playback_history',
});
```

## Pagination

For large result sets, use pagination:

```javascript
const pageSize = 50;
let offset = 0;
let hasMore = true;

while (hasMore) {
  const response = await client.query({
    connectionId,
    query: `SELECT * FROM playlists LIMIT ${pageSize} OFFSET ${offset}`,
  });

  if (response.data.length < pageSize) {
    hasMore = false;
  }

  // Process response.data
  console.log(`Fetched ${response.data.length} playlists`);

  offset += pageSize;
}
```
