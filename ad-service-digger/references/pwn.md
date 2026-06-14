# Pwn

## Read This For

- native services where memory corruption is already visible or strongly suspected
- stack overflow, format string, heap corruption, integer truncation, or race-to-corruption paths
- binary services where the remaining blocker is turning a crash or primitive into reliable flag access

Do not start here if you still do not understand what the binary does.
Use [reverse.md](reverse.md) first when implementation discovery is the main blocker.

## First Pass

Do the smallest triage that tells you whether the service is realistically exploitable:

- `checksec --file ./binary`
- confirm architecture and libc assumptions with `file` and `ldd`
- identify the input surface: line-based TCP, length-prefixed frames, file parser, HTTP-to-native bridge
- classify the primitive: stack overwrite, format string, heap misuse, integer bug, race
- confirm what you control: bytes written, read length, repeated attempts, reconnect behavior, fork-per-request

In A/D, a clean shell is optional.
A stable flag read, role flip, or data dump is usually enough.

## Protection Triage

Use protections to choose direction early:

- no PIE: fixed code addresses make ret2win and GOT targets easier
- partial RELRO: GOT overwrite may be the fastest path
- full RELRO: prefer return-address control, function pointers, heap metadata, or vtables
- NX enabled: plan for ROP, ret2libc, or non-code-exec data abuse
- stack canary present: leak it first, avoid the stack, or use a non-return-address primitive

Do not over-invest in elegant chains before you know which protection blocks actually matter.

## Primitive To Direction Map

### Stack Overflow

Start with:

- exact offset
- whether RIP is controlled
- whether the crash happens before function return
- whether the service restarts or forks

Typical directions:

- ret2win when symbols are fixed
- ret2libc when you can leak libc
- short ROP to call `open`, `read`, `write` if shell interaction is unnecessary
- stack pivot when overflow space is too small for a full chain

### Format String

Start with:

- whether you control the format string itself
- leak surface: stack, PIE base, libc, canary
- whether writes are possible with `%n`

Typical directions:

- leak canary and return later with a stack overflow
- overwrite GOT only if RELRO allows it
- use one-shot arbitrary write for auth flags, role bits, or function pointers

### Heap Corruption

Start with:

- allocation pattern
- free pattern
- reuse behavior
- whether libc version matters

Typical directions:

- UAF for read/write on sensitive objects
- double free or tcache poisoning for pointer redirection
- adjacent struct corruption to flip auth state or replace callbacks

In A/D, a structure-field overwrite that exposes flags is often better than a full `system("/bin/sh")`.

### Integer Or Sign Bugs

Start with:

- length truncation
- signed-to-unsigned conversion
- multiplication or addition overflow
- mismatch between validation type and copy size

Typical directions:

- under-sized allocation followed by overflow
- negative index or OOB access
- bypassed balance or quota checks leading to privileged paths

### Race Or TOCTOU

Start with:

- shared global state
- temporary files
- unlink/rename windows
- multi-threaded auth or quota updates

Typical directions:

- flip validation-result state
- swap checked paths or objects
- win repeated attempts if the server forks or the race window is consistent

## A/D Priorities

- prefer flag read or targeted data exfil over shell if it is faster and more reliable
- prefer a two-stage exploit with one leak and one action over a fragile all-in-one chain
- document exact offsets, gadgets, libc assumptions, and reconnect behavior for teammates
- if a crash only proves corruption but cannot reach flag flow, pivot back and look for a cheaper trust-boundary bug

## When To Pivot

- back to [reverse.md](reverse.md) if the binary behavior is still unclear
- to [web.md](web.md) if the real exploit path depends more on auth or request routing than on native corruption
- to [protocols-and-storage.md](protocols-and-storage.md) if memory corruption is real but not the fastest route to flags

## Handoff Minimum

If another agent will write the exploit, capture at least:

- binary name and architecture
- protection state from `checksec`
- primitive type
- exact crash or control offset
- confirmed leak sources
- target function, object, or syscall plan
- whether the path is `locally-confirmed`, `live-confirmed`, or already `farm-ready`
