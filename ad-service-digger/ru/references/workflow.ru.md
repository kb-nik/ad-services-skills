# Workflow

## Основной цикл

Если нет сильной причины делать иначе, иди в таком порядке:

1. Прочитать исходники и deployment layout
2. Найти хранилища, mounts и internal-only components
3. Восстановить поведение чекера по исходникам, pcap, логам, скриптам или farm traces
4. Понять, где флаг пишется, как трансформируется, кешируется и возвращается
5. Построить exploit-гипотезы по коду и trust boundaries
6. Завершить один статический coverage inventory для выбранного lens
7. Сделать самую лёгкую локальную или внешнюю валидацию, которая отсечёт слабые гипотезы
8. Использовать coverage pass, чтобы не закончить на первом подтверждённом path
9. Использовать live-проверки только на самых сильных кандидатах
10. Патчить самую узкую сломанную trust boundary
11. Перепроверить функциональность и неэксплуатируемость

Если исходников нет или они явно не соответствуют live, откатывайся к старому порядку и начинай с внешнего probe.

Если задача в `research mode`, останавливайся после шага с live-check или раньше и рапортуй findings.
К шагам фикса и перепроверки переходи только в `patch mode` или при явной live-defense задаче.

## Шаг 1: Сначала исходники и deployment

Если код и конфиги доступны, начинай именно с них.

- определи форму репозитория
- найди entrypoints, routers, handlers, workers, cron jobs, sidecars, binaries
- прочитай `docker-compose.yml`, Dockerfiles, entrypoints, nginx/caddy configs, supervisord files
- найди secrets, signing keys, broker creds, admin bootstrap и debug toggles
- пометь user-controlled fields, которые пересекают trust boundaries

Что надо выяснить:

- какой компонент является auth authority
- какой компонент является data authority
- какие routes, commands, jobs или helper binaries привилегированы
- какие code paths похожи на checker-facing

## Шаг 2: Data и internal components

Найди реальные хранилища и привилегированные пути как можно раньше.

- Postgres/MySQL/SQLite schemas
- Redis, ClickHouse, Elastic, files, object storage, Kafka topics
- mounted folders вроде `storage`, `uploads`, `repos`, `s3data`
- таблицы или поля с notes, answers, artifacts, messages, metadata, callback responses, access tokens
- sidecars, workers, brokers, cron jobs, analytics jobs, cleanup jobs

Смотри на:

- ownership columns
- raw tokens в открытом виде
- поля, которые похожи на место хранения флага
- generated names, которые чекер потом читает
- компоненты, которые видят больше, чем public API

## Шаг 3: Реконструкция чекера

Часто это самый дорогой по пользе шаг.

Используй:

- checker scripts
- pcap
- exploit harnesses типа `start_sploit.py`
- логи или сохранённые запросы
- code paths, которые создают, меняют или читают flag-bearing objects

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

## Шаг 4: Карта флага и построение гипотез

Строй exploit ideas из реальных trust boundaries, а не из абстрактного checklist.

Хорошие вопросы:

- могу ли я стать другой identity?
- могу ли я заставить сервис действовать от моего имени, но с большими правами?
- могу ли я превратить internal-only flow в public one?
- могу ли я спутать ownership между object A и container B?
- могу ли я переиспользовать checker retrieval path?
- могу ли я дотянуться до privileged backend через SSRF, worker jobs, broker RPC, forged events или native primitives?

Сортируй гипотезы по:

- вероятному количеству флагов
- цене проверки
- зависимости от source/live match
- farmability

До live-проверок собери короткий список кандидатов:

- `H1`, `H2`, `H3`
- точные файлы, routes, functions или objects
- какой proof подтвердит или убьёт каждую идею

## Шаг 5: Статический coverage inventory

До local validation собери компактный inventory по выбранному lens.

Для web-сервисов нужно покрыть хотя бы:

- все auth routes
- все object create/read/list/update/delete routes
- все render sinks в шаблонах или frontend-коде
- все модели и поля, которые похожи на носители флага или ownership
- все очевидные body/query entrypoints, которые доходят до ORM lookups
- все опубликованные data-plane порты вроде Mongo, Redis, broker, sidecar, admin, raw socket
- все очевидные источники секретов: hardcoded session secrets, JWT/HMAC keys, default creds, env fallbacks

Inventory не обязан быть длинным, но должен быть достаточно полным, чтобы агент мог сказать:

- какие routes были просмотрены
- какие sinks были просмотрены
- какие классы ещё не покрыты
- какие sinks подтверждённо escaped, частично escaped или вообще unescaped
- какие internal/data-plane порты были просмотрены и торчат ли они наружу
- какие secret/config классы были просмотрены и они exploitable, weak-only или отсутствуют

Пиши этот inventory на языке пользователя: заголовок секции, название таблицы, названия колонок и короткие статусы тоже должны совпадать с языком запроса.
Для web-сервисов таблица verdict по sink'ам обязательна: одна строка на каждый просмотренный sink.
Если релевантных template/frontend sink'ов не нашлось, всё равно добавь одну явную строку `не найдено`.

