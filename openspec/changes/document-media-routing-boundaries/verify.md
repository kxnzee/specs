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
- **Среда и данные:** ветка `master` репозиториев `jicofo` и
  `jitsi-videobridge`; тестовые данные и запуск приложения не нужны — изменение
  документационное.

## Сценарии ручной проверки

| Сценарий | Действия проверяющего | Ожидаемый результат | Подтверждение агента | Решение человека |
| --- | --- | --- | --- | --- |
| Acceptance case: выбор моста в Jicofo | Открыть [документ Jicofo](https://github.com/kxnzee/jicofo/blob/master/doc/bridge-control-boundary.md) и пройти по ссылкам на `BridgeSelector`, `ColibriV2SessionManager` и `Colibri2Session`. | Понятно, какой код выбирает мост и отправляет create/modify-запрос выбранному JVB. | `PASS` — анкеры сверены с кодом на `master`. | `PASS` |
| Acceptance case: приём запроса в JVB | Открыть [документ JVB](https://github.com/kxnzee/jitsi-videobridge/blob/master/doc/bridge-control-boundary.md) и пройти путь `Conference` → `ColibriQueue` → `Colibri2ConferenceHandler`. | Понятно, где мост принимает и обрабатывает управляющий запрос от Jicofo. | `PASS` — анкеры сверены с кодом на `master`. | `PASS` |
| Acceptance case: граница читается целиком | Перейти по взаимным ссылкам между двумя документами. | Каждый документ объясняет свою сторону границы, а вместе они дают полный путь без повторной трассировки кода. | `PASS` — обе кросс-ссылки проверены. | `PASS` |
| Acceptance case: протокол не дублируется | Сопоставить новые документы с [conference request](https://github.com/kxnzee/jicofo/blob/master/doc/conference-request.md) и [REST Colibri2](https://github.com/kxnzee/jitsi-videobridge/blob/master/doc/rest-colibri2.md). | Новые документы ссылаются на протокольные описания, но не копируют wire-format. | `PASS` — дублирования не найдено. | `PASS` |

## Автоматические проверки

| Проверка | Результат | Evidence |
| --- | --- | --- |
| Валидация OpenSpec (strict) | `PASS` | Change проходит строгую валидацию без ошибок. |
| Состояние задач | `PASS` | [tasks.md](./tasks.md) — 5 из 5 задач завершены. |
| Опубликованный кандидат | `PASS` | [jicofo PR #2](https://github.com/kxnzee/jicofo/pull/2) и [jitsi-videobridge PR #1](https://github.com/kxnzee/jitsi-videobridge/pull/1) смёржены; версии зафиксированы в карте реализации. |
| Границы изменения | `PASS` | В обоих PR добавлен только `doc/bridge-control-boundary.md`; `jitsi-meet` не менялся. |
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
  независимо восстановить только по репозиторию: формальное GitHub Review не
  зафиксировано, а журнал вызовов обязательных Superpowers skills отсутствует.
- **Evidence:** оба PR смёржены ([jicofo #2](https://github.com/kxnzee/jicofo/pull/2),
  [jitsi-videobridge #1](https://github.com/kxnzee/jitsi-videobridge/pull/1)),
  но их review не оформлен как формальный GitHub Review; в
  [implementation-map.yaml](./implementation-map.yaml) нет записи о вызове
  обязательных Superpowers skills при реализации.

## Открытые вопросы

- Для этого кандидата блокеров нет. В следующих изменениях стоит фиксировать
  ручное ревью в PR и сохранять ссылку на использованный Apply workflow.
