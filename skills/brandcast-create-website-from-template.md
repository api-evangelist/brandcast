---
name: brandcast-create-website-from-template
description: Create a new Brandcast website from an existing Design Studio template, then verify it exists. Use when an agent needs to stand up a branded microsite, proposal or campaign page from an approved template.
api: Brandcast API
base_url: https://api.brandcast-prod.io/v1
generated: '2026-08-13'
method: generated
source: openapi/brandcast-templates-openapi.yml, openapi/brandcast-websites-openapi.yml
operations:
  - getTemplates
  - getTemplate
  - createWebsite
  - getWebsite
---

# Create a Brandcast website from a template

Every Brandcast website starts from a template that already exists in the account. There is no
"create a template" operation in the API — templates are authored in Design Studio.

## Preconditions

- An API key issued by Brandcast, tied to the account that owns the template.
- Send it on **every** request as `x-api-key: <key>`, over HTTPS only. There is no other
  credential a third party can use.

## Steps

1. **List the templates available to this account** — `getTemplates`
   `GET /templates`
   Returns the templates for the account behind the API key. There are no pagination or filter
   parameters on this operation, so the whole set comes back at once.

2. **Confirm the template you intend to use** — `getTemplate`
   `GET /templates/{templateId}`
   Do this before creating anything. `templateId` values are opaque; Brandcast publishes no id
   format, so never construct one — only use an id returned by step 1.

3. **Create the website** — `createWebsite`
   `POST /websites?templateId=...&name=...&subdomain=...&newId=...&description=...`
   All five inputs are **query parameters**, not a JSON body. `templateId` is the template to copy.
   `subdomain` determines the address the site will publish to, so treat it as the one value the
   requester must approve.

4. **Read the created website back** — `getWebsite`
   `GET /websites/{websiteId}`
   Confirm the site exists and capture its `websiteId` for every later call.

## Rules an agent must follow

- **Do not retry step 3 blindly.** The Brandcast API supports no idempotency key on any
  operation (`conventions/brandcast-conventions.yml`). A retried `createWebsite` after a timeout
  can produce a second website. On an ambiguous failure, call `getWebsites` and look for the
  `subdomain` you requested before retrying.
- **Errors are nearly undocumented.** No operation declares any 4xx or 5xx response. The only
  verified error shape is `403 {"message":"Missing Authentication Token"}`, which is returned both
  for a bad key and for an unrecognised path — see `errors/brandcast-problem-types.yml`.
- **No rate limits are published** (`rate-limits/brandcast-rate-limits.yml`). Use conservative
  pacing and exponential backoff; there is no `Retry-After` contract to rely on.
- Creating a website does **not** put it on the Internet. Publishing is a separate, asynchronous
  flow — see `brandcast-publish-website`.
