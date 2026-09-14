## Why

Support engineers investigating conference issues need the exact build
identifier of the serving Focus (`jitsi-control`, the Jicofo role) to confirm
whether a specific fix is present, but today Conference info shows nothing
like it. Showing the identifier in Conference info would make it available
to the participant helping support investigate the issue.

## What Changes

- `jitsi-control`'s existing focus-presence `ConferenceProperties` object
  (already built and published by `JitsiMeetConferenceImpl`) can carry an
  additional optional `focus-build-id` entry, carrying a public Jicofo build
  identifier. Design must confirm the existing source and suitability for public
  display; this remains an open question in Intake §9, not a verified runtime
  fact. This clarification follows the Intake review and preserves the intended
  public nature of the value. The accepted `display-conference-focus-version` Change
  chose the same transport, but its open implementation work is not treated as
  a current runtime fact. Older or unmodified Jicofo instances that omit the
  new property are unaffected.
- `lib-jitsi-meet` relies on its existing generic MUC presence
  `conference-properties` pass-through (`ChatRoom` into
  `JitsiConference.properties`, with no per-key allow-list) to expose
  `focus-build-id` when provided. No parsing or storage logic changes; this
  change adds one focused contract test confirming the pass-through carries
  `focus-build-id` end-to-end.
- `jitsi-web`'s Conference info shows a "Focus build id" row built from that
  value. The row is present only when the value was provided by the current
  conference; it is absent — no row, no placeholder, no error state — when
  the value was not provided.
- Audience: the row is visible to all participants, not restricted to
  moderators or support, consistent with the precedent row for
  `focus-version`.

No breaking changes: the field is optional end-to-end, and its absence
reproduces today's behavior (no "Focus build id" row) exactly.

## Capabilities

### New Capabilities
- `conference/focus-build-id-visibility`: participant- and support-facing
  visibility of the Focus (`jitsi-control`) build identifier serving the
  current conference, shown conditionally in Conference info.

### Modified Capabilities
(none)

## Repository Impact

| Repository | Capabilities |
| --- | --- |
| `jitsi-control` | `conference/focus-build-id-visibility` |
| `lib-jitsi-meet` | `conference/focus-build-id-visibility` |
| `jitsi-web` | `conference/focus-build-id-visibility` |

## Impact

- `jitsi-control`: adds one optional entry to the existing `ConferenceProperties`
  object already published in focus presence; no new transport, no change to
  conference allocation, bridge selection, or authentication.
- `lib-jitsi-meet`: no production code change; adds one focused contract test
  on the existing generic presence pass-through.
- `jitsi-web`: adds one conditional Conference info row consuming the
  existing `JitsiConference.getProperty('focus-build-id')` read path; no
  change to access control.
- Non-goal: `jitsi-videobridge` is out of scope — "Focus" names the
  `jitsi-control` role, not the bridge.
- Non-goal: any change to conference allocation, bridge selection, media
  routing, or authentication.
- Non-goal: new storage, logging, or analytics beyond holding the value as
  part of the current conference's in-memory properties in `lib-jitsi-meet`.
