# CostTrack API, API Reference (Sample)

> **Portfolio sample, not a live product.** Written by Sriram Balasubramanian as a
> developer-documentation sample for technical-writer applications (Mercor Technical
> Writer, AfterQuery Technical Writer Expert, Ethos AI Technical Writer).
> CostTrack is a fictional bill-of-quantities / cost-estimation API, chosen so the
> examples use real quantity-surveying knowledge, BOQ items, rates, quantities and
> cost summaries, rather than invented tech-company experience.
> Status: draft for Sriram's review, 18 Nov 2026. Not published anywhere yet.

## Overview

The CostTrack API lets an application read and create bill-of-quantities (BOQ)
items and cost estimates for a construction project. Use it to pull a project's
priced BOQ into a reporting tool, or to submit a new estimate for review.

- **Base URL:** `https://api.costtrack.example/v1`
- **Format:** JSON for every request and response.
- **Auth:** Bearer token in the `Authorization` header. Get a token from the
  CostTrack dashboard under **Settings → API keys**. Tokens expire after 90 days.
- **Rate limit:** 100 requests per minute per token. Exceeding it returns `429`.
  See [Errors](#errors).

```http
Authorization: Bearer <your-api-token>
Content-Type: application/json
```

## Quick start

```bash
curl https://api.costtrack.example/v1/projects/proj_01J8X/boq \
  -H "Authorization: Bearer <your-api-token>"
```

Returns the priced BOQ for project `proj_01J8X`, see
[List BOQ items](#list-boq-items).

---

## List BOQ items

Get every priced BOQ item for a project, in bill order.

```http
GET /projects/{project_id}/boq
```

### Path parameters

| Parameter | Type | Description |
|---|---|---|
| `project_id` | string | The project ID, e.g. `proj_01J8X`. |

### Query parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number, starting at 1. |
| `per_page` | integer | `50` | Items per page. Maximum `100`. |
| `trade` | string | - | Filter by trade, e.g. `concrete`, `steel`, `finishes`. |

### Example request

```bash
curl "https://api.costtrack.example/v1/projects/proj_01J8X/boq?trade=concrete&per_page=2" \
  -H "Authorization: Bearer <your-api-token>"
```

### Example response, `200 OK`

```json
{
  "project_id": "proj_01J8X",
  "currency": "SGD",
  "page": 1,
  "per_page": 2,
  "total_items": 37,
  "items": [
    {
      "item_no": "3.1",
      "description": "Grade 40 reinforced concrete, columns",
      "unit": "m3",
      "quantity": 120.5,
      "rate": 185.00,
      "amount": 22292.50,
      "trade": "concrete"
    },
    {
      "item_no": "3.2",
      "description": "Grade 40 reinforced concrete, beams",
      "unit": "m3",
      "quantity": 86.0,
      "rate": 185.00,
      "amount": 15910.00,
      "trade": "concrete"
    }
  ]
}
```

`amount` is always `quantity × rate`, rounded to 2 decimal places. Do not
recalculate it client-side, use the value returned.

---

## Get a BOQ item

Get one BOQ item by its item number.

```http
GET /projects/{project_id}/boq/{item_no}
```

### Path parameters

| Parameter | Type | Description |
|---|---|---|
| `project_id` | string | The project ID. |
| `item_no` | string | The BOQ item number, e.g. `3.1`. URL-encode it if it contains spaces. |

### Example request

```bash
curl https://api.costtrack.example/v1/projects/proj_01J8X/boq/3.1 \
  -H "Authorization: Bearer <your-api-token>"
```

### Example response, `200 OK`

```json
{
  "item_no": "3.1",
  "description": "Grade 40 reinforced concrete, columns",
  "unit": "m3",
  "quantity": 120.5,
  "rate": 185.00,
  "amount": 22292.50,
  "trade": "concrete",
  "updated_at": "2026-11-10T09:30:00+08:00"
}
```

Returns `404` if the item number does not exist in that project.

---

## Create a cost estimate

Submit a new cost estimate for a project. The estimate is saved as **draft**
and must be reviewed in the CostTrack dashboard before it becomes the live
estimate, creating one never overwrites the current priced BOQ.

```http
POST /projects/{project_id}/estimates
```

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | A short name, e.g. `"November re-measure"`. Max 120 characters. |
| `items` | array | Yes | At least one BOQ item. See below. |
| `items[].description` | string | Yes | What is being priced. |
| `items[].unit` | string | Yes | Unit of measurement, e.g. `m3`, `m2`, `nr`. |
| `items[].quantity` | number | Yes | Must be greater than 0. |
| `items[].rate` | number | Yes | Unit rate in the project currency. Must be 0 or greater. |
| `notes` | string | No | Free-text notes for the reviewer. Max 1,000 characters. |

### Example request

```bash
curl -X POST https://api.costtrack.example/v1/projects/proj_01J8X/estimates \
  -H "Authorization: Bearer <your-api-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "November re-measure",
    "items": [
      {
        "description": "Grade 40 reinforced concrete, columns",
        "unit": "m3",
        "quantity": 120.5,
        "rate": 185.00
      }
    ],
    "notes": "Re-measured from Revit model rev C."
  }'
```

### Example response, `201 Created`

```json
{
  "estimate_id": "est_9K2M4Q",
  "project_id": "proj_01J8X",
  "title": "November re-measure",
  "status": "draft",
  "total_amount": 22292.50,
  "currency": "SGD",
  "created_at": "2026-11-18T09:00:00+08:00"
}
```

`total_amount` is the sum of `quantity × rate` across all items. If any item
fails validation, the whole request is rejected, no partial estimate is saved.

---

## Get a project cost summary

Get the rolled-up cost of a project: the live estimate total, broken down by
trade, with the draft estimate total shown separately if one exists.

```http
GET /projects/{project_id}/summary
```

### Example request

```bash
curl https://api.costtrack.example/v1/projects/proj_01J8X/summary \
  -H "Authorization: Bearer <your-api-token>"
```

### Example response, `200 OK`

```json
{
  "project_id": "proj_01J8X",
  "currency": "SGD",
  "live_total": 1842500.00,
  "by_trade": [
    { "trade": "concrete", "total": 642300.00 },
    { "trade": "steel", "total": 518400.00 },
    { "trade": "finishes", "total": 681800.00 }
  ],
  "draft_estimate": {
    "estimate_id": "est_9K2M4Q",
    "total_amount": 22292.50
  }
}
```

`draft_estimate` is `null` when no draft exists. `by_trade` totals always add
up to `live_total`.

---

## Errors

Errors use the same JSON shape for every endpoint:

```json
{
  "error": {
    "code": "invalid_quantity",
    "message": "quantity must be greater than 0.",
    "field": "items[0].quantity"
  }
}
```

| Status | Code | Meaning / what to do |
|---|---|---|
| `400` | `invalid_quantity`, `invalid_rate`, `missing_field`, `invalid_field` | A field failed validation. Fix the named `field` and retry. The request was not saved. `invalid_field` covers values that are present but out of range, e.g. a title over 120 characters or notes over 1,000 characters. |
| `401` | `unauthorized` | Missing, expired or revoked token. Get a new token from the dashboard. |
| `404` | `not_found` | The project or item number does not exist, or you do not have access to it. |
| `429` | `rate_limited` | Over 100 requests/minute. Wait and retry; the response includes a `Retry-After` header in seconds. |
| `500` | `server_error` | Something went wrong on our side. Retry once; if it persists, contact support with the request ID from the response headers. |

## Changelog

| Date | Change |
|---|---|
| 2026-11-18 | First draft: 4 endpoints, error table, quick start. |
