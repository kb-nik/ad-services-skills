# Traffic Triage

## Когда читать этот файл

- при анализе pcap или производных от pcap
- при разборе traffic PDF
- при live-вопросах вида "у нас прямо сейчас воруют флаги?"
- при сравнении трафика с уже известными exploit pack'ами

Это файл именно для attack-defense theft triage, а не для общего packet analysis.
Главный вопрос тут не "есть ли где-то строка, похожая на флаг", а "родился ли флаг в ответе сервиса".

## Базовое правило

Не называй что-то утечкой, пока не отделил:

- checker traffic
- флаги, которые атакующий сам прислал, а сервис просто отзеркалил
- mirror/export duplication
- настоящий server-origin leak

Самая короткая полезная модель такая:

- `server_flags - client_flags` = кандидаты на утечку
- `server_flags ∩ client_flags` = скорее echo или replay, пока не доказано обратное

## Workflow

1. Определи окно захвата и тип входных данных.
2. Сопоставь порты с сервисами и протоколами.
3. Найди checker IP, наши хосты и mirror/export channels.
4. Посчитай направленный flag flow по stream'ам.
5. Классифицируй каждый подозрительный случай как real leak, checker, echo, noise или backlog candidate.
6. Сверь это с уже существующими exploit-заметками перед тем как называть что-то новым.

## Что фиксировать

- окно захвата
- checker IP или пометку `unknown`
- карту `service/port/protocol`
- `src` и `dst` для важных stream'ов
- кусок запроса и ответа
- был ли флаг сначала в request
- является ли stream checker-origin

## Классификация

- `confirmed leak`: не-checker флаг пришёл из ответа сервиса и не был раньше в запросе
- `echo/no theft`: флаг был в запросе и просто отзеркалился
- `checker traffic`: легитимный checker put/get или validation flow
- `probe/noise`: попытка эксплуатации без server-origin flag
- `new exploit gap`: путь выглядит реальным и ещё не покрыт локальными заметками
- `known/covered`: уже есть в backlog эксплуатации или защиты
- `unconfirmed`: PDF или статический hint без достаточного подтверждения

## Частые ошибки

1. Считать любой flag-like string в ответе кражей.
2. Забывать, что checker сам переносит флаги по сети.
3. Считать real leak'ом флаг, который атакующий сам создал и сервис просто вернул.
4. Игнорировать mirrored/exported traffic с дубликатами payload'ов.
5. Называть PDF finding новым, не сверившись с текущими exploit notes.
6. Игнорировать длину capture window и делать слишком сильные выводы по короткому срезу.

## Форма ответа

Держи summary коротким:

- Window
- Confirmed stolen flags
- Services with real leaks
- Echo или probe noise
- New backlog candidates
- Blockers
