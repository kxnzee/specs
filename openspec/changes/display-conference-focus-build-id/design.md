## Context

Accepted Proposal capability: `conference/focus-build-id-visibility` (new
capability), spanning `jitsi-control`, `lib-jitsi-meet`, `jitsi-web`
(proposal.md - Repository Impact); `jitsi-videobridge` is out of scope. The
accepted Specs (`specs/conference/focus-build-id-visibility/spec.md`) fix
four Requirements: row shown when the value has been provided; row/
placeholder/error absent when not provided; a Focus that omits the value
(including an older/unmodified one) treated the same as "not provided"; and
visibility to all participants, not gated on moderator role.

Intake §9 originally left one Design-blocking open question (the exact
Jicofo build-id source and its public-display suitability, resolved below)
and three deferrable ones. The owner has since decided two of those three
directly during this Design stage — blank-value handling and long-value
presentation (Intake §9 items 2 and 3, updated in place below with that
provenance; see Decision 7). Only the client-version test-compatibility
matrix (Intake §9 item 4) remains deferred to Tasks.

The sibling `display-conference-focus-version` change's Design proposed the
same three-repository transport and is used here only as a lead, not as
authority — it was itself corrected once after a confirmed Apply-gap, so it
is evidence of a *researched pattern*, not a current-state source. Every
claim below was re-verified directly against the current registered
checkouts for this Change.

**Directly verified in `jitsi-control`
(`jicofo/src/main/java/org/jitsi/jicofo/...`):**

- `version/CurrentVersionImpl.java:91-92`: `NIGHTLY_BUILD_ID` is a
  `public static final String`, parsed at class-load time from the JAR
  manifest's `Implementation-Version` (format `major.minor-buildid`) via
  `CurrentVersionImpl.java:41-64`, falling back to the literal `"build.git"`
  when the manifest doesn't set it (`CurrentVersionImpl.java:92`, e.g.
  non-packaged/dev builds). It is one process-wide value for the running
  Jicofo instance, not conference-specific.
- `conference/JitsiMeetConferenceImpl.java:642-646`: on room join, a
  `ComponentVersionsExtension` presence stanza already advertises
  `CurrentVersionImpl.VERSION.toString()` — which embeds `NIGHTLY_BUILD_ID`
  (`CurrentVersionImpl.java:94-100`) — to every participant in the room
  today, independent of this change. This directly establishes that
  `NIGHTLY_BUILD_ID` is already public to all conference participants via
  existing presence, resolving the public-suitability half of Intake §9 Q1
  without relying on the sibling Design's separate `/about/version`
  HTTP-route claim (not re-verified here, and not needed).
- `JitsiMeetConferenceImpl.java:648-662`: the focus-presence
  `ConferenceProperties` object (`org.jitsi.xmpp.extensions.jitsimeet.
  ConferenceProperties`, imported at line 50 — an external dependency, not
  part of any registered checkout for this Change) is populated by
  `private void setConferenceProperty(String key, String value, boolean
  updatePresence)` calls (e.g. using `ConferenceProperties.
  KEY_SUPPORTS_SESSION_RESTART`, `KEY_VISITORS_ENABLED`) into a
  `Map<String,String>` field, then serialized generically by
  `createConferenceProperties()` (`JitsiMeetConferenceImpl.java:722-727`)
  and attached to `presenceExtensions` (line 662) before the stanza is sent.
  No fixed key allow-list exists on this path. This confirms the transport
  the sibling Design also identified, verified independently here.

**Directly verified in `lib-jitsi-meet`:**

- `modules/xmpp/ChatRoom.ts:1123-1136`: the `conference-properties`
  presence-stanza case (guarded only by `member.isFocus`) reads every
  `<property key=... value=.../>` child generically into a plain object and
  emits `XMPPEvents.CONFERENCE_PROPERTIES_CHANGED` with it — no per-key
  allow-list.
