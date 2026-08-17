# Release process

Релиз должен продвигать тот же Snapshot и поставляемый результат, которые прошли
проверку. Замена commit или artifact создаёт нового кандидата и требует повторной
проверки.

## Среды и продвижение

<!-- TODO
question: Какие среды существуют и как подтверждённый Snapshot продвигается между ними?
owner: unassigned
expected_source: Deployment policy, runbooks or maintainer confirmation
-->

## Миграции и управление включением

<!-- TODO
question: Как выполняются миграции и управляется постепенное включение изменений?
owner: unassigned
expected_source: Runbooks, deployment policy or accepted ADRs
-->

## Наблюдение и откат

- До Gate 3 должны быть определены наблюдаемые сигналы успешности и условия отката
  для всего затронутого пользовательского сценария.
- Конкретные метрики, endpoints, команды и процедуры отдельного компонента принадлежат
  соответствующему Code Repository.

<!-- TODO
question: Какие общие сигналы останавливают поставку и кто принимает решение об откате?
owner: unassigned
expected_source: Monitoring policy, runbooks, incidents or maintainer confirmation
-->

## Archive

- Archive разрешён только после завершения всех реализаций и обязательной ручной
  проверки.
- Штатный OpenSpec Archive остаётся владельцем применения Delta Specs к Master Specs
  и перемещения Change.
- При расхождении источником истины остаётся архивная Git revision OpenSpec Store.
