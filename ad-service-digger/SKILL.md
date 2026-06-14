---
name: ad-service-digger
description: Attack-defense CTF service analysis and hardening workflow. Use when Codex needs to inspect an unfamiliar service source tree, docker-compose stack, exploit folder, patch set, pcap, binary, or mixed deployment to determine how flags are stored, how the checker interacts with the service, which vulnerabilities are most likely exploitable, how to build a fast proof or farm exploit, and how to apply minimal fixes without breaking checker SLA.
---

# A/D Service Digger

Use this skill to work like an attack-defense teammate, not a generic auditor.
The goal is to answer five questions quickly:

1. What does the service actually do?
2. Where do flags live at rest and in transit?
3. What trust boundaries or invariants look weakest?
4. What is the fastest reliable exploit path?
5. What is the smallest safe fix that keeps the checker happy?

## Modes

Choose the mode from the user's wording before doing any edits:

- `research mode`: the user asks to analyze, inspect, dig, review, map, or find vulnerabilities.
- `patch mode`: the user explicitly asks to fix, patch, harden, close, or validate a fix.

Default to `research mode`.
In `research mode`, do not edit service files, do not silently patch, and do not switch into fixing just because one or two bugs were found.
Only move into `patch mode` when the user explicitly asks for code changes or when the task is clearly live defense.

## Output Language

Match the user's language in the final answer.
If the request is in Russian, write the report in Russian, including section headers and finding labels, unless the user explicitly asks for English.
Apply this to section titles, table titles, column labels, and status labels, not just the prose paragraphs.

## Quick Start

1. Read [references/workflow.md](references/workflow.md) first.
2. Read [references/categories.md](references/categories.md) to classify the service before diving deep.
3. Pick one primary lens:
   - HTTP/UI/API heavy service: read [references/web.md](references/web.md)
   - S3, Kafka, SSH, queues, internal workers, custom protocols: read [references/protocols-and-storage.md](references/protocols-and-storage.md)
   - ELF, `.so`, `i64`, obfuscated sidecar, emulator, crypto challenge: read [references/reverse.md](references/reverse.md)
   - native memory-corruption path, crash triage, exploit-direction work: read [references/pwn.md](references/pwn.md) after [references/reverse.md](references/reverse.md)
4. Read [references/questions.md](references/questions.md) if you need a universal question set for an unfamiliar service.
5. Read [references/traffic-triage.md](references/traffic-triage.md) when the task includes pcap, traffic PDFs, or "are they stealing now?" questions.
6. Read [references/farm-packaging.md](references/farm-packaging.md) when confirmed findings must turn into a clean farm script or teammate-ready archive.
7. Read [references/operations.md](references/operations.md) when the task includes active rounds, flag theft, preflight, service health, exploit farming, or incident response.
8. Read [references/handoff.md](references/handoff.md) when one agent is digging the service and another agent will later write the exploit or patch from a structured artifact.
9. If the task includes patching or validating fixes, also read [references/defense.md](references/defense.md).
10. If you need examples of patterns that appeared in real A/D digs, read [references/case-studies.md](references/case-studies.md). Treat them as examples, not as an exhaustive service list.

## Operating Rules

- Start from flag flow, not from bug classes. The first win is understanding where the checker writes and reads secrets.
- Respect the current mode. In `research mode`, stay in analysis and evidence gathering. In `patch mode`, propose or apply the narrowest safe fix after the findings are clear.
- If source is available, do `source-first` analysis before touching live targets. Generate candidate exploit paths from code and deployment evidence first, then validate locally, and only then do narrow live checks.
- Do not call the context "sufficient" for local validation until static coverage is reasonably complete for the chosen lens. For web services, that means route, model, and render-sink coverage first.
- Treat checker traffic, compose files, DB schemas, mounted volumes, and internal jobs as first-class evidence.
- Prefer confirmed exploit chains over long vulnerability inventories.
- Do not stop after the first one or two confirmed bugs if the service is still small enough to cover. Run one explicit coverage pass before calling the dig complete.
- Separate `confirmed on live`, `confirmed in source only`, and `hypothesis`. Attack-defense services often differ between source and deployed binary.
- Prioritize exploit paths that are fast to validate and easy to farm.
- When fixing, prefer minimal changes that break the trust boundary instead of broad refactors.

## Default Output Shape

Produce results in this order unless the user asks differently:

1. Architecture map and flag path
2. Top exploit hypotheses, ordered by payoff and confidence
3. Minimal validation plan from `source -> local -> live`
4. Static coverage inventory as a compact table
5. Confirmed findings as a compact table, with exploit amplifiers called out separately
6. Negative checks summary as a compact table
7. Coverage pass summary: what classes were checked, what remains weak, config-only, or unverified
8. Exploit strategy or proof outline
9. Fix direction only if the user asked for fixes or the task is live defense
10. Regression checks that would protect checker SLA if a patch is applied

When handing off to another agent, also produce a structured handoff file using the template in `assets/templates/service-handoff.md`.

For medium or larger findings sets, prefer compact Markdown tables over prose for:

- confirmed findings
- sink inventory
- negative checks

For web services, a sink inventory table is required whenever templates, HTML rendering, JS rendering, or frontend sinks exist.
If none were found, still include the table with one explicit `none found` row instead of silently omitting it.
Keep the table title, column names, and status words in the user's language.

Recommended shapes:

```md
| ID | Class | Evidence | Impact | Path/Farmability | Location |
|---|---|---|---|---|---|
| F1 | auth bypass | locally-confirmed | account takeover | no-username | auth.ts:39 |
```

```md
| Sink | Field | Status | Notes |
|---|---|---|---|
| item-detail.ejs innerHTML | bodyPart | unescaped | stored XSS candidate |
```

```md
| Check | Result | Notes |
|---|---|---|
| session secret | weak-only | hardcoded |
| Mongo port | exposed | published in compose |
```

## Common Priorities

Investigate these early because they repeatedly pay off across many A/D services:

- Hardcoded or predictable secrets
- JWT/session confusion and trust in unsigned data
- IDOR and ownership mismatches
- Error paths that continue execution
- SSRF into internal-only endpoints or metadata services
- Path traversal into mounted storage
- Trust in internal headers or sidecar-only routes
- Background jobs that run with stronger privileges than the user-facing API
- Weak ACL checks in brokers, object stores, or SSH helpers
- Predictable RNG, weak crypto, or challenge-state logic
- LiveView/WebSocket/event handlers that skip server-side authorization
- Source/live mismatch where obvious source bugs are dead but the deployed binary still has a different reachable path

## Scope

Do not assume future tasks will resemble any specific prior service.
Generalize from prior material into reusable patterns:

- how to classify the service
- how to reconstruct checker behavior
- how to map trust boundaries
- how to choose exploit direction
- how to patch with minimum SLA risk

Use concrete case names only as optional supporting examples after the generic reasoning is already clear.
