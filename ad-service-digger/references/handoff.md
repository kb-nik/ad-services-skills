# Handoff

## Purpose

Use this file when one agent does the reconnaissance and analysis, and a later agent should write an exploit, patch, or farm script without repeating the whole dig from scratch.

The handoff artifact is not a competition writeup for organizers.
It is an internal engineering artifact for teammates or other agents.

## When To Produce A Handoff

Produce a handoff when any of these are true:

- the service dig is already deep and repeating it would waste time
- another agent will write the exploit
- another agent will write the fix
- the current round may continue later and the context should survive
- source, traffic, and local repro results are spread across too many ad hoc notes

## What A Good Handoff Enables

A second agent should be able to answer these without re-digging:

- what the service is
- where the flag lives
- how the checker writes and reads it
- what exploit paths are confirmed versus hypothetical
- what the highest-value next action is
- what exact files, routes, ports, and objects matter

## Required Sections

Include at least:

1. Service summary
2. Architecture and ports
3. Flag flow
4. Identities and trust boundaries
5. Findings table
6. Evidence level for each finding
7. Best exploit direction
8. Fix direction
9. Open questions
10. Concrete next actions

## Evidence Labels

Use explicit labels per finding:

- `source-suspected`
- `locally-confirmed`
- `checker-matched`
- `live-confirmed`
- `farm-ready`
- `patched-closed`

Do not collapse these into one vague severity paragraph.

## Writing Rules

- Prefer facts over prose.
- Include exact paths, route names, ports, headers, object names, field names, and role names.
- Separate confirmed facts from guesses.
- If an exploit failed, say that clearly so the next agent does not waste time.
- If local and live differ, state that at the top.
- If the checker behavior is known, put it near the top, not buried later.

## Suggested File Naming

Use a deterministic name so teammates can find it quickly:

- `service-handoff.md`
- or `handoff-<service>.md`

For round-specific notes:

- `handoff-<service>-round-<n>.md`

## Template

Use:

- [assets/templates/service-handoff.md](../assets/templates/service-handoff.md)
