---
name: Create and submit a repair
description: >-
  Create a repair request on the Upsie Partner Network API — either in a single
  all-in-one payload or piece by piece — and submit it for routing through the
  repair network.
api: openapi/upsie-partner-network-openapi.yml
operations: [partnerLogin, generateApiAccessToken, listRepairCategories, listRepairItemTemplates, createRepair, createRepairItem, updateRepair, getRepair]
generated: '2026-07-21'
method: generated
---

# Create and submit a repair

Base URL: `https://api.upsie.com` (sandbox: `https://stage-api.upsie.com`). Every
request carries a JWT in a `token` header — see
`../authentication/upsie-authentication.yml`.

## Authenticate

1. `partnerLogin` — `POST /partner/auth/login` with username/password to get a
   user JWT (or use a partner-scoped token from the Partner Portal).
2. `generateApiAccessToken` — `POST /partner/auth/apiaccess` (with the user
   token in the `token` header) to mint a 30-day API access token. Use this
   token for all following calls.

## Option A — one payload

3. `createRepair` — `POST /partner/repairs` with the full payload: `customer`
   (firstName, lastName, phoneNumber, email), `category` (e.g. `Smartphones`),
   `authorizedLimit`, `meta` (policyNumber, model, serial, imei, color,
   storage), `repairLocationDetails` (address, radiusInMiles), `items[]`
   (each either `{"repairItemTemplateId": n}` or `{"description": "..."}`),
   and `repairStatus: "SUBMITTED"`.

## Option B — piece by piece

3. `createRepair` — `POST /partner/repairs` with `{}` creates an empty DRAFT
   repair; note the returned `id`/`identifier`.
4. `updateRepair` — `PUT /partner/repairs/{id}` to add customer, category,
   meta, and location details.
5. Find work lines: `listRepairCategories` (`GET /partner/repaircategories`)
   then `listRepairItemTemplates`
   (`GET /partner/repairitemtemplates?where[category_id]=<id>`).
6. `createRepairItem` — `POST /partner/repairitems` with
   `{"repairId": <id>, "repairItemTemplateId": <templateId>}` or a free-text
   `description`. Repeat per item.
7. `updateRepair` — `PUT /partner/repairs/{id}` with
   `{"repairStatus": "SUBMITTED"}` to submit.

## Rules

- After submission, core repair attributes become read-only for the requestor;
  repair notes can always be added.
- Filtering uses `where[field]=value`; relation expansion uses
  `?includes=customer`. No idempotency keys or pagination are documented — see
  `../conventions/upsie-conventions.yml`.
- Verify state anytime with `getRepair` — `GET /partner/repairs/{id}`
  (`repairStatus`, `assignmentStatus`).
