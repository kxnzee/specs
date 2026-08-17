# Domain glossary

Включены только термины, необходимые для одинакового понимания продукта и
межсистемных границ. Детали реализации принадлежат Code Repositories.

| Термин | Подтверждённое определение | Источник |
|---|---|---|
| Conference | Видеовстреча, объединяющая участников, сигнализацию и медиапотоки | `../src/jitsi-control/README.md:13-20` |
| Participant / Endpoint | Участник конференции и его подключение к конференции | `../src/jitsi-control/README.md:13-15` |
| Focus | Роль `jitsi-control`: координирует конференцию и соединения участников | `../src/jitsi-control/README.md:13-17` |
| JVB | `jitsi-videobridge`: компонент, пересылающий медиапотоки между участниками | `../src/jitsi-videobridge/README.md:3-4` |
| MUC | Комната конференции в XMPP, объединяющая участников и управляющие компоненты | `../src/jitsi-control/README.md:52-66` |
| Visitor | Участник, который принимает медиапотоки, но не отправляет собственные | `../src/jitsi-control/jicofo/src/main/resources/reference.conf:180-222` |
| External API | Публичный контракт для встраивания Jitsi Meet в сторонний продукт | `../src/jitsi-web/modules/API/external/index.js:1-3` |
| Relay / Octo | Межмостовое соединение для конференций, распределённых между несколькими JVB | `../src/jitsi-videobridge/doc/relay.md:4-53` |
