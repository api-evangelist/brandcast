---
name: brandcast-publish-website
description: Publish a Brandcast website to the Internet as an asynchronous job, poll it to completion, and unpublish or export the result. Use when an agent needs to take a prepared Brandcast site live, check publishing status, or produce a downloadable ZIP of a published site.
api: Brandcast API
base_url: https://api.brandcast-prod.io/v1
generated: '2026-08-13'
method: generated
source: openapi/brandcast-websites-openapi.yml
operations:
  - createPublishJob
  - getPublishJob
  - getPublishJobs
  - unpublishWebsite
  - createExportJob
  - getExportJob
  - getExportList
  - protectWebsite
---

# Publish, unpublish and export a Brandcast website

Publishing is **asynchronous**. Brandcast ships no webhooks and no callbacks, so the only way to
learn the outcome is to poll a job id.

## Preconditions

- `x-api-key: <key>` on every request, over HTTPS.
- A `websiteId` from `getWebsites` or `createWebsite`.

## Publish

1. **Start the publish job** — `createPublishJob`
   `POST /websites/{websiteId}/publish`
   Optional JSON body: the `CustomVars` object — a free-form map (`key` holds arbitrary JSON). It
   is the only user-defined metadata surface in the whole API.
   The response carries a **`publishId`** and an initial status.

2. **Poll the job** — `getPublishJob`
   `GET /websites/{websiteId}/publish/{publishId}`
   Repeat until the status settles. Brandcast publishes no expected duration, no terminal-status
   vocabulary and no polling interval, so pick a conservative interval (start at a few seconds,
   back off) and impose your own timeout.

3. **Review history if needed** — `getPublishJobs`
   `GET /websites/{websiteId}/publish` returns the site's publishing history.

## Take a site down

- `unpublishWebsite` — `POST /websites/{websiteId}/unpublish`
  This is destructive to a live URL. Treat it as an operation that needs explicit human approval.

## Restrict access instead of unpublishing

- `protectWebsite` — `PUT /websites/{websiteId}/protection?username=...&password=...`
  Adds or removes password protection. Credentials are passed as **query parameters**, so they
  will appear in any request log or proxy trace — never send a credential you use anywhere else.

## Export a publish as a ZIP

1. `createExportJob` — `POST /websites/{websiteId}/publish/{publishId}/export?fileName=...`
   ZIPs all HTML and assets for that publish. If the file name already exists, Brandcast renames
   it to `filename(1)` rather than failing — so a retry after a timeout produces a **duplicate
   export**, not an overwrite.
2. `getExportJob` — `GET /websites/{websiteId}/publish/{publishId}/export/{fileName}`
   Poll for status; the result is delivered as an S3 signed URL.
3. `getExportList` — `GET /websites/{websiteId}/export` lists prior exports.

> The three export operations also accept an `Authorization` JWT plus an `x-account-id` header.
> Brandcast documents that JWT as "only required when called by Design Studio" and publishes no
> token endpoint for it — a third-party agent uses `x-api-key` alone. See
> `authentication/brandcast-authentication.yml`.

## Rules an agent must follow

- **No idempotency keys exist.** `createPublishJob` and `createExportJob` are both unsafe to blind
  retry. Before retrying, call `getPublishJobs` / `getExportList` to check whether the first
  attempt actually landed.
- **Unpublishing and deleting are irreversible from the API's point of view.** There is no undo
  operation and no versioning of published output.
- Only `403 {"message":"Missing Authentication Token"}` is a documented failure; every other error
  shape is undiscoverable without a live key (`errors/brandcast-problem-types.yml`).
