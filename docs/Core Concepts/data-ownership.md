---
title: Data Portability
excerpt: Context Gateway is built on the principle that users own their data.
deprecated: true
hidden: true
---
Context Gateway is built on the principle that users own their data. This page explains what that means in practice and how it affects your application.

## Ownership principles

**The user is the data owner**
The user controls what sources connect, what data is accessed, which applications can query it, and when access is revoked. Your application is a temporary custodian, not an owner.

**No credential sharing**
User credentials never leave the source system. Your application never receives the user's password, never stores API keys or tokens, and cannot impersonate the user to the source.

## What your app can and cannot store

Context Gateway ensures your application does not become a data silo. In practice:

| You can | You cannot |
|---|---|
| Use data returned from a query within a session | Store raw user data in your database indefinitely |
| Cache data briefly for performance within a request lifecycle | Build persistent secondary indexes on user data |
| Log that a query occurred, with a timestamp | Retain query results after the user session ends |
| Store the connectionId to re-query later | Continue using cached data after consent is revoked |

If your application needs to display user data across sessions, re-query the Personal Server at session start. This keeps your application in sync with the user's current consent state and the latest source data.

## Trust boundaries

| Boundary | Data Flow | Trust Model |
|---|---|---|
| User to Source | Credentials, authorisation | User trusts source with credentials |
| User to Personal Server | Ownership, access grants, revocation | User controls via passkey |
| App to Context Gateway | API key, connection requests, queries | Mutual authentication via API key |
| Context Gateway to Personal Server | Requests, query results | Authenticated, scope-limited access |

## Reduced compliance burden

Your application does not need to encrypt user data at rest, maintain backups of user data, implement access logs for user data, or comply with data residency regulations. Context Gateway handles these. Your app focuses on application logic.

## Portability in practice

```
1. User authenticates with Instagram
   > User owns Instagram data and Personal Server

2. User grants your app read:posts scope
   > Your app can query posts from the Personal Server

3. Your app uses the data within the session
   > Do not persist this to your own database

4. User starts a new session
   > Re-query the Personal Server for fresh data

5. User revokes access
   > Your app can no longer query. Revocation is immediate.

6. User re-grants access later
   > Your app can query again with no reconciliation needed
```