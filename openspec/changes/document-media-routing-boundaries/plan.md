# Implementation Plan

> **Execution:** Для реализации этого OpenSpec Change (`document-media-routing-boundaries`)
> вызови установленный штатный OpenSpec Apply (`/opsx:apply` или `/opsx-apply`) и следуй
> актуальным инструкциям схемы `superspec-multirepo`. Superpowers executor запускается
> внутри Apply по этим инструкциям (`superpowers:using-superpowers`,
> `superpowers:using-git-worktrees`, `superpowers:subagent-driven-development` или
> `superpowers:executing-plans` как fallback). Создание этого Plan не запускает реализацию.

**Goal:** Publish two cross-linked, evidence-backed developer docs describing the
bridge-selection and colibri2 control-request boundary between `jitsi-control` and
`jitsi-videobridge`, with no runtime behavior change.

**Accepted inputs:** `proposal.md` (Why, What Changes, Impact), `design.md` (Context
evidence, Decisions, Repository Implementation Map), `tasks.md` (coarse tasks 1.1, 1.2,
2.1, 2.2, 3.1). `skip_specs: true` — no Delta Specs to trace.

## Repository: `jitsi-control`

**Repository result:** `doc/bridge-control-boundary.md` exists, cites the current
`BridgeSelector.selectBridge` and `ColibriV2SessionManager` / `Colibri2Session`
anchors, and cross-links `jitsi-videobridge/doc/bridge-control-boundary.md` and the
existing `doc/conference-request.md`.

**Depends on:** none (documentation-only; no shared state with `jitsi-videobridge`
during authoring — cross-links are added once both files exist, see Cross-repository
convergence).

### Task 1: Confirm current bridge-selection anchor

**Traces to:** Task 1.1; `design.md` - Context (BridgeSelector evidence).

- [ ] Step 1: Read `jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/BridgeSelector.kt`
  around the Planning-time anchor (`selectBridge`, line ~161) and confirm it still exists
  at that location and signature.
- [ ] Step 2: Read `jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java`
  around the Planning-time anchor (constructor field `bridgeSelector`, line ~317/336) and
  confirm the current file:line. Expected result: an exact, current file:line for both
  anchors, matching or superseding the Planning-time citation in `design.md`.

### Task 2: Confirm colibri2 outbound control-session anchor

**Traces to:** Task 1.1; `design.md` - Context (ColibriV2SessionManager / Colibri2Session
evidence).

- [ ] Step 1: Read `jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/colibri/Colibri2Session.kt`
  around the Planning-time anchor (line ~64) and confirm the current class responsible for
  sending the conference create/modify control request to the selected bridge.
- [ ] Step 2: Read `jicofo-selector/src/main/kotlin/org/jitsi/jicofo/bridge/colibri/ColibriV2SessionManager.kt`
  and confirm it is the caller of `BridgeSelector.selectBridge`. Expected result: an exact,
  current file:line for both anchors.

### Task 3: Write and cross-link the `jitsi-control` doc

**Traces to:** Tasks 1.1, 1.2.

- [ ] Step 1: Write `jitsi-control/doc/bridge-control-boundary.md` describing (a) the
  bridge-selection step with its confirmed anchor, (b) the outbound colibri2 control
  request with its confirmed anchor, and (c) a one-line summary of the receiving side
  (full detail owned by the `jitsi-videobridge` doc).
- [ ] Step 2: Add a link to `../../jitsi-videobridge/doc/bridge-control-boundary.md`
  (or the repository's actual cross-repo doc-link convention) and a link to the existing
  `doc/conference-request.md`. Verify: render the markdown file locally (e.g.
  `markdownlint doc/bridge-control-boundary.md` or the repository's existing doc-lint
  command, if any) and confirm no duplicated wire-format content versus
  `conference-request.md` by reading both side by side. Expected result: valid markdown,
  no duplication, both links present.

## Repository: `jitsi-videobridge`

**Repository result:** `doc/bridge-control-boundary.md` exists, cites the current
`Conference` / `ColibriQueue` / `Colibri2ConferenceHandler.handleConferenceModifyIQ`
anchors, and cross-links `jitsi-control/doc/bridge-control-boundary.md` and the existing
`doc/rest-colibri2.md`.

**Depends on:** none for authoring (documentation-only); the cross-link target
(`jitsi-control/doc/bridge-control-boundary.md`) is finalized in Cross-repository
convergence below.

### Task 1: Confirm current colibri2 receiving-path anchor

**Traces to:** Task 2.1; `design.md` - Context (Conference / ColibriQueue /
Colibri2ConferenceHandler evidence).

- [ ] Step 1: Read `jvb/src/main/java/org/jitsi/videobridge/Conference.java` around the
  Planning-time anchor (constructor, `Colibri2ConferenceHandler` at line ~324,
  `ColibriQueue` request handler at line ~325-351, dispatch to
  `colibri2Handler.handleConferenceModifyIQ` at line ~349) and confirm the current
  file:line for the construction and the dispatch call. Expected result: an exact,
  current file:line for both, matching or superseding the Planning-time citation in
  `design.md`.

### Task 2: Write and cross-link the `jitsi-videobridge` doc

**Traces to:** Tasks 2.1, 2.2.

- [ ] Step 1: Write `jitsi-videobridge/doc/bridge-control-boundary.md` describing the
  colibri2 request-handling path with its confirmed anchor, and a one-line summary of the
  selecting side (full detail owned by the `jitsi-control` doc).
- [ ] Step 2: Add a link to the existing `doc/rest-colibri2.md` and a link to
  `../../jitsi-control/doc/bridge-control-boundary.md` (or the repository's actual
  cross-repo doc-link convention). Verify: render the markdown file locally and confirm
  no duplicated wire-format content versus `rest-colibri2.md` by reading both side by
  side. Expected result: valid markdown, no duplication, both links present.

## Cross-repository convergence

- Ordering: the two repositories' Task 1 (anchor confirmation) and doc-writing steps have
  no shared state and may run independently (`superpowers:dispatching-parallel-agents`
  permitted per `openspec-orch.yaml` scope). Finalize the exact cross-link path convention
  (relative path vs. full remote URL) by checking each repository's own existing
  cross-repo-doc-link pattern, if any, before writing Step 2 in either repository, so both
  links use the same convention.
- Compatibility check: after both docs exist, re-read both files together (Task 3 / Task 2
  Step 2 in each repository above already includes this) and confirm each doc's one-line
  summary of "the other side" is still accurate and neither doc duplicates the other's
  wire-format or wording beyond that summary.
- Evidence handoff: record the final confirmed file:line anchors used in each published
  doc as part of Task 3.1 (PR review evidence) in `tasks.md`; do not open the PRs or
  request review until the docs are actually written and rendered.
- This Plan does not cover `jitsi-web`: confirmed no-change in `proposal.md` - Impact.
