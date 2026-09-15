## Context

Accepted Proposal scope: `conference/focus-version-visibility` (new
capability), spanning `jitsi-control`, `lib-jitsi-meet`, `jitsi-web`.
`jitsi-videobridge` is out of scope.

Targeted CodeGraph exploration of the three registered checkouts confirms
the following three-repository path; no application was run to establish it.

- `JitsiConference.properties` is kept current from MUC presence:
  `ChatRoom.ts:1123-1135` reads every property under
  `conference-properties` and emits `CONFERENCE_PROPERTIES_CHANGED`;
  `JitsiConference.ts:673` subscribes `_updateProperties` directly to that
  event. This path has no per-key allow-list. Jicofo's
  `JitsiMeetConferenceImpl` already builds a `ConferenceProperties` object
  and publishes it in the focus presence — the same presence path already
  cited below for `ComponentVersionsExtension` — independent of the
  conference-allocation response. The corrected transport for this change is
  that existing focus-presence `ConferenceProperties` path, not the
  allocation IQ response. `JitsiMeetConferenceImpl.java:642-663` constructs
  the initial focus presence extensions and publishes
  `createConferenceProperties()`; the helper itself is defined at
  `JitsiMeetConferenceImpl.java:722-727`.
- `jitsi-control` already computes a public version string
  (`CurrentVersionImpl.VERSION`, `jicofo/src/main/java/org/jitsi/jicofo/version/CurrentVersionImpl.java`)
  and already exposes it multiple ways, including a `ComponentVersionsExtension`
  presence extension sent on room join (`JitsiMeetConferenceImpl.joinTheRoom()`,
  `jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java:642-646`)
  and an unauthenticated HTTP route (`GET /about/version`,
  `jicofo/src/main/kotlin/org/jitsi/jicofo/ktor/Application.kt:123-134`).
  The corrected target for this change is a third existing surface in the
  same focus presence: the `ConferenceProperties` object
  `JitsiMeetConferenceImpl` already builds and publishes there, distinct
  from both of the above.
- The conference-allocation success response is built in
  `ConferenceIqHandler.doHandleConferenceIq`
  (`jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt:106-124`),
  which already attaches optional string properties to the response
  (`addProperty(ConferenceIq.Property("authentication", ...))`,
  `"externalAuth"`, `"sipGatewayEnabled"`, and later `"visitors-supported"`,
  `"live"`). This remains an established pattern for that response, but it
  is not this change's target. It must continue to omit `focus-version`, and
  its existing response test should retain an explicit absence assertion.
- `lib-jitsi-meet`'s `ChatRoom` and `JitsiConference` already relay MUC
  presence `conference-properties` generically into
  `JitsiConference.properties` (`ChatRoom.ts:1123-1135` emits the change;
  `JitsiConference.ts:673` binds it to `_updateProperties`,
  `JitsiConference.ts:2126-2184`), with no per-key allow-list on this path —
  unlike `xmpp/moderator.js`'s conference-allocation-response parsing
  (`_parseConferenceIq`, `modules/xmpp/moderator.js:261-291`), which is not
  this change's target. Consumers read a value via
  `JitsiConference.getProperty(key)` (`JitsiConference.ts:4710-4712`),
  unchanged.
- `jitsi-web`'s conference details header
  (`react/features/conference/components/web/ConferenceInfo.tsx`) already
  renders a config-driven, conditional set of small info rows/labels (e.g.
  `E2EELabel`, `VisitorsCountLabel`, `InsecureRoomNameLabel`) from a
  `COMPONENTS` list, gated by which ids appear in the effective
  `alwaysVisible` / `autoHide` id lists. `getConferenceInfo`
  (`react/features/conference/components/functions.any.ts`) falls back to
  the default `alwaysVisible` / `autoHide` arrays defined in
  `react/features/conference/components/constants.ts` when
  `config.conferenceInfo` does not override them. A component id absent
  from both the config override and these `constants.ts` defaults is never
  eligible to render, regardless of the underlying value — so the new
  label's id must be added to one of the `constants.ts` default arrays
  (`autoHide` is the closer fit: the row should disappear when the value is
  absent like other optional/informational rows, not persist unconditionally
  like `alwaysVisible` entries). Existing labels in this list already follow
  an internal "render nothing when the underlying value/condition is absent"
  pattern (e.g. `VisitorsCountLabel` only makes sense when there are
  visitors).
- `jitsi-web` already receives `JitsiConferenceEvents.PROPERTIES_CHANGED` in
  `react/features/base/conference/actions.any.ts` and stores the complete map
  in `state['features/base/conference'].properties` through the existing base
  conference reducer. UI components select data from Redux; the local
  `IJitsiConference` interface does not expose `getProperty`. The new label
  therefore reads the optional key from that existing Redux map and adds no
  state container or key-specific reducer.

This confirms the three-repository shape assumed in Proposal is accurate,
and identifies an established per-repository pattern to extend rather than
a new mechanism to invent.

## Goals / Non-Goals

Goals: implement `conference/focus-version-visibility` exactly as scoped in
Proposal, reusing each repository's existing optional-property /
conditional-label pattern instead of introducing a new cross-repository
mechanism.

Non-Goals (carried from Proposal, restated for Design):
`jitsi-videobridge` version display; any change to conference allocation,
bridge selection, or media routing; new storage, logging, or analytics for
the value.

## Decisions and alternatives

