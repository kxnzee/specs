# Cross-system invariants

Фиксируйте только условия, которые должны сохраняться сразу между несколькими
системами или определяют общую границу ответственности.

| Инвариант | Область | Основание |
|---|---|---|
| `jitsi-control` координирует конференцию, но не обрабатывает медиапотоки | jitsi-control ↔ jitsi-videobridge | `../src/jitsi-control/README.md:15-17` |
| Управление конференцией на медиамосте должно использовать согласованный обеими сторонами контракт | jitsi-control ↔ jitsi-videobridge | `../src/jitsi-control/README.md:19-20`, `../src/jitsi-videobridge/doc/rest-colibri2.md:9-40` |
| Мосты разных версий не смешиваются в одном relay | jitsi-videobridge, multi-bridge Conference | `../src/jitsi-videobridge/doc/relay.md:29-53` |
| При включённой аутентификации клиент и управляющая сторона используют согласованную модель доступа | jitsi-web ↔ jitsi-control | `../src/jitsi-web/config.js:1677-1697`, `../src/jitsi-control/jicofo/src/main/kotlin/org/jitsi/jicofo/auth/AuthConfig.kt:80-84` |
| Создание и сопровождение комнаты конференции требует согласованных прав между управляющей и сигнальной системами | jitsi-control ↔ xmpp-signaling | `../src/jitsi-control/README.md:52-66` |

<!-- TODO
question: Какие дополнительные наблюдаемые условия должны сохраняться сразу в нескольких системах?
owner: unassigned
expected_source: Contracts, maintained requirements, tests or accepted ADRs
-->
