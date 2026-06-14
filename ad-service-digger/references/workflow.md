# Workflow

## Core Loop

Use this sequence unless there is a strong reason to reorder it:

1. Read the source tree and deployment layout.
2. Find data stores, mounts, and internal-only components.
3. Reconstruct checker behavior from source, pcap, logs, scripts, or farm traces.
4. Map where flags are written, transformed, cached, and returned.
5. Generate exploit hypotheses from code and trust boundaries.
6. Finish one static coverage inventory for the chosen lens.
7. Do the lightest local or external validation that separates good hypotheses from bad ones.
8. Run one explicit coverage pass so the dig does not end at the first confirmed path.
9. Use live checks only on the strongest candidates.
10. Patch the narrowest broken trust boundary.
11. Recheck functionality and exploitability.

If source is absent or clearly stale, fall back to the old order and use external probing first.

If the task is in `research mode`, stop after step 9 and report findings.
Only do steps 10 and 11 in `patch mode` or explicit live-defense work.

## Step 1: Source And Deployment First

Start from code and service layout whenever they exist.

- classify the repository shape
- identify entrypoints, routers, handlers, workers, cron jobs, sidecars, binaries
- read `docker-compose.yml`, Dockerfiles, entrypoints, nginx/caddy configs, supervisord files
- find secrets, signing keys, broker creds, admin bootstrap, debug toggles
- mark user-controlled fields that cross trust boundaries

Questions to answer:

- Which component is the auth authority?
- Which component is the data authority?
- Which routes, commands, jobs, or helper binaries look privileged?
- Which code paths are likely checker-facing?

## Step 2: Data And Internal Components

Find the real storage and privileged paths early.

- Postgres/MySQL/SQLite schemas
- Redis, ClickHouse, Elastic, files, object storage, Kafka topics
- mounted folders like `storage`, `uploads`, `repos`, `s3data`
- tables or fields with notes, answers, artifacts, messages, metadata, callback responses, access tokens
- sidecars, workers, brokers, cron jobs, analytics jobs, cleanup jobs

Look for:

- ownership columns
- raw tokens stored in cleartext
- fields likely to hold flags
- generated names that checker later fetches
- components that can read more than the public API can

## Step 3: Checker Reconstruction

This is often the highest-value step.

Use:

- checker scripts
- pcap files
- exploit harnesses like `start_sploit.py`
- logs or captured requests
- code paths that create, transform, or read flag-bearing objects

Extract:

- full happy-path write flow
- full happy-path read flow
- object naming pattern
- auth method used by checker
- internal services contacted during the round

Prefer concrete statements such as:

- "checker creates a private repo, writes the flag into wiki, then fetches it by token"
- "checker uploads an S3 object and later reads it through analytics CSV"
- "checker becomes detective before reading all suspects"

## Step 4: Flag Map And Hypothesis Generation

Generate exploit ideas from real trust boundaries, not generic checklists.

Good questions:

- Can I become another identity?
- Can I make the service act on my behalf with stronger privileges?
- Can I turn an internal-only flow into a public one?
- Can I confuse ownership between object A and container B?
- Can I reuse the checker's own retrieval path?
- Can I hit the privileged backend through SSRF, worker jobs, broker RPC, forged events, or native primitives?

Rank hypotheses by:

- expected flag yield
- validation cost
- dependency on source/live match
- farmability

Produce a short candidate list before touching live:

- `H1`, `H2`, `H3`
- exact files, routes, functions, or objects involved
- what proof would confirm or kill each idea
- whether each idea is:
  - a direct finding
  - an exploit amplifier like public usernames, sequential IDs, or leaked object names
  - a weak config or hygiene issue that matters but is not the primary path

## Step 5: Static Coverage Inventory

Before local validation, make a compact inventory for the chosen lens.

For web services, cover at least:

- all auth routes
- all object create/read/list/update/delete routes
- all render sinks in templates or frontend code
- all models and the fields likely to carry flags or ownership
- all obvious query/body entrypoints that reach ORM lookups
- all published data-plane ports like Mongo, Redis, broker, sidecar, admin, raw socket
- obvious secret sources: hardcoded session secrets, JWT/HMAC keys, default creds, env fallbacks

The inventory does not need to be verbose, but it must be complete enough that the agent can say:

- which routes were reviewed
- which sinks were reviewed
- which classes are still unreviewed
- which sinks were confirmed escaped versus unescaped
- which internal or data-plane ports were reviewed and whether they are exposed
- which secret/config classes were reviewed and whether they were exploitable, weak-only, or absent