- **Transport**: add `focus-version` to the `ConferenceProperties`
  object Jicofo's `JitsiMeetConferenceImpl` already builds and publishes in
  the focus presence, following the same pattern already used for that
  object's other entries. A `ConferenceIq.Property` on the
  conference-allocation success response, parsed by `moderator.js`, is
  rejected because `_handleSuccess` filters that parsed result before it
  reaches `JitsiConference.properties`; adding the field there would not make
  it visible to `jitsi-web`. A separate request/response (e.g. reusing the
  `/about/version` HTTP route) remains not chosen for the same reason as
  before: a second round-trip and value source for no accepted benefit.
- **lib-jitsi-meet**: no parser change. `ChatRoom` and
  `JitsiConference` already relay MUC presence
  `conference-properties` generically into `JitsiConference.properties`
  through the existing `_updateProperties` path, with no per-key allow-list,
  so `focus-version` reaches `JitsiConference.properties` automatically once
  `jitsi-control` publishes it in the focus presence — no new state
  container, no `moderator.js` change. This change's `lib-jitsi-meet` scope
  is limited to contract regression coverage confirming that existing
  generic pass-through actually carries `focus-version` end-to-end; it does
  not add parsing logic. Consumers read it via the existing
  `getProperty('focus-version')`, unchanged.
- **jitsi-web placement and conditionality**: add a new label to the
  existing `ConferenceInfo` `COMPONENTS` list (its header info-row
  mechanism) rather than building a separate "conference details" surface,
  and register its id in the default `autoHide` array in
  `react/features/conference/components/constants.ts` (alongside the other
  default-`autoHide` ids) so the row is eligible to render without
  requiring a deploy-time `config.conferenceInfo` override — omitting this
  registration means `ConferenceInfo` never renders the component by
  default, independent of the underlying value. The new label component
  selects `focus-version` from the existing
  `state['features/base/conference'].properties` map and renders nothing when
  the selected value is absent or not a non-empty string — mirroring the
  existing "render nothing when the underlying value is absent" pattern — so
  once registered, the row's conditional appearance is driven only by value
  presence, not by needing further deploy-time configuration. This directly
  implements Alternative B from brainstorm.md (conditional row, no
  placeholder) and the approved all-participants audience: the label is not
  gated on moderator role.
- **Optionality and backward compatibility**: every hop (Jicofo presence
  property → lib-jitsi-meet's existing generic presence pass-through →
  jitsi-web label) already treats the value as optional/absent-tolerant in
  its existing pattern, so an older Jicofo that omits `focus-version`
  reproduces today's behavior end-to-end with no code path change beyond
  "value not present."

## Repository Implementation Map

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-control` | Add the already-public Focus version to the existing `ConferenceProperties` published in the focus presence (`JitsiMeetConferenceImpl`); keep the conference-allocation IQ response unchanged and explicitly covered as not containing `focus-version`. | Produces the `focus-version` value consumed by `lib-jitsi-meet` via presence; no inbound dependency from this change. |
| `lib-jitsi-meet` | Add contract regression coverage confirming the existing generic MUC presence `conference-properties` pass-through (`ChatRoom` into `JitsiConference`) carries `focus-version` into the current conference's exposed properties; no parser or storage change. | Depends on `jitsi-control` publishing the property in focus presence; is depended on by `jitsi-web` for reading it. |
| `jitsi-web` | Select `focus-version` from the existing base-conference Redux properties map and conditionally render the "Focus version" row for all participants, using a new `lang/main.json` translation key `info.focusVersion` (`"Focus version: {{version}}"`). | Depends on the existing `JitsiConferenceEvents.PROPERTIES_CHANGED` bridge from `lib-jitsi-meet`; no new state and no downstream dependency. |

## Risks / Trade-offs

- Low risk: purely additive, optional data on an existing response/property
  path in each repository; no existing behavior is modified when the value
  is absent.
- Disclosure risk (accepted): this makes Jicofo's version — already public
  via other channels — visible to every participant in a UI surface they
  routinely see, not just to those who would seek it out. The human owner
  accepted this for the approved audience (see brainstorm.md Key
  decisions); no further mitigation is in scope for this change.
- Trade-off: reusing `ConferenceInfo`'s existing label/config mechanism
  means the row's eligibility is still technically subject to deploy-time
  `config.conferenceInfo.alwaysVisible`/`autoHide` overrides, consistent
  with every other label in that list. This change registers the new
  label's id in the default `autoHide` array in `constants.ts` so the
  conditional-on-value behavior described in Goals is what participants
  observe out of the box, without requiring a deploy-time override; a
  deployment that explicitly overrides `config.conferenceInfo.autoHide` to
  omit `focus-version` can still hide the row even when the value is
  present — an accepted, pre-existing property of this configuration
  mechanism, not new to this change.

## Migration, rollout and rollback

- No data migration. Rollout is a normal three-repository deploy; the
  `focus-version` property, its already-generic presence relay, and its
  rendering are each independently no-ops when absent, so the repositories
  can roll out in any order without an intermediate broken state — an older
  `lib-jitsi-meet` or `jitsi-web` simply continues to show no row against a
  newer `jitsi-control`, and a newer `lib-jitsi-meet`/`jitsi-web` show no
  row against an older `jitsi-control`.
- Rollback is symmetric: reverting any one repository returns that hop to
  "value absent," which the other hops already treat as the normal case.

## Open Questions

None outstanding for Planning. Brainstorm's UX-approval and disclosure
Open questions are resolved (see brainstorm.md). The wire property key
(`focus-version`, used identically by `jitsi-control`'s presence
`ConferenceProperties`, `lib-jitsi-meet`'s relayed `properties` map, and
`jitsi-web`'s base-conference Redux properties read) and the new `ConferenceInfo` component id
(`focus-version`) are fixed by this corrected Design and carried
consistently through Tasks. The path was confirmed by
targeted CodeGraph inspection; Apply still must verify its tests and actual
behavior. No application was run during this correction.
