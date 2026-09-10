## Context

Both affected repositories already keep a `doc/` folder with topic-scoped markdown
files (`jitsi-web/doc/api.md`, `jitsi-web/doc/quick-install.md`;
`jitsi-control/doc/conference-request.md`, `jitsi-control/doc/health-checks.md`,
`jitsi-control/doc/reservation.md`). `jitsi-control/doc/conference-request.md`
already documents the conference-request **wire protocol** (HTTP and XMPP payload
shapes) but does not name the code that sends or receives it. See proposal.md - Why
for the underlying motivation; no restatement here.

Planning-time evidence that a client-side join entry point and a matching
focus-service receiving entry point actually exist and are reachable was confirmed
via CodeGraph in this session; the exact anchors used for that confirmation are
recorded in `intake.md` §9 as evidence with provenance, not restated here as a
normative Design decision — this document specifies the observable role each
documented entry point must cover, not its current internal symbol or line number.

## Goals / Non-Goals

**Goals:**
- Document the client-side entry point in `jitsi-web` that starts a conference join
  and initiates the signaling exchange with the focus service.
- Document the entry point in `jitsi-control` that receives that request and the
  step that creates or looks up the resulting conference.
- Cross-link the new docs to each other and to the existing
  `jitsi-control/doc/conference-request.md` protocol reference, instead of duplicating
  its wire-format content.

**Non-Goals:**
- Documenting the media path to `jitsi-videobridge` (out of scope per Proposal;
  covered by the separate `document-media-routing-boundaries` change).
- Documenting the internal lib-jitsi-meet implementation of the join call — it is an
  external dependency of `jitsi-web`, not part of this repository's indexed source.
- Any change to code, configuration, or tests.

## Decisions

- **New dedicated file per repository, not an edit to `conference-request.md`.**
  `conference-request.md` documents the wire protocol; this change documents which
  code sends/receives it. Mixing the two would make the protocol doc harder to keep
  stable. Alternative considered: append a "code entry points" section to
  `conference-request.md` directly — rejected because `jitsi-web` has no equivalent
  protocol doc to extend, so the two repositories would end up documenting the same
  path inconsistently.
- **File paths:** `jitsi-web/doc/conference-entrypoint.md` and
  `jitsi-control/doc/conference-entrypoint.md`, matching each repository's existing
  `doc/<topic>.md` convention.
- **Exact symbol and file:line anchors are Apply-time, repository-owned content, not
  a Store-level decision.** The published doc in each repository must cite the
  current entry-point function/class with its file:line, but that citation is
  confirmed and written during Apply directly in the Code Repository (re-verified
  against current source, since it is exactly the kind of implementation detail that
  belongs to the Repository, not this Store). Design only fixes the observable role
  each doc must cover and the failure branch it must describe (the client's handling
  of an unavailable focus service), both already confirmed reachable per `intake.md`
  §9.

## Repository Implementation Map

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-web` | Add `doc/conference-entrypoint.md` documenting the client-side function that starts a conference join and triggers the signaling exchange with the focus service, and the failure branch triggered when the focus service is unavailable. Cite the current entry point's file:line, confirmed against source at Apply time. Link to `jitsi-control/doc/conference-entrypoint.md`. | No code contract change. Documentation-only, so no dependency on other repositories' release state. |
| `jitsi-control` | Add `doc/conference-entrypoint.md` documenting the XMPP entry point that receives the client's initial conference request and the step that creates or looks up the resulting conference. Cite the current entry point's file:line, confirmed against source at Apply time. Link to the existing `doc/conference-request.md` protocol reference and to `jitsi-web/doc/conference-entrypoint.md`. | No code contract change. References the existing conference-request protocol types without altering them. |
| `jitsi-videobridge` | No implementation task. | Confirmed no-change in Proposal - Impact; not represented in this map beyond that confirmation. |

## Risks / Trade-offs

- [Referenced code moves or is refactored, silently making the new docs stale] →
  Mitigation: each doc cites an exact file:line anchor, confirmed at Apply time and
  re-checkable on every later review, so staleness is easy to detect; no automated
  drift-detection is introduced by this change (out of scope).
- [Two separate files per repository could drift out of sync with each other] →
  Mitigation: each doc explicitly cross-links the other and states which side of the
  boundary it owns, keeping duplication to a one-line summary of the other side.
