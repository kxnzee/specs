## Problem and success criteria

Jicofo (the `jitsi-control` service that plays the domain "Focus" role per
`openspec/context/02-domain.md`: "координирует конференцию и соединения
участников") already has a public version string, but nothing surfaces which
Focus version is serving a given conference to a participant or to support.
Today that requires server log access, which is unavailable to participants
and slows down support investigations that are specific to a Focus version
(for example, a behavior already fixed in a newer Jicofo release).

Success is observed when: Jicofo's conference-allocation success response can
carry an optional `focus-version` field with its already-public version;
`lib-jitsi-meet` retains that value for the current conference; and
`jitsi-web`'s conference details UI shows a "Focus version" line built from
it — present only when the value was provided, absent (no line at all) when
it was not. A participant or support agent can then read the serving Focus
version directly from the conference details UI, without server log access.

## Constraints and non-goals

- The field is optional on the allocation response. An older or unmodified
  Jicofo that omits it must not break `lib-jitsi-meet` or `jitsi-web`; the
  common case (no field) must render no "Focus version" line and no error
  state.
- The value is Jicofo's already-public version string — this introduces no
  new version-negotiation or capability-handshake data, so it does not by
  itself touch the authentication/security-contract conditions in
  `openspec/context/05-constraints.md`. It is nonetheless a new disclosure of
  diagnostic metadata into the client-facing UI, and `05-constraints.md`
  records conference metadata classification as unresolved (open TODO); this
  change cannot claim on its own that no security/privacy review is needed —
  see Open questions.
- Non-goal: `jitsi-videobridge` (JVB) version display. Per domain.md, "Focus"
  names the `jitsi-control` role specifically, not the bridge; JVB is not in
  scope for this change.
- Non-goal: any change to conference allocation, bridge selection, or media
  routing behavior — this is read-only visibility of an existing value.
- Non-goal: new storage, logging, or analytics for the value beyond holding
  it as part of the current conference's in-memory state in
  `lib-jitsi-meet`.

## Alternatives considered

### Alternative A: Always show a "Focus version" row, with a placeholder (e.g. "unknown") when absent

- Approach: `jitsi-web` always renders the "Focus version" row in conference
  details, substituting placeholder text when `lib-jitsi-meet` has no value.
- Advantages: fixed UI layout; no conditional row logic.
- Trade-offs: contradicts the stated requirement that the row must not show
  at all when the value is absent; would show a confusing placeholder for
  every conference served by a Jicofo that doesn't yet send the field, which
  is expected to be the common case at rollout.

### Alternative B: Conditionally render the row only when focus-version is present (chosen)

- Approach: `lib-jitsi-meet` exposes `focus-version` as part of the current
  conference's properties only when Jicofo included it in the allocation
  response; `jitsi-web`'s conference details UI reads that property and
  renders the "Focus version" row only when it is defined.
- Advantages: matches the stated requirement exactly; degrades gracefully
  against older Jicofo versions; no misleading placeholder.
- Trade-offs: the conference details panel's row count varies by serving
  Focus version — acceptable, since that variability is the explicit
  requirement.

### Alternative C: Surface focus-version only in a technical/debug surface instead of the conference details panel

- Approach: keep the value in `lib-jitsi-meet`, but only expose it through a
  developer-facing surface (e.g. stats overlay or console), not the everyday
  conference details UI participants and support already use.
- Advantages: keeps the participant-facing conference details panel scoped
  to participant-facing data only.
- Trade-offs: does not meet the stated value — support and participants are
  not expected to open developer tooling; the whole point is visibility
  without server log access through a surface they already use.

## Agreed approach

Pending human approval for this smoke test (see Open questions): Alternative
B — conditional rendering, no placeholder — is recommended as the closest
match to the explicit requirement and value statement supplied for this
change.

## Key decisions

- Candidate Repository Impact for Proposal: `jitsi-control` (Jicofo emits
  `focus-version` in the conference-allocation success response),
  `lib-jitsi-meet` (stores/exposes the value for the current conference),
  `jitsi-web` (conditionally renders "Focus version" in conference details).
  `jitsi-videobridge` is not impacted — "Focus" is the `jitsi-control` role,
  not the bridge, per `openspec/context/02-domain.md`.
- The field carries an already-public version string and introduces no new
  negotiation/capability data, so it does not trigger the
  `openspec/context/05-constraints.md` human-review condition for
  authentication/security-contract changes specifically. It is still a new
  surfacing of diagnostic metadata to participants and support in the UI;
  whether that disclosure and its audience are acceptable is a separate
  human decision, not settled by this brainstorm (see Open questions).
- Absence of `focus-version` is the expected default case and must render no
  row and no error state in `jitsi-web`, not a placeholder.

## Open questions

- Approve Alternative B (conditional row, no placeholder) as the agreed
  approach, or prefer different UX (naming, placement within conference
  details) — human decision required before Proposal.
- Is it acceptable to surface Jicofo's Focus version as new diagnostic
  metadata in the participant-facing conference details UI, and to which
  audience (all participants, or only moderators/support)? This is a
  distinct disclosure question from "is the version itself already public"
  — `openspec/context/05-constraints.md` leaves conference-metadata
  classification unresolved, so this change cannot assume no
  security/privacy review is warranted. Human decision required before
  Proposal.
- The registered `jitsi-control`, `lib-jitsi-meet` and `jitsi-web` checkouts
  are available and were reachable via CodeGraph in this session, which
  confirmed that a matching conference-allocation response path, a current-
  conference properties store, and a conference-details UI surface exist in
  each repository respectively. The exact response field shape on the
  Jicofo side and the precise code paths to change remain to be pinned down
  as current-state anchors during Design/Apply, not decided here.