Для frontend/template sink'ов не пиши общие фразы вроде "XSS checked" или "output escaped", если ты не перечислил конкретные sink'и.
Лучше писать так:

- `item-detail.ejs`: `description` escaped, `bodyPart` unescaped
- `index.ejs`: username escaped
- `feed.ejs`: description escaped

Покрытие route'ов и sink'ов должно быть достаточно явным, чтобы другой агент видел, что именно ещё могли пропустить.

Пока большие группы route'ов или sink'ов не прочитаны, в local validation не переходи.

## Шаг 6: Внешняя рекогносцировка

Только теперь используй внешний взгляд, чтобы уточнить или отсеять source-based гипотезы.

Пойми, что сервис реально экспонирует, и совпадает ли это с ожиданиями по коду.

- выпиши порты и протоколы
- пройди очевидные HTTP/UI endpoints
- попробуй регистрацию, логин, создание объекта, sharing, export, search, upload, webhook, profile flows
- для non-web сервисов пойми banner, handshake, auth scheme, framing и error behavior
- сохрани все user-controlled identifiers, которые потом могут стать path names, topic names, repo names или DB keys

Что надо выяснить:

- какие identity реально доступны
- какие object types реально существуют
- какие действия stateful
- какие endpoints/commands пахнут admin, internal, export, preview, debug, import, analytics, callback или worker hooks
- какие source-гипотезы переживают контакт с реальной exposed surface

## Шаг 7: Доказательство

Сначала подтверждай путь самым дешёвым безопасным proof.

- один `curl` лучше, чем 300 строк эксплойта
- один forged token лучше, чем полный account workflow, если он доказывает то же самое
- для reverse-задач сначала подтвердить один primitive: file read, role flip, decrypted response
- если возможно, предпочитай local или lab confirmation до live confirmation

Помечай статус как:

- confirmed
- partially confirmed
- inferred

И отдельно фиксируй, где это подтверждено:

- source only
- local deployment
- live target
- multi-target farm

Используй live-проверки только для короткого списка кандидатов, которые уже пережили source review и local proof.
Не надо делать широкий live fuzzing, если код уже резко сузил поиск.

## Шаг 8: Coverage pass

Перед тем как называть сервис "разобранным", сделай residual pass по оставшимся классам.

Для маленького или среднего web-сервиса явно спроси себя:

- проверили ли мы auth type confusion во всех login-like path
- проверили ли ownership отдельно на read, write, list и indirect render path
- проверили ли sequential IDs, public listings и object enumeration
- проверили ли query/operator injection там, где body fields идут в ORM lookups
- проверили ли UI/API mismatch и hidden-but-accepted fields
- проверили ли, нельзя ли усилить strongest finding, например из `needs username` в `no-username` или из `needs flag_id` в `no-flag-id`

Если класс не проверен, помечай его как `not reviewed yet`, а не замалчивай.
Если вещь реальная, но вторичная, помечай её как:

- `exploit amplifier`
- `weak config`
- `checked but not promoted`

а не мешай с main confirmed exploit list.

Также принудительно добавляй короткий negative-check report для:

- hardcoded secrets checked: `confirmed | weak-only | not found`
- cookie/session flags checked: `weak | acceptable | not reviewed yet`
- published DB/broker/internal ports checked: `exposed | internal-only | not reviewed yet`

## Шаг 9: Live-проверка

Live validation идёт последней, а не первой.

Она нужна только для оставшихся дорогих вопросов:

- совпадает ли local proof с deployed target
- реальна ли source/live mismatch
- ведёт ли exploit path к тому же flag flow на реальной команде
- достаточно ли стабилен путь, чтобы считать его `live-confirmed`

Держи live-проверки узкими:

- подтверждай один route, object или primitive
- останавливайся, как только гипотеза доказана или убита
- если live расходится с source, фиксируй mismatch явно

## Шаг 10: Фикс

В этот шаг входи только если пользователь явно попросил patch/fix/hardening, либо если задача идёт как активная защита в live-раунде.

Патчь самый узкий сломанный инвариант.

Примеры:

- перевыпустить секрет и отключить legacy verification
- enforce `resource.owner_id == current_user.id`
- запретить redirect в private IP space
- убрать trust в `X-Internal-Request`
- вырезать dangerous fields из mass assignment
- нормализовать path и запрещать `..`
- заменить public signing endpoint на server-side session

## Шаг 11: Перепроверка

Всегда валидируй обе стороны:

- сервис всё ещё делает то, что нужно чекеру
- exploit path умер

Минимальные проверки:

- registration/login всё ещё работают, если чекер их использует
- создание объекта всё ещё работает
- легитимный owner всё ещё читает свой флаг
- исходный exploit proof или его уменьшенная версия больше не работают
