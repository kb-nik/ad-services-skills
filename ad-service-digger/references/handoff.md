# Handoff

## Purpose

Use this file when one agent does the reconnaissance and analysis, and a later agent should write an exploit, patch, or farm script without repeating the whole dig from scratch.

The handoff artifact is not a competition writeup for organizers.
It is an internal engineering artifact for teammates or other agents.

Treat one handoff as one service dossier.
If the service has multiple vulnerabilities, keep them in one file unless they truly belong to different deployments or different round states.

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
- which candidate exploit paths came from source review before any live checks
- which findings are primary, secondary, or defense-critical
- which paths are `no-flag-id`, `needs flag_id`, `needs victim`, or otherwise not farm-ready
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
7. Findings ranking
8. Ranked candidate exploit paths from source review
9. Primary and secondary exploit paths
10. Fix direction and fix order
11. Exploit amplifiers and weak config
12. Checked-but-not-promoted items
13. Open questions
14. Concrete next actions
15. Coverage pass residuals

## Evidence Labels

Use explicit labels per finding:

- `source-suspected`
- `locally-confirmed`
- `checker-matched`
- `live-confirmed`
- `farm-ready`
- `patched-closed`

Do not collapse these into one vague severity paragraph.
Also keep findings separate from exploit paths. A single exploit path may depend on one finding or on a chain of findings.

## Writing Rules

- Prefer facts over prose.
- Include exact paths, route names, ports, headers, object names, field names, and role names.
- Separate confirmed facts from guesses.
- Give each finding a stable ID such as `F1`, `F2`, `F3`.
- Give each exploit path a stable ID such as `P1`, `P2`.
- Label farmability explicitly per finding or per path when it matters.
- If an exploit failed, say that clearly so the next agent does not waste time.
- If local and live differ, state that at the top.
- If the checker behavior is known, put it near the top, not buried later.
- If live checks have not started yet, say that clearly instead of implying confirmation.
- If a finding is not the best attack path but still dangerous for defense, mark it as `defense-critical`.
- Add a short residual list of classes that were checked and classes that still were not reviewed.
- Keep exploit amplifiers and weak config in separate sections instead of inflating them into primary findings.

## Suggested File Naming

Use a deterministic name so teammates can find it quickly:

- `service-handoff.md`
- or `handoff-<service>.md`

For round-specific notes:

- `handoff-<service>-round-<n>.md`

## Template

Use:

- [assets/templates/service-handoff.md](../assets/templates/service-handoff.md)
