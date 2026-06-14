# Workflow

## Основной цикл

Если нет сильной причины делать иначе, иди в таком порядке:

1. Потыкать сервис снаружи
2. Прочитать compose/deployment layout
3. Найти хранилища, mounts и internal-only components
4. Восстановить поведение чекера по pcap, логам, скриптам или farm traces
5. Понять, где флаг пишется, как трансформируется, кешируется и возвращается
6. Выбрать самый дешёвый exploit path к тем же данным
7. Патчить самую узкую сломанную trust boundary
8. Перепроверить функциональность и неэксплуатируемость

## Шаг 1: Внешняя рекогносцировка

Сначала пойми, что сервис заявляет о себе до чтения кода.

- выпиши порты и протоколы
- пройди очевидные HTTP/UI endpoints
- попробуй регистрацию, логин, создание объекта, sharing, export, search, upload, webhook, profile flows
- для non-web сервисов пойми banner, handshake, auth scheme, framing и error behavior
- сохрани все user-controlled identifiers, которые потом могут стать path names, topic names, repo names или DB keys

Что надо выяснить:

- какие есть identity
- какие есть object types
- какие действия stateful
- какие endpoints/commands пахнут admin, internal, export, preview, debug, import, analytics, callback или worker hooks

## Шаг 2: Deployment и архитектура

Дальше читай layout стека.

- `docker-compose.yml`, Dockerfiles, entrypoints, supervisord configs, nginx/caddy configs
- internal ports против published ports
- sidecars, workers, brokers, cron jobs, analytics jobs, cleanup jobs
- volumes и bind mounts
- env vars для секретов, admin passwords, signing keys, broker creds, seed data

Собери короткую карту:

- public surface
- кто является auth authority
- кто является data authority
- какие background components привилегированы
- где путь до storage с флагами

## Шаг 3: Data и storage

Найди, где лежат ценные данные.

- Postgres/MySQL/SQLite schemas
- Redis, ClickHouse, Elastic, files, object storage, Kafka topics
- mounted folders вроде `storage`, `uploads`, `repos`, `s3data`
- таблицы или поля с notes, answers, artifacts, messages, metadata, callback responses, access tokens

Смотри на:

- ownership columns
- raw tokens в открытом виде
- поля, которые похожи на место хранения флага
- generated names, которые чекер потом читает

## Шаг 4: Реконструкция чекера

Часто это самый дорогой по пользе шаг.

Используй:

- pcap
- checker scripts
- exploit harnesses типа `start_sploit.py`
- логи или сохранённые запросы

Вытащи:

- полный happy-path записи
- полный happy-path чтения
- схему именования объектов
- auth method чекера
- какие внутренние сервисы он трогает в раунде

Стремись к конкретным формулировкам:

- "checker creates a private repo, writes the flag into wiki, then fetches it by token"
- "checker uploads an S3 object and later reads it through analytics CSV"
- "checker becomes detective before reading all suspects"

## Шаг 5: Построение гипотез

Придумывай exploit ideas из реальных trust boundaries, а не из абстрактного checklist.

Хорошие вопросы:

- могу ли я стать другой identity?
- могу ли я заставить сервис действовать от моего имени, но с большими правами?
- могу ли я превратить internal-only flow в public one?
- могу ли я спутать ownership между object A и container B?
- могу ли я переиспользовать checker retrieval path?
- могу ли я дотянуться до privileged backend через SSRF, worker jobs, broker RPC или forged events?

Сортируй гипотезы по:

- вероятному количеству флагов
- цене проверки
- зависимости от source/live match
- farmability

## Шаг 6: Доказательство

Сначала подтверждай путь самым дешёвым безопасным proof.

- один `curl` лучше, чем 300 строк эксплойта
- один forged token лучше, чем полный account workflow, если он доказывает то же самое
- для reverse-задач сначала подтвердить один primitive: file read, role flip, decrypted response

Помечай статус как:

- confirmed
- partially confirmed
- inferred

## Шаг 7: Фикс

Патчь самый узкий сломанный инвариант.

Примеры:

- перевыпустить секрет и отключить legacy verification
- enforce `resource.owner_id == current_user.id`
- запретить redirect в private IP space
- убрать trust в `X-Internal-Request`
- вырезать dangerous fields из mass assignment
- нормализовать path и запрещать `..`
- заменить public signing endpoint на server-side session

## Шаг 8: Перепроверка

Всегда валидируй обе стороны:

- сервис всё ещё делает то, что нужно чекеру
- exploit path умер

Минимальные проверки:

- registration/login всё ещё работают, если чекер их использует
- создание объекта всё ещё работает
- легитимный owner всё ещё читает свой флаг
- исходный exploit proof или его уменьшенная версия больше не работают
