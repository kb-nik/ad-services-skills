# Case Studies

## How To Use This File

Read this only after the generic workflow and category are already clear.
These are examples of patterns from prior A/D digs, not a canonical list of service types and not a template that future services must match.

Use each example like this:

- identify the underlying pattern
- reuse the pattern if the new service really fits
- ignore the service name if it does not

## s3waaas

- Checker traffic revealed the real flag path: random bucket, random object, analytics flow.
- The high-value angle was not just the UI; it was the privileged analytics job and trust in an internal header.
- Bucket and key names also touched filesystem boundaries, making traversal relevant.

Generic pattern:
Trace background jobs and internal trust before spending too long on public CRUD endpoints.

## pontiger

- HTTP controller methods emitted an error but kept executing and serialized sensitive data anyway.
- Search leaked targets, HTTP leaked tokens, and SSH auth helper failed to bind repo name to owner.

Generic pattern:
In repo-like services, compare HTTP auth logic, SSH auth logic, and object ownership separately.

## kakafka

- Public web UI was only one layer; external Kafka on `9092` and RPC topic behavior mattered.
- Secrets and Flask session trust were as important as broker ACLs.

Generic pattern:
If users can create protocol-level identities, inspect the broker side directly and not only the web layer.

## pony-stark

- Binary used Unicorn/emulation semantics; the route to flags was through emulated syscalls and storage access.
- Strings, hooks, syscall lists, and output behavior gave more value than blind shellcode attempts.

Generic pattern:
For emulator-style services, identify the syscall mediation model before crafting payloads.

## mspd2

- Source and live deployment did not match.
- Two obvious source bugs were dead on live; the real path was a detective-role challenge implemented in a deployed binary sidecar.

Generic pattern:
Keep live validation separate from source analysis. A dead source bug is not a live exploit.

## ServiceDesk

- Fast wins came from secrets, mass assignment, SSTI, SSRF, and share-token determinism.
- Patch order mattered: rotate secrets first, then remove privilege escalation and renderer abuse.

Generic pattern:
When time is short, prioritize bugs that collapse many downstream exploit paths at once.

## typevault

- Public search/gallery, callback SSRF, LiveView auth gaps, and frontend XSS all mattered.
- The service mixed API, rendering, event handlers, and internal status endpoints.

Generic pattern:
In rich web apps, do not stop at REST routes. Check event handlers, preview features, and internal controllers.

## pdfgen

- Public signing service and weak crypto made auth forgery possible.
- Print/render side service created SSRF into internal-only export paths.

Generic pattern:
Any public helper service that can sign, render, fetch, or print should be treated as a privilege boundary.

## identityhub

- Practical defense required patching `.so` binaries directly.
- Byte-level patching plus explicit byte verification was the realistic response path.

Generic pattern:
For binary-only components, "surgical patch plus verify bytes" is a valid A/D workflow, not a last resort.
