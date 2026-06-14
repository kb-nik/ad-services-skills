# Pwn

## Когда читать этот файл

- native services, где memory corruption уже видна или очень вероятна
- stack overflow, format string, heap corruption, integer truncation или race-to-corruption paths
- бинарные сервисы, где главный блокер уже не понимание логики, а превращение крэша или primitive в надёжный доступ к флагу

Не начинай отсюда, если ты всё ещё не понимаешь, что делает бинарь.
В таком случае сначала открой `reverse.ru.md`.

## Первый проход

Сделай самый маленький triage, который показывает, реалистична ли эксплуатация:

- `checksec --file ./binary`
- подтверждение архитектуры и предположений о libc через `file` и `ldd`
- понимание input surface: line-based TCP, length-prefixed frames, file parser, HTTP-to-native bridge
- классификация primitive: stack overwrite, format string, heap misuse, integer bug, race
- подтверждение того, чем ты управляешь: байты записи, длина чтения, число повторов, reconnect behavior, fork-per-request

В A/D полноценная shell не обязательна.
Стабильный flag read, role flip или data dump обычно уже достаточно.

## Triage защит

Используй protections, чтобы рано выбрать направление:

- no PIE: фиксированные адреса кода упрощают ret2win и GOT targets
- partial RELRO: GOT overwrite может быть самым быстрым путём
- full RELRO: лучше смотреть в сторону return-address control, function pointers, heap metadata или vtables
- NX enabled: планируй ROP, ret2libc или non-code-exec data abuse
- stack canary present: сначала утечка, либо уход со stack-примитива, либо non-return-address primitive

Не вкладывайся слишком рано в красивую цепочку, пока не ясно, какая защита реально мешает.

## Карта primitive -> направление

### Stack Overflow

Сначала выясни:

- точный offset
- контролируется ли RIP
- происходит ли крэш до возврата из функции
- перезапускается ли сервис или форкается

Типовые направления:

- ret2win, если символы и адреса фиксированы
- ret2libc, если можно утечь libc
- короткий ROP на `open`, `read`, `write`, если shell не нужна
- stack pivot, если места под полноценную цепочку мало

### Format String

Сначала выясни:

- контролируешь ли ты сам format string
- что можно утечь: stack, PIE base, libc, canary
- доступны ли записи через `%n`

Типовые направления:

- утечь canary и потом вернуться к stack overflow
- перезаписать GOT, только если RELRO это позволяет
- использовать single-shot arbitrary write для auth flags, role bits или function pointers

### Heap Corruption

Сначала выясни:

- паттерн выделений
- паттерн освобождений
- поведение reuse
- важна ли конкретная версия libc

Типовые направления:

- UAF ради read/write на чувствительных объектах
- double free или tcache poisoning для pointer redirection
- порча соседней структуры ради смены auth state или callback’а

В A/D перезапись поля структуры, которая открывает флаги, часто лучше, чем полный `system("/bin/sh")`.

### Integer Or Sign Bugs

Сначала выясни:

- truncation длины
- signed-to-unsigned conversion
- overflow в multiplication или addition
- mismatch между типом валидации и размером копирования

Типовые направления:

- маленькая allocation, за которой следует overflow
- negative index или OOB access
- обход balance/quota checks с выходом на privileged path

### Race Or TOCTOU

Сначала выясни:

- общее глобальное состояние
- временные файлы
- unlink/rename windows
- multi-threaded auth или quota updates

Типовые направления:

- flip validation-result state
- подмена уже проверенных paths или objects
- выигрыш на повторных попытках, если сервер форкается или окно стабильно

## Приоритеты для A/D

- предпочитай flag read или targeted data exfil вместо shell, если это быстрее и надёжнее
- предпочитай двухстадийный exploit с одной утечкой и одним действием, а не хрупкую all-in-one цепочку
- документируй точные offsets, gadgets, предположения о libc и reconnect behavior для команды
- если крэш только доказывает corruption, но не ведёт к flag flow, откатывайся и ищи более дешёвый trust-boundary bug

## Когда переключаться

- обратно в `reverse.ru.md`, если поведение бинаря всё ещё неясно
- в `web.ru.md`, если реальный exploit path больше зависит от auth или request routing, чем от native corruption
- в `protocols-and-storage.ru.md`, если memory corruption реальна, но это не самый быстрый путь к флагам

## Минимум для handoff

Если exploit будет писать другой агент, зафиксируй хотя бы:

- имя бинаря и архитектуру
- состояние защит из `checksec`
- тип primitive
- точный crash/control offset
- подтверждённые leak sources
- целевую функцию, объект или syscall plan
- находится ли путь на уровне `locally-confirmed`, `live-confirmed` или уже `farm-ready`
