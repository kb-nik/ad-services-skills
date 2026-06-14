# Reverse

## Когда читать этот файл

- ELF services
- `.so` libraries, которые нужно патчить
- `.i64` databases или decompiler artifacts
- garble-obfuscated Go binaries
- emulator, VM, crypto или challenge-sidecars

Не начинай с reverse-ракурса, если баг уже понятен, а дальше нужна только эксплуатация или патч. Используй его тогда, когда именно понимание реализации является главным блокером.

## Первый проход

Сначала делай самую дешёвую разведку:

- `file`, `strings`, `checksec`
- imported symbols
- очевидные file paths вроде `/storage/flag`, `/app`, `/tmp`, `/proc`
- protocol markers, JSON keys, route names, role names
- syscall numbers или libc wrappers

Для Go или stripped binaries ищи:

- route strings
- JSON field names
- database table names
- role names
- error strings
- challenge field names

## Компактный toolkit

Используй самый маленький инструмент, который отвечает на следующий вопрос:

- layout и metadata: `file`, `checksec`, `readelf`, `objdump`, `nm`
- быстрые semantic hints: `strings`
- статический reverse: Ghidra, IDA, Binary Ninja, radare2/Cutter
- syscalls и runtime behavior: `strace`, `ltrace`
- debugger-level подтверждение: `gdb`, `lldb`
- помощь по Go-бинарям: GoReSym, redress
- dynamic instrumentation, если статика встала: Frida
- path exploration или constraint solving по необходимости: angr, z3

Старайся сначала брать лёгкие наблюдения, а потом уже тяжёлые фреймворки.

## Что искать

### Capability Leaks

- arbitrary file read через эмулируемые syscalls
- command execution через helper calls
- signature или crypto misuse
- challenge endpoints, которые gate’ят role escalation

### State Machine Mistakes

- сервер хранит challenge state per-user и игнорирует client-supplied fields
- успех виден только по side effect вроде смены роли
- source logic и deployed binary расходятся

### Patchability

- одно- или мало-байтные branch patch’и для убийства auth bypass
- early return в exported helper functions
- server-side session replacement, если client token format скомпрометирован

### Reverse-heavy паттерны

- custom VM или bytecode interpreter
- anti-debug или anti-analysis logic
- signal-handler driven control flow
- self-modifying или self-checking code
- packers, staged loaders или runtime decryptors
- custom syscall mediation
- validation logic, спрятанная в арифметике или constraint checks
- side-channel oracle через timing, signals, exits или counters

## Reverse workflow

1. Подтвердить внешний protocol surface.
2. Вытащить strings и type hints.
3. Найти один primitive, который открывает дорогу к флагу: file read, role flip, data dump, token forge.
4. Подтвердить этот primitive самым маленьким proof.
5. Только потом делать чистый exploit.

## Как патчить бинарники

Когда исходников нет или они неактуальны:

- сначала сделать backup бинаря
- патчить самый маленький branch или function prologue
- после патча проверить точные bytes
- документировать offsets и ожидаемые byte patterns
- перезапустить и сделать smoke test

Не предполагай, что source patch имеет смысл, если live service собран из другой версии.
