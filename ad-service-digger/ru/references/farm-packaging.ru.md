# Farm Packaging

## Когда читать этот файл

- когда пишется exploit script для команды или фермы
- когда нужно упаковать no-victim или no-flag-id exploit
- когда после подтверждения уязвимости надо собрать чистый архив

Этот файл начинается после основной vuln research.
Не надо сначала паковать, а потом только проверять идею.

## Базовый farm contract

Если пользователь не сказал иначе:

- цель передаётся через `argv[1]`
- в stdout печатаются только флаги, по одному на строку
- диагностика и прогресс идут в stderr
- состояние хранится per-target, а не одним глобальным файлом на все команды
- не хардкодь live-секреты, если их можно достать во время выполнения
- когда возможно, предпочитай stdlib-only реализацию

## Workflow упаковки

1. Подтверди scope эксплойта:
   - кража флагов
   - no-victim или нужен victim
   - no-flag-id или нужен flag id
   - DoS/state-corruption вместо theft
2. Напиши самый маленький надёжный скрипт.
3. Сделай local smoke test с synthetic flag-like marker.
4. Проверь, что stdout содержит только флаги.
5. Упакуй только нужные файлы.
6. Зафиксируй точную команду запуска и оставшиеся blocker'ы.

## Farmability gate

Явно классифицируй путь:

- `no-flag-id`
- `needs flag_id`
- `needs user_id`
- `needs victim`
- `DoS/state-corruption`
- `non-flag`

Не называй путь farm-ready, пока эта метка не ясна.

## Hygiene архива

Не включай:

- `__pycache__/`, `*.pyc`
- `.env`, secrets, tokens, SSH keys
- DB dumps, WAL/SHM файлы
- логи, pcap, browser artifacts
- старые неудачные PoC
- нерелевантные helper scripts

## Частые ошибки

1. Печатать debug JSON или ID в stdout и ломать ферму.
2. Забывать про flush для флагов.
3. Держать один watermark/state file на все хосты.
4. Паковать victim-required browser bait как no-victim exploit.
5. Отправлять архив, не проверив его содержимое.
6. Называть `flag_id` endpoint farmable, не имея способа получить `flag_id`.

## Что передавать в handoff

Если exploit path уходит другой нейронке, приложи:

- целевой порт и протокол
- ожидаемое auth state
- flag-bearing field
- farmability label
- run command
- статус local smoke-test
