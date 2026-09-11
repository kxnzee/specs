# Приёмка изменения

**Изменение:** `display-conference-focus-region`

<!-- SCENARIO_VERIFICATION_CONTRACT_V1_START -->
## Краткий вывод агента

- **Готовность к ручной проверке:** `BLOCKED`
- **Проверено автоматически:** Backend формирует поле по контракту;
  lib-jitsi-meet разбирает и фиксирует первоначальное значение; web-код проходит
  ESLint, TypeScript и проверку локализации.
- **Проверить человеку:** В запущенной web-конференции подтвердить показ региона и
  отсутствие label при ненастроенном регионе.
- **Риски:** Визуальная проверка исключена из текущего прогона по решению
  пользователя.

## Объект проверки

- **Требования:** `openspec/changes/display-conference-focus-region/specs/conference/region-diagnostics/spec.md`
- **Кандидат:** `openspec/changes/display-conference-focus-region/implementation-map.yaml`.
- **Среда и данные:** Совместимые checkout `jitsi-control`, `lib-jitsi-meet` и
  `jitsi-web`; для положительного кейса задать `jicofo.local-region=eu-west`, для
  отрицательного — убрать настройку или оставить пустой.

## Сценарии ручной проверки

| Сценарий | Действия проверяющего | Ожидаемый результат | Подтверждение агента | Решение человека |
| --- | --- | --- | --- | --- |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-001` | Настроить Jicofo с `jicofo.local-region=eu-west`, создать конференцию и проверить ответ создания. | Ответ содержит `focus-region=eu-west`. | `PASS` — backend-тест положительного ответа. | `PENDING` |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-002` | Убрать регион, затем повторить с пустым значением и создать конференцию в каждом случае. | В обоих ответах нет `focus-region`. | `PASS` — backend-тесты отсутствующего и пустого значения. | `PENDING` |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-003` | С настроенным регионом открыть web-конференцию и посмотреть верхнюю информационную строку. | Показан `Conference region: eu-west` рядом с существующими элементами. | `PENDING` — код и статические проверки готовы, UI не запускался. | `PENDING` |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-004` | Открыть web-конференцию без региона; отдельно передать пустое или некорректное значение клиенту. | Region label отсутствует, сообщение об ошибке не появляется. | `PENDING` — разбор и условие скрытия проверены по коду, UI не запускался. | `PENDING` |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-005` | После создания конференции с регионом сменить обслуживающий focus; повторить для конференции, созданной без региона. | Первоначальное значение или его отсутствие не меняется. | `PASS` — browser-тесты обоих вариантов снимка. | `PENDING` |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-006` | Передать значение вроде `<b>eu-west</b>` и открыть информационную строку. | Локализованная подпись содержит значение буквально; разметка не создаётся, label не интерактивен. | `PASS` — значение передаётся в текстовый prop React Label, строка берётся из i18n. | `PENDING` |
| `SC-CONFERENCE-REGION-DIAGNOSTICS-007` | Открыть тот же сценарий в mobile-клиенте. | Новый region label отсутствует; поведение mobile не изменено. | `PASS` — изменены только web-компоненты и общий интерфейс типа, mobile-компоненты не затронуты. | `PENDING` |

## Автоматические проверки

| Проверка | Результат | Evidence |
| --- | --- | --- |
| `openspec validate display-conference-focus-region --strict` | `PASS` | Change прошёл строгую валидацию. |
| Backend contract tests | `PASS` | `jitsi-control/.../ConferenceIqHandlerTest.kt`: заполненное, отсутствующее и пустое значения. |
| lib-jitsi-meet lint and browser tests | `PASS` | ESLint и TypeScript без ошибок; native и polyfill suites — по 582 успешных теста. |
| jitsi-web static checks | `PASS` | ESLint изменённых файлов, `tsc:web` и JSON parsing без ошибок. |

<!-- SCENARIO_VERIFICATION_CONTRACT_V1_END -->

<!-- FEATURE_ACCEPTANCE_CONTRACT_V1_START -->
## Решение о приёмке

- **Решение:** `PENDING`
- **Комментарий:** Человек ещё не прошёл два UI-состояния в запущенной конференции.

<!-- FEATURE_ACCEPTANCE_CONTRACT_V1_END -->

## Соблюдение процесса Superspec

- **Результат:** `PASS`
- **Кратко:** Требования трассируются к задачам, опубликованным code-кандидатам и
  автоматическим проверкам; ручная приёмка корректно остаётся отдельным шагом.
- **Evidence:** `proposal.md`, `design.md`, `tasks.md`, локальные результаты проверок
  трёх затронутых репозиториев.

## Открытые вопросы

- Пройти ручные UI-кейсы перед решением о приёмке.
