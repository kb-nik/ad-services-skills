# Protocols And Storage

## Когда читать этот файл

- S3-подобные object stores
- Kafka, Redis streams, MQ или RPC topics
- SSH-backed services
- background jobs с внутренним транспортом
- сервисы, где баг сидит в ACL, broker auth, path mapping или worker trust

## Повторяющиеся паттерны

### Object Storage и Files

- bucket names или object names переиспользуются как filesystem paths
- traversal через bucket names, keys, export names или archive paths
- analytics, cleanup, thumbnail, sync jobs лезут в storage с большими правами
- internal headers или internal tokens bypass’ят ACL

### Brokers и Queues

- public broker port торчит наружу и использует user-created credentials
- RPC topic или command topic доступен обычным пользователям
- ACL generation bug выдаёт слишком широкие права
- topic names предсказуемо строятся из usernames или IDs

### SSH и Repo Services

- auth helper валидирует имя repo, но не owner
- token возвращается в repository metadata
- search endpoints раскрывают private repo names или IDs
- HTTP и SSH authorization расходятся

### Workers и Sidecars

- пользователь контролирует destination ARN, webhook base, callback host или job parameters
- worker отправляет internal requests от имени атакующего
- worker может читать privileged storage и писать в attacker-controlled sink

## Порядок исследования

1. Выписать все non-HTTP published ports и что на них говорит.
2. Прочитать ACL/auth/policy code раньше, чем exploit code.
3. Проследить путь user input в path, topic, repo или ARN construction.
4. Проверить background jobs, которые соединяют две trust zone.
5. Сравнить user-facing permissions и worker-side permissions.

Не считай опубликованный Mongo, Redis, broker или admin port побочной заметкой.
Если он торчит наружу, явно рапортуй его как одно из:

- direct finding
- exploit amplifier
- weak exposure без proof пока

## Конкретные вопросы

- Может ли пользователь влиять на internal destination?
- Может ли пользователь заставить privileged component прочитать victim data?
- Может ли path или name выйти из своей namespace?
- Достаётся ли ресурс только по name там, где должен быть owner+name?
- Может ли broker credential дотянуться до command channel или system topic?

## Быстрые выигрыши

- Переиспользуй checker naming conventions для topics, buckets и repos.
- Ищи `X-Internal-Request`, `internal`, `admin`, `analytics`, `cleanup`, `rpc`, `export`, `token`.
- Сравни creation authorization и read authorization: mismatch там часто и есть баг.
