> **Correction note:** revised after a confirmed Apply-gap — the transport
> accepted when 1.1/1.2 were implemented (conference-allocation IQ
> response) does not reach `JitsiConference.properties` (see design.md
> Context, Correction). 1.1 and 1.2 are kept below unchanged as the
> historical record of what was actually implemented; 1.3/1.4 are the
> follow-up correction. 2.1/2.2 are redirected from a `moderator.js` parser
> change to a regression test of the existing generic presence
> pass-through. No application was run to produce this correction.

## 1. `jitsi-control`

- [x] 1.1 Attach the existing public Focus version string as one optional
      property on the conference-allocation success response, following the
      same optional-property pattern already used for other properties on
      that response. Required result: the response carries the property
      when the conference is created/joined; no other field on the response
      changes. Traces to: `conference/focus-version-visibility` —
      Requirement "Conference details UI shows the serving Focus version
      when known" (SC-FOCUS-VERSION-001), Requirement "Conference details
      UI omits the Focus version row when the value is unknown"
      (SC-FOCUS-VERSION-002). Verify by: repository's local unit/integration
      test suite covering the conference-allocation response builder,
      asserting the new property is present with the expected value.
- [x] 1.2 Confirm no other conference-allocation response behavior changes
      (existing properties, `ready`/`focusJid`/error handling unaffected).
      Verify by: repository's existing local test suite for the
      conference-allocation request/response path passes unmodified in
      assertions unrelated to this property.
- [x] 1.3 Follow-up correction: remove the `focus-version` property added
      to the conference-allocation success response in 1.1 — this transport
      does not reach `JitsiConference.properties` (see design.md Context,
      Correction). Required result: the conference-allocation response no
      longer carries `focus-version`; no other property on that response
      changes. Traces to: `conference/focus-version-visibility` (corrected
      transport). Verify by: repository's local test suite for
      `ConferenceIqHandler` no longer asserting the property, and the
      class's existing assertions otherwise unaffected.
- [ ] 1.4 Add the already-public Focus version string as one entry on the
      existing `ConferenceProperties` object that `JitsiMeetConferenceImpl`
      already builds and publishes in the focus presence, following the
      same pattern already used for that object's other entries. Required
      result: the focus presence carries a `focus-version` conference
      property when the conference is created/joined; no other entry on
      `ConferenceProperties` or other presence content changes. Traces to:
      `conference/focus-version-visibility` — SC-FOCUS-VERSION-001,
      SC-FOCUS-VERSION-002. Verify by: repository's local unit/integration
      test suite covering `JitsiMeetConferenceImpl`'s `ConferenceProperties`
      construction/publication, asserting the new entry is present with the
      expected value.

## 2. `lib-jitsi-meet`

- [ ] 2.1 Correction: no `moderator.js` parser change. Confirm, and add a
      regression test for, the existing generic MUC presence
      `conference-properties` pass-through in `ChatRoom` and
      `JitsiConference`, which already carries arbitrary
      presence-sourced properties into `JitsiConference.properties` with no
      per-key allow-list. Required result: a test demonstrates that when the
      focus presence carries a `focus-version` conference property (per
      jitsi-control task 1.4), `JitsiConference.getProperty('focus-version')`
      returns it; when the presence carries no such property, it is
      `undefined`. This supersedes the original 2.1, which targeted parsing
      the conference-allocation response in `moderator.js` — that transport
      does not reach `JitsiConference.properties` (see design.md Context,
      Correction), and any in-progress local change made against it should
      not be carried forward. Traces to: `conference/focus-version-visibility`
      — both SC-FOCUS-VERSION-001 and SC-FOCUS-VERSION-002. Verify by:
      repository's local unit test suite covering presence-driven
      `JitsiConference.properties` updates, with cases for the property
      present and absent in presence.
- [ ] 2.2 Confirm no new storage mechanism was introduced (in-memory only,
      per Proposal Non-Goals) and that `_updateProperties`/`getProperty`
      remain unchanged and key-agnostic. Verify by: code inspection
      confirming `_updateProperties` assigns whatever keys the presence
      `conference-properties` element carries (including `focus-version`
      per 2.1) into `this.properties` unchanged, and that `getProperty(key)`
      reads back from that same map with no key-specific branching — no
      production code change is needed for this task, only the regression
      test in 2.1.
