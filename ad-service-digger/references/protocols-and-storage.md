# Protocols And Storage

## Read This For

- S3-like object stores
- Kafka, Redis streams, MQ or RPC topics
- SSH-backed services
- background jobs with internal transport
- services where the bug is in ACL, broker auth, path mapping, or worker trust

## Recurring Patterns

### Object Storage and Files

- bucket or object names reused as filesystem paths
- traversal through bucket names, keys, export names, or archive paths
- analytics, cleanup, thumbnail, or sync jobs that access storage with elevated trust
- internal headers or tokens that bypass ACL checks

### Brokers and Queues

- public broker port exposed with user-created credentials
- RPC topic or command topic reachable by ordinary users
- ACL creation bugs that grant access wider than intended
- guessable topic names derived from usernames or IDs

### SSH and Repo Services

- auth helper validates repo name but not owner
- token returned in repository metadata
- search endpoints leak private repo names or IDs
- HTTP and SSH authorization logic disagree

### Workers and Sidecars

- user controls destination ARN, webhook base, callback host, or job parameters
- worker sends internal requests on behalf of the attacker
- worker can read from privileged storage and write to attacker-controlled sink

## Investigation Order

1. Identify every published non-HTTP port and what speaks on it.
2. Read ACL, auth, and policy code before exploit code.
3. Follow data from user input into path, topic, repo, or ARN construction.
4. Inspect background jobs that bridge two trust zones.
5. Compare user-facing permissions with worker-side permissions.

Do not treat a published Mongo, Redis, broker, or admin port as a side note.
If it is exposed, report it explicitly as one of:

- direct finding
- exploit amplifier
- weak exposure with no proof yet

## Concrete Questions

- Can a user influence an internal destination?
- Can a user cause a privileged component to read victim data?
- Can a path or name escape its intended namespace?
- Can a resource lookup be done by name only when it should be by owner plus name?
- Can a broker credential reach a command channel or system topic?

## Fast Wins

- Reuse checker naming conventions for topics, buckets, or repos.
- Search for `X-Internal-Request`, `internal`, `admin`, `analytics`, `cleanup`, `rpc`, `export`, `token`.
- Compare creation authorization with read authorization; the mismatch is often the bug.
