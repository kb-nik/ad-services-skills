# Questions

## Purpose

Use this file when you need a compact, reusable question set for a service you have never seen before.
The goal is to force early clarity before you sink time into the wrong layer.

## Identity

- Who can register?
- Who can authenticate?
- Which roles exist?
- Which roles are only implied in UI but enforced on the server?
- Which component is the real source of truth for identity?

## Data

- What objects exist?
- Which fields are private, secret, or likely to contain flags?
- Which object relationships encode ownership?
- Which data is stored in DB versus filesystem versus broker versus memory?

## Flag Flow

- How does the checker create the object that receives the flag?
- How is the flag named, indexed, or linked?
- Which exact endpoint, protocol, or worker reads it back?
- Is the checker using direct readback or an indirect flow like export, report, analytics, webhook, or claim?

## Trust Boundaries

- Which component has more privilege than the public interface?
- Which component is assumed internal but is reachable?
- Which DB, cache, broker, or admin port is published and reachable from outside?
- Which headers, tokens, env vars, or helper ports are treated as trusted?
- Which user-controlled values cross into file paths, queries, topics, repo names, or internal URLs?

## Validation

- What proof would confirm the bug with the least effort?
- Can the finding be confirmed on live, or only in source/local?
- What observable side effect proves success: role change, returned secret, object visibility, worker output, log entry?
- What evidence is missing right now?
- Which high-value classes were checked and came back negative or weak-only?

## Exploitability

- Does the path scale across teams?
- Does it depend on guessed IDs, leaked names, or checker traffic?
- Is it fast enough for farming?
- Does it survive partial fixes or restarts?

## Defense

- What single trust assumption should be broken first?
- Can the bug be killed by secret rotation, owner check, path normalization, route closure, or worker restriction?
- Which fix is smallest while keeping checker behavior intact?
- What exact positive and negative tests must pass after patching?
