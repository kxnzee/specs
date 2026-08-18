## 1. jitsi-control: verify admission path before building on top of it

This section runs first even though Design's Repository implementation map
lists `jitsi-web` as the primary solution owner (order 1): the evidence this
section gathers determines whether task 1.3 is needed at all, so it must be
resolved before other work depends on an unverified assumption about jicofo's
role in admission.

- [x] 1.1 Trace the actual reconnect exchange for an existing, working approve-flow promotion (raise hand → approve → visitor reconnects as participant): capture whether the reconnect uses a MUC presence + room password (`mod_visitors_component.lua:505-509`) or a `ConferenceIq` handled by `ConferenceIqHandler`, and record which one actually admits the visitor into the main room. (Design — Open Questions, Risk 1)
- [x] 1.2 From 1.1, confirm whether jicofo's `isAllowedInMainRoom`/`isPreferredInMainRoom` (`ChatRoomImpl.kt:222-240`) participates in admission for visitor promotion at all, or whether admission is fully handled at the prosody/MUC level. Record the finding as evidence for Design's Open Question. (Design — Open Questions)
- [x] 1.3 Corrected by 1.1/1.2 evidence: the relevant gate is `redirectVisitor()` (`JitsiMeetConferenceImpl.java:1979-2043`), not `isAllowedInMainRoom`/`isPreferredInMainRoom`. Implemented the minimal jicofo-side change: removed `visitorsAlreadyUsed` from `redirectVisitor`'s forced-redirect condition, so a reconnecting client that isn't itself requesting visitor mode and isn't over the participants soft limit is admitted to the main room instead of being "stuck" in visitor mode just because the conference has used visitors before — this covers a moderator-promoted visitor's reconnect the same way it already needs to cover an approve-flow-promoted visitor's reconnect. (Design — Repository implementation map, Decision 2)

## 2. jitsi-web: prosody "visitors" component — moderator-initiated promotion

- [ ] 2.1 Add a new moderator-initiated action type to the stanza dispatcher in `mod_visitors_component.lua` (currently accepts only `"promotion-response"` and `"demote-request"`, `mod_visitors_component.lua:625`), addressed directly at a target visitor id, without requiring a pending promotion-request record. (Design — Decision 3; Spec `SC-VISITOR-PROMOTION-001`)
- [ ] 2.2 Implement the handler for the new action so it writes into the same promotion-allowed state (`visitors_promotion_map` / `metadata.visitors.promoted`, `mod_visitors_component.lua:511,529-537`) that `process_promotion_response` already writes on approve, and sends the visitor the same kind of response that already triggers `CONNECTION_REDIRECTED` on the client. (Design — Decision 2/3)
- [ ] 2.3 Make the handler idempotent against a concurrent self-initiated `promotion-request` for the same visitor id, so the two entry paths cannot leave conflicting state. (Design — Risks, race condition)
- [ ] 2.4 Verify (or, if missing, add) a server-side check that the sender of the new moderator-initiated action actually holds moderator affiliation in the room, mirroring however the existing dispatcher already authorizes `"promotion-response"`/`"demote-request"` — do not rely on UI-level hiding (task 3.1) as the only access control. (Spec `SC-VISITOR-PROMOTION-004`)

## 3. jitsi-web: moderator-facing UI and action

- [ ] 3.1 Add an "include in conversation" action to each item of `CurrentVisitorsList`, visible only to participants holding the moderator role that already manages mute/kick/demote. (Spec `SC-VISITOR-PROMOTION-001`, `SC-VISITOR-PROMOTION-004`)
- [ ] 3.2 Add a new action creator in `react/features/visitors/actions.ts`, symmetric to `demoteRequest(id)`, that sends the new moderator-initiated promote message from task 2.1 through the existing visitors messaging channel. (Design — Decision 3)
- [ ] 3.3 Wire the UI action from 3.1 to dispatch the new action creator from 3.2.

## 4. jitsi-web: promoted-visitor client behavior (verification of reused mechanism)

- [ ] 4.1 Verify the existing `CONNECTION_REDIRECTED` → `redirect()` cycle (`base/connection/actions.any.ts:238-240,316-328`, `base/conference/actions.any.ts:1082-1150`) correctly fires for a visitor promoted through the new moderator-initiated message from task 2.2, not only for the existing approve-flow. (Design — Decision 1)
- [ ] 4.2 Verify the promoted participant's microphone starts muted and no audio is sent before the participant unmutes themselves. (Spec `SC-VISITOR-PROMOTION-002`)
- [ ] 4.3 Verify the promoted participant's video remains disabled after promotion. No fallback task is defined here (unlike 4.5 for audio): promotion never requests video capability in the first place, so there is no default-enabled state the reused mechanism could produce that would need correcting. (Spec `SC-VISITOR-PROMOTION-003`)
- [ ] 4.4 Verify the promoted visitor sees no rejoin screen, "you left" message, or loss of meeting state during the reconnect cycle. (Spec `SC-VISITOR-PROMOTION-005`)
- [ ] 4.5 If 4.2 shows the microphone is not muted by default through the reused reconnect path, add explicit mute-on-promote handling in the promotion flow.

## 5. jitsi-web: regression checks

All three scenarios below live entirely within `jitsi-web` (client code plus
`mod_visitors_component.lua`, both part of that repository per Design's
Context); `jitsi-control` has no task in this section.

- [ ] 5.1 Verify the existing `demoteRequest(id)` reverts a visitor promoted through the new moderator-initiated path back to visitor state, with no dedicated revert mechanism added. (Spec `SC-VISITOR-PROMOTION-007`)
- [ ] 5.2 Verify a moderator-promoted, inactive participant is not automatically reverted to visitor state by any existing timer or inactivity mechanism. (Spec `SC-VISITOR-PROMOTION-008`)
- [ ] 5.3 Verify the existing raise-hand → `promotion-request` → approve/deny flow behaves unchanged with the new action present, and that neither flow is a precondition for the other. (Spec `SC-VISITOR-PROMOTION-006`)

## 6. Tests and evidence preparation

- [ ] 6.1 Add a unit test for the new action creator in `react/features/visitors/actions.ts` (task 3.2) using the project's existing JS/TS test setup, covering the message payload sent for a moderator-initiated promote. (Spec `SC-VISITOR-PROMOTION-001`)
- [ ] 6.2 Define a manual QA script covering `SC-VISITOR-PROMOTION-001` through `SC-VISITOR-PROMOTION-008`, since `mod_visitors_component.lua` has no existing automated test framework in this repository (no `.busted`/rockspec setup found) — this closes Design's Lua test-coverage risk without inventing new test infrastructure.
- [ ] 6.3 Assemble PR Review evidence for `jitsi-web` (diff plus references to Design Decisions 1-3 and the relevant Requirement/Scenario IDs) and for `jitsi-control` (task 1.1/1.2 findings, plus the 1.3 change if any was needed).
- [ ] 6.4 Assemble IFT/QA evidence package from the manual QA script results (task 6.2) and the cross-repository regression checks (section 5); do not mark external PR Review, IFT, or QA as complete in this Change — only prepare the evidence for those to review.