- [ ] 2.3 Cross-repository contract check: verify `lib-jitsi-meet`'s
      presence-driven pass-through accepts the exact property name and
      value format `jitsi-control` publishes in `ConferenceProperties`
      (task 1.4) via a shared fixture or contract test using a sample focus
      presence payload. Evidence owner: `lib-jitsi-meet` (it owns this
      contract test). Traces to: `conference/focus-version-visibility` —
      SC-FOCUS-VERSION-001. Verify by: a contract-level test asserting
      that, given a fixture presence payload shaped like `jitsi-control`'s
      task 1.4 output, `JitsiConference.properties` ends up with the
      expected `focus-version` entry.

## 3. `jitsi-web`

- [ ] 3.1 Add a "Focus version" row to the conference details UI, visible
      to all participants (no moderator/support restriction), reading the
      value exposed by `lib-jitsi-meet`, rendered via a new translation key
      `info.focusVersion` (`"Focus version: {{version}}"`) added to
      `lang/main.json`, and register the new component's id in the default
      `autoHide` array in
      `react/features/conference/components/constants.ts` (the array
      `getConferenceInfo` in `functions.any.ts` falls back to when
      `config.conferenceInfo` does not override it) — without this
      registration, `ConferenceInfo` never renders the new component by
      default, regardless of the value. Required result: the row renders
      with the value when present, using only the default configuration.
      Traces to: `conference/focus-version-visibility`
      — SC-FOCUS-VERSION-001, "Conference details UI shows the serving
      Focus version is available for all participants" (SC-FOCUS-VERSION-003).
      Verify by: this repository's existing static checks (type-check,
      lint, and `npm run lint:lang` for the new `lang/main.json` key)
      passing on the new/changed files; manual confirmation in the running
      app is deferred to Verify (no component/unit test harness exists for
      `react/features` in this repository — `npm test` here is WebdriverIO
      end-to-end, out of scope for this narrow change).
- [ ] 3.2 Ensure the row is entirely omitted — no row, no placeholder text,
      no error state — when the value is absent. Required result: rendering
      the conference details UI with no Focus-version property produces no
      trace of the row (no empty label, no "unknown" text). Traces to:
      `conference/focus-version-visibility` — SC-FOCUS-VERSION-002. Verify
      by: code review confirming the component returns `null` when
      `getProperty('focus-version')` is `undefined`, following the same
      "render nothing when absent" pattern as sibling labels; manual
      confirmation in the running app is deferred to Verify (see 3.1 for
      why no automated component test is used).
- [ ] 3.3 Confirm the row is not gated by moderator role or any other
      participant-role check. Verify by: code review confirming the new
      component reads only `getProperty('focus-version')` with no
      role/permission check; manual confirmation in the running app for a
      non-moderator participant is deferred to Verify (see 3.1).
- [ ] 3.4 Cross-repository contract check: verify `jitsi-web`'s read of the
      Focus-version property matches the exact key `lib-jitsi-meet` exposes
      in task 2.1/2.2 (present and absent cases). Evidence owner:
      `jitsi-web` (it owns the UI-facing consumption contract check).
      Traces to: `conference/focus-version-visibility` —
      SC-FOCUS-VERSION-001, SC-FOCUS-VERSION-002. Verify by: code review
      confirming the literal key string `'focus-version'` passed to
      `getProperty` in the new component matches the key `lib-jitsi-meet`
      exposes per task 2.1; manual confirmation in the running app
      (present and absent cases) is deferred to Verify — no component/unit
      test harness exists for this area to automate the check (see 3.1).

No application run is included in or required by these Planning tasks,
including this correction. `jitsi-control` and `lib-jitsi-meet`
verification is each repository's local test suite (its existing
runner/pattern) plus the cross-repository contract check owned by
`lib-jitsi-meet` (2.3). `jitsi-web` has no component/unit test harness for
`react/features`, so its tasks (3.1-3.4) are verified now by this
repository's existing static checks, with manual UI confirmation —
including the `jitsi-web`-owned contract check (3.4) — deferred to Verify.
1.1/1.2 are kept as the historical record of what was implemented under the
originally accepted (now superseded) transport; 1.3/1.4 are the follow-up
correction, and 2.1/2.2 have been redirected from a `moderator.js` parser
change to a regression test of the existing generic presence pass-through.
No application run happens during Planning or Apply.
