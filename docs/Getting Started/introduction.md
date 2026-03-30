---
title: Introduction
excerpt: >-
  Context Gateway gives your users portable, user-owned data, without you
  building the infrastructure.
---
Context Gateway is a data portability API for platform builders. It lets your users connect their accounts from other services (Spotify, Netflix, GitHub, and more) and bring that data into your app, without your platform ever touching credentials or taking on liability for storing it.

## Why Context Gateway exists

Building data portability in-house means managing OAuth integrations, credential storage, sync reliability, and platform-specific edge cases. It's expensive to build and expensive to maintain.

Platforms that hold user data on their users' behalf take on trust and legal liability. Users increasingly expect to own and control their own data.

Without portability, every app starts cold for every user. There's no way to make use of data the user already has elsewhere.

Context Gateway solves all three problems with a single integration.

## Who it's for

Context Gateway is built for **application and platform builders** who want to offer their users seamless data portability as a feature. You don't want to build or maintain the infrastructure, handle user credential management, or take on the legal and trust surface of holding external user data.

Your users are end users of your app. They own their own data. Your platform is never in the middle.

## Key principles

| Principle                              | Description                                                                                                                                                     |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **No install for end users**           | The end user never downloads or installs anything. The connect flow is embedded in your app.                                                                    |
| **Platform never touches credentials** | The user authenticates directly with the source. Your platform and Context Gateway never see or store login credentials.                                        |
| **User owns the data**                 | Each user's Personal Server is provisioned in their name, controlled by their passkey. Neither Context Gateway nor your platform can access it without consent. |
| **Access is scoped and revocable**     | Each platform only sees what it connected. Users can revoke access at any time.                                                                                 |

## What's coming soon

Context Gateway is designed for periodic sync, not real-time streaming. Push/event-based emission and consumer-facing data management UIs are on the roadmap but not part of the current API surface.
