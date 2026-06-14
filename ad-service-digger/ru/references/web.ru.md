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

## Вопросы, сфокусированные на флаге

- В каком поле вероятнее всего лежит флаг: note, answer, artifact, design note, resolution, callback response?
- Можно ли заставить public или low-privilege flow отрендерить это поле косвенно?
- Можно ли триггернуть privileged fetch и прочитать ответ назад?
- Можно ли overwrite/publish victim-owned object, а потом его прочитать?
