---
name: Monitor repair events with webhooks
description: >-
  Subscribe to Upsie Partner Network events (repair_status_updated) by managing
  webhook subscriptions, and track repair progress with notes.
api: openapi/upsie-partner-network-openapi.yml
operations: [generateApiAccessToken, whoAmI, createWebhook, listWebhooks, updateWebhook, deleteWebhook, listRepairNotes, createRepairNote]
generated: '2026-07-21'
method: generated
---

# Monitor repair events with webhooks

Base URL: `https://api.upsie.com`; JWT in the `token` header on every request.

## Steps

1. `generateApiAccessToken` — `POST /partner/auth/apiaccess` to mint an API
   access token; confirm identity with `whoAmI` — `GET /partner/auth/whoami`.
2. `createWebhook` — `POST /partner/partnerorganizationwebhooks` with a body
   like:

   ```json
   {
     "url": "https://your-app.example.com/webhooks",
     "meta": { "events": ["repair_status_updated"] },
     "description": "repair status listener"
   }
   ```

   `repair_status_updated` is the event name published in the official
   documentation examples.
3. `listWebhooks` — `GET /partner/partnerorganizationwebhooks/` to audit
   subscriptions; `updateWebhook` (`PUT .../{id}`) to change the destination
   URL; `deleteWebhook` (`DELETE .../{id}`) to remove one.
4. On an event, fetch context: `listRepairNotes` —
   `GET /partner/repairnotes?where[repairnotes__dot__repair_id]=<repairId>`
   (note the `__dot__` escape for nested columns), and reply with
   `createRepairNote` — `POST /partner/repairnotes`.

## Rules

- Webhook endpoints require a partner API access token (30-day expiry;
  refresh via `POST /partner/auth/apirefresh`).
- Serve the destination URL over HTTPS.
- See `../asyncapi/upsie-webhooks.yml` for the captured webhook surface.
