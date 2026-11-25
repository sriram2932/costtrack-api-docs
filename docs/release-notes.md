# CostTrack release notes — v1.1.0 (18 November 2026)

> **Portfolio sample, not a live product.** CostTrack is a fictional bill-of-quantities and cost-estimation API. These notes are a documentation sample.

## What is new

### Draft estimates

You can now submit a cost estimate through the API (`POST /projects/{project_id}/estimates`) and it is saved as a **draft** for review. Drafts never overwrite the live priced BOQ. Review and approve a draft in the CostTrack dashboard before it becomes the live estimate. If any item in a submission fails validation, the whole estimate is rejected and nothing is saved.

### Project cost summary

A new endpoint, `GET /projects/{project_id}/summary`, returns the live estimate total broken down by trade, with the draft estimate total shown separately when one exists. Use it to check a project total without pulling the full BOQ.

### CostTrack CLI

The `costtrack` command-line tool is now available. List and read BOQ items, check a project summary, and submit a draft estimate without opening the dashboard. See `cli-help-and-errors.md` for the full help text. The CLI uses the same error codes as the API, so a fix found in one doc works in the other.

## Changes

- `GET /projects/{project_id}/boq` now accepts a `trade` filter (`concrete`, `steel`, `finishes`) and a `per_page` of up to 100 items.
- `amount` on BOQ items is always `quantity × rate`, rounded to 2 decimal places. Use the returned value instead of recalculating it client-side.
- API tokens now expire after 90 days. Create a new token under **Settings → API keys** when the API returns `401 unauthorized`.

## Known issues

- The dashboard does not yet show a history of rejected draft estimates. Rejected submissions are not saved anywhere; resubmit with the corrected fields.
- `by_trade` totals in the summary are rounded per trade, so in rare cases the displayed trade totals can differ from the live total by a few cents. The live total is the correct figure.

## Previous release

### v1.0.0 (10 November 2026)

First release. Read BOQ items for a project (`GET /projects/{project_id}/boq` and `GET /projects/{project_id}/boq/{item_no}`), Bearer token auth, 100 requests per minute per token.
