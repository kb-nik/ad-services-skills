# Service Handoff

## Metadata

- Service:
- Category:
- Date:
- Analyst:
- Source/live match:
- Current status: `unknown|vulnerable|closed|broken|checker-broken`

## Summary

- What the service does:
- Main likely flag container:
- Best current exploit direction:
- Best current fix direction:
- Best current farmability label:

## Architecture

- Public ports:
- Internal ports:
- Main components:
- Privileged components:
- Storage:

## Checker Flow

- How checker creates data:
- Where checker stores flag:
- How checker reads flag back:
- Object naming pattern:
- Traffic/source evidence:

## Trust Boundaries

- Identity authority:
- Data authority:
- Internal-only assumptions:
- User-controlled inputs that cross trust boundaries:

## Findings

| ID | Title | Class | Evidence | Attack role | Farmability | Impact | Location | Notes |
|---|---|---|---|---|---|---|---|---|
| F1 |  |  | source-suspected | primary | no-flag-id |  |  |  |
| F2 |  |  | source-suspected | secondary | needs flag_id |  |  |  |

## Findings Ranking

| Rank | Finding ID | Why it matters now | Validation cost | Farmability | Defense priority |
|---|---|---|---|---|---|
| 1 | F1 |  |  |  | high |

## Ranked Candidates From Source Review

| Rank | Path ID | Findings used | Farmability | Path | Why it may work | Fastest local proof | Live check needed |
|---|---|---|---|---|---|---|---|
| 1 | P1 | F1 | no-flag-id |  |  |  |  |
| 2 | P2 | F2 + F3 | needs flag_id |  |  |  |  |

## Primary Exploit Path

- Path ID:
- Findings used:
- Farmability:
- Goal:
- Preconditions:
- Minimal proof:
- Current stage: `source-suspected|locally-confirmed|checker-matched|live-confirmed|farm-ready`
- Why this path is best now:

## Secondary Exploit Paths

| Path ID | Findings used | Farmability | Why lower priority | What would promote it |
|---|---|---|---|---|
| P2 | F2 + F3 | needs user_id |  |  |

## Defense-Critical Findings

| Finding ID | Why it must be fixed even if not primary | Suggested order |
|---|---|---|
| F3 |  | 1 |

## Exploit Amplifiers

| ID | Type | Why it helps exploitation | Location | Notes |
|---|---|---|---|---|
| A1 | public usernames |  |  |  |
| A2 | sequential IDs |  |  |  |

## Weak Config Or Hygiene

| ID | Issue | Why it matters | Location | Notes |
|---|---|---|---|---|
| W1 | weak cookie flags |  |  |  |

## Checked But Not Promoted

| Class | Why it looked interesting | Why not promoted yet | Next proof if needed |
|---|---|---|---|
| operator injection |  |  |  |

## Fix Direction

- Smallest safe fix:
- Fix order:
- Files to patch:
- Risk to checker/SLA:
- Validation needed:

## Open Questions

- Q1:
- Q2:

## Coverage Pass Residuals

| Class | Status | Notes |
|---|---|---|
| auth type confusion | checked |  |
| ownership on detail/list/readback | checked |  |
| operator injection | not reviewed yet |  |
| enumeration / sequential IDs | checked |  |
| UI/API mismatch | not reviewed yet |  |

## Negative Checks

| Class | Status | Notes |
|---|---|---|
| hardcoded secrets | weak-only | session secret hardcoded |
| cookie/session flags | weak |  |
| published DB/broker/internal ports | checked | Mongo exposed on compose |
| direct DB exploitability | not reviewed yet |  |

## Next Actions

1. 
2. 
3. 

## Artifacts

- Relevant files:
- Pcaps:
- Logs:
- Exploit drafts:
- Patch drafts:
