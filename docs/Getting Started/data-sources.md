---
title: Data Sources
excerpt: Supported data sources, available scopes, and example response schemas.
---
## Supported sources

We're adding sources weekly. Don't see a source that you need? Get in touch hello@opendatalabs.xyz

| Source       | Status      | Scopes                                               |
| ------------ | ----------- | ---------------------------------------------------- |
| Instagram    | Available   | `read:profile`, `read:following`, `read:ads`         |
| iCloud Notes | Available   | `read:notes`, `read:folders`                         |
| GitHub       | Available   | `read:profile`, `read:repositories`, `read:starred`  |
| Oura Ring    | Available   | `read:readiness`, `read:sleep`, `read:activity`      |
| Spotify      | Coming soon | `read:profile`, `read:savedTracks`, `read:playlists` |

***

<br />

## Instagram

**Source ID:** `instagram`

### Scopes

| Scope            | Description                                           |
| ---------------- | ----------------------------------------------------- |
| `read:profile`   | Username, bio, follower and following counts          |
| `read:following` | List of accounts the user follows                     |
| `read:ads`       | Advertisers seen and ad topics based on user activity |

### Example response

```json
{
  "platform": "instagram",
  "username": "janedoe",
  "followers": 1240,
  "following": 480,
  "media_count": 97,
  "following_accounts": [
    { "username": "natgeo", "full_name": "National Geographic", "label": null },
    { "username": "nasa", "full_name": "NASA", "label": null }
  ],
  "ads": {
    "advertisers": [
      { "name": "Acme Corp" },
      { "name": "Brand Co" }
    ],
    "ad_topics": [
      { "name": "Travel" },
      { "name": "Technology" }
    ],
    "targeting_categories": [
      { "name": "Small business owners" }
    ]
  }
}
```

***

## iCloud Notes

**Source ID:** `icloud_notes`

### Scopes

| Scope          | Description                         |
| -------------- | ----------------------------------- |
| `read:notes`   | Notes content, titles, and metadata |
| `read:folders` | Notes folder structure              |

### Example response

```json
{
  "platform": "icloud_notes",
  "userName": "Jane Doe",
  "notes": [
    {
      "title": "Project ideas",
      "textContent": "1. Build a personal data app\n2. Automate weekly review",
      "snippet": "1. Build a personal data app",
      "folder": "Work",
      "modified": "2024-11-15T09:32:00Z"
    },
    {
      "title": "Grocery list",
      "textContent": "Eggs, milk, bread",
      "snippet": "Eggs, milk, bread",
      "folder": "Personal",
      "modified": "2024-11-20T18:00:00Z"
    }
  ],
  "folders": [
    { "title": "Work", "recordName": "folder-abc123" },
    { "title": "Personal", "recordName": "folder-def456" }
  ]
}
```

***

## GitHub

**Source ID:** `github`

### Scopes

| Scope               | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `read:profile`      | Username, bio, location, and follower counts                 |
| `read:repositories` | Repository list with metadata (language, stars, update time) |
| `read:starred`      | Repositories the user has starred                            |

### Example response

```json
{
  "platform": "github",
  "github.profile": {
    "username": "janedoe",
    "fullName": "Jane Doe",
    "bio": "Building developer tools",
    "company": "Acme Corp",
    "location": "San Francisco, CA",
    "website": "https://janedoe.dev",
    "avatarUrl": "https://avatars.githubusercontent.com/u/12345678",
    "followers": 320,
    "following": 85,
    "repositoryCount": 42,
    "profileUrl": "https://github.com/janedoe"
  },
  "github.repositories": {
    "repositories": [
      {
        "name": "my-project",
        "url": "https://github.com/janedoe/my-project",
        "description": "An open source tool for developers",
        "language": "TypeScript",
        "stars": 214,
        "forks": 18,
        "visibility": "public",
        "topics": ["cli", "developer-tools"],
        "updatedAt": "2024-11-10T14:22:00Z"
      }
    ]
  },
  "github.starred": {
    "starred": [
      {
        "fullName": "vercel/next.js",
        "url": "https://github.com/vercel/next.js",
        "description": "The React Framework",
        "language": "JavaScript",
        "stars": 118000,
        "updatedAt": "2024-11-19T08:00:00Z"
      }
    ]
  }
}
```

***

## Oura Ring

**Source ID:** `oura`

### Scopes

| Scope            | Description                                       |
| ---------------- | ------------------------------------------------- |
| `read:readiness` | Daily readiness scores, HRV, and recovery metrics |
| `read:sleep`     | Sleep duration, phases, heart rate, and HRV data  |
| `read:activity`  | Daily step counts, calories, and activity levels  |

### Example response

```json
{
  "platform": "oura",
  "oura.readiness": {
    "days": [
      { "day": "2024-11-20", "score": 82, "temperatureDeviation": -0.12 },
      { "day": "2024-11-21", "score": 76, "temperatureDeviation": 0.05 }
    ]
  },
  "oura.sleep": {
    "dailyScores": [
      { "day": "2024-11-20", "score": 79 },
      { "day": "2024-11-21", "score": 85 }
    ],
    "sleepPeriods": [
      {
        "day": "2024-11-20",
        "type": "long_sleep",
        "bedtimeStart": "2024-11-19T23:10:00Z",
        "bedtimeEnd": "2024-11-20T07:05:00Z",
        "totalSleepDuration": 27600,
        "efficiency": 88,
        "averageHeartRate": 56,
        "averageHrv": 48
      }
    ]
  },
  "oura.activity": {
    "days": [
      {
        "day": "2024-11-20",
        "score": 71,
        "steps": 9234,
        "activeCalories": 420,
        "totalCalories": 2180,
        "highActivityTime": 22
      }
    ]
  }
}
```

***

## Spotify _(coming soon)_

**Source ID:** `spotify`

### Scopes

| Scope              | Description                   |
| ------------------ | ----------------------------- |
| `read:profile`     | Display name and account info |
| `read:savedTracks` | Liked songs                   |
| `read:playlists`   | Playlists and their tracks    |
