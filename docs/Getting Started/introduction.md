---
title: Introduction
excerpt: >-
  Open Data Labs lets your users connect external accounts in your app without
  you storing credentials or building source-specific integrations.
---
<br />

<Callout icon="rocket" theme="info">
  **After speed?** Start with the [Quickstart](/docs/quickstart), then get your API key from the [dashboard](https://dashboard.opendatalabs.com).
</Callout>

Open Data Labs gives product teams an embedded Connect flow for consented data access. Your app initiates a Connect session on the server, Open Data Labs hosts the user-facing connection flow, and your users choose what to share.

The result is a simpler integration model:

* Your server holds a secret API key
* Your frontend launches a hosted Connect session
* Open Data Labs handles the source-specific automation
* Your users stay in control of which account and scopes they authorize

## Why teams use Open Data Labs

Building this in-house means taking on:

* source-specific auth and automation logic
* brittle browser edge cases
* consent UI and lifecycle handling
* credential-handling risk

Open Data Labs gives you one API and one embedded flow instead.

## What the current product does

Today, Open Data Labs supports:

* hosted Connect sessions
* approved-domain enforcement
* server API keys
* account-level configuration in the dashboard
* available sources:
  * Instagram
  * iCloud Notes

Additional sources like Spotify and GitHub are visible in the roadmap, but they are not yet generally available in the production API.

## Integration model

The current integration has four parts:

1. Your team gets an API key from the [dashboard](https://dashboard.opendatalabs.com).
2. You approve the domains where Connect can be launched.
3. Your server creates a Connect session through the API.
4. Your frontend opens the returned hosted Connect URL in a modal or iframe.

This keeps your backend in control of the integration while avoiding long-lived secrets in the browser.

## Security model

| Principle                                       | What it means                                                                                             |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Your server holds the secret**                | Use your API key only from trusted backend code.                                                          |
| **Your frontend launches short-lived sessions** | The browser should open hosted Connect sessions, not call the API with a long-lived secret.               |
| **Approved domains are enforced**               | Connect sessions only work from domains you have explicitly approved.                                     |
| **Credentials are not stored by your app**      | The user authenticates during the hosted flow; your app should not capture or persist source credentials. |

## What comes next

The current public API is focused on embedded Connect and account configuration. Durable connection-management APIs and broader SDK surfaces are planned, but the best current integration path is:

* server-side API key
* approved domains
* hosted Connect session creation
