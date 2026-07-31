---
title: Data Sources
excerpt: Get the current source catalog and available scopes.
---
The available sources and scopes can change. Get the current catalog from the API instead of copying it into your app.

## List sources

```bash
curl https://api.opendatalabs.com/api/v1/sources \
  -H "Authorization: Bearer $OPENDATALABS_API_KEY"
```

Use the source identifier in your Connect controller.

## List scopes

```bash
curl https://api.opendatalabs.com/api/v1/sources/instagram/scopes \
  -H "Authorization: Bearer $OPENDATALABS_API_KEY"
```

Replace `instagram` with a source identifier from the catalog. Use the returned scopes in your Connect controller.
