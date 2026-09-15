# Приёмка изменения

**Изменение:** `document-media-routing-boundaries`

<!-- SCENARIO_VERIFICATION_CONTRACT_V1_START -->
## Краткий вывод агента

- **Готовность к ручной проверке:** `READY`
- **Проверено автоматически:** оба документа опубликованы, code-anchors
  соответствуют текущему коду, кросс-ссылки и границы репозиториев подтверждены.
- **Проверить человеку:** даёт ли пара документов однозначный ответ, где Jicofo
  выбирает мост и где JVB принимает управляющий запрос.
- **Риски:** file:line-ссылки могут устареть после будущего рефакторинга.

## Объект проверки

- **Требования:** Acceptance cases из раздела
  [Constraints and success criteria](./proposal.md#constraints-and-success-criteria);
  Delta Specs не создавались, потому что поведение системы не меняется.
- **Кандидат:** [implementation-map.yaml](./implementation-map.yaml)
- **Среда и данные:** checkout кандидата `jitsi-control` и `jitsi-videobridge`,
  подготовленные через Change Tracking; тестовые данные и запуск приложения не
  нужны — изменение документационное.

## Сценарии ручной проверки

| Сценарий | Действия проверяющего | Ожидаемый результат | Подтверждение агента | Решение человека |
| --- | --- | --- | --- | --- |
| Acceptance case: выбор моста в Jicofo | Открыть `jitsi-control:doc/bridge-control-boundary.md` и пройти по ссылкам на `BridgeSelector`, `ColibriV2SessionManager` и `Colibri2Session`. | Понятно, какой код выбирает мост и отправляет create/modify-запрос выбранному JVB. | `PASS` — анкеры сверены с кодом кандидата. | `PASS` |
| Acceptance case: приём запроса в JVB | Открыть `jitsi-videobridge:doc/bridge-control-boundary.md` и пройти путь `Conference` → `ColibriQueue` → `Colibri2ConferenceHandler`. | Понятно, где мост принимает и обрабатывает управляющий запрос от Jicofo. | `PASS` — анкеры сверены с кодом кандидата. | `PASS` |
| Acceptance case: граница читается целиком | Перейти по взаимным ссылкам между двумя документами. | Каждый документ объясняет свою сторону границы, а вместе они дают полный путь без повторной трассировки кода. | `PASS` — обе кросс-ссылки проверены. | `PASS` |
| Acceptance case: протокол не дублируется | Сопоставить новые документы с `jitsi-control:doc/conference-request.md` и `jitsi-videobridge:doc/rest-colibri2.md`. | Новые документы ссылаются на протокольные описания, но не копируют wire-format. | `PASS` — дублирования не найдено. | `PASS` |

## Автоматические проверки

| Проверка | Результат | Evidence |
| --- | --- | --- |
| Валидация OpenSpec (strict) | `PASS` | Change проходит строгую валидацию без ошибок. |
| Состояние задач | `PASS` | [tasks.md](./tasks.md) — 5 из 5 задач завершены. |
| Кандидат в checkout | `PASS` | Change Tracking подготовил candidate checkout; оба документа присутствуют по указанным путям. |
| Границы изменения | `PASS` | В checkout кандидата добавлены только два документа по указанным путям; `jitsi-meet` не менялся. |
<!-- SCENARIO_VERIFICATION_CONTRACT_V1_END -->

<!-- FEATURE_ACCEPTANCE_CONTRACT_V1_START -->
## Решение о приёмке

- **Решение:** `PASS`
- **Комментарий:** сохранено явное решение пользователя по этому же кандидату;
  все применимые сценарии подтверждены.
<!-- FEATURE_ACCEPTANCE_CONTRACT_V1_END -->

## Соблюдение процесса Superspec

- **Результат:** `PASS_WITH_WARNINGS`
- **Кратко:** приёмка проходит, но два процессных подтверждения нельзя
  независимо восстановить только по Store: решение ревьюера не зафиксировано,
  а журнал вызовов обязательных Superpowers skills отсутствует.
- **Evidence:** candidate checkout подготовлен, но решение ревьюера не записано
  в Store; в
  [implementation-map.yaml](./implementation-map.yaml) также нет записи о
  вызове обязательных Superpowers skills при реализации.

## Открытые вопросы

- Для этого кандидата блокеров нет. В следующих изменениях стоит фиксировать
  решение ревьюера и использованный Apply workflow в Store.
