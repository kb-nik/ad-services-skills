# ad-service-digger

`ad-service-digger` is a reusable skill for analyzing unfamiliar attack-defense services and turning messy service material into a practical exploit or hardening plan.

The skill is built around a simple idea:

1. Reconstruct what the service does.
2. Find where flags live and how the checker touches them.
3. Map weak trust boundaries.
4. Choose the fastest exploit direction.
5. Propose the smallest safe fix that preserves checker SLA.

## What Is Inside

- `SKILL.md` - main English skill for the model
- `agents/openai.yaml` - agent-facing configuration
- `references/workflow.md` - default digging flow
- `references/categories.md` - how to classify service types
- `references/web.md` - web, HTTP, API, session, SSRF, IDOR, auth
- `references/protocols-and-storage.md` - queues, brokers, storage, SSH, workers, sidecars
- `references/reverse.md` - binaries, libraries, protocol reversing, crypto/state logic
- `references/operations.md` - live rounds, validation, exploit farming, incident mode
- `references/defense.md` - minimal fixes and SLA-safe patching
- `references/handoff.md` - how to prepare a clean handoff for another agent or teammate
- `references/case-studies.md` - recurring A/D patterns from real digs
- `assets/templates/service-handoff.md` - structured handoff template
- `ru/` - Russian mirror of the skill and reference materials

## Quick Start

For the model:

1. Read `SKILL.md`.
2. Read `references/workflow.md`.
3. Classify the target with `references/categories.md`.
4. Pick the right lens:
   - `references/web.md`
   - `references/protocols-and-storage.md`
   - `references/reverse.md`
5. Use `references/operations.md` and `references/defense.md` when the task shifts from digging to active exploitation or patching.

For humans:

1. Read `ru/SKILL.ru.md` if you want the Russian overview first.
2. Use `ru/references/*.ru.md` as the team-readable mirror.
3. If one person digs and another writes the exploit or fix, use the handoff flow in `references/handoff.md`.

## Recommended Output

The default result shape is:

1. Architecture map and flag path
2. Top exploit hypotheses
3. Minimal validation plan
4. Exploit strategy or proof outline
5. Minimal hardening plan
6. Regression checks for checker safety

If the work should be passed to someone else, also produce a structured handoff file from `assets/templates/service-handoff.md`.

## Language Notes

- English files are the primary model-facing version.
- Russian files are the team-facing mirror for easier review and onboarding.
- The structure is intentionally kept close between both versions.
