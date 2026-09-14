## Context

Accepted Proposal scope: `conference/focus-version-visibility` (new
capability), spanning `jitsi-control`, `lib-jitsi-meet`, `jitsi-web`.
`jitsi-videobridge` is out of scope.

CodeGraph exploration of the three registered checkouts confirms the
following current state:

- `jitsi-control` already computes a public version string
  (`CurrentVersionImpl.VERSION`, `jicofo/src/main/java/org/jitsi/jicofo/version/CurrentVersionImpl.java`)
  and already exposes it two ways: a `ComponentVersionsExtension` presence
  extension sent on room join (`JitsiMeetConferenceImpl.joinTheRoom()`,
  `jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java:642-646`),
  and an unauthenticated HTTP route (`GET /about/version`,
  `jicofo/src/main/kotlin/org/jitsi/jicofo/ktor/Application.kt:123-134`).
  Neither is the conference-allocation response the Proposal targets.
- The conference-allocation success response is built in
  `ConferenceIqHandler.doHandleConferenceIq`
  (`jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt:106-124`),
  which already attaches optional string properties to the response
  (`addProperty(ConferenceIq.Property("authentication", ...))`,
  `"externalAuth"`, `"sipGatewayEnabled"`, and later `"visitors-supported"`,
  `"live"`). This is an established, existing pattern for adding one more
  optional property.
- `lib-jitsi-meet`'s `xmpp/moderator.js` already parses a fixed allow-list of
  named `<property>` elements out of the conference-allocation response IQ
  into a plain properties object (`_parseConferenceIq`,
  `modules/xmpp/moderator.js:261-291`, e.g. the `"authentication"` and
  `"sipGatewayEnabled"` checks). `JitsiConference._updateProperties`
  (`JitsiConference.ts:2126-2184`) is the single place that assigns the
  conference's `properties` map and emits `PROPERTIES_CHANGED`; consumers
  read a value via `JitsiConference.getProperty(key)`
  (`JitsiConference.ts:4710-4712`).
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

- **Transport**: add `focus-version` as one more optional
  `ConferenceIq.Property` on the existing conference-allocation success
  response (`ConferenceIqHandler.doHandleConferenceIq`), following the same
  shape as `authentication` / `sipGatewayEnabled`. Alternative — a separate
  request/response (e.g. reusing the `/about/version` HTTP route from
  `lib-jitsi-meet`) was not chosen: it would add a second network
  round-trip and a second value source for a single-field, already-public
  value, for no accepted benefit.
- **lib-jitsi-meet parsing**: extend `_parseConferenceIq`'s existing named
  `<property>` allow-list with `focus-version`, so it flows into
  `JitsiConference.properties` through the existing `_updateProperties`
  path with no new state container. Consumers read it via the existing
  `getProperty('focus-version')`.
- **jitsi-web placement and conditionality**: add a new label to the
  existing `ConferenceInfo` `COMPONENTS` list (its header info-row
  mechanism) rather than building a separate "conference details" surface,
  and register its id in the default `autoHide` array in
  `react/features/conference/components/constants.ts` (alongside the other
  default-`autoHide` ids) so the row is eligible to render without
  requiring a deploy-time `config.conferenceInfo` override — omitting this
  registration means `ConferenceInfo` never renders the component by
  default, independent of the underlying value. The new label component
  internally renders nothing when `getProperty('focus-version')` is
  undefined — mirroring the existing "render nothing when the underlying
  value is absent" pattern already used by sibling labels in that list — so
  once registered, the row's conditional appearance is driven only by value
  presence, not by needing further deploy-time configuration. This directly
  implements Alternative B from brainstorm.md (conditional row, no
  placeholder) and the approved all-participants audience: the label is not
  gated on moderator role.
- **Optionality and backward compatibility**: every hop (Jicofo property →
  lib-jitsi-meet parsed property → jitsi-web label) already treats the
  value as optional/absent-tolerant in its existing pattern, so an older
  Jicofo that omits `focus-version` reproduces today's behavior end-to-end
  with no code path change beyond "value not present."

## Repository Implementation Map

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-control` | Attach the already-public Focus version as an optional property on the conference-allocation success response. | Produces the `focus-version` value consumed by `lib-jitsi-meet`; no inbound dependency from this change. |
| `lib-jitsi-meet` | Parse the optional `focus-version` property from the conference-allocation response and expose it as a current-conference property. | Depends on `jitsi-control` emitting the property; is depended on by `jitsi-web` for reading it. |
| `jitsi-web` | Conditionally render the "Focus version" row in the conference details UI for all participants, only when `lib-jitsi-meet` exposes a value, using a new `lang/main.json` translation key `info.focusVersion` (`"Focus version: {{version}}"`). | Depends on `lib-jitsi-meet` exposing the property; no downstream dependency. |

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
  `focus-version` property, its parsing, and its rendering are each
  independently no-ops when absent, so the repositories can roll out in any
  order without an intermediate broken state — an older `lib-jitsi-meet` or
  `jitsi-web` simply continues to show no row against a newer
  `jitsi-control`, and a newer `lib-jitsi-meet`/`jitsi-web` show no row
  against an older `jitsi-control`.
- Rollback is symmetric: reverting any one repository returns that hop to
  "value absent," which the other hops already treat as the normal case.

## Open Questions

None outstanding for Planning. Brainstorm's UX-approval and disclosure
Open questions are resolved (see brainstorm.md). The wire property key
(`focus-version`, used identically by `jitsi-control`'s
`ConferenceIq.Property`, `lib-jitsi-meet`'s parsed `properties` map, and
`jitsi-web`'s `getProperty` read) and the new `ConferenceInfo` component id
(`focus-version`) are already fixed by this Design and carried consistently
through Tasks and Plan — not an open question left for Apply.
