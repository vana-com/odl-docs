---
title: How It Works
excerpt: Understand the current hosted Connect session model.
---
# How Open Data Labs Works

Open Data Labs uses a hosted Connect model. Your backend creates short-lived Connect sessions, and your frontend opens the returned hosted URL in an embedded flow.

## The current flow

### Step 1: Your server creates a Connect session

Your backend calls the API with:

* your server API key
* the source to connect
* the scopes you want to request
* the exact origin where the flow will run

### Step 2: Open Data Labs validates the request

Before returning a Connect URL, Open Data Labs checks:

* that the source is available
* that the request is authenticated
* that the requested origin is approved for your account

### Step 3: The hosted Connect flow runs

The user completes the connection flow in an Open Data Labs-hosted interface. Your application does not need to build the source-specific auth and automation flow itself.

### Step 4: Your app receives lifecycle events

The hosted flow posts events back to the embedding page, including:

* `ready`
* `success`
* `exit`

## Trust model

| Component          | Responsibility                                             |
| ------------------ | ---------------------------------------------------------- |
| **Your server**    | Holds the API key and creates Connect sessions             |
| **Your frontend**  | Opens the hosted Connect flow                              |
| **Open Data Labs** | Hosts the Connect UI and executes the source-specific flow |
| **The user**       | Authenticates and grants access                            |

## Approved domains

Open Data Labs enforces approved domains as part of Connect session creation.

That means:

* the browser origin must match an approved domain exactly
* local development domains can be added separately
* unsupported or mismatched origins are rejected before the session is created

## Supported sources

The public source catalog currently includes:

* Instagram
* iCloud Notes

Roadmap sources may appear in product demos or catalog responses as `coming_soon`, but only `available` sources should be used in production integrations.

<br />

<br />

**Questions? Get in touch!** [hello@opendatalabs.com](mailto:hello@opendatalabs.com)
