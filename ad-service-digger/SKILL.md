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

## Quick Start

1. Read [references/workflow.md](references/workflow.md) first.
2. Read [references/categories.md](references/categories.md) to classify the service before diving deep.
3. Pick one primary lens:
   - HTTP/UI/API heavy service: read [references/web.md](references/web.md)
   - S3, Kafka, SSH, queues, internal workers, custom protocols: read [references/protocols-and-storage.md](references/protocols-and-storage.md)
   - ELF, `.so`, `i64`, obfuscated sidecar, emulator, crypto challenge: read [references/reverse.md](references/reverse.md)
   - native memory-corruption path, crash triage, exploit-direction work: read [references/pwn.md](references/pwn.md) after [references/reverse.md](references/reverse.md)
4. Read [references/questions.md](references/questions.md) if you need a universal question set for an unfamiliar service.
5. Read [references/operations.md](references/operations.md) when the task includes active rounds, flag theft, preflight, service health, exploit farming, or incident response.
6. Read [references/handoff.md](references/handoff.md) when one agent is digging the service and another agent will later write the exploit or patch from a structured artifact.
7. If the task includes patching or validating fixes, also read [references/defense.md](references/defense.md).
8. If you need examples of patterns that appeared in real A/D digs, read [references/case-studies.md](references/case-studies.md). Treat them as examples, not as an exhaustive service list.

## Operating Rules

- Start from flag flow, not from bug classes. The first win is understanding where the checker writes and reads secrets.
- Treat checker traffic, compose files, DB schemas, mounted volumes, and internal jobs as first-class evidence.
- Prefer confirmed exploit chains over long vulnerability inventories.
- Separate `confirmed on live`, `confirmed in source only`, and `hypothesis`. Attack-defense services often differ between source and deployed binary.
- Prioritize exploit paths that are fast to validate and easy to farm.
- When fixing, prefer minimal changes that break the trust boundary instead of broad refactors.

## Default Output Shape

Produce results in this order unless the user asks differently:

1. Architecture map and flag path
2. Top exploit hypotheses, ordered by payoff and confidence
3. Minimal validation plan
4. Exploit strategy or proof outline
5. Minimal hardening plan
6. Regression checks that protect checker SLA

When handing off to another agent, also produce a structured handoff file using the template in `assets/templates/service-handoff.md`.

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
