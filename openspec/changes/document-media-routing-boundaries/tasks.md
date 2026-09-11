## 1. `jitsi-control`

- [ ] 1.1 Add `doc/bridge-control-boundary.md` documenting the bridge-selection
  step (`BridgeSelector.selectBridge`) and the outbound colibri2 control
  request sent via `ColibriV2SessionManager` / `Colibri2Session` to the
  selected bridge. Confirm the current entry point location by reading the
  relevant source at Apply time (Planning-time confirmation recorded in
  `design.md` - Context, but the published anchor must reflect current
  source) and cite its file:line in the doc; verify by confirming the cited
  anchor matches current source and the doc renders as valid markdown.
- [ ] 1.2 Cross-link `doc/bridge-control-boundary.md` to
  `jitsi-videobridge/doc/bridge-control-boundary.md` and to the existing
  `doc/conference-request.md`, without duplicating wire-format content already
  documented there; verify by reading all three docs together and confirming
  no content is duplicated.

## 2. `jitsi-videobridge`

- [ ] 2.1 Add `doc/bridge-control-boundary.md` documenting the colibri2
  request-handling path in `Conference` (`ColibriQueue` →
  `Colibri2ConferenceHandler.handleConferenceModifyIQ`) that receives and
  processes the control request from `jitsi-control`. Confirm the current
  entry point location by reading the relevant source at Apply time
  (Planning-time confirmation recorded in `design.md` - Context, but the
  published anchor must reflect current source) and cite its file:line in the
  doc; verify by confirming the cited anchor matches current source and the
  doc renders as valid markdown.
- [ ] 2.2 Cross-link `doc/bridge-control-boundary.md` to the existing
  `doc/rest-colibri2.md` protocol reference and to
  `jitsi-control/doc/bridge-control-boundary.md`, without duplicating
  wire-format content already documented there; verify by reading both docs
  together and confirming no content is duplicated.

## 3. Cross-repository review

- [ ] 3.1 Prepare PR review evidence for both docs: open a PR in each of
  `jitsi-control` and `jitsi-videobridge` referencing this Change
  (`document-media-routing-boundaries`), and request review confirming the
  documented boundary and cited file:line anchors are accurate; owner: primary
  solution owner is whoever authors the doc PR in each repository (no single
  cross-repo owner is designated, since both sides are documentation-only and
  independently reviewable). Do not mark this task complete until the PRs
  actually exist and review is actually requested — do not record an external
  review as done in advance.
