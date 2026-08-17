# Domain model

## Сущности и отношения

- **Conference** объединяет участников и связанные с ними сигнальные и медиасессии.
- **Participant** присоединяется к Conference и может отправлять или получать медиа.
- **Visitor** — ограниченный Participant, который получает медиа без отправки
  собственного аудио и видео.
- **Bridge** обслуживает медиапотоки участников Conference; одна Conference может
  использовать несколько совместимых Bridge.
- **External application** встраивает Conference через публичный External API.

Источники: `../src/jitsi-control/README.md:13-20`,
`../src/jitsi-videobridge/README.md:3-4`,
`../src/jitsi-web/modules/API/external/index.js:1-3`.

## Состояния и переходы

- Участник может присоединиться к конференции, участвовать в ней и покинуть её.
- Конференция может получить медиамост, расшириться на дополнительные совместимые
  мосты и завершиться.
- Недоступный мост не должен использоваться для новых подключений участников.

<!-- TODO
question: Какие состояния Conference и Participant являются нормативными для продукта и какие переходы должны быть отражены в Requirements и Scenarios?
owner: unassigned
expected_source: Maintained requirements or product documentation
-->

## Жизненные циклы

<!-- TODO
question: Какой подтверждённый жизненный цикл конференции должен сохраняться от создания до полного завершения на всех системах?
owner: unassigned
expected_source: Maintained requirements or verified cross-system behavior
-->
