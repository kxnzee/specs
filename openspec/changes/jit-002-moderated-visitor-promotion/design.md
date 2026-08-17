## Context

Proposal (`proposal.md`) left two open questions unresolved: (1) is the existing
messaging channel enough to reach a visitor who never requested promotion, or is
`VisitorsManager`/`VisitorsIq` in `jitsi-control` required; (2) does the client-side
promote transition need an internal reconnect like demote, or can it be in-place.
Both are resolved below with code evidence gathered specifically for this Design.

Confirmed current behavior (existing, working, self-promotion via
raise-hand → approve):
- A visitor sends `promotion-request`; the prosody "visitors" component —
  `resources/prosody-plugins/mod_visitors_component.lua` (shipped inside the
  `jitsi-web` repository, not `jitsi-control`) — records the request and, on
  moderator approval, runs `process_promotion_response(room, id, approved)`
  (`mod_visitors_component.lua:377-420`), which updates a promotion-allowed record
  for that visitor and sends back a `promotion-response` stanza.
- The visitor's own client reconnects through a **generic, already-existing**
  mechanism: the XMPP connection layer fires `CONNECTION_REDIRECTED`
  (`react/features/base/connection/actions.any.ts:238-240,316-328`, comment at
  `:321`: "This is after promotion from visitor to main participant"), which
  dispatches `redirect()` (`react/features/base/conference/actions.any.ts:1082-1150`):
  `disconnect(true)` → `overwriteConfig`/`setIAmVisitor(false)` →
  `destroyLocalTracks()` → `connect()`. This is the same reconnect-cycle shape
  already used by demote (`visitors/actions.ts:96-103`,
  `visitors/middleware.ts:97-117`: `disconnect(true)` → `setPreferVisitor(true)` →
  `connect()`).
- `preferVisitor` is a connect-time-only option in both `lib-jitsi-meet`
  (`modules/xmpp/moderator.js:195-198`) and `jitsi-web`
  (`react/features/base/connection/actions.any.ts:116-160,222`) — there is no
  mid-session, in-place API to become send-capable in either the client or the
  external `lib-jitsi-meet` dependency. The transition always goes through a
  disconnect/connect cycle.
- `jitsi-control` (jicofo) has no working code on this approve→redirect path
  today. Its only visitor-promotion-shaped code is
  `VisitorsManager.processStanza` (`VisitorsManager.kt:100-111`), which only logs
  and ends in `// TODO pass to the conference` — confirmed not on the critical
  path of the flow described above. jicofo's actual visitor-related role is
  limited to (a) the initial, synchronous join-time decision to redirect a
  connecting client to a visitor node (`ConferenceIqHandler.kt:180-201`,
  `JitsiMeetConferenceImpl.java:1979-2043`), the opposite direction from
  promotion, and (b) MUC presence role bookkeeping that explicitly rejects a
  `VISITOR` ↔ non-`VISITOR` transition outside the first presence
  (`ChatRoomMemberImpl.kt:196-204`, logs `"... - not supported!"`).

This refines Proposal's preliminary Repository impact guess for `jitsi-control`:
the Proposal named `VisitorsManager.processStanza` as the "probable entry point"
for routing a moderator's message to a visitor; evidence now shows the
functionally equivalent, already-working approve-flow does not use it at all.

## Goals / Non-Goals

**Goals:**
- Let a moderator trigger, for a visitor who has not sent a promotion-request,
  the same effective promoted state that approval already produces today, and
  let the existing generic reconnect mechanism carry the visitor into the
  conversation.
- Keep the existing request/approve and demote flows unchanged.

**Non-Goals:**
- Building a new client-side reconnect mechanism — the existing
  `CONNECTION_REDIRECTED` → `redirect()` path is reused as-is.
- Changing jicofo's MUC presence role-transition handling — it already
  (correctly, for its current purpose) rejects direct presence-based
  transitions; this Change does not rely on presence to carry the transition.
- Bulk/multi-select promotion (explicitly out of MVP per Proposal).

## Decisions

