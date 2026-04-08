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
## What to do next

1. Create or select an app in your <a href="https://dashboard.opendatalabs.com/" target="_blank" rel="noreferrer">OpenDataLabs Dashboard</a>.
2. Generate a data encryption secret in Dashboard → App Settings → Data Encryption Secret, and store it as `OPENDATALABS_ENCRYPTION_SECRET` in your server environment.
3. Add your app domain where you wish to launch a [Connect Session](https://dev.opendatalabs.com/reference/createconnectsession).
4. Call `POST /connect/sessions` from your server to start a [Connect Session](https://dev.opendatalabs.com/reference/createconnectsession) for a source.
5. You can also list available [Sources](https://dev.opendatalabs.com/reference/listsources) and their [Scopes](https://dev.opendatalabs.com/reference/listsourcescopes).

For a complete walkthrough, continue to the [Quickstart](https://dev.opendatalabs.com/docs/quickstart) and [integration guides](https://dev.opendatalabs.com/docs/integrating-connect-flow) in the main docs.