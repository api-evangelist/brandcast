---
name: brandcast-update-website-content
description: Read a Brandcast website's content items, page through them, add or upload media, and update individual content items. Use when an agent needs to change copy, imagery or video on an existing Brandcast site without touching its design.
api: Brandcast API
base_url: https://api.brandcast-prod.io/v1
generated: '2026-08-13'
method: generated
source: openapi/brandcast-websites-openapi.yml, openapi/brandcast-templates-openapi.yml
operations:
  - getWebsiteContent
  - updateWebsiteContent
  - getTemplateContent
  - getWebsiteMediaLibrary
  - addToWebsiteMediaLibrary
  - uploadToWebsiteMediaLibrary
  - getTemplateMediaLibrary
---

# Update content and media on a Brandcast website

## Preconditions

- `x-api-key: <key>` on every request, over HTTPS.
- A `websiteId`.

## Read the content first — always

`getWebsiteContent` — `GET /websites/{websiteId}/content`

This is the **only** way to learn what is on the site and what type each item is. Brandcast
publishes no response schema anywhere in its contract, so the shape of a content item is
discoverable only by reading it. The provider says so explicitly on the update operation: *"The
JSON body depends on the type of content being updated (e.g., text, embed, images, video, etc.),
which is known by getting the website's content."*

Query parameters:

| Parameter | Purpose |
|---|---|
| `name` | Filters the result by name. |
| `filterLockedContent` | Enables/disables the Locked Content Filter. Defaults to `true`. |
| `limit` | Limits the size of the result set. |
| `offset` | Pass the `next-offset` value from the previous response to get the next page. |
| `tags` | **Deprecated by Brandcast.** Do not use in new work. |

**Pagination:** loop while the response carries a `next-offset` value, feeding it back as `offset`.
These two content endpoints are the only paginated operations in the API — `getWebsites`,
`getTemplates`, `getPublishJobs` and `getExportList` have no paging parameters at all.

`getTemplateContent` — `GET /templates/{templateId}/content` — behaves identically for a template,
which is useful for discovering what a template exposes before creating a site from it.

## Update one content item

`updateWebsiteContent` — `PUT /websites/{websiteId}/content/{contentId}`

Send the JSON body matching the item's type as read in the previous step. Update **one item at a
time**; there is no batch operation and no partial-failure envelope.

## Add media

Two paths, and they are not interchangeable:

1. **By reference** — `addToWebsiteMediaLibrary`
   `POST /websites/{websiteId}/medialibrary`
   Supported media: `video/youtube` and `video/vimeo` only. Returns a media ID.

2. **By upload** — `uploadToWebsiteMediaLibrary`
   `POST /websites/{websiteId}/medialibrary/upload?fileName=...`
   Supported media: images, video, PDFs and ZIP files. This does **not** accept the bytes. It
   returns a media ID and an **S3 presigned URL**; the client then PUTs the file to that URL
   directly. Only after the upload completes is the media ID usable.

In both cases the returned **media ID is then used in an `updateWebsiteContent` body** to place the
asset on the site. `getWebsiteMediaLibrary` / `getTemplateMediaLibrary` list what is already there.

## Rules an agent must follow

- **Read before write, every time.** Writing a body of the wrong content type has no documented
  validation error to catch it.
- **No idempotency keys.** A retried `addToWebsiteMediaLibrary` can duplicate a library entry.
  Call `getWebsiteMediaLibrary` before retrying.
- Content changes are **not live until the site is published** — see `brandcast-publish-website`.
- The presigned S3 URL is a short-lived credential. Do not log it, and do not hand it to another
  system.
