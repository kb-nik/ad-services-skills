# Reverse

## Read This For

- ELF services
- `.so` libraries that need patching
- `.i64` databases or decompiler artifacts
- garble-obfuscated Go binaries
- emulator, VM, crypto, or challenge-style sidecars

Do not start with the reverse lens when the bug is already understood and the remaining work is just exploitation or patching. Use it when understanding the implementation is the blocker.

## First Pass

Do the cheapest information gathering first:

- `file`, `strings`, `checksec`
- imported symbols
- obvious file paths like `/storage/flag`, `/app`, `/tmp`, `/proc`
- protocol markers, JSON keys, route names, role names
- syscall numbers or libc wrappers

For Go or stripped binaries, search for:

- route strings
- JSON field names
- database table names
- role names
- error strings
- challenge field names

## Compact Toolkit

Use the smallest tool that answers the next question:

- layout and metadata: `file`, `checksec`, `readelf`, `objdump`, `nm`
- quick semantic hints: `strings`
- static reverse: Ghidra, IDA, Binary Ninja, radare2/Cutter
- syscall and runtime behavior: `strace`, `ltrace`
- debugger-level confirmation: `gdb`, `lldb`
- Go rebuild assistance: GoReSym, redress
- dynamic instrumentation when static stalls: Frida
- path exploration or constraint solving when needed: angr, z3

Prefer lightweight observation before heavyweight frameworks.

## What To Hunt

### Capability Leaks

- arbitrary file read through emulated syscalls
- command execution through helper calls
- signature or crypto misuse
- challenge endpoints that gate role escalation

### State Machine Mistakes

- server stores challenge state per user and ignores client-supplied fields
- success is only observable through side effects like role change
- source logic and deployed binary differ

### Patchability

- one-byte or short-byte branches to kill auth bypasses
- early returns in exported helper functions
- server-side session replacement when a client token format is compromised

### Reverse-Heavy Patterns

- custom VM or bytecode interpreter
- anti-debug or anti-analysis logic
- signal-handler driven control flow
- self-modifying or self-checking code
- packers, staged loaders, or runtime decryptors
- custom syscall mediation
- validation logic hidden in arithmetic or constraint checks
- side-channel style oracles from timing, signals, exits, or counters

## Reverse Workflow

1. Confirm the externally visible protocol.
2. Extract strings and type hints.
3. Identify the one primitive that would expose flags: file read, role flip, data dump, token forge.
4. Test that primitive with the smallest proof.
5. Only then build a clean exploit.

## Binary Patch Guidance

When source is absent or irrelevant:

- back up the binary first
- patch the smallest branch or function prologue possible
- verify exact bytes after patching
- document offsets and expected byte patterns
- restart and smoke test

Do not assume source patches matter if the live service is a different build.
