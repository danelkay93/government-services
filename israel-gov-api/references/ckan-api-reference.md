# data.gov.il CKAN API Reference

## Base URL
`https://data.gov.il/api/3/`

## Authentication
None required (public API).

## Key Endpoints

### Dataset Discovery
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/action/package_search` | GET | Search datasets by keyword |
| `/action/package_show` | GET | Get full dataset details |
| `/action/package_list` | GET | List all dataset IDs |
| `/action/group_list` | GET | List dataset categories |
| `/action/organization_list` | GET | List publishing organizations |
| `/action/tag_list` | GET | List all tags |

### Resource Access
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/action/resource_show` | GET | Get resource metadata |
| `/action/datastore_search` | GET | Query tabular data |

### Search Parameters
- `q` -- Search query string
- `fq` -- Filter query (e.g., `organization:lamas`)
- `rows` -- Number of results (default 10)
- `start` -- Offset for pagination
- `sort` -- Sort field and order

### Datastore Query Parameters
- `resource_id` -- Resource ID (required)
- `limit` -- Maximum records per page (default 100). There is no hard ceiling, but deep `offset` paging gets progressively slower.
- `offset` -- Record offset; the response also returns `_links.next` for the next page. `filters` is exact-match only, so there is no `_id` keyset paging THROUGH `filters`. For a full extraction use `datastore_search_sql` with `WHERE _id > N ORDER BY _id`, which is true keyset paging and does not degrade with depth, or pull `records_format=csv`.
- `fields` -- Comma-separated field names
- `filters` -- JSON object of field:value pairs
- `q` -- Full-text search within resource
- `sort` -- Sort field and order (e.g. `sort=_id asc` for a stable ordering across offset pages)
- `records_format` -- One of `objects` (default), `lists`, `csv`, `tsv`. `lists` and `csv` are noticeably faster for large pulls.

### Endpoint status notes
| Endpoint | Status | Notes |
|----------|--------|-------|
| `/action/datastore_search_sql` | Historical observation, 2026-08-27 | Several SQL forms returned data; some patterns returned HTML 404. That response alone cannot establish its cause. Do not rewrite or probe a denied query. For permitted requests, `datastore_search` supplies a matching-record `total` without SQL. |

## Common Organization IDs

Verified against `organization_show?id=<id>` on 2026-06-16. Stale aliases like `cbs`, `mot`, `moh`, `moe`, `ita` return 404 and must not be used.

| Organization | ID | Hebrew |
|--------------|-----|--------|
| Central Bureau of Statistics | `lamas` | הלשכה המרכזית לסטטיסטיקה |
| Ministry of Transportation | `ministry_of_transport` | משרד התחבורה |
| Ministry of Health | `ministry-health` | משרד הבריאות |
| Ministry of Education | `ministry_of_education` | משרד החינוך |
| Israel Tax Authority | `taxes-authority` | רשות המסים |
| Israel Land Authority | `the_israel_lands_administration` | רשות מקרקעי ישראל |
| Ministry of Interior | `interior_affairs` | משרד הפנים |

Run `curl -s "https://data.gov.il/api/3/action/organization_list" | head -200` to enumerate the canonical IDs; ministry IDs change without notice.

## Response Format
Successful CKAN responses normally return JSON with the following envelope. HTTP errors may contain a useful JSON `error` (including HTTP 409) or a non-JSON body; inspect status and shape before parsing:
```json
{
  "success": true/false,
  "result": {
    "total": 0,
    "records": [],
    "fields": []
  },
  "error": { ... }
}
```

The `result.total` field reports the count matching that request's filters (not the page); use it to plan pagination. Requests do not retain earlier filters. For `lists`, map each row position to `result.fields` in order. CSV/TSV bodies are headerless: write a quoted header from those fields with the corresponding comma/tab delimiter. The helper emits delimited exports without a prose preamble; `--json` preserves the raw result.

## Rate Limits
No current rate limit is verified here. Historical successful bursts do not rule out throttling now. Follow documented limits and Retry-After when supplied. An explicit security/access denial (including 403 "Security Violation") requires stopping the operation, reporting it and contacting official support. Do not change cookies, User-Agent, client, proxy or query shape to evade it. An ambiguous HTML 404 calls for checking existing metadata and official documentation, not probing alternative queries. Ordinary structured validation errors can be corrected; documented alternative access is usable only if it does not evade a denial.
