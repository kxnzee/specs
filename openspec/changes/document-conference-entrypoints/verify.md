# Приёмка изменения

**Изменение:** `document-conference-entrypoints`

<!-- SCENARIO_VERIFICATION_CONTRACT_V1_START -->
## Объект проверки

- **Требования:** Delta Specs для этого Change отсутствуют (`skip_specs: true`,
  обоснование — `proposal.md` § Capabilities/Impact: изменение не затрагивает
  наблюдаемое поведение системы, только developer-документацию). Признак успеха
  зафиксирован в `intake.md` § 2.1 и раскрыт сценариями `intake.md` § 2.2–2.4;
  design-контракт анкеров — `design.md` § Decisions (третий пункт).
- **Проверяемый результат (повторная проверка после синхронизации checkout с
  `origin/master`):**
  - `jitsi-web` — `git fetch origin master` выполнен в этой сессии;
    checkout `finalize/workspace/src/jitsi-web` находится в состоянии
    `HEAD detached at origin/master`, `HEAD` = `origin/master` =
    `28f9bd63cb80a16e4985c7d6d3c644cabc412e71` (PR
    https://github.com/kxnzee/jitsi-meet/pull/1, merge второго родителя —
    doc-коммит `a865eecd10f91cc0540f241ef00a77d87aa2fdc0`), рабочее дерево
    чистое. Точное совпадение `HEAD` и `origin/master` подтверждено `git log -1`
    по обеим ссылкам в одной команде.
  - `jitsi-control` — `git fetch origin master` выполнен в этой сессии;
    checkout `finalize/workspace/src/jitsi-control` находится в состоянии
    `HEAD detached at origin/master`, `HEAD` = `origin/master` =
    `03ece5e4ee705c39e6e1e55d8ab50027065383cd` (merge независимого Change
    `document-media-routing-boundaries`, PR
    https://github.com/kxnzee/jicofo/pull/2), рабочее дерево чистое. Проверяемый
    для этого Change merge-коммит `e380c14a41078e7aa5b70933aab3f8ed48a0a298`
    (PR https://github.com/kxnzee/jicofo/pull/1, doc-коммит
    `7134f75c2044d3857aeebe72e9451a884e45d259`) подтверждён как предок текущего
    `HEAD`: `git merge-base --is-ancestor e380c14a HEAD` → успех. Файл
    `doc/conference-entrypoint.md`, добавленный этим коммитом, присутствует и
    не изменялся более поздним merge (проверено построчным чтением анкеров
    ниже против текущего `HEAD`).
  - `jitsi-videobridge` — `git fetch origin master` выполнен в этой сессии;
    checkout находится в состоянии `HEAD detached at origin/master`, `HEAD` =
    `origin/master` = `a2f5fba8c5bd018216a352e6db2cb0c70c635d91` (merge
    независимого Change `document-media-routing-boundaries`, не относящийся к
    этому Change). No-change подтверждён повторно: `find ... -iname
    "*conference-entrypoint*"` в этом checkout → пусто.
  - Источник соответствия — `implementation-map.yaml` в Store (записи
    `repository_id: jitsi-web` / `jitsi-control`, `version: 1`); указанные там
    commit-хеши (`28f9bd63...`, `e380c14a...`) совпадают с зафиксированными выше
    merge-коммитами этого Change и подтверждены как текущий `HEAD` (`jitsi-web`)
    либо как предок текущего `HEAD` (`jitsi-control`) после синхронизации с
    `origin/master`.
- **Условия проверки:** повторная post-merge проверка в smoke-окружении,
  изолированном от продакшена, после синхронизации всех трёх временных
  Code Repository checkout с точным `origin/master` (см. выше). Проверка
  выполнена прямым чтением текущих файлов и `git fetch`/`git log`/`git
  show`/`git merge-base` в checkout `jitsi-web`, `jitsi-control` и
  `jitsi-videobridge` (без CodeGraph — индекс не использовался для этой
  проверки, применялось прямое чтение исходников по точным анкерам из
  опубликованных документов). Прямые сетевые вызовы к GitHub API не
  выполнялись; PR-идентификация подтверждена локальной историей merge-коммитов
  после `git fetch` от реального `origin`.

## Покрытие сценариев

| Scenario ID или точная ссылка | Подтверждение выполненной проверки | Результат |
| --- | --- | --- |
| `intake.md` § 2.2 «Основной сценарий» (путь `jitsi-web` → `jitsi-control`, ролевые анкеры и failure-branch) | Повторно прочитаны оба опубликованных документа после синхронизации checkout с `origin/master`: `jitsi-web/doc/conference-entrypoint.md:11-31`, `jitsi-control/doc/conference-entrypoint.md:12-28`. Каждый цитируемый анкер сверен построчно с исходным кодом на текущем `HEAD` (= `origin/master`): `conference.js:233,347,351,259,286-290,300` (jitsi-web) и `ConferenceIqHandler.kt:47,80,97`, `FocusManager.kt:48,90,100-120,129` (jitsi-control) — все совпадают буквально. Расхождение с Planning-time анкером `FocusManager.kt:88` из `intake.md` § 9 объяснимо и допустимо: `design.md` прямо требует повторной проверки анкера в Apply-time против текущего источника, а не переноса Planning-значения; опубликованный `:90` — актуальная строка и остаётся актуальной после синхронизации. | `PASS` |
| `intake.md` § 2.3, пункт 1 (документ читаем в одиночку и кросс-линкует смежный) | Оба документа содержат раздел «See also» с корректной ссылкой на файл и репозиторий другой стороны: `jitsi-web/doc/conference-entrypoint.md:33-36` → `jicofo/doc/conference-entrypoint.md`; `jitsi-control/doc/conference-entrypoint.md:30-33` → `jitsi-meet/doc/conference-entrypoint.md`. Путь и репозиторий в обеих ссылках соответствуют фактическому расположению файлов. | `PASS` |
| `intake.md` § 2.3, пункт 2 (устаревание документации при будущих изменениях кода) | Не тестируемый сценарий поведения — это принятый риск, а не критерий приёмки. Митигация зафиксирована в `design.md` § Risks/Trade-offs (анкер file:line, без автоматического обнаружения дрейфа), реализация её не отменяет. | `N/A` — вне области проверяемого результата; сценарий описывает будущий риск, а не текущее поведение. |
| `design.md` § Decisions, пункт 3 (без дублирования wire-format `conference-request.md`) | Прочитаны `jitsi-control/doc/conference-request.md` (61 строка, HTTP/XMPP payload-форматы) и `jitsi-control/doc/conference-entrypoint.md` — пересечения по JSON/XMPP payload-содержимому не обнаружено; entrypoint-документ ссылается на protocol-документ вместо повторения содержимого. | `PASS` |
| `tasks.md` 1.1 / 2.1 (доки не меняют код/конфигурацию/тесты) | `git show --stat` повторно выполнен для обоих merge-коммитов (`28f9bd63`, `e380c14a`) на синхронизированном `origin/master`: каждый добавляет ровно один файл `doc/conference-entrypoint.md` (36 строк), без изменений в других файлах. | `PASS` |

Полный список сценариев Delta Specs отсутствует, так как `proposal.md` устанавливает
`skip_specs: true` с явным обоснованием (документация не меняет наблюдаемое
поведение системы). В качестве замещающего критерия приёмки использованы
единственные зафиксированные в Planning сценарии из `intake.md` § 2.2–2.3 и
дополнительный контракт из `design.md` § Decisions — оба источника приняты в
рамках этого Change и не пересматривались после Apply.
<!-- SCENARIO_VERIFICATION_CONTRACT_V1_END -->

<!-- FEATURE_ACCEPTANCE_CONTRACT_V1_START -->
## Дополнительные проверки

| Проверка | Подтверждение | Результат |
| --- | --- | --- |
| Все задачи `tasks.md` завершены | `openspec instructions apply --change "document-conference-entrypoints" --json` → `state: "all_done"`, `progress: {"total":5,"complete":5,"remaining":0}` (повторно проверено в этой сессии) | `PASS` |
| Строгая валидация Store | `openspec validate document-conference-entrypoints --strict --no-interactive` → `Change 'document-conference-entrypoints' is valid` (повторно выполнено в этой сессии) | `PASS` |
| OpenSpec Graph impact | `get_spec_change_impact(change_id: "document-conference-entrypoints")` → `state: "ready"`, `summary: {"errors":0,"warnings":2}`; оба warning (`REPOSITORY_IMPACT_MISSING`, `UNLINKED_MASTER_SPEC`) относятся к архивному Change `2026-08-18-jit-002-moderated-visitor-promotion` и не касаются `document-conference-entrypoints` (для этого Change: `delta_specs: []`, `repositories: []`, `edges: []` — ожидаемо при `skip_specs: true`) | `PASS` |
| Синхронизация checkout с `origin/master` | `git fetch origin master` выполнен для всех трёх репозиториев в этой сессии; `jitsi-web`, `jitsi-control`, `jitsi-videobridge` — все три в состоянии `HEAD detached at origin/master` с `HEAD` буквально равным `origin/master` (проверено `git log -1` по обеим ссылкам) | `PASS` |
| Соответствие `implementation-map.yaml` фактическому checkout после синхронизации | Commit-хеш `jitsi-web` (`28f9bd63...`) совпадает с текущим `HEAD` = `origin/master`; commit-хеш `jitsi-control` (`e380c14a...`) подтверждён как предок текущего `HEAD` = `origin/master` (`git merge-base --is-ancestor` → успех), поскольку поверх него смёржен более поздний независимый Change; PR-номера (`pull/1` в обоих репозиториях) совпадают с текстом merge-коммитов («Merge pull request #1») | `PASS` |
| `jitsi-videobridge` — подтверждённое no-change | Повторно проверено на синхронизированном `origin/master`: `find .../jitsi-videobridge -iname "*conference-entrypoint*"` → пусто; `HEAD` = merge независимого Change `document-media-routing-boundaries` | `PASS` |
| Markdown валиден | Оба файла повторно прочитаны как обычный markdown на синхронизированном `HEAD` (заголовки, списки, code-spans), синтаксических ошибок не обнаружено | `PASS` |
| Ручное ревью через PR (задача 3.1) | Out-of-band evidence из текущей сессии: пользователь запросил ссылки на PR словами «я проверю и залью», ссылки на https://github.com/kxnzee/jitsi-meet/pull/1 и https://github.com/kxnzee/jicofo/pull/1 были переданы пользователю, после проверки пользователь сообщил «все залил», а локальная git-история после `git fetch` от реального `origin` подтверждает merge обоих PR (merge-коммиты `28f9bd63...` и `e380c14a...`; второй является предком текущего `03ece5e...`, см. выше). Это не формальное GitHub review event (approve/comment через API не выполнялся и не проверялся) — фиксируется только выполнение условия задачи (PR создан, ссылка на review запрошена и передана, PR смёржен), а не содержательное решение ревьюера | `PASS` |

Строгая валидация выполнена для текущего кандидата (`document-conference-entrypoints`,
merge-коммиты выше) на состоянии checkout, синхронизированном с `origin/master` в этой
сессии; при изменении проверяемого варианта требуется повтор.

## Решение о приёмке

- **Решение:** `PASS`
- **Комментарий:** Пользователь явно подтвердил Feature Acceptance: PASS для
  этого Change повторно, в рамках текущей post-merge Verify сессии после
  синхронизации всех трёх временных Code Repository checkout с точным
  `origin/master` (аргумент команды `/opsx:verify`). Решение опирается на
  собранные выше свидетельства, полностью переснятые в этой сессии на
  синхронизированном состоянии: все 5 задач `tasks.md` завершены,
  `openspec validate --strict` проходит без ошибок, OpenSpec Graph сообщает
  только диагностики, не относящиеся к этому Change, оба документа
  присутствуют в соответствующих Code Repositories на `origin/master` с
  анкерами, дословно совпадающими с текущим исходным кодом, кросс-ссылки
  корректны, дублирования wire-format контента нет, изменения кода отсутствуют
  во всех трёх репозиториях. Единственный `N/A` (будущий дрейф документации)
  обоснован как принятый риск вне тестируемой области этого Change. Ревью
  задачи 3.1 подтверждено out-of-band evidence текущей сессии (см. таблицу
  выше), а не формальным GitHub review event.

Приёмка не выполняет архивирование, UAT или выпуск. Перед архивированием изменения
через PR в Store требуется `PASS`. Архивирование не разрешает выпуск: для него
по-прежнему нужны UAT и отдельное решение.
<!-- FEATURE_ACCEPTANCE_CONTRACT_V1_END -->

## Соблюдение процесса

`NOT_APPLICABLE` — схема `spec-driven-extended` не предписывает TDD, рабочие деревья
Git или конкретный порядок ревью реализации. Это не отменяет обязательный вызов
scout для адресного исследования кода и правила проверки планирования из Extension.

**Проблемы и следующий шаг:** Критических проблем не обнаружено при повторной
проверке. Реализация полностью соответствует `proposal.md`, `design.md` и
`tasks.md`; все анкеры повторно сверены и актуальны на состоянии checkout,
синхронизированном с `origin/master` в этой сессии. Следующий шаг — человеческое решение об Archive
этого Change через Store PR (вне рамок данной Verify-сессии); Archive, Release
и push не выполнялись в рамках этой проверки.
