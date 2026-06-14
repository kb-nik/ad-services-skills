# Defense

## Goal

Preserve checker functionality while killing the cheapest exploit path.

## Patch Order

Apply fixes in this order when time is short:

1. Rotate or replace exposed secrets.
2. Break the main trust-boundary bug.
3. Remove dangerous public or low-privilege data exposure.
4. Add narrow validation around internal fetches, paths, and ownership checks.
5. Rebuild and smoke test.

## Minimal Fix Patterns

### Secrets

- replace hardcoded JWT, HMAC, admin, broker, or webhook secrets
- reject legacy verification branches once rotated

### Ownership

- enforce owner checks at the final object lookup, not only at the parent container
- forbid mass-assignment of role or workspace-like fields

### Internal Trust

- stop trusting internal-only headers from user-reachable code paths
- pin worker destinations or whitelist them
- block private and loopback IPs for callbacks, webhooks, renderers, and previews

### Paths and Names

- reject `..`, slashes, separators, and normalize before filesystem access
- ensure repo, bucket, or topic lookups include owner identity where required

### Source/Live Mismatch

- if live differs from source, patch the live trust boundary that is actually reachable
- keep source-only bugs documented, but do not spend defense time on dead paths first

## SLA Checks

After patching, test the features the checker most likely needs:

- auth still works
- create/write still works
- legitimate readback still works
- background jobs still run if they are part of normal service behavior
- the original exploit proof is dead

## Practical A/D Notes

- A half-fix that preserves SLA is usually better than an elegant refactor that risks downtime.
- If only binaries are available, byte-patching plus verification is acceptable.
- If a worker or sidecar is the only broken component and cannot be rebuilt, move the guard to the public-facing service.
