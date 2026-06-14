# Web

## Read This For

- Classic HTTP APIs
- Flask/FastAPI/Go/Nim/Phoenix/Node web services
- Frontend plus API stacks
- LiveView, WebSocket, server-pushed UI, preview/export flows

Keep the first pass short:

- map the app
- confirm the trust boundary
- identify likely flag-bearing objects
- only then dive into deeper technique-specific analysis

## High-Value Surfaces

Check these before exotic ideas:

- registration and login
- JWT/session parsing
- profile update and role-bearing fields
- search, listing, gallery, export, preview, share, report, webhook, callback
- object read/write endpoints with both `project_id` and `object_id`
- admin panels, hidden routes, internal API namespaces
- file upload, SVG/PDF rendering, template rendering

## First-Pass Recon

Before deeper analysis:

- inspect page source for comments and hidden hints
- read JS bundles for hidden endpoints and accepted fields
- compare what the UI sends with what the API accepts
- check common paths like `/robots.txt`, `/sitemap.xml`, `/.well-known/`, `/admin`, `/api`, `/debug`
- note custom headers, auth hints, and alternate content types
- try direct API calls that bypass the frontend entirely

## Typical Web Bug Families In A/D

### Auth and Session

- hardcoded JWT secret
- algorithm confusion
- trusting unsigned payload fields
- missing expiry checks
- legacy or alternate verification paths
- service-role or `kind=service` bypasses
- public or weakly protected admin login/bootstrap routes
- cookie seeding or privileged cookies set by public flows
- weak secret or partial-hash validation using `slice`, `substring`, or `startsWith`

### Ownership and IDOR

- endpoint checks container access but not object ownership
- `project_id` is checked, `font_id` or `ticket_id` is not tied back to it
- hidden or internal fields leak through export/share endpoints
- public search or gallery lists private object identifiers

### Server-Side Execution or Fetching

- SSTI in report, template, preview, or message rendering
- SSRF in webhook, callback URL, print service, preview renderer, fetch helper
- following redirects into private address space
- internal-only endpoints reachable via SSRF

### Frontend Trust

- `innerHTML` with user-controlled fields
- LiveView or WebSocket events that only validate access on mount
- hidden buttons do not equal server-side authorization
- client-side access gates based on URL params, globals, or frontend-only checks
- content hidden by CSS or JS but still present in raw HTML or API output

### Query and Search

- raw interpolation in search, filter, sort, or weight parameters
- alternate parsing where partial numeric coercion still leaves attacker input
- error branches that continue into success serialization

### Routing and Edge Behavior

- encoded slash or path normalization mismatches like `%2F`
- middleware applied to one route form but not another
- method confusion: `TRACE`, `PUT`, `PATCH`, `DELETE` succeed where `GET`/`POST` are blocked
- proxy or ingress regex mismatches between allowlists and the application router

## Web Investigation Order

1. Trace auth/session creation and verification.
2. Compare route guards with actual object fetches.
3. Check mass-assignment or profile update schemas.
4. Inspect search and list endpoints for SQL-like interpolation or excessive data return.
5. Inspect export/preview/report/callback/webhook logic.
6. Inspect frontend event handlers only after server handlers are mapped.
7. Check alternate route forms, methods, encodings, and direct API access that bypass intended UI flows.

## Static Coverage Gate Before Validation

Do not start local validation just because two strong hypotheses already exist.
For compact web services, finish this minimal static gate first:

- auth routes fully read
- object CRUD and list/detail routes fully read
- model fields carrying ownership or flags identified
- template sinks and `innerHTML`-style render sinks reviewed
- obvious body/query to ORM lookup paths reviewed
- each reviewed sink classified as escaped, partially escaped, or unescaped
- compose or runtime port exposure reviewed for DB, Redis, broker, admin, or sidecar services
- secret sources reviewed: session secret, JWT/HMAC key, default creds, permissive env fallback

Only after that say the context is sufficient for local validation.
If the gate is incomplete, keep reading source instead of moving to Docker or runtime checks.

## Secret And Internal Surface Checks

Do not leave these implicit.
For small web stacks, explicitly report:

- session secret hardcoded or not
- cookie flags weak or acceptable
- JWT/HMAC/admin secrets present or absent
- whether Mongo/Redis/broker/admin/sidecar ports are published
- if a DB or broker is exposed, whether it appears usable as a direct exploit surface or only as an amplifier

Even when nothing strong is found, say that these were checked.

## Sink Review Rules

When reviewing XSS or render issues:

- enumerate the actual sinks
- name the exact field reaching each sink
- state whether escaping happens at the sink, before the sink, or not at all
- treat string concatenation into `innerHTML` as suspicious until each interpolated field is checked
- report the result as a sink verdict table with one row per reviewed sink
- if there are no relevant template or frontend sinks, include one explicit `none found` row instead of dropping the section

Do not collapse this into one sentence like "frontend XSS checked".
That wording is too lossy and is exactly how real bugs get missed.

## Coverage Pass For Small Web Services

If the service is compact enough to read fully, do not stop at the first confirmed bug.
Run a short residual checklist and report each item as `checked`, `confirmed`, or `not reviewed yet`:

- login and register type confusion
- ORM/operator injection through body or query objects
- ownership on list, detail, and create-linked readback flows
- enumeration via sequential IDs or public counters
- hidden fields accepted by the API but not shown in the UI
- session secret issues, but only after verifying whether the session store makes them exploitable or merely bad hygiene
- strongest chain upgrade:
  - can a path that currently needs a username become `no-username`?
  - can a path that currently needs `flag_id` become `no-flag-id`?

This pass is specifically meant to catch "the report is correct, but incomplete" failures.

## How To Report Web Findings Cleanly

For compact services, separate four buckets:

- `confirmed finding`: directly exploitable bug or clearly flag-relevant weakness
- `exploit amplifier`: public usernames, sequential IDs, leaked object names, predictable counters
- `weak config`: real but lower-value security weakness such as weak cookie flags or hardcoded secrets with no immediate path yet
- `checked but not promoted`: suspicious class reviewed but not confirmed enough to call it a finding

This usually produces better reports than forcing everything into one severity list.

Prefer tables for the final report:

- findings table
- sink inventory table
- negative checks table

For sink inventory, use one row per reviewed sink instead of one summary sentence.
Keep section headers, table titles, column names, and short status labels in the user's language.

## Flag-Focused Questions

- Which field likely stores the flag: note, answer, artifact, design note, resolution, callback response?
- Can public or low-privilege flows render that field indirectly?
- Can an attacker trigger a privileged fetch and read the response back?
- Can an attacker overwrite or publish a victim-owned object, then read it?
