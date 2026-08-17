# Quality gates

Gate — явное решение людей по проверяемому evidence. Skill или subagent может
подготовить review, но не принимает Gate автоматически.

## Gate 1 — Planning accepted

| Evidence | Минимальное условие |
|---|---|
| Jira и Change | Jira Story связана с `change-id` |
| OpenSpec | Proposal, Delta Specs, Design и Tasks согласованы и валидны |
| Scope | Capability, системы и Code Repositories определены |
| Repository impact | По каждому `repository-id` определены impact, Design scope, Tasks и evidence |
| Verification | Каждый новый или изменённый Scenario имеет план проверки |
| Решения | Нет вопроса, меняющего Specs, Design или Tasks |

## Gate 2 — Implementation candidate

| Evidence | Минимальное условие |
|---|---|
| PR | Требуемые реализации прошли review и связаны с Change |
| Repository checks | В каждом затронутом Code Repository выполнены его локальные обязательные проверки |
| Snapshot | Зафиксированы точные commits всех затронутых репозиториев |
| Artifact | Определён поставляемый результат |
| Deviations | Отклонения от OpenSpec отсутствуют или явно разрешены |

Локальные команды build/test/lint, CI-конфигурация и технические критерии хранятся в
соответствующих Code Repositories и не дублируются в Store.

## Gate 3 — Release ready

| Evidence | Минимальное условие |
|---|---|
| IFT | Выполнен на Snapshot Gate 2 |
| QA | OpenSpec Scenarios проверены на том же Snapshot |
| Defects | Нет блокирующих дефектов |
| Release | Подтверждены rollout, наблюдение и rollback |

## Инвалидация

- Новая planning revision или новый состав репозиториев требует повторного Gate 1.
- Новый implementation commit или artifact инвалидирует Gate 2, IFT, QA и Gate 3.
- Изменение наблюдаемого поведения возвращает Change в Planning.

## Критичные сценарии

<!-- TODO
question: Какие продуктовые сценарии обязательно проверяются при затрагивающем их изменении?
owner: unassigned
expected_source: Test strategy, maintained requirements or incidents
-->

## Исключения

Для исключения обязательны причина, владелец решения, scope, срок действия и
компенсирующая проверка. Исключение без владельца или срока блокирует Gate.
