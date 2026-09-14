---
owner: unassigned
updated: 2026-09-14
---
# Границы продукта и окружение

## Граница продукта

В продукт входят пользовательский клиент, клиентская библиотека сигнализации и
медиа, управление конференциями и пересылка медиапотоков. XMPP-инфраструктура,
запись, стриминг и SIP-интеграции взаимодействуют с продуктом как отдельные стороны.

Точный реестр Code Repositories хранится только в `openspec-orch.yaml`; этот раздел
описывает ответственность компонентов и не дублирует реестр.

## Ответственность сторон

| Сторона | За какой бизнес-результат отвечает |
|---|---|
| `jitsi-web` | Пользовательское взаимодействие с конференцией и встраивание видеосвязи в сторонние продукты |
| `lib-jitsi-meet` | Клиентская сигнализация и управление медиасессией, используемые `jitsi-web` |
| `jitsi-control` | Координация конференции, участников и выбор медиамостов; не обрабатывает медиапотоки |
| `jitsi-videobridge` | Приём и пересылка медиапотоков участников, управление качеством пересылаемого медиа, поддержка конференций на нескольких медиамостах |
| `xmpp-signaling` | Комнаты конференций и сигнальное взаимодействие клиентов и управляющих компонентов; внешняя система, не зарегистрированная как отдельный Code Repository этого Store |

Источники: `../src/jitsi-web/README.md:1-4`,
`../src/jitsi-web/react/features/base/lib-jitsi-meet/_.native.ts:1-7`,
`../src/lib-jitsi-meet/JitsiConnection.ts:98-122`,
`../src/jitsi-control/README.md:13-20`, `../src/jitsi-videobridge/README.md:3-4`,
`../src/jitsi-control/README.md:52-66`.

## Обмен информацией

| Отправитель | Получатель | Бизнес-информация |
|---|---|---|
| `jitsi-web` | `lib-jitsi-meet` | Пользовательские действия и настройки подключения к конференции |
| `lib-jitsi-meet` | `xmpp-signaling` | Подключение клиента и запрос конференции |
| `jitsi-control` | `xmpp-signaling` | Координация конференции и доступных компонентов |
| `lib-jitsi-meet` через XMPP | `jitsi-control` | Запрос на создание конференции или подключение к ней |
| `jitsi-control` | `jitsi-videobridge` | Создание и изменение конференции на выбранном медиамосте |
| `lib-jitsi-meet` | `jitsi-videobridge` | Передача и приём клиентских медиапотоков через выбранный медиамост |

Источники: `../src/jitsi-web/react/features/base/lib-jitsi-meet/_.native.ts:1-7`,
`../src/lib-jitsi-meet/JitsiConnection.ts:98-122`,
`../src/lib-jitsi-meet/modules/xmpp/moderator.js:295-325`,
`../src/jitsi-control/README.md:52-66`, `../src/jitsi-control/doc/conference-request.md:13-52`,
`../src/jitsi-control/README.md:19-20`, `../src/jitsi-videobridge/doc/rest-colibri2.md:9-40`,
`../src/jitsi-videobridge/doc/web-sockets.md:6-19`.

Действующие межсистемные условия и инварианты — в [ограничениях](05-constraints.md).
При необходимости добавьте обзорную схему здесь. Цели участников — в
[назначении](01-product-context.md), последовательность действий — в
[процессах](04-business-processes.md), причины значимых решений — в [ADR](ADR/README.md).
