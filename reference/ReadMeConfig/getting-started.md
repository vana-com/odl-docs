---
title: Getting Started
excerpt: Make your first authenticated Open Data Labs API call from server-side code.
api:
  file: opendatalabs-api.json
  operationId: listSources
api_config: getting-started
hidden: false
icon: icon-book1
---

Use this page to make your first authenticated request to the Open Data Labs API.

We recommend starting with `GET /sources`. It returns the currently available sources for your account and is a simple way to verify that your API key is working.

## First call

```bash
export OPENDATALABS_API_KEY=YOUR_OPENDATALABS_API_KEY

curl --request GET \
  --url https://api.opendatalabs.com/api/v1/sources \
  --header "Authorization: Bearer $OPENDATALABS_API_KEY"
```

If this request succeeds, you are ready to create Connect sessions from your server.

## What to do next

1. Create or select an app in the dashboard.
2. Add the domain where you will launch Connect.
3. Call `POST /connect/sessions` from your server to start a Connect session for a source.

For a complete walkthrough, continue to the Quickstart and integration guides in the main docs.