## Why

Contributors and incident responders working across the `jitsi-control`
(Jicofo) and `jitsi-videobridge` (JVB) boundary have no verified, code-anchored
description of how a conference selects a bridge and how conference-control
commands (create/modify) reach that bridge. `openspec/context/03-architecture.md`
records the relationship only at responsibility level
(`jitsi-control -> jitsi-videobridge`, `type: media-control`), without code
anchors. This change adds verified developer documentation of that control
boundary so contributors do not have to re-trace it from scratch. Observable
runtime behavior does not change.

## What Changes

- Add developer documentation in `jitsi-control` describing the bridge-selection
  step (`BridgeSelector.selectBridge`) that picks a `jitsi-videobridge` instance
  for a conference, and how the resulting colibri2 control session
  (`ColibriV2SessionManager` / `Colibri2Session`) sends the conference
  create/modify control request to the selected bridge.
- Add developer documentation in `jitsi-videobridge` describing the receiving
  side: the colibri2 request queue in `Conference` that hands each request to
  `Colibri2ConferenceHandler.handleConferenceModifyIQ`, creating or updating the
  conference on that bridge.
- No code, configuration, tests, or build changes in any repository.
- Exact code identifiers were confirmed during Planning via CodeGraph and are
  recorded with provenance in `design.md` - Context; this proposal describes
  each side by its observable role rather than restating those identifiers,
  since a Code Repository's internal symbols and file:line are Repository-owned
  detail, not a Store-level decision (see `design.md` - Decisions).

## Capabilities

### New Capabilities

None. This change adds developer documentation only; it does not introduce or
modify externally observable system behavior, so no capability spec is created.

### Modified Capabilities

None — no spec-level behavior changes. `skip_specs: true` is set in
`.openspec.yaml` per the schema's Capabilities rule: specs describe behavior,
and behavior does not change here.

## Impact

Known incompatibility, reported rather than papered over: this proposal omits a
`## Repository Impact` section. `skip_specs` is set and this change declares no
capabilities (see Capabilities above), so the OpenSpec Graph's capability-keyed
`Repository | Capabilities` table contract cannot be satisfied — every row
would require at least one real capability identifier, and this change has
none to list. A `## Repository Impact` heading with an invalid or fabricated
table would fail Graph inspection (`openspec-graph inspect`); omitting the
heading avoids both the fabrication and the Graph error while
`openspec validate --strict` still passes under `skip_specs`. Per-repository
impact type and reason are given below instead.

- `jitsi-control` (documentation): add a markdown doc describing bridge
  selection and the outbound colibri2 control request to the selected bridge.
  Reason: holds the code that decides which bridge handles a conference and
  issues the control request.
- `jitsi-videobridge` (documentation): add a markdown doc describing the
  colibri2 control-request receiving path that creates/updates the conference
  on the bridge. Reason: holds the code that receives and processes that
  control request.
- `jitsi-web` (no-change): verified that the client join/entry path (documented
  separately by `document-conference-entrypoints`) does not perform or
  influence bridge selection; the client is not part of the control boundary
  documented here.
- No affected APIs, dependencies, runtime configuration, or tests in any
  repository.
- Evidence that these code paths exist and are reachable was collected via
  CodeGraph (`openspec-orch plugin exec codegraph explore`) against the indexed
  `jitsi-control` and `jitsi-videobridge` checkouts; exact queries and code
  anchors are recorded with provenance in `design.md` - Context, not restated
  here (Repository symbol/file:line detail belongs to the Repository, not this
  Store proposal).

## Constraints and success criteria

- Documentation only; no runtime behavior change in any repository.
- Each doc must cite an exact, currently-verified function/class and file:line
  for the boundary it documents.
- Success: a developer can read the `jitsi-control` doc and the
  `jitsi-videobridge` doc and obtain a verified answer to "which code selects a
  bridge, and which code on the bridge receives the resulting control request",
  without independently re-tracing the call graph.
