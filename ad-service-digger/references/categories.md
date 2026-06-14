# Categories

## Purpose

Use this file to classify a new service before going deep.
Most A/D tasks are mixed, but one category usually dominates the first exploit path.

## Category Map

### 1. Web CRUD

Typical shape:

- registration, login, profiles
- objects with owner fields
- classic REST or JSON APIs
- templates, exports, previews, share links

Start with:

- auth/session handling
- ownership checks
- search/list/export leaks
- SSRF/SSTI/XSS/file upload

Read:

- [web.md](web.md)

### 2. Web Plus Realtime

Typical shape:

- LiveView, WebSocket, server events, reactive frontend
- state checked on mount but not on event

Start with:

- forged events
- missing authorization in handlers
- client-side trust and `innerHTML`

Read:

- [web.md](web.md)

### 3. Web Plus Storage

Typical shape:

- HTTP UI backed by S3-like storage, files, blobs, archives, repos
- object names or bucket names become paths

Start with:

- traversal
- namespace confusion
- internal workers that touch storage
- metadata or analytics paths

Read:

- [protocols-and-storage.md](protocols-and-storage.md)
- [web.md](web.md)

### 4. Broker Or Queue Service

Typical shape:

- Kafka, RabbitMQ, Redis Streams, custom RPC topics
- user-facing app creates broker identities or ACLs

Start with:

- ACL generation
- system topics
- command channels
- predictable topic names

Read:

- [protocols-and-storage.md](protocols-and-storage.md)

### 5. Repo Or SSH Service

Typical shape:

- Git-like app
- HTTP metadata plus SSH push/pull
- separate auth helper for SSH

Start with:

- owner/name mismatches
- leaked access tokens
- disagreement between HTTP and SSH authorization

Read:

- [protocols-and-storage.md](protocols-and-storage.md)

### 6. Binary Network Service

Typical shape:

- raw TCP
- custom framing
- banner or challenge protocol
- minimal or no source

Start with:

- protocol discovery
- strings and syscall surface
- file-read or role-flip primitive

Read:

- [reverse.md](reverse.md)

### 7. Binary Sidecar Behind Web

Typical shape:

- web app proxies to local binary or obfuscated helper
- source only covers the public half

Start with:

- trust boundary between web tier and sidecar
- exact data passed to the sidecar
- source/live mismatch

Read:

- [reverse.md](reverse.md)
- [web.md](web.md)

### 8. Challenge-Crypto Hybrid

Typical shape:

- login or role gate depends on challenge response
- math, crypto, emulator, or stateful puzzle

Start with:

- state machine
- challenge storage
- side effects that prove success
- shortcuts around the challenge

Read:

- [reverse.md](reverse.md)

### 9. Mixed Service Mesh

Typical shape:

- UI + API + worker + broker + DB + storage
- no single obvious entrypoint

Start with:

- [workflow.md](workflow.md)
- classify each component
- find the strongest component and the weakest boundary between them

## Fast Classification Questions

Ask these early:

1. Is the main trust boundary user-to-HTTP, component-to-component, or network-to-binary?
2. Where is the flag likely stored: DB row, file, object, topic message, repo content, memory-backed session?
3. Which component reads the flag back for the checker?
4. Which component has more privilege than the user-facing surface?
5. Is the most realistic exploit path read, impersonation, path escape, worker abuse, or protocol break?

## Default Order By Category

- `web` first for CRUD and UI-heavy tasks
- `protocols-and-storage` first for S3/Kafka/SSH/internal jobs
- `reverse` first for binary and obfuscated components
- `defense` once you already have a concrete exploit path or a live incident