### Decision 1: Reuse the existing `redirect()` / `CONNECTION_REDIRECTED` reconnect cycle client-side — no new reconnect logic
Evidence: `base/connection/actions.any.ts:238-240,316-328`,
`base/conference/actions.any.ts:1082-1150`, `moderator.js:195-198`.
Rationale: this is the exact mechanism already in production for the
approve-flow and structurally identical to demote's mechanism; it already
satisfies the Spec's "no visible rejoin" requirement
(`SC-VISITOR-PROMOTION-005`) because that guarantee is already proven for this
code path.
Alternative considered: an in-place, no-reconnect media upgrade. Rejected — no
evidence such an API exists in `lib-jitsi-meet` (`preferVisitor` is
connect-time-only); higher implementation risk for no observable product
benefit, since reconnect is already invisible to the user.
This resolves Proposal's open question about in-place vs. reconnect: reconnect,
confirmed as the only mechanism that exists.

### Decision 2: Implement the moderator-initiated trigger by extending the prosody "visitors" component (in the `jitsi-web` repository), not `jitsi-control`
Evidence: `mod_visitors_component.lua:377-420` (`process_promotion_response`),
`mod_visitors_component.lua:625` (action dispatch), `VisitorsManager.kt:100-111`
(TODO, not on critical path).
Rationale: mirrors the already-working approve-flow exactly and keeps a single
authority for "who is allowed to become send-capable," avoiding a second,
parallel mechanism in `jitsi-control` that would need to stay in sync with the
prosody component's promotion state.
Alternative considered (Proposal's initial guess): complete jicofo's
`VisitorsManager.processStanza` TODO as the entry point. Rejected as the
primary path — evidence shows the equivalent, working flow bypasses it
entirely; building a second promotion authority there risks divergence between
jicofo's and prosody's view of who is promoted.

### Decision 3: Add a new, symmetric moderator-initiated action to `mod_visitors_component.lua`, addressed directly at the target visitor by id
Evidence: `mod_visitors_component.lua:625` currently accepts only
`"promotion-response"` and `"demote-request"`.
Rationale: `process_promotion_response(room, id, approved)` assumes a pending
request record for `id`; the new action must not require a request to have
existed, since Proposal requires the two entry paths to "coexist
independently" (`proposal.md` — request/approve unaffected by the new action).
Alternative considered: synthesize a fake pending-request entry so the
existing `process_promotion_response` code path can be reused unchanged.
Rejected — leaks an internal modeling detail (a request that was never sent)
into what must behave as an independent action.

## Repository implementation map

| repository-id | Часть решения | Изменяемые границы | Входящие контракты | Исходящие контракты | Зависимости | Порядок |
|---|---|---|---|---|---|---|
| `jitsi-web` | (a) Действие "включить в разговор" на `CurrentVisitorsList`; (b) новый action в `visitors/actions.ts`, симметричный `demoteRequest`, отправляющий moderator-initiated promote-сообщение через существующий канал; (c) новый action-тип в `mod_visitors_component.lua`, симметричный `process_promotion_response`, без требования предшествующего request; (d) на стороне промотируемого зрителя — без изменений: существующий `CONNECTION_REDIRECTED` → `redirect()` уже обрабатывает переход. | `react/features/participants-pane/components/web/CurrentVisitorsList.tsx`, `react/features/visitors/actions.ts`, `resources/prosody-plugins/mod_visitors_component.lua` | Клик модератора в UI | XMPP/IQ-сообщение к prosody-компоненту visitors; `promotion-response`-подобный ответ промотируемому зрителю | — | 1 |
| `jitsi-control` | Изменений кода не предполагается. Подтверждено, что `isAllowedInMainRoom`/`isPreferredInMainRoom` (`ChatRoomImpl.kt:222-240`) читают state store (`participants`/`moderators` из `room_metadata`-компонента), не связанный в коде с `visitors_promotion_map`, который пишет approve-flow — то есть jicofo, вероятно, вообще не участвует в admission для этого потока. Требуется verification-задача: эмпирически подтвердить это и убедиться, что moderator-initiated reconnect проходит тем же путём, что и уже работающий approve-flow. | нет | — | — | `jitsi-web` (Decision 2/3) | 2 — verification only |
| `jitsi-videobridge` | Без изменений — подтверждено дважды независимо (Proposal и повторная проверка); кода, специфичного для visitor promote/demote, не найдено. | — | — | — | — | — |

Это уточняет Repository impact из Proposal: тип влияния `jitsi-control` фактически
сводится к verification, а не implementation основного пути; `jitsi-web` несёт
как клиентскую, так и prosody-компонентную часть решения, поскольку
`mod_visitors_component.lua` физически размещён в этом репозитории.

## Risks / Trade-offs

- [Risk] Подтверждено: `visitors_promotion_map` (пишет `mod_visitors_component.lua`
  при approve) и `participants`/`moderators` (читает jicofo через
  `isAllowedInMainRoom`, наполняется отдельным `mod_room_metadata_component.lua`)
  — раздельные state store без обнаруженной синхронизации между ними →
  Mitigation: verification-задача в Tasks для `jitsi-control`/`jitsi-web` —
  эмпирически проследить reconnect уже работающего approve-flow (какой
  stanza/IQ реально допускает зрителя в main room) и воспроизвести тот же
  путь для moderator-initiated promote, а не полагаться на предположение об
  участии jicofo `isAllowedInMainRoom` в admission.
- [Risk] Race между самостоятельным `promotion-request` зрителя и
  одновременным moderator-initiated promote того же зрителя → Mitigation:
  новый обработчик в `mod_visitors_component.lua` должен использовать общее
  состояние допуска (не создавать второй, независимый источник истины) и вести
  себя идемпотентно.
- [Risk] `mod_visitors_component.lua` — Lua/prosody-код вне типичного
  JS/TS test coverage `jitsi-web` → Mitigation: задача в Tasks на расширение
  существующих Lua-тестов (если есть) или явный ручной QA-сценарий.

## Open Questions

- Подтверждено (отдельной независимой проверкой): join-time проверка jicofo
  (`ConferenceIqHandler.kt:167-168,180`, `ChatRoomImpl.kt:212,220,222-240,378-392`)
  читает `participants`/`moderators` из state store, который наполняется
  `RoomMetadataHandler` (`RoomMetadataHandler.kt:50-71,86-96`) из отдельного
  prosody-модуля `mod_room_metadata_component.lua`
  (`mod_room_metadata_component.lua:103-116,183-196`). Approve-flow при этом
  пишет совершенно в другое состояние — `visitors_promotion_map`/
  `metadata.visitors.promoted` в `mod_visitors_component.lua:511,529-537`. Кода,
  синхронизирующего эти два state store, не найдено ни в `jitsi-control`, ни в
  прочитанной части `jitsi-web`.
  Рабочая гипотеза: реальный допуск reconnect для уже одобренного зрителя
  сегодня обеспечивается не проверкой jicofo `isAllowedInMainRoom`, а MUC room
  password, который `mod_visitors_component.lua` формирует на основе
  `visitors_promotion_map` при join (`mod_visitors_component.lua:505-509`), то
  есть jicofo может вообще не быть участником admission-решения для этого
  потока. Это не опровергает и не меняет Decision 2/3 (мы по-прежнему
  зеркалируем именно тот механизм, который уже производит рабочий reconnect
  сегодня, каким бы он ни был) — но означает, что Repository impact для
  `jitsi-control` в Repository implementation map может оказаться не
  "verification-only", а буквально "no-change" после подтверждения.
  Не разрешимо дальнейшим чтением исходников в рамках Design — требует
  эмпирической трассировки reconnect-запроса зрителя (`disconnect()`→
  `connect()`) в Tasks: зафиксировать, какой именно stanza/IQ реально
  предъявляется при повторном join (MUC presence с паролем vs. новый
  `ConferenceIq` через `ConferenceIqHandler`), и подтвердить или опровергнуть
  участие jicofo в admission для нового moderator-initiated пути тем же
  способом.
