# CostTrack API docs

Developer documentation for CostTrack, a bill-of-quantities and cost-estimation API. This is a **portfolio sample, not a live product**: CostTrack is fictional, and the docs were written to show developer-documentation craft in a domain I know well, quantity surveying and cost estimation.

Written by Sriram Balasubramanian, Singapore. Quantity surveyor and AI training specialist. Native Tamil speaker. Google Data Analytics certified.

## What is in here

| File | What it is |
|---|---|
| `docs/api-reference.md` | Full API reference: 4 endpoints, auth, quick start, request and response examples, error table |
| `docs/cli-help-and-errors.md` | CLI help text for every command, a worked example, and error messages that say what went wrong and how to fix it |
| `docs/how-to-price-a-boq.md` | Step-by-step guide: price a BOQ with the API, from first token to a reviewed draft estimate |
| `docs/release-notes.md` | Release notes for CostTrack v1.1.0 |
| `openapi/costtrack-openapi.yaml` | OpenAPI 3.0 specification for the same 4 endpoints |
| `examples/estimate.json` | Example estimate payload used in the API reference and the how-to guide |

The API reference, the CLI help and the OpenAPI spec all describe the same API: same project (`proj_01J8X`), same data, same error codes. If a detail differs between them, the API reference is the source of truth.

## The API in one paragraph

CostTrack lets an application read and create bill-of-quantities (BOQ) items and cost estimates for a construction project. Base URL `https://api.costtrack.example/v1`, JSON everywhere, Bearer token auth, 100 requests per minute per token. Creating an estimate saves it as a draft for review; it never overwrites the live priced BOQ.

## Why this exists

Technical-writer roles ask for a developer-docs portfolio: an API reference, CLI help text, and a how-to guide, all in Markdown. I built this one around cost estimation because that is my real background: 2 years in construction quantity surveying, cost estimation, tender preparation and contract administration. The examples use real QS knowledge, BOQ items, rates, quantities and cost summaries, rather than invented tech-company experience.

The working API that matches these docs is in the [costtrack-api](https://github.com/sriram2932/costtrack-api) repo. Same project, same data, same error codes.

## Status

Draft for review, November 2026. The API, the CLI and the company are fictional. Nothing here connects to a real service.
