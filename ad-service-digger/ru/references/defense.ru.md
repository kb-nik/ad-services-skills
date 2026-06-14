# Defense

## Цель

Сохранить функциональность для чекера и убить самый дешёвый exploit path.

## Порядок фиксов

Если времени мало, иди так:

1. Заменить или перевыпустить утёкшие секреты.
2. Перекрыть главный trust-boundary bug.
3. Убрать опасную public/low-privilege data exposure.
4. Добавить узкую валидацию вокруг internal fetches, paths и ownership checks.
5. Пересобрать и сделать smoke test.

## Минимальные паттерны фиксов

### Secrets

- заменить hardcoded JWT, HMAC, admin, broker или webhook secrets
- после ротации убрать legacy verification branches

### Ownership

- enforce owner checks в точке финального object lookup, а не только на parent container
- запретить mass-assignment полей вроде role, workspace, scope

### Internal Trust

- убрать trust к internal-only headers из user-reachable code paths
- pin/whitelist destinations для workers
- блокировать private и loopback IP для callbacks, webhooks, renderers и previews

### Paths и Names

- запрещать `..`, slash separators и делать normalizing перед filesystem access
- ensure repo, bucket или topic lookups учитывают owner identity, если это нужно по модели

### Source/Live Mismatch

- если live отличается от source, патчить живую trust boundary, а не мёртвую из исходников
- source-only bugs документировать, но не тратить на них defence time в первую очередь

## SLA-проверки

После патча проверь то, что чекер, скорее всего, использует:

- auth всё ещё работает
- create/write всё ещё работает
- легитимное readback всё ещё работает
- background jobs всё ещё живы, если они часть штатной логики
- исходный exploit proof умер

## Практические заметки для A/D

- Полуфикс, который сохраняет SLA, часто лучше красивого рефакторинга с риском дауна.
- Если есть только бинарь, byte-patching с проверкой байт — нормальный A/D workflow.
- Если worker или sidecar нельзя быстро пересобрать, ставь guard на public-facing service.
