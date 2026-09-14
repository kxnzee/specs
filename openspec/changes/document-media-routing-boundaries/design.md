## Context

`jitsi-control/doc/` and `jitsi-videobridge/doc/` already keep topic-scoped
markdown files (e.g. `jitsi-control/doc/conference-request.md`,
`jitsi-videobridge/doc/rest-colibri2.md`). `openspec/context/03-architecture.md`
already records `jitsi-control -> jitsi-videobridge` as a media-control relationship
at a responsibility level (sources: `jitsi-control/README.md:19-20`,
`jitsi-videobridge/doc/rest-colibri2.md:9-40`), without code-level anchors. See
`proposal.md` - Why for the underlying motivation; no restatement here.

Planning-time CodeGraph evidence confirming the control boundary exists and is
reachable, collected in this session:

- CodeGraph explore, repository `jitsi-control`, checkout
  `/private/tmp/openspec-jitsi-pilot.qjWHIr/workspace/src/jitsi-control`, query
  "BridgeSelector selecting a Jitsi Videobridge instance for a new conference
  and how JitsiMeetConferenceImpl or the colibri client uses it to establish
  media routing to the selected bridge", invoked via
  `openspec-orch plugin exec --repo jitsi-control codegraph explore`. Confirmed
  `BridgeSelector` (`jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/BridgeSelector.kt:40`)
  is held by `JitsiMeetConferenceImpl` as a constructor parameter/field
  (`jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java:317,336`).
- CodeGraph explore, repository `jitsi-control`, same checkout, query "where
  JitsiMeetConferenceImpl calls BridgeSelector.selectBridge to pick a
  videobridge for a new conference and how ColibriSessionManager or
  Colibri2Session then sends the conference-create request to that selected
  bridge", invoked the same way. Confirmed
  `BridgeSelector.selectBridge(...)`
  (`jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/BridgeSelector.kt:161-171`),
  called from `ColibriV2SessionManager`
  (`jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/colibri/ColibriV2SessionManager.kt`),
  and `Colibri2Session`
  (`jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/colibri/Colibri2Session.kt:64`)
  as the object that carries the colibri2 control session to a selected bridge.
- CodeGraph explore, repository `jitsi-videobridge`, checkout
  `/private/tmp/openspec-jitsi-pilot.qjWHIr/workspace/src/jitsi-videobridge`,
  query "IQ handler in jitsi-videobridge that receives a colibri2
  conference-modify/create request from jicofo and creates a Conference and
  Endpoint, and how it reports back load/stats used by bridge selection",
  invoked via
  `openspec-orch plugin exec --repo jitsi-videobridge codegraph explore`.
  Confirmed `Conference`
  (`jvb/src/main/java/org/jitsi/videobridge/Conference.java:300-351`)
  constructs a `Colibri2ConferenceHandler` (line 324) and a `ColibriQueue`
  (line 325) whose request handler receives each colibri2 request (logged as
  "RECV colibri2 request: ...", line 346) and dispatches it to
  `colibri2Handler.handleConferenceModifyIQ(request.getRequest())` (line 349).

These anchors confirm reachability and existence; they are recorded here as
evidence with provenance, not restated as a normative Design decision — Apply
re-verifies each cited anchor against current source before publishing the
doc, since file:line detail is Repository-owned, not Store-level.

## Goals / Non-Goals

**Goals:**

- Document, in `jitsi-control`, the bridge-selection step and the outbound
  colibri2 control request that follows it.
- Document, in `jitsi-videobridge`, the receiving side of that colibri2
  control request.
- Cross-link the two new docs to each other and to the existing
  `jitsi-control/doc/conference-request.md` and
  `jitsi-videobridge/doc/rest-colibri2.md` protocol references, instead of
  duplicating their wire-format content.

**Non-Goals:**

- Documenting the RTP media transport itself (packet-level forwarding) — out
  of scope; this documents only the control boundary (bridge selection +
  colibri2 control session), already partially covered at protocol level by
  `jitsi-videobridge/doc/web-sockets.md`.
- Documenting the client join/entry path — covered separately by
  `document-conference-entrypoints`.
- Any change to code, configuration, or tests.

## Decisions and alternatives

- **One dedicated file per repository, not an edit to existing protocol
  docs.** `conference-request.md` and `rest-colibri2.md` document wire
  protocols; this change documents which code selects a bridge and which code
  receives the resulting control request. Alternative considered: a single
  combined doc in `jitsi-control` covering both sides — rejected because it
  would force `jitsi-control`'s doc to describe `jitsi-videobridge`-owned
  internals it cannot keep current (see `brainstorm.md` - Alternatives
  considered).
- **File paths:** `jitsi-control/doc/bridge-control-boundary.md` and
  `jitsi-videobridge/doc/bridge-control-boundary.md`, matching each
  repository's existing `doc/<topic>.md` convention and using a shared
  filename so the cross-link pairing is obvious.
- **Exact symbol and file:line anchors are Apply-time, repository-owned
  content, not a Store-level decision.** The published doc in each repository
  must cite the current entry-point function/class with its file:line,
  confirmed against current source at Apply time (re-verified, since it is
  exactly the kind of implementation detail that belongs to the Repository,
  not this Store). Design only fixes the observable role each doc must cover
  (see Repository Implementation Map), already confirmed reachable per Context
  above.

## Repository Implementation Map

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-control` | Add `doc/bridge-control-boundary.md` documenting the bridge-selection step (`BridgeSelector.selectBridge`) used when a conference needs a bridge, and the colibri2 control session (`ColibriV2SessionManager` / `Colibri2Session`) that sends the resulting conference create/modify control request to the selected bridge. Cite the current entry point's file:line, confirmed against source at Apply time. Link to `jitsi-videobridge/doc/bridge-control-boundary.md` and to the existing `doc/conference-request.md`. | No code contract change. Documentation-only, so no dependency on other repositories' release state. |
| `jitsi-videobridge` | Add `doc/bridge-control-boundary.md` documenting the colibri2 request-handling path in `Conference` (`ColibriQueue` → `Colibri2ConferenceHandler.handleConferenceModifyIQ`) that receives and processes the control request from `jitsi-control`. Cite the current entry point's file:line, confirmed against source at Apply time. Link to the existing `doc/rest-colibri2.md` protocol reference and to `jitsi-control/doc/bridge-control-boundary.md`. | No code contract change. References the existing colibri2 protocol without altering it. |
| `jitsi-web` | No implementation task. | Confirmed no-change in `proposal.md` - Impact; not represented in this map beyond that confirmation. |

## Risks / Trade-offs

- [Referenced code moves or is refactored, silently making the new docs stale]
  → Mitigation: each doc cites an exact file:line anchor, confirmed at Apply
  time and re-checkable on every later review; no automated drift-detection is
  introduced by this change (out of scope).
- [Two separate files per repository could drift out of sync with each other]
  → Mitigation: each doc explicitly cross-links the other and states which
  side of the boundary it owns, and both use the same filename
  (`bridge-control-boundary.md`) to make the pairing obvious.

## Migration, rollout and rollback

Not applicable — documentation-only change with no runtime behavior,
configuration, or deployment impact. Rollback is a plain revert of the added
markdown files in each repository.

## Open Questions

None — repository impact and the code boundary were confirmed via CodeGraph
evidence in this session (see Context) before Proposal.
