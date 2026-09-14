## Why

Jicofo (the `jitsi-control` service that plays the domain "Focus" role: it
coordinates the conference and participant connections) already has a
public version string, but nothing surfaces which Focus version is serving
a given conference to a participant or to support. Today, finding that out
requires server log access, which participants do not have and which slows
down support investigations that are specific to a Focus version (for
example, a behavior already fixed in a newer Jicofo release).

Why now: this is a small, additive visibility change with no change to
conference allocation, bridge selection, or media routing — it exposes an
existing, already-public value through a surface participants and support
already use, removing a log-access dependency from routine version checks.

Expected improvement: a participant or support agent can read the serving
Focus version directly from the conference details UI, without server log
access.

## What Changes

- Jicofo's conference-allocation success response can carry an optional
  `focus-version` value, carrying its already-public version string. Older
  or unmodified Jicofo instances that omit it are unaffected.
- `lib-jitsi-meet` retains the `focus-version` value, when provided, as part
  of the current conference's exposed properties.
- `jitsi-web`'s conference details UI shows a "Focus version" row built
  from that value. The row is present only when the value was provided by
  the current conference; it is absent — no row, no placeholder, no error
  state — when the value was not provided. This is the common case at
  rollout, when the serving Jicofo does not yet send the field.
- Audience: the "Focus version" row is visible to all participants in the
  conference details UI, not restricted to moderators or support.

No breaking changes: the field is optional end-to-end, and its absence
reproduces today's behavior (no "Focus version" row) exactly.

## Capabilities

### New Capabilities

- `conference/focus-version-visibility`: participant- and support-facing
  visibility of the Focus (`jitsi-control`) version serving the current
  conference, shown conditionally in the conference details UI.

### Modified Capabilities

(none)

## Repository Impact

| Repository | Capabilities |
| --- | --- |
| `jitsi-control` | `conference/focus-version-visibility` |
| `lib-jitsi-meet` | `conference/focus-version-visibility` |
| `jitsi-web` | `conference/focus-version-visibility` |

## Constraints and success criteria

- The field is optional on the allocation response; an older or unmodified
  Jicofo that omits it must not break `lib-jitsi-meet` or `jitsi-web`, and
  must render no "Focus version" row and no error state — this is the
  expected default case, not an edge case.
- The value is Jicofo's already-public version string; this change
  introduces no new version-negotiation or capability-handshake data.
- Non-goal: `jitsi-videobridge` (JVB) version display — "Focus" names the
  `jitsi-control` role specifically, not the bridge.
- Non-goal: any change to conference allocation, bridge selection, or media
  routing behavior — this is read-only visibility of an existing value.
- Non-goal: new storage, logging, or analytics for the value beyond holding
  it as part of the current conference's in-memory state in
  `lib-jitsi-meet`.
- Success is observed when: the conference-allocation success response can
  carry `focus-version`; `lib-jitsi-meet` retains it for the current
  conference; and `jitsi-web`'s conference details UI shows the "Focus
  version" row for all participants only when the value is present, with
  no row and no error state when it is absent.