Write the inventory in the user's language, including section title, table title, column names, and short status labels.
For web services, the sink inventory must be a table with one row per reviewed sink.
If no relevant template or frontend sink exists, include one explicit row stating that none were found.

For frontend or template sinks, do not write blanket claims like "XSS checked" or "output is escaped" unless you can enumerate the actual sinks you reviewed.
Prefer statements like:

- `item-detail.ejs`: `description` escaped, `bodyPart` unescaped
- `index.ejs`: username escaped
- `feed.ejs`: description escaped

Route and sink coverage should be explicit enough that another agent can spot what you may have missed.

Do not move to local validation while major route groups or sinks are still unread.

## Step 6: External Probe

Only now use the outside view to refine or reject source-based hypotheses.

Learn what the service actually exposes and whether it matches the source assumptions.

- list ports and protocols
- hit the obvious HTTP/UI endpoints
- try registration, login, object creation, sharing, export, search, upload, webhook, and profile flows
- for non-web services, identify the banner, handshake, auth scheme, framing, and error behavior
- capture any user-controlled identifiers that may later become path names, topic names, repo names, or DB keys

Questions to answer:

- What identities exist in practice?
- What objects exist in practice?
- Which actions look stateful?
- What endpoints or commands smell like admin, internal, export, preview, debug, import, analytics, callback, or worker hooks?
- Which source hypotheses survive contact with the real exposed surface?

## Step 7: Proof

Confirm the path with the smallest safe proof first.

- One curl request is better than a 300-line exploit.
- One forged token is better than a full account workflow if both prove the same bug.
- For reverse tasks, confirm one primitive like file read, role flip, or decrypted response before polishing.
- Prefer local or lab confirmation before live confirmation whenever the environment allows it.

Record evidence as:

- confirmed
- partially confirmed
- inferred

Also record where it is confirmed:

- source only
- local deployment
- live target
- multi-target farm

Only use live checks on the shortest list of candidates that survived source review and local proof.
Avoid broad live fuzzing when code already narrowed the search space.

## Step 8: Coverage Pass

Before calling the service "done", force one residual pass across the remaining classes.

For a small or medium web service, explicitly ask:

- Did we check auth type confusion on every login-like path?
- Did we check ownership on read, write, list, and indirect render paths separately?
- Did we check sequential IDs, public listings, and object enumeration?
- Did we check query/operator injection, especially where body fields go straight into ORM lookups?
- Did we check UI/API mismatch and hidden-but-accepted fields?
- Did we check whether the strongest finding can be made even stronger, for example `needs username` to `no-username`, or `needs flag_id` to `no-flag-id`?

Do not inflate the report with fake certainty.
If a class was not checked, mark it as `not reviewed yet` instead of silently omitting it.
If something is real but secondary, label it explicitly as:

- `exploit amplifier`
- `weak config`
- `checked but not promoted`

instead of mixing it into the main confirmed exploit list.

Also force a short negative-check report for:

- hardcoded secrets checked: `confirmed | weak-only | not found`
- cookie/session flags checked: `weak | acceptable | not reviewed yet`
- published DB/broker/internal ports checked: `exposed | internal-only | not reviewed yet`

## Step 9: Live Check

Live validation comes last, not first.

Use it to answer only the remaining high-value questions:

- does the local proof match the deployed target
- is the source/live mismatch real
- does the exploit path reach the same flag flow on a real team
- is the path stable enough to become `live-confirmed`

Keep live checks narrow:

- confirm one route, object, or primitive
- stop after the hypothesis is proven or killed
- record exact mismatch if live diverges from source

## Step 10: Fix

Enter this step only when the user explicitly asked for a patch/fix/hardening pass, or when the task is active defense during a live round.

Patch the narrowest broken invariant.

Examples:

- rotate a secret and reject legacy verification
- enforce `resource.owner_id == current_user.id`
- stop following redirects to private IP space
- remove trust in `X-Internal-Request`
- block dangerous fields from mass assignment
- enforce path normalization and reject `..`
- replace public signing endpoint with server-side session

## Step 11: Recheck

Always validate both sides:

- the service still performs the checker's expected workflow
- the exploit path is dead

Minimum checks:

- registration/login still works if the checker uses it
- object creation still works
- flag retrieval path still works for the legitimate owner
- original exploit or a reduced proof no longer works
