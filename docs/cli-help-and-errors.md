# CostTrack CLI, Help Text and Error Messages (Sample)

> **Portfolio sample, not a live product.** Written by Sriram Balasubramanian as a
> developer-documentation sample for technical-writer applications (Mercor Technical
> Writer, AfterQuery Technical Writer Expert, Ethos AI Technical Writer).
> CostTrack is a fictional bill-of-quantities / cost-estimation tool, chosen so the
> examples use real quantity-surveying knowledge, BOQ items, rates, quantities and
> cost summaries, rather than invented tech-company experience. It is the companion
> to the CostTrack API reference (`api-reference-boq-api.md`): same project
> (`proj_01J8X`), same data, same error codes where they overlap.
> Status: draft for Sriram's review, 18 Nov 2026. Not published anywhere yet.

## What the CLI is

`costtrack` is a command-line tool for working with a project's priced BOQ
without opening the dashboard. Read BOQ items, check a project cost summary,
and submit a cost estimate as a draft for review, creating an estimate never
overwrites the live priced BOQ.

Install it, then sign in once with `costtrack auth login`. The CLI stores the
token locally; tokens expire after 90 days.

```bash
costtrack auth login
```

---

## `costtrack --help`

This is the full top-level help output, exactly as a user would see it:

```text
costtrack, work with a project's BOQ from the command line

Usage:
  costtrack <command> [options]

Commands:
  auth      Sign in and manage your API token
  boq       List and read BOQ items for a project
  estimate  Create a cost estimate (saved as a draft)
  summary   Show a project's rolled-up cost by trade
  help      Show help for a command

Options:
  -h, --help     Show this help and exit
  -v, --version  Show the costtrack version and exit

Run `costtrack <command> --help` for the options of one command.
Examples:
  costtrack boq list --project proj_01J8X --trade concrete
  costtrack summary --project proj_01J8X
```

### `costtrack boq --help`

```text
costtrack boq, read BOQ items for a project

Usage:
  costtrack boq list --project <id> [--trade <trade>] [--page <n>] [--per-page <n>]
  costtrack boq get  --project <id> --item <item-no>

Options:
  --project <id>   Project ID, e.g. proj_01J8X (required)
  --trade <trade>  Filter list by trade, e.g. concrete, steel, finishes
  --page <n>       Page number, starting at 1 (default: 1)
  --per-page <n>   Items per page, 1-100 (default: 50)
  --item <item-no> BOQ item number, e.g. 3.1 (required for `get`)

Examples:
  costtrack boq list --project proj_01J8X --trade concrete --per-page 2
  costtrack boq get --project proj_01J8X --item 3.1
```

### `costtrack estimate --help`

```text
costtrack estimate, create a cost estimate for a project

Usage:
  costtrack estimate create --project <id> --title <text> --file <path>

Options:
  --project <id>  Project ID, e.g. proj_01J8X (required)
  --title <text>  Short name for the estimate, e.g. "November re-measure"
                  (required, max 120 characters)
  --file <path>   JSON file with the estimate items (required).
                  Each item needs description, unit, quantity and rate.
                  quantity must be greater than 0; rate must be 0 or greater.

The estimate is saved as a draft and must be reviewed in the dashboard
before it becomes the live estimate. If any item fails validation, the
whole estimate is rejected, no partial estimate is saved.

Example:
  costtrack estimate create --project proj_01J8X \
    --title "November re-measure" --file estimate.json
```

### `costtrack summary --help`

```text
costtrack summary, show a project's rolled-up cost

Usage:
  costtrack summary --project <id>

Options:
  --project <id>  Project ID, e.g. proj_01J8X (required)

Shows the live estimate total, broken down by trade, with the draft
estimate total shown separately if one exists.

Example:
  costtrack summary --project proj_01J8X
```

### `costtrack auth --help`

```text
costtrack auth, sign in and manage your API token

Usage:
  costtrack auth login
  costtrack auth logout

`login` opens the CostTrack dashboard so you can approve the CLI, then
stores the token on this machine. Tokens expire after 90 days. Run
`login` again when the CLI tells you the token has expired.
```

---

## Worked example

List the concrete BOQ items, then check the project total:

```bash
$ costtrack boq list --project proj_01J8X --trade concrete --per-page 2

item_no  description                              unit  quantity  rate     amount
3.1      Grade 40 reinforced concrete, columns    m3    120.5     185.00   22,292.50
3.2      Grade 40 reinforced concrete, beams      m3    86.0      185.00   15,910.00

Page 1 of 19 · 37 items · currency SGD
```

```bash
$ costtrack summary --project proj_01J8X

Project: proj_01J8X (SGD)
Live total:      1,842,500.00
  concrete:        642,300.00
  steel:           518,400.00
  finishes:        681,800.00
Draft estimate:  est_9K2M4Q, 22,292.50 (November re-measure)
```

`amount` is always `quantity × rate`, rounded to 2 decimal places.

---

## Error messages

Each error says what went wrong, why, and what to do next. The `code` in
parentheses matches the API error code in `api-reference-boq-api.md`, so a
user searching either doc finds the same fix.

### 1. Not signed in (`unauthorized`)

```text
Error: you are not signed in. (unauthorized)

Your API token is missing or has expired (tokens last 90 days).
Run `costtrack auth login` and try again.
```

### 2. Project or item not found (`not_found`)

```text
Error: project proj_01J8X was not found. (not_found)

Check the project ID, it is case-sensitive, and that you have access
to the project. To list the projects you can access, open the CostTrack
dashboard.
```

For a missing BOQ item, the same shape applies:

```text
Error: BOQ item 9.9 was not found in project proj_01J8X. (not_found)

Check the item number and project ID. Use
`costtrack boq list --project proj_01J8X` to see the items that exist.
```

### 3. Invalid estimate item (`invalid_quantity`)

```text
Error: estimate not created, quantity must be greater than 0.
(invalid_quantity, field: items[0].quantity)

Fix the quantity in estimate.json and run the command again. Nothing
was saved: if any item fails validation, the whole estimate is rejected.
```

The same pattern covers `invalid_rate` ("rate must be 0 or greater") and
`missing_field` (names the missing field, e.g. `items[0].description`).

### 4. Rate limited (`rate_limited`)

```text
Error: too many requests, you have exceeded 100 requests per minute.
(rate_limited)

Wait 30 seconds and try again. If you are running the CLI in a script,
add a short pause between calls instead of retrying immediately.
```

## Changelog

| Date | Change |
|---|---|
| 2026-11-18 | First draft: top-level and per-command help, worked example, 4 error messages. |
