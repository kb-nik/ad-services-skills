# Farm Packaging

## Read This For

- writing exploit scripts for teammates or a farm
- packaging a no-victim or no-flag-id exploit
- preparing a clean archive after a vulnerability is already confirmed

This file starts after the vulnerability research is mostly done.
Do not package first and validate later.

## Default Farm Contract

Unless the user specifies another platform:

- accept target from `argv[1]`
- print only flags to stdout, one per line
- send diagnostics and progress to stderr
- keep per-target state, not one global state for all teams
- avoid hardcoded live secrets when they can be discovered at runtime
- prefer stdlib-only tooling when practical

## Packaging Workflow

1. Confirm the exploit scope:
   - flag theft
   - no-victim or victim-required
   - no-flag-id or needs flag id
   - DoS/state-corruption instead of theft
2. Write the smallest reliable script.
3. Smoke-test locally with a synthetic flag-like marker.
4. Verify stdout contains only flags.
5. Package only intended files.
6. Document exact run command and remaining blockers.

## Farmability Gate

Classify the path explicitly:

- `no-flag-id`
- `needs flag_id`
- `needs user_id`
- `needs victim`
- `DoS/state-corruption`
- `non-flag`

Do not call a path farm-ready until this label is clear.

## Archive Hygiene

Do not include:

- `__pycache__/`, `*.pyc`
- `.env`, secrets, tokens, SSH keys
- DB dumps, WAL/SHM files
- logs, pcaps, browser artifacts
- old failed PoCs
- unrelated helper scripts

## Common Pitfalls

1. Printing debug JSON or IDs to stdout and breaking the farm.
2. Forgetting flush behavior for flags.
3. Sharing one watermark or state file across all hosts.
4. Packaging victim-required browser bait as a no-victim exploit.
5. Sending an archive before checking its contents.
6. Calling a `flag_id` endpoint farmable without a way to obtain the `flag_id`.

## Handoff Notes

When handing an exploit path to another agent, include:

- target port and protocol
- expected auth state
- flag-bearing field
- farmability label
- run command
- local smoke-test status
