## Why

Developers investigating the client-to-focus boundary have no verified, code-anchored
description of the conference entry path — only a responsibility-level link in
`openspec/context/03-architecture.md`. This change adds verified developer documentation of that path so
contributors do not have to re-trace it from scratch. Observable runtime behavior does
not change.

## What Changes

- Add developer documentation in `jitsi-web` describing the client-side conference
  entry point that starts a conference join and how it triggers the signaling
  exchange that reaches the focus service.
- Add developer documentation in `jitsi-control` describing the focus-service entry
  point that receives the client's request and creates or looks up the resulting
  conference.
- No code, configuration, tests, or build changes in any repository.
- Exact code identifiers for these entry points were confirmed during Planning via
  CodeGraph and are recorded with provenance in `intake.md` §9; this proposal
  describes each entry point by its observable role rather than restating those
  identifiers, since a Code Repository's internal symbols and file:line are
  Repository-owned detail, not a Store-level decision (see design.md - Decisions).

## Capabilities

### New Capabilities
None. This change adds developer documentation only; it does not introduce or modify
externally observable system behavior, so no capability spec is created.

### Modified Capabilities
None — no spec-level behavior changes. `skip_specs: true` is set in `.openspec.yaml`
per the schema's Capabilities rule: specs describe behavior, and behavior does not
change here.

## Impact

Known incompatibility, reported rather than papered over: this proposal omits a
`## Repository Impact` section. `skip_specs` is set and this change declares no
capabilities (see Capabilities above), so the OpenSpec Graph's capability-keyed
`Repository | Capabilities` table contract cannot be satisfied — every row would
require at least one real capability identifier, and this change has none to list.
A `## Repository Impact` heading with an invalid or fabricated table would fail
Graph inspection (`openspec-graph inspect`); omitting the heading avoids both the
fabrication and the Graph error while `openspec validate --strict` still passes
under `skip_specs`. Per-repository impact type and reason are given below instead.

- `jitsi-web` (documentation): add a markdown doc describing the client-side
  conference entry point. Reason: holds the code that initiates the conference join.
- `jitsi-control` (documentation): add a markdown doc describing the focus-service
  entry point. Reason: holds the code that receives and processes the client's
  conference request.
- `jitsi-videobridge` (no-change): verified during Intake that the join/entry path
  does not reach videobridge code; media routing is documented separately by
  `document-media-routing-boundaries`.
- No affected APIs, dependencies, runtime configuration, or tests in any repository.
- Evidence that these entry points exist and are reachable was collected via
  CodeGraph (`openspec-orch plugin exec codegraph explore` / `query`) against the
  indexed `jitsi-web` and `jitsi-control` checkouts; exact queries and code anchors
  are recorded with provenance in `intake.md` §9, not restated here (Repository
  symbol/file:line detail belongs to the Repository, not this Store proposal).
