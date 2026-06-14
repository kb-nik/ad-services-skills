# Web

## Когда читать этот файл

- обычные HTTP API
- Flask/FastAPI/Go/Nim/Phoenix/Node web-сервисы
- frontend + API стеки
- LiveView, WebSocket, server-pushed UI, preview/export flows

Держи первый проход коротким:

- быстро отрисовать карту приложения
- подтвердить trust boundary
- понять, какие объекты похожи на носители флага
- и только потом нырять в глубокие техники

## Поверхности с высокой отдачей

Проверь это раньше экзотики:

- registration и login
- JWT/session parsing
- profile update и поля, влияющие на role/scope
- search, listing, gallery, export, preview, share, report, webhook, callback
- object read/write endpoints, где одновременно есть `project_id` и `object_id`
- admin panels, hidden routes, internal API namespaces
- file upload, SVG/PDF rendering, template rendering

## Recon первого прохода

До глубокого анализа:

- посмотри page source на comments и hidden hints
- прочитай JS bundles на hidden endpoints и accepted fields
- сравни, что отправляет UI, и что реально принимает API
- проверь common paths вроде `/robots.txt`, `/sitemap.xml`, `/.well-known/`, `/admin`, `/api`, `/debug`
- зафиксируй custom headers, auth hints и alternate content types
- попробуй direct API calls в обход фронтенда

## Типовые web bug families в A/D

### Auth и Session

- hardcoded JWT secret
- algorithm confusion
- trust в unsigned payload fields
- отсутствие expiry checks
- legacy или alternate verification paths
- bypass через service-role или `kind=service`
- public или слабо защищённые admin login/bootstrap routes
- cookie seeding, когда public flow выставляет привилегированную cookie
- слабая проверка секрета или хэша через `slice`, `substring`, `startsWith`

### Ownership и IDOR

- endpoint проверяет доступ к контейнеру, но не ownership объекта
- `project_id` проверяется, а `font_id` или `ticket_id` не связывается с ним
- hidden/internal fields утекают через export/share endpoints
- public search или gallery раскрывают private object identifiers

### Server-Side Execution и Fetching

- SSTI в report/template/preview/message rendering
- SSRF в webhook, callback URL, print service, preview renderer, fetch helper
- redirect в private address space
- доступ к internal-only endpoints через SSRF

### Frontend Trust

- `innerHTML` с user-controlled data
- LiveView или WebSocket events проверяют доступ только на mount
- скрытая кнопка в UI не заменяет server-side authorization
- client-side access gates на URL params, globals или frontend-only checks
- контент скрыт CSS/JS, но реально лежит в HTML или API output

### Query и Search

- raw interpolation в search, filter, sort, weight parameters
- частичный numeric parsing, после которого attacker input всё ещё проходит
- error branch, после которого код доходит до success serialization

### Routing и Edge Behavior

- encoded slash или path normalization mismatch вроде `%2F`
- middleware висит только на одной форме route, а не на другой
- method confusion: `TRACE`, `PUT`, `PATCH`, `DELETE` проходят там, где `GET`/`POST` блокируются
- proxy/ingress regex mismatch между allowlist и application router

## Порядок исследования web-сервиса

1. Проследить создание и верификацию auth/session.
2. Сравнить route guards с тем, как реально достаются объекты.
3. Проверить mass-assignment и profile update schemas.
4. Проверить search/list endpoints на SQL-like interpolation и excessive data return.
5. Проверить export/preview/report/callback/webhook logic.
6. К frontend event handlers идти после того, как понятны server handlers.
7. Проверить alternate route forms, methods, encodings и direct API access, которые обходят intended UI flow.

## Статический coverage gate до валидации

Не переходи в local validation только потому, что уже есть две сильные гипотезы.
Для компактного web-сервиса сначала закрой минимальный static gate:

