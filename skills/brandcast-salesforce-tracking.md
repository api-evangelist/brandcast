---
name: brandcast-salesforce-tracking
description: Read, configure and disable the Salesforce tracking attached to a Brandcast website, which is what makes a Brandcast site personalized and trackable from Salesforce. Use when an agent needs to wire a Brandcast microsite into a Salesforce campaign or turn that tracking off.
api: Brandcast API
base_url: https://api.brandcast-prod.io/v1
generated: '2026-08-13'
method: generated
source: openapi/brandcast-salesforce-openapi.yml
operations:
  - getTracking
  - putTracking
  - deleteTracking
---

# Configure Salesforce tracking on a Brandcast website

Brandcast's Salesforce integration is what turns a published site into a personalized, trackable
one. The API exposes it as a single tracking resource hung off a website id.

## Preconditions

- `x-api-key: <key>` on every request, over HTTPS.
- A `websiteId` for a site that already exists (`getWebsites` / `createWebsite`).

## Steps

1. **Read the current configuration** — `getTracking`
   `GET /salesforce/tracking/{websiteId}`
   Returns the tracking configured for this website. Do this first: `putTracking` replaces a
   **collection**, so you need to know what is there before you write.

2. **Create or replace the configuration** — `putTracking`
   `PUT /salesforce/tracking/{websiteId}`
   Body: a collection of Tracking configurations as JSON. Brandcast does not publish the schema in
   its contract — it points at its own internal page, *"For details of the Tracking JSON, refer to
   Salesforce App Features"*, which is on a Brandcast Atlassian wiki that is not publicly
   reachable. **Do not invent a body.** Derive it from what `getTracking` returned, or obtain the
   shape from Brandcast directly.

3. **Disable tracking** — `deleteTracking`
   `DELETE /salesforce/tracking/{websiteId}`
   Disables tracking for this website. There is one tracking resource per website, so this is a
   full removal, not a per-rule delete.

## Rules an agent must follow

- **`putTracking` is a full replace of a collection.** Merge client-side against the result of
  `getTracking`; do not send a partial set expecting a patch.
- **The request schema is not published.** This is the one Brandcast flow where an agent cannot
  construct a valid request from the contract alone. Treat an unverified body as unsafe.
- **No idempotency key exists**, but `putTracking` and `deleteTracking` are naturally idempotent
  because they target a single resource — a repeat is safe in a way `createWebsite` and
  `createPublishJob` are not.
- Tracking changes affect a live site's analytics. Confirm with the requester before disabling.