- `JitsiConference.ts:673`: `_updateProperties` (defined at
  `JitsiConference.ts:2126-2184`) is bound to that event and unconditionally
  replaces `this.properties` with the full incoming object; only a handful
  of already-known keys (`audio-limit-reached`, `video-limit-reached`,
  `bridge-count`, `visitor-codecs`, `visitor-count`) get extra side-effect
  handling — an unrecognized key such as `focus-build-id` simply becomes
  readable, with no extra code path required.
- `JitsiConference.ts:4710-4712`: `getProperty(key)` returns
  `this.properties[key]` directly — `undefined` for a key never sent, and
  whatever string was sent (including `""`) otherwise.

**Directly verified in `jitsi-web`
(`react/features/conference/components/...`):**

- `constants.ts:1-15` (`CONFERENCE_INFO`): the current default `autoHide`
  array does **not** contain a `focus-version` or `focus-build-id` id yet —
  current source inspection therefore does not treat the sibling Design as
  already-applied behavior. This Change's `jitsi-web` registration is
  independent and cannot assume a shared id is already present.
- `functions.any.ts:11-22` (`getConferenceInfo`): returns
  `state['features/base/config'].conferenceInfo` overrides when set, else
  falls back to `CONFERENCE_INFO` from `constants.ts`. A component id absent
  from both is never eligible to render.
- `web/ConferenceInfo.tsx:75-134` (`COMPONENTS`) and `174-196`
  (`_renderAutoHide`): renders exactly the ids present in both `COMPONENTS`
  and the effective `autoHide`/`alwaysVisible` lists; the existing
  `visitors-count` entry (`VisitorsCountLabel`) follows a "component renders
  `null` when its underlying value is falsy" idiom, confirmed directly at
  `react/features/visitors/components/web/VisitorsCountLabel.tsx:25-34`
  (`visitorsCount > 0 ? (<Label .../>) : null`).

## Goals / Non-Goals

**Goals:** implement `conference/focus-build-id-visibility` exactly as
scoped in Proposal/Specs, reusing each repository's existing generic
property-relay / conditional-label mechanism; resolve Intake §9 open
question 1 (source and public-suitability) and the empty-value/compatibility
implementation approach so Tasks can proceed without new architectural
decisions.

**Non-Goals:** (carried from Proposal, restated for Design) any new
transport, request/response, storage, logging, or analytics beyond holding
the value as an in-memory conference property; `jitsi-videobridge`; changes
to conference allocation, bridge selection, authentication, or access
control; a custom truncation or bespoke accessibility treatment for long
values, beyond Conference info's existing row presentation (Decision 7); a
client-version compatibility test matrix beyond general project rules
(Intake §9 Q4, deferred to Tasks).

## Decisions

1. **Build-id source (resolves Intake §9 Q1):** use
   `CurrentVersionImpl.NIGHTLY_BUILD_ID` (`CurrentVersionImpl.java:91-92`)
   unchanged, with no reformatting. Alternative considered:
   `CurrentVersionImpl.VERSION.toString()` (the combined
   `major.minor-buildid` form already sent via `ComponentVersionsExtension`)
   — rejected because Proposal/Specs call for "the build identifier"
   specifically, and the combined string would duplicate the
   already-visible `ComponentVersionsExtension` version rather than adding
   the distinct, targeted value support engineers need to match a fix to a
   specific build.

2. **Public-suitability boundary:** `NIGHTLY_BUILD_ID` is already sent to
   every participant in the room today, embedded in `VERSION.toString()`
   inside the existing `ComponentVersionsExtension` presence stanza
   (`JitsiMeetConferenceImpl.java:642-646`). Publishing the same raw value
   under a second key does not add data or widen the technical audience, but
   showing it in Conference info makes the value easier to discover. That
   increased discoverability is the accepted purpose of the all-participants
   UI requirement, not evidence that the value was previously visible there.

