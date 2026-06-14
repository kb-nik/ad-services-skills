# Traffic Triage

## Read This For

- pcap or pcap-derived analysis
- traffic PDF reports
- live "are they stealing flags right now?" questions
- comparing observed traffic against known exploit packs

This file is for attack-defense theft triage, not generic packet analysis.
The key question is whether flags originated from the service response, not whether a flag-like string appears anywhere in traffic.

## Core Rule

Do not call something a leak until you separate:

- checker traffic
- attacker-supplied flags echoed back by the service
- mirror/export duplication
- real server-origin flag disclosure

The shortest useful model is:

- `server_flags - client_flags` = candidate leaks
- `server_flags ∩ client_flags` = likely echo or replay until proven otherwise

## Workflow

1. Define the capture window and input type.
2. Map ports to services and protocols.
3. Identify checker IPs, our own hosts, and mirror/export channels.
4. Count directional flag flow per stream.
5. Classify each suspected leak as real, checker, echo, noise, or backlog candidate.
6. Compare against existing exploit notes before calling it new.

## What To Record

- capture window
- checker IPs or `unknown`
- service/port/protocol map
- src and dst per relevant stream
- request excerpt and response excerpt
- whether the flag appeared in request first
- whether the stream is checker-origin

## Classification

- `confirmed leak`: non-checker server-origin flag in response not present in request
- `echo/no theft`: flag appears in request and is echoed back
- `checker traffic`: legitimate checker put/get or validation flow
- `probe/noise`: exploit attempt with no server-origin flag
- `new exploit gap`: observed path looks real and is not covered by existing local notes
- `known/covered`: already in exploit or defense backlog
- `unconfirmed`: PDF or static hint without enough evidence

## Common Pitfalls

1. Counting any flag-like string in a response as theft.
2. Forgetting that checker traffic naturally moves flags.
3. Treating attacker-created flags echoed by create/update endpoints as real leaks.
4. Ignoring mirrored or exported traffic that duplicates payloads.
5. Reporting a PDF finding as new without checking existing exploit notes.
6. Ignoring the capture window and over-claiming from a short slice.

## Output Shape

Keep summaries short:

- Window
- Confirmed stolen flags
- Services with real leaks
- Echo or probe noise
- New backlog candidates
- Blockers
