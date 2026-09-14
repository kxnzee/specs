## Problem and success criteria

Contributors investigating or maintaining the boundary between `jitsi-control`
(Jicofo) and `jitsi-videobridge` (JVB) have no verified, code-anchored description
of how a conference selects a bridge and how conference-control commands
(create/modify) reach the selected bridge. `openspec/context/03-architecture.md`
records this relationship only at responsibility level
(`jitsi-control -> jitsi-videobridge`), without code-level
anchors. This creates onboarding and incident-response friction at exactly the
boundary most likely to be involved in bridge-selection or overload incidents.

Success is observed when a developer can read one pair of cross-linked documents
and get an evidence-backed answer to "which code selects a bridge for a
conference, and which code on the bridge receives the resulting control
request", without re-tracing the call graph themselves. Runtime behavior does
not change — this is developer documentation only.

## Constraints and non-goals

- Documentation only; no code, configuration, test, or build changes in
  `jitsi-control` or `jitsi-videobridge`.
- `jitsi-web` is no-change: the client join/entry path is documented separately
  by `document-conference-entrypoints` and does not perform or influence bridge
  selection, so it is not part of this control boundary.
- Scope is the *control* boundary (bridge selection + the colibri2 control
  session), not the RTP media transport itself (per-packet forwarding), which is
  a distinct concern already partially covered at protocol level by
  `jitsi-videobridge/doc/web-sockets.md`.
- Must cite exact, currently-verified code anchors (file:line), not architecture
  inferred solely from README-level text.

## Alternatives considered

### Alternative A: Single shared document in `jitsi-control` only

- Approach: add one markdown file in `jitsi-control/doc/` describing both the
  bridge-selection logic and, informationally, what `jitsi-videobridge` does
  with the resulting control request.
- Advantages: a single place to read; no cross-repository linking needed.
- Trade-offs: forces `jitsi-control`'s doc to describe `jitsi-videobridge`
  internals it does not own, so it drifts silently when `jitsi-videobridge`
  changes without the `jitsi-control` maintainer noticing.

### Alternative B: One dedicated doc per repository, cross-linked (chosen)

- Approach: `jitsi-control/doc/...md` documents bridge selection and the
  outbound colibri2 control request; `jitsi-videobridge/doc/...md` documents the
  receiving side (colibri2 request handling that creates/updates the
  conference). Each doc cross-links the other.
- Advantages: each repository documents only what it owns and can keep current;
  matches the existing per-repository `doc/<topic>.md` convention already used
  in both repositories.
- Trade-offs: a reader wanting the full picture must open two documents;
  mitigated by explicit cross-links and a one-line summary of the other side in
  each doc.

## Agreed approach

Alternative B. Add one documentation file to `jitsi-control` describing
conference-level bridge selection (`BridgeSelector`) and how the selected
bridge is told to create/modify the conference (the colibri2 control session),
and one documentation file to `jitsi-videobridge` describing the receiving side
(colibri2 request handling in `Conference`). `jitsi-web` is explicitly
no-change.

## Key decisions

- File placement follows each repository's existing `doc/<topic>.md`
  convention (matching `jitsi-control/doc/conference-request.md`,
  `jitsi-videobridge/doc/rest-colibri2.md`).
- Scope is the control boundary (bridge selection + colibri2 control session),
  explicitly excluding RTP media transport itself.
- Exact function/class names and file:line citations were confirmed via
  CodeGraph in this session; citations are re-verified against current source
  at Apply time, since they are Repository-owned implementation detail, not a
  Store-level decision (see design.md - Context and Decisions).

## Open questions

None — repository impact (`jitsi-control`, `jitsi-videobridge`: documentation;
`jitsi-web`: no-change) and the existence/reachability of the code boundary
were confirmed via CodeGraph evidence in this session before Proposal.