3. **Presence property key:** the literal string `"focus-build-id"`,
   matching the wire key already fixed by the accepted Proposal and used
   identically by all four accepted Scenarios. `ConferenceProperties` is
   defined in the external `org.jitsi.xmpp.extensions.jitsimeet` dependency
   (imported at `JitsiMeetConferenceImpl.java:50`), not part of any
   registered Code Repository for this Change, so a new `KEY_*` constant
   cannot be added there. `setConferenceProperty(String key, String value,
   boolean updatePresence)` already accepts an arbitrary `String` key with
   no restriction on the underlying map, so a literal is the only available
   approach here, not merely a style choice.

4. **`jitsi-control` wiring:** add one call —
   `setConferenceProperty("focus-build-id", CurrentVersionImpl.
   NIGHTLY_BUILD_ID, false)` — alongside the existing property calls at
   `JitsiMeetConferenceImpl.java:648-660`, before `createConferenceProperties()`
   (line 662) builds and attaches the presence extension. This includes the
   property from the first presence stanza sent on room join; no separate
   update path is needed because the value does not change during a
   conference's lifetime.

5. **`lib-jitsi-meet` (generic relay, confirms no change needed):** the
   existing generic path — `conference-properties` presence case →
   `CONFERENCE_PROPERTIES_CHANGED` → `_updateProperties` → `this.properties`
   (`ChatRoom.ts:1123-1136`; `JitsiConference.ts:673`, `2126-2184`) — already
   carries any new key, including `focus-build-id`, with no allow-list to
   extend. This Change's `lib-jitsi-meet` scope is limited to one contract
   regression test (planned in Tasks, not created by this Design)
   confirming `getProperty('focus-build-id')` returns the value after a
   presence stanza containing it — mirroring the equivalent test planned
   for `focus-version`.

6. **`jitsi-web` placement and conditionality:** add a new label id
   `focus-build-id` to the `COMPONENTS` array in `ConferenceInfo.tsx`
   (alongside `visitors-count`, `insecure-room`, etc.) and to the default
   `autoHide` array in `constants.ts` (`CONFERENCE_INFO.autoHide`) —
   `autoHide` is correct because the row must disappear by default when the
   value is absent, like other optional/informational rows, not persist
   unconditionally like `alwaysVisible` entries. This registration is
   independent of the sibling `focus-version` change's `jitsi-web` work
   landing first, or at all (see Context correction) — the two ids are
   separate entries in the same existing arrays.

7. **Blank-value handling and unchanged display (Owner decision, recorded at
   Design; resolves Intake §9 items 2 and 3):** the owner decided — if
   `focus-build-id` is absent, empty, or whitespace-only, the row is not
   shown; for any other string, the row shows that value unchanged; long
   values use Conference info's existing row presentation behavior, with no
   custom truncation. This is a normative accepted behavior, not an inferred
   implementation reading: it refines the first two Delta Spec Requirements
   and adds the explicit blank-value Scenario
   `display-conference-focus-build-id-005`. Intake §9 items 2 and 3 are
   updated in place with the same provenance.

   Implementation shape: the new label component follows the same
   presence-check idiom already used by `VisitorsCountLabel.tsx:25`
   (`condition ? (<Label .../>) : null`), with the condition
   `getProperty('focus-build-id')?.trim().length > 0` — this renders nothing
   for `undefined`, `null`, `""`, or a whitespace-only string (same as the
   "not provided" case, now also covered by the new blank-value
   Requirement), and otherwise renders the value exactly as returned by
   `getProperty` (`JitsiConference.ts:4710-4712`, which passes through with
   no server-side validation), with no truncation, length limit, or other
   transformation — using the same row component and styling as every other
   entry in `COMPONENTS`, so no new presentation work is needed for long
   values.

8. **Compatibility with a Focus that omits the value (confirms
   Requirement 3):** every hop is optional end-to-end — `jitsi-control`
   simply does not add the key, `lib-jitsi-meet`'s relay carries whatever
   keys are present with no required-keys check, and `jitsi-web`'s new
   label renders nothing when `getProperty` returns `undefined`. An older or
   unmodified Focus therefore reproduces exactly today's behavior (no row),
   verified directly at each hop above rather than assumed, with no
   compatibility shim needed anywhere.

