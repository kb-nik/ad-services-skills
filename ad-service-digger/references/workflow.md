# Workflow

## Core Loop

Use this sequence unless there is a strong reason to reorder it:

1. Probe the service from the outside.
2. Read the compose or deployment layout.
3. Find data stores, mounts, and internal-only components.
4. Reconstruct checker behavior from pcap, logs, scripts, or farm traces.
5. Map where flags are written, transformed, cached, and returned.
6. Choose the cheapest exploit path that reaches the same data.
7. Patch the narrowest broken trust boundary.
8. Recheck functionality and exploitability.

## Step 1: External Probe

Learn what the service claims to be before reading code.

- List ports and protocols.
- Hit the obvious HTTP/UI endpoints.
- Try registration, login, object creation, sharing, export, search, upload, webhook, and profile flows.
- For non-web services, identify the banner, handshake, auth scheme, framing, and error behavior.
- Capture any user-controlled identifiers that may later become path names, topic names, repo names, or DB keys.

Questions to answer:

- What identities exist?
- What objects exist?
- Which actions look stateful?
- What endpoints or commands smell like admin, internal, export, preview, debug, import, analytics, callback, or worker hooks?

## Step 2: Deployment and Architecture

Read the stack layout next.

- `docker-compose.yml`, Dockerfiles, entrypoints, supervisord configs, nginx/caddy configs
- Internal ports versus published ports
- Sidecars, workers, brokers, cron jobs, analytics jobs, cleanup jobs
- Volumes and bind mounts
- Env vars for secrets, admin passwords, signing keys, broker creds, seed data

Build a small mental map:

- public surface
- auth authority
- data authority
- background privileged components
- storage path for flags

## Step 3: Data and Storage

Find where valuable data lives.

- Postgres/MySQL/SQLite schemas
- Redis, ClickHouse, Elastic, files, object storage, Kafka topics
- mounted folders like `storage`, `uploads`, `repos`, `s3data`
- tables or fields with notes, answers, artifacts, messages, metadata, callback responses, access tokens

Look for:

- ownership columns
- raw tokens stored in cleartext
- fields likely to hold flags
- generated names that checker later fetches

## Step 4: Checker Reconstruction

This is often the highest-value step.

Use:

- pcap files
- checker scripts
- exploit harnesses like `start_sploit.py`
- logs or captured requests

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

## Step 5: Hypothesis Generation

Generate exploit ideas from real trust boundaries, not generic checklists.

Good questions:

- Can I become another identity?
- Can I make the service act on my behalf with stronger privileges?
- Can I turn an internal-only flow into a public one?
- Can I confuse ownership between object A and container B?
- Can I reuse the checker's own retrieval path?
- Can I hit the privileged backend through SSRF, worker jobs, broker RPC, or forged events?

Rank hypotheses by:

- expected flag yield
- validation cost
- dependency on source/live match
- farmability

## Step 6: Proof

Confirm the path with the smallest safe proof first.

- One curl request is better than a 300-line exploit.
- One forged token is better than a full account workflow if both prove the same bug.
- For reverse tasks, confirm one primitive like file read, role flip, or decrypted response before polishing.

Record evidence as:

- confirmed
- partially confirmed
- inferred

Also record where it is confirmed:

- source only
- local deployment
- live target
- multi-target farm

## Step 7: Fix

Patch the narrowest broken invariant.

Examples:

- rotate a secret and reject legacy verification
- enforce `resource.owner_id == current_user.id`
- stop following redirects to private IP space
- remove trust in `X-Internal-Request`
- block dangerous fields from mass assignment
- enforce path normalization and reject `..`
- replace public signing endpoint with server-side session

## Step 8: Recheck

Always validate both sides:

- the service still performs the checker's expected workflow
- the exploit path is dead

Minimum checks:

- registration/login still works if the checker uses it
- object creation still works
- flag retrieval path still works for the legitimate owner
- original exploit or a reduced proof no longer works
