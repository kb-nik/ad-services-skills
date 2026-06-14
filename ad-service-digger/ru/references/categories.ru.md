# Категории

## Зачем этот файл

Используй его, чтобы классифицировать новый сервис до глубокого анализа.
Большинство A/D задач смешанные, но обычно один тип сервиса задаёт первый реальный путь к флагу.

## Карта категорий

### 1. Web CRUD

Обычно это:

- регистрация, логин, профиль
- сущности с owner fields
- обычные REST/JSON API
- шаблоны, exports, previews, share links

С чего начинать:

- auth/session handling
- ownership checks
- search/list/export leaks
- SSRF/SSTI/XSS/file upload

Основной файл:

- `web.ru.md`

### 2. Web Plus Realtime

Обычно это:

- LiveView, WebSocket, server events, reactive frontend
- состояние проверяется на mount, но не на event

С чего начинать:

- forged events
- отсутствующая авторизация в handlers
- client-side trust и `innerHTML`

Основной файл:

- `web.ru.md`

### 3. Web Plus Storage

Обычно это:

- HTTP UI поверх S3-подобного storage, файлов, blob’ов, архивов, реп
- bucket names или object names превращаются в пути

С чего начинать:

- traversal
- namespace confusion
- внутренние worker/job-компоненты, которые трогают storage
- metadata и analytics paths

Основные файлы:

- `protocols-and-storage.ru.md`
- `web.ru.md`

### 4. Broker Or Queue Service

Обычно это:

- Kafka, RabbitMQ, Redis Streams, custom RPC topics
- пользовательский сервис создаёт broker identities или ACL

С чего начинать:

- ACL generation
- system topics
- command channels
- predictable topic names

Основной файл:

- `protocols-and-storage.ru.md`

### 5. Repo Or SSH Service

Обычно это:

- Git-подобный сервис
- HTTP metadata плюс SSH push/pull
- отдельный auth helper для SSH

С чего начинать:

- owner/name mismatch
- утечки access token
- расхождение между HTTP и SSH authorization

Основной файл:

- `protocols-and-storage.ru.md`

### 6. Binary Network Service

Обычно это:

- raw TCP
- custom framing
- banner или challenge protocol
- мало или совсем нет исходников

С чего начинать:

- protocol discovery
- strings и syscall surface
- primitive вроде file-read или role-flip
- если memory corruption уже видна, после первого reverse-pass переключаться в `pwn.ru.md`

Основной файл:

- `reverse.ru.md`

### 7. Binary Sidecar Behind Web

Обычно это:

- web app проксирует в локальный бинарь или обфусцированный helper
- исходники покрывают только публичную половину логики

С чего начинать:

- trust boundary между web tier и sidecar
- точные данные, которые web tier отдаёт в sidecar
- source/live mismatch
- в `pwn.ru.md` переключаться только если exploit реально держится на native corruption, а не на request forgery или auth confusion

Основные файлы:

- `reverse.ru.md`
- `web.ru.md`

### 8. Challenge-Crypto Hybrid

Обычно это:

- логин или роль завязаны на challenge-response
- математика, крипта, эмулятор, stateful puzzle

С чего начинать:

- state machine
- где хранится challenge state
- какие side effects доказывают успех
- shortcuts вокруг challenge

Основной файл:

- `reverse.ru.md`

### 9. Mixed Service Mesh

Обычно это:

- UI + API + worker + broker + DB + storage
- нет одной очевидной точки входа

С чего начинать:

- `workflow.ru.md`
- классифицировать каждый компонент
- искать самый сильный компонент и самую слабую границу между компонентами

## Быстрые вопросы для классификации

Задай себе в начале:

1. Главная trust boundary здесь user-to-HTTP, component-to-component или network-to-binary?
2. Где, скорее всего, лежит флаг: DB row, file, object, topic message, repo content, in-memory session?
3. Какой компонент читает флаг назад для чекера?
4. Какой компонент имеет больше прав, чем user-facing surface?
5. Самый реалистичный exploit path здесь — read, impersonation, path escape, worker abuse или protocol break?

## Базовый порядок по категориям

- `web` сначала для CRUD и UI-heavy задач
- `protocols-and-storage` сначала для S3/Kafka/SSH/internal jobs
- `reverse` сначала для бинарных и обфусцированных компонентов
- `pwn` после `reverse`, когда бинарь уже понятен и остаётся надёжно разыграть memory-corruption primitive
- `defense` после того, как уже есть конкретный exploit path или live incident