## Repository Implementation Map

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-control` | Add `focus-build-id` = `CurrentVersionImpl.NIGHTLY_BUILD_ID` to the existing focus-presence `ConferenceProperties` object (`JitsiMeetConferenceImpl`), alongside `KEY_SUPPORTS_SESSION_RESTART`/`KEY_VISITORS_ENABLED`. | Produces the value consumed by `lib-jitsi-meet` via existing MUC presence; no inbound dependency from this change. |
| `lib-jitsi-meet` | Add one contract regression test confirming the existing generic `conference-properties` presence pass-through (`ChatRoom` → `JitsiConference`) carries `focus-build-id` into `getProperty('focus-build-id')`; no parser, allow-list, or storage change. | Depends on `jitsi-control` publishing the property in focus presence; depended on by `jitsi-web` for reading it. |
| `jitsi-web` | Add a `focus-build-id` entry to `ConferenceInfo`'s `COMPONENTS` list and to the default `autoHide` array in `constants.ts`; new label renders nothing when the value is empty/absent, and the raw value otherwise, visible to all participants (no moderator gate). | Depends on `lib-jitsi-meet` exposing the property via `getProperty`; no downstream dependency. |

## Risks / Trade-offs

- Low risk: additive, optional data on an existing property/relay/label
  path in each repository; no existing behavior changes when the value is
  absent — verified, not assumed, at every hop above.
- [Increased discoverability] → `NIGHTLY_BUILD_ID` is already transmitted to
  every participant via `ComponentVersionsExtension`, but the new UI row makes
  it substantially easier to find → accepted by the all-participants
  Requirement; no new data or audience is introduced (Decision 2).
- [Sibling-change coupling] → the `focus-version` and `focus-build-id`
  capabilities share the same `jitsi-web` mechanism (`COMPONENTS`,
  `autoHide`) but are independent entries; a merge race between the two
  changes' edits to `constants.ts`/`ConferenceInfo.tsx` is a normal
  source-control concern, not a design risk → mitigated by keeping each id
  and array insertion self-contained, not assuming shared line edits.
- [Manifest fallback] → in a non-packaged/dev Jicofo build,
  `NIGHTLY_BUILD_ID` falls back to the literal `"build.git"`
  (`CurrentVersionImpl.java:92`) rather than being absent; the row would
  then show that literal, not omit itself. This is existing, pre-change
  behavior of the value's source, not a new failure mode introduced here,
  and is consistent with the accepted Requirement ("whenever that value has
  been provided" — the field is always set, just sometimes to a placeholder
  string in dev builds).

## Migration Plan

No data migration. Every hop (`jitsi-control` property, `lib-jitsi-meet`
relay, `jitsi-web` label) is independently a no-op when its input is absent
(verified above), so the three repositories can roll out in any order with
no intermediate broken state: an older `lib-jitsi-meet`/`jitsi-web` against a
newer `jitsi-control` continues to show no row (property present but
nothing reads it yet); a newer `lib-jitsi-meet`/`jitsi-web` against an older
`jitsi-control` shows no row (property never set). Rollback is symmetric:
reverting any one repository returns that hop to "value absent," which the
other hops already treat as the normal case.

## Open Questions

- Client-version compatibility test coverage beyond general project rules
  (Intake §9 Q4) — deferred to Tasks, where the actual test matrix is
  decided, per Intake's own routing of this question.

Intake §9 item 3 (long-value presentation) is resolved, not deferred: the
owner decided long values use Conference info's existing row presentation
behavior with no custom truncation (Decision 7) — reusing the same row
component and styling as every other `COMPONENTS` entry, so no separate
accessibility treatment is introduced beyond what that existing presentation
already provides.
