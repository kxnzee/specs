## 1. `jitsi-web`

- [x] 1.1 Add `doc/conference-entrypoint.md` documenting the client-side function that starts a conference join and triggers the signaling exchange with the focus service, and the failure branch triggered when the focus service is unavailable. Confirm the current entry point and failure-branch location by reading `conference.js` at Apply time (Planning-time confirmation is recorded in `intake.md` §9 for reference, but the published anchor must reflect current source) and cite its file:line in the doc; verify by confirming the cited anchor matches current source and the doc renders as valid markdown.
- [x] 1.2 Link `doc/conference-entrypoint.md` to `jitsi-control/doc/conference-entrypoint.md`; verify the link target path is correct relative to the published doc locations agreed with the `jitsi-control` maintainer.

## 2. `jitsi-control`

- [x] 2.1 Add `doc/conference-entrypoint.md` documenting the XMPP entry point that receives the client's initial conference request and the step that creates or looks up the resulting conference. Confirm the current entry point location by reading the relevant source at Apply time (Planning-time confirmation is recorded in `intake.md` §9 for reference, but the published anchor must reflect current source) and cite its file:line in the doc; verify by confirming the cited anchor matches current source and the doc renders as valid markdown.
- [x] 2.2 Cross-link `doc/conference-entrypoint.md` to the existing `doc/conference-request.md` protocol reference and to `jitsi-web/doc/conference-entrypoint.md`, without duplicating the wire-format content already in `conference-request.md`; verify by reading both docs together and confirming no content is duplicated.

## 3. Cross-repository review

- [x] 3.1 Prepare PR review evidence for both docs: open a PR in each of `jitsi-web` and `jitsi-control` referencing this Change (`document-conference-entrypoints`), and request review confirming the documented entry points and cited file:line anchors are accurate; owner: primary solution owner is whoever authors the doc PR in each repository (no single cross-repo owner is designated, since both sides are documentation-only and independently reviewable). Do not mark this task complete until the PRs actually exist and review is actually requested — do not record an external review as done in advance.