- auth routes прочитаны целиком
- object CRUD и list/detail routes прочитаны целиком
- model fields, связанные с ownership и flag storage, найдены
- template sinks и `innerHTML`-подобные render sinks просмотрены
- очевидные пути `body/query -> ORM lookup` просмотрены
- каждый просмотренный sink помечен как escaped, partially escaped или unescaped
- exposure по compose/runtime для DB, Redis, broker, admin или sidecar сервисов просмотрен
- источники секретов просмотрены: session secret, JWT/HMAC key, default creds, permissive env fallback

Только после этого можно говорить, что контекст достаточен для local validation.
Если gate не закрыт, продолжай читать исходники, а не переходи к Docker/runtime-проверкам.

## Проверки секретов и внутренних поверхностей

Не оставляй это между строк.
Для маленьких web-стеков явно рапортуй:

- hardcoded session secret или нет
- cookie flags слабые или приемлемые
- JWT/HMAC/admin secrets есть или нет
- опубликованы ли Mongo/Redis/broker/admin/sidecar порты
- если БД или broker торчит наружу, это direct exploit surface, exploit amplifier или пока только weak exposure

Даже если ничего сильного не нашли, всё равно пиши, что эти классы были проверены.

## Правила review для sink'ов

Когда проверяешь XSS или render issues:

- перечисляй конкретные sink'и
- называй точное поле, которое доходит до каждого sink
- отдельно указывай, происходит escaping в самом sink, до sink или не происходит вообще
- считай string concatenation в `innerHTML` подозрительной, пока не проверено каждое интерполируемое поле
- рапортуй результат в виде таблицы verdict по sink'ам: одна строка на каждый просмотренный sink
- если релевантных template/frontend sink'ов нет, всё равно оставляй таблицу с одной явной строкой `не найдено`

Не своди это к одной фразе вроде "frontend XSS checked".
Именно из-за такой сжатости и пропускаются реальные баги.

## Coverage pass для маленьких web-сервисов

Если сервис достаточно компактный, чтобы его можно было прочитать целиком, не останавливайся на первой подтверждённой дыре.
Сделай короткий residual checklist и для каждого пункта пометь `checked`, `confirmed` или `not reviewed yet`:

- type confusion в login и register
- ORM/operator injection через body или query-объекты
- ownership в list, detail и create-linked readback flows
- enumeration через последовательные ID или публичные счётчики
- hidden fields, которые API принимает, хотя UI их не показывает
- проблемы session secret, но только после проверки, делает ли store это реально эксплуатируемым, или это пока просто плохая hygiene
- усиление strongest chain:
  - можно ли путь, который сейчас требует username, сделать `no-username`?
  - можно ли путь, который сейчас требует `flag_id`, сделать `no-flag-id`?

Этот проход нужен специально для случаев, когда отчёт вроде правильный, но неполный.

## Как аккуратно рапортовать web-находки

Для компактных сервисов разделяй четыре корзины:

- `confirmed finding`: прямо эксплуатируемый баг или явно flag-relevant weakness
- `exploit amplifier`: публичные usernames, последовательные ID, утечки object names, предсказуемые счётчики
- `weak config`: реальная, но менее ценная слабость вроде слабых cookie flags или hardcoded secret без немедленного exploit path
- `checked but not promoted`: подозрительный класс проверили, но пока недостаточно оснований называть это finding

Такой формат обычно лучше, чем пытаться впихнуть всё в один severity-список.

Предпочитай таблицы в финальном отчёте:

- таблица findings
- таблица sink inventory
- таблица negative checks

Для sink inventory делай по одной строке на sink, а не одну общую фразу на весь frontend.
Заголовки секций, названия таблиц, колонки и короткие статусы тоже держи на языке пользователя.

## Вопросы, сфокусированные на флаге

- В каком поле вероятнее всего лежит флаг: note, answer, artifact, design note, resolution, callback response?
- Можно ли заставить public или low-privilege flow отрендерить это поле косвенно?
- Можно ли триггернуть privileged fetch и прочитать ответ назад?
- Можно ли overwrite/publish victim-owned object, а потом его прочитать?
