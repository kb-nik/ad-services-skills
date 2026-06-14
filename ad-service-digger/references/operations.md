# Operations

## Read This For

- active attack-defense rounds
- suspected ongoing flag theft
- preflight and health diagnostics
- exploit farming readiness
- service-closed or post-patch validation
- incident response while the round is live

## Operating Mindset

A strong source finding is not enough during a live round.
Default to `source-first` reasoning when source exists, but delay live interaction until you have a ranked candidate list.
Track four separate states:

1. `source-suspected`
2. `locally-confirmed`
3. `live-confirmed`
4. `farm-ready`

Do not treat these as interchangeable.

## Evidence Ladder

Use the strongest available evidence:

- source-only reasoning
- local reproduction
- checker traffic match
- live exploit proof
- repeated multi-team farming success

Explicitly label which level you have.

Default progression:

1. source reasoning
2. local reproduction
3. checker match
4. live proof
5. farming

Do not skip straight from vague source suspicion to wide live probing unless incident pressure leaves no safer option.

## When Flags Are Being Stolen

Switch from general auditing to incident triage.

Questions:

- Which service is actually losing flags?
- Is the loss confirmed by telemetry like `flag_out`, stream logs, or repeated checker failure?
- Is the service still vulnerable after the latest patch?
- Is the issue exploitability, availability, or checker incompatibility?

Immediate actions:

- confirm the victim service and round timing
- correlate incoming exploit traffic with the suspected path
- prefer a narrow emergency mitigation over a broad refactor
- retest legitimate checker flow after any emergency change

## Preflight And Health

Treat preflight as supporting evidence, not as proof of exploitability.

Useful signals:

- service up/down
- disk pressure
- docker compose status
- NTP drift
- key monitoring or farm endpoints
- published service availability

Do not confuse:

- warning-level observability failures
- with real exploit paths
- or with the root cause of flag theft

## Service States

Classify the service status explicitly:

- `vulnerable`
- `closed`
- `unknown`
- `broken`
- `checker-broken`

Use `closed` only after both are true:

- the exploit path no longer works
- the legitimate service behavior still works

## Exploit Lifecycle

An exploit becomes operational in stages:

1. primitive works
2. flag path works once
3. path matches checker reality
4. exploit survives network noise and timing
5. exploit scales across teams

Before calling something farm-ready, check:

- target discovery
- retries and timeouts
- duplicate flag handling
- graceful failure logging
- stability after partial service degradation

For packaging details after the path is confirmed, use [farm-packaging.md](farm-packaging.md).

## Traffic-Guided Reasoning

If traffic or stream logs are available, use them to answer:

- what exact object names checker uses
- whether attackers are reading or overwriting
- whether exploitation is HTTP, TCP, SSH, broker, or worker-driven
- whether the attacker cleans up artifacts after use

Traffic often reveals the real exploit path faster than static review.
For stricter leak versus echo classification, use [traffic-triage.md](traffic-triage.md).

## Patch Map

When defending under time pressure, build a patch map:

- file
- line range
- bug class
- minimal change
- user-facing risk
- validation command or proof

This helps avoid random edits and speeds rollback reasoning.

## Live Commit Hygiene

If a live code change must be committed or handed to teammates:

- stage only intended files
- never commit runtime data, DB files, uploads, logs, caches, or secrets
- verify compose, syntax, and a minimal health check before calling the fix safe
- prefer one service per commit, not one giant mixed live commit

## Reporting During The Round

Summaries should answer:

- which service is affected
- what is confirmed
- what is still hypothesis
- whether flags are currently leaking
- whether a fix is deployed
- whether the checker still passes

Keep round reports short and stateful.
