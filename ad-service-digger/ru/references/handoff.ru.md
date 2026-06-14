# Handoff

## Зачем этот файл

Используй его, когда один агент делает раскопку сервиса, а другой потом пишет сплойт, фикс или farm script без повторного полного анализа.

Этот артефакт не является writeup для оргов.
Это внутренний инженерный handoff для команды или другой нейронки.

Считай один handoff одним dossier по сервису.
Если в сервисе несколько уязвимостей, держи их в одном файле, если только это не разные deployment-state или не разные раундовые состояния.

## Когда нужно делать handoff

Делай handoff, если верно хотя бы одно:

- раскопка уже глубокая, и повторять её дорого
- другая нейронка будет писать exploit
- другая нейронка будет писать fix
- раунд продолжится позже и контекст надо сохранить
- source, traffic и local repro уже размазаны по случайным заметкам

## Что должен давать хороший handoff

Второй агент должен понять без переанализа:

- что делает сервис
- где лежит флаг
- как checker его пишет и читает
- какие exploit path подтверждены, а какие пока гипотезы
- какие candidate exploit paths появились по source review ещё до live-проверок
- какие findings являются primary, secondary или defense-critical
- какие пути относятся к `no-flag-id`, `needs flag_id`, `needs victim` или вообще не подходят для фермы
- какое следующее действие самое выгодное
- какие файлы, routes, ports и object names реально важны

## Обязательные секции

Минимум должны быть:

1. Краткое summary сервиса
2. Архитектура и порты
3. Flag flow
4. Identities и trust boundaries
5. Таблица findings
6. Уровень evidence для каждого finding
7. Ранжирование findings
8. Ранжированный список candidate exploit paths из source review
9. Primary и secondary exploit paths
10. Направление фикса и порядок фикса
11. Exploit amplifiers и weak config
12. Checked-but-not-promoted items
13. Open questions
14. Concrete next actions
15. Остатки после coverage pass

## Метки evidence

Для каждого finding используй явные лейблы:

- `source-suspected`
- `locally-confirmed`
- `checker-matched`
- `live-confirmed`
- `farm-ready`
- `patched-closed`

Не своди это в один расплывчатый абзац про severity.
И не смешивай findings с exploit paths. Один exploit path может использовать один finding или цепочку из нескольких.

## Правила написания

- Предпочитай факты, а не длинную прозу.
- Указывай точные paths, route names, ports, headers, object names, field names и role names.
- Отделяй подтверждённые факты от предположений.
- Давай каждому finding стабильный ID вроде `F1`, `F2`, `F3`.
- Давай каждому exploit path стабильный ID вроде `P1`, `P2`.
- Явно помечай farmability для finding или path, когда это важно.
- Если exploit path уже пробовали и он не взлетел, напиши это явно.
- Если local и live различаются, вынеси это наверх.
- Если checker behavior известен, пиши это ближе к началу, а не глубоко внизу.
- Если live-checks ещё не начинались, напиши это явно и не создавай ложное ощущение подтверждения.
- Если finding не является лучшим путём атаки, но его всё равно важно закрыть при защите, помечай его как `defense-critical`.
- Добавляй короткий список классов, которые были проверены, и классов, которые ещё не успели проверить.
- Держи exploit amplifiers и weak config отдельными секциями, а не раздувай их до primary findings.

## Как называть файл

Используй предсказуемое имя:

- `service-handoff.md`
- или `handoff-<service>.md`

Если заметка привязана к раунду:

- `handoff-<service>-round-<n>.md`

## Шаблон

Используй:

- `assets/templates/service-handoff.md`
