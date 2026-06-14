# Case Studies

## Как использовать этот файл

Читай его только после того, как уже понятен общий workflow и категория сервиса.
Это примеры паттернов из реальных A/D раскопок, а не канонический список типов сервисов.

Используй каждый пример так:

- выдели базовый паттерн
- переиспользуй паттерн только если новый сервис реально похож
- игнорируй имя сервиса, если сходства нет

## s3waaas

- checker traffic показал реальный flag path: random bucket, random object, analytics flow
- главный угол атаки был не только в UI, а в privileged analytics job и доверии к internal header
- bucket names и object keys затрагивали filesystem boundaries, поэтому traversal тоже был релевантен

Общий паттерн:
Сначала проверь background jobs и internal trust, а не залипай только в public CRUD endpoints.

## pontiger

- HTTP controller methods отправляли ошибку, но продолжали выполнение и сериализовали sensitive data
- search раскрывал цели, HTTP тек access token, SSH auth helper не связывал repo name с owner

Общий паттерн:
В repo-like сервисах сравнивай HTTP auth logic, SSH auth logic и object ownership отдельно.

## kakafka

- public web UI был только одной гранью; внешняя Kafka на `9092` и RPC topic behavior были не менее важны
- secrets и Flask session trust были так же важны, как broker ACLs

Общий паттерн:
Если пользователи создают protocol-level identities, обязательно проверь сам broker, а не только web-слой.

## pony-stark

- бинарь был завязан на Unicorn/emulation semantics; путь к флагу шёл через emulated syscalls и storage access
- strings, hooks, syscall list и output behavior дали больше пользы, чем слепые shellcode attempts

Общий паттерн:
Для emulator-style сервисов сначала пойми syscall mediation model, а потом пиши payload.

## mspd2

- source и live deployment не совпадали
- два очевидных source bug были мертвы на live, а реальный путь сидел в detective-role challenge внутри deployed sidecar

Общий паттерн:
Всегда отделяй live validation от source analysis. Мёртвый source bug не равен live exploit.

## ServiceDesk

- быстрые выигрыши пришли из secrets, mass assignment, SSTI, SSRF и share-token determinism
- порядок фиксов был важен: сначала ротация секретов, потом privesc и renderer abuse

Общий паттерн:
Когда времени мало, чини сначала баги, которые схлопывают сразу несколько downstream exploit path.

## typevault

- public search/gallery, callback SSRF, LiveView auth gaps и frontend XSS все были релевантны
- сервис смешивал API, rendering, event handlers и internal status endpoints

Общий паттерн:
В rich web apps не останавливайся на REST routes. Проверяй event handlers, preview features и internal controllers.

## pdfgen

- public signing service и слабая крипта делали auth forgery реальным
- print/render side service давал SSRF в internal-only export paths

Общий паттерн:
Любой публичный helper, который умеет sign/fetch/render/print, считай privilege boundary.

## identityhub

- практическая защита потребовала прямого patch’а `.so` бинарей
- byte-level patching плюс точная проверка байт были реалистичным способом защиты

Общий паттерн:
Для binary-only компонентов "surgical patch plus verify bytes" — это нормальный A/D workflow, а не крайняя мера.
