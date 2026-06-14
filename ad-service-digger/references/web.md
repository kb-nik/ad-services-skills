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

## Flag-Focused Questions

- Which field likely stores the flag: note, answer, artifact, design note, resolution, callback response?
- Can public or low-privilege flows render that field indirectly?
- Can an attacker trigger a privileged fetch and read the response back?
- Can an attacker overwrite or publish a victim-owned object, then read it?
