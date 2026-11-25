# How to price a BOQ with the CostTrack API

> **Portfolio sample, not a live product.** Part of the CostTrack API docs sample set. CostTrack is fictional. The examples use the same project (`proj_01J8X`) and the same data as the API reference (`api-reference.md`) and the CLI help (`cli-help-and-errors.md`).

**What you will do:** take a project's priced BOQ, build a new cost estimate from re-measured quantities, submit it as a draft, and check the project total. About 10 minutes, once you have a token.

**What you need:** an API token and `curl`. If you prefer the command line, the same steps work with the CostTrack CLI; see `cli-help-and-errors.md`.

## 1. Get an API token

1. Open the CostTrack dashboard and go to **Settings → API keys**.
2. Create a key and copy the token. Tokens expire after 90 days.
3. Keep the token safe. Every request below sends it as a Bearer token:

```http
Authorization: Bearer <your-api-token>
```

If a request returns `401 unauthorized`, the token is missing, expired or revoked. Create a new one and try again.

## 2. List the project's BOQ items

Pull the current priced BOQ so you are working from the live figures:

```bash
curl "https://api.costtrack.example/v1/projects/proj_01J8X/boq?per_page=100" \
  -H "Authorization: Bearer <your-api-token>"
```

The response lists each item with its item number, description, unit, quantity, rate and amount. `amount` is always `quantity × rate`. Use the returned value; do not recalculate it yourself.

If the project has more than 100 items, page through them with `?page=2`, `?page=3`, and so on. You can also filter by trade, for example `?trade=concrete`.

## 3. Build the estimate file

Write the re-measured items into a JSON file. A ready-made example is in this repo at `examples/estimate.json`:

```json
{
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
}
```

Rules that will reject the whole estimate if broken:

- Every item needs `description`, `unit`, `quantity` and `rate`.
- `quantity` must be greater than 0. `rate` must be 0 or greater.
- `title` is required, maximum 120 characters. `notes` is optional, maximum 1,000 characters.

If any item fails validation, nothing is saved. Fix the named field and submit again.

## 4. Submit the estimate

```bash
curl -X POST https://api.costtrack.example/v1/projects/proj_01J8X/estimates \
  -H "Authorization: Bearer <your-api-token>" \
  -H "Content-Type: application/json" \
  -d @examples/estimate.json
```

A successful submit returns `201 Created` with an estimate ID:

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

The estimate is a **draft**. Submitting one never overwrites the live priced BOQ.

## 5. Check the project total

```bash
curl https://api.costtrack.example/v1/projects/proj_01J8X/summary \
  -H "Authorization: Bearer <your-api-token>"
```

The summary shows the live total broken down by trade, with your draft total listed separately:

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

`draft_estimate` is `null` when no draft exists.

## 6. Review the draft in the dashboard

The last step happens in the dashboard, not the API. Open the project, review the draft estimate line by line, and approve it to make it the live estimate, or reject it with a note. Until it is approved, the live priced BOQ does not change.

## If something goes wrong

| Problem | What it means | Fix |
|---|---|---|
| `401 unauthorized` | Token missing, expired or revoked | Create a new token under Settings → API keys |
| `404 not_found` | Wrong project ID or item number, or no access | Check the ID (it is case-sensitive) and your access |
| `400 invalid_quantity` | A quantity is 0 or negative | Fix the named field, e.g. `items[0].quantity`, and resubmit |
| `400 missing_field` | A required field is absent | Add the named field to the estimate file |
| `429 rate_limited` | Over 100 requests per minute | Wait for the `Retry-After` header (seconds) and retry |

For the full error table, see `api-reference.md`. For the same errors in CLI form, see `cli-help-and-errors.md`.
