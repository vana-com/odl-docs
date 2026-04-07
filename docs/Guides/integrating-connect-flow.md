---
title: Integrating the Connect Flow
excerpt: >-
  The Connect flow is the core integration surface for Context Gateway. Your
  backend creates a short-lived session. Your frontend embeds the hosted Connect
  URL.
---
The Connect flow is the core integration surface for Context Gateway. Your backend creates a short-lived session. Your frontend embeds the hosted Connect URL.

## Recommended architecture

| Layer | Responsibility |
|---|---|
| Server | Stores \`OPENDATALABS_API_KEY\`. Creates Connect sessions for a specific app. |
| Frontend | Uses the public app ID. Requests a session from your backend. Opens the returned \`connectUrl\`. Handles \`postMessage\` events from the hosted flow. |

In the dashboard, create an app for each integration surface where you embed Connect. Domains are approved per app, not globally for the account.

## SDK option

For React apps, use \`@opendatalabs/connect-js\` and keep session-creation logic on your backend:

```javascript
import { OpenDataLabsProvider, useConnect }
  from '@opendatalabs/connect-js/react';
```

## Create a Connect session

```javascript
export async function createConnectSession() {
  const response = await fetch(
    'https://api.opendatalabs.com/api/v1/connect/sessions',
    {
      method: 'POST',
      headers: {
        Authorization: \`Bearer \${process.env.OPENDATALABS_API_KEY}\`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        appId: 'odl_app_123',
        source: 'instagram',
        scopes: ['read:user_profile', 'read:posts'],
        origin: 'https://yourapp.com',
      }),
    }
  );
  const data = await response.json();
  if (!response.ok) {
    throw new Error(data.message || 'Failed to create Connect session');
  }
  return data;
}
```

## Request fields

| Field | Required | Description |
|---|---|---|
| source | Yes | Source identifier, e.g. \`instagram\` or \`icloud_notes\` |
| origin | Yes | Exact embedding origin, e.g. \`https://yourapp.com\` |
| scopes | No | Scopes to request for the source |
| appId | No | Public app ID. If omitted, uses the default app for the account. |
| redirectUrl | No | Return URL for flows that need to hand control back to your app. |

## Open the flow in a modal

```javascript
const session = await fetch('/api/connect-session')
  .then(res => res.json());

const iframe = document.createElement('iframe');
iframe.src = session.connectUrl;
iframe.style.width = '100%';
iframe.style.height = '720px';
iframe.style.border = '0';

modalBody.appendChild(iframe);
modal.open();
```

## Handle lifecycle events

```javascript
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://dashboard.opendatalabs.com') {
    return;
  }
  switch (event.data?.type) {
    case 'ready':
      console.log('Connect ready');
      break;
    case 'success':
      console.log('Connected:', event.data.connectionId);
      break;
    case 'exit':
      console.log('User exited Connect');
      break;
  }
});
```

## Domain approval errors

If the origin is not approved for the selected app, session creation fails with \`origin_not_allowed\`. Fix this by adding the exact embedding origin to the matching app in the dashboard.

Valid formats:
- \`https://app.example.com\`
- \`https://staging.example.com\`
- \`http://localhost:3000\`

Do not include paths, query strings, or fragments.

## Best practices

- Keep your API key on the server.
- Treat the hosted Connect session as short-lived.
- Validate your frontend event origin before acting on messages.
- Use the live source catalogue from \`/api/v1/sources\`. Do not hard-code sources.

Questions, feature requests, or support: hello@opendatalabs.com