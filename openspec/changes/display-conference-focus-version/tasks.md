## 1. `jitsi-control`

- [ ] 1.1 Add a `focus-version` entry to the existing `ConferenceProperties`
      object that `JitsiMeetConferenceImpl` builds and publishes in the
      focus presence, following the same pattern used for that object's
      other entries. Remove the `focus-version` property from the
      conference-allocation success response. Required result: the focus
      presence carries a `focus-version` conference property when the
      conference is created/joined; the conference-allocation success
      response carries no `focus-version` property; no other entry on
      `ConferenceProperties`, other presence content, or other field on
      the conference-allocation response changes. Traces to:
      `conference/focus-version-visibility` — Requirement "Conference
      details UI shows the serving Focus version when known"
      (SC-FOCUS-VERSION-001), Requirement "Conference details UI omits the
      Focus version row when the value is unknown" (SC-FOCUS-VERSION-002).
      Verify by: repository's local unit/integration test suite covering
      `JitsiMeetConferenceImpl`'s `ConferenceProperties`
      construction/publication, asserting the new entry is present with the
      expected value; and the local test suite for `ConferenceIqHandler` not
      asserting a `focus-version` property on the conference-allocation
      response.
- [ ] 1.2 Confirm no other conference-allocation response behavior changes
      (existing properties, `ready`/`focusJid`/error handling unaffected)
      and no other entry on `ConferenceProperties` or other presence content
      changes. Verify by: repository's existing local test suite for the
      conference-allocation request/response path passes unmodified in
      assertions unrelated to this property, and the existing local test
      suite for focus-presence/`ConferenceProperties` content passes
      unmodified for entries other than `focus-version`.

## 2. `lib-jitsi-meet`

- [ ] 2.1 Add a regression test for the existing generic MUC presence
      `conference-properties` pass-through in `ChatRoom` and
      `JitsiConference`, which already carries arbitrary presence-sourced
      properties into `JitsiConference.properties` with no per-key
      allow-list; no parser change is required. Required result: a test
      demonstrates that when the focus presence carries a `focus-version`
      conference property (per jitsi-control task 1.1),
      `JitsiConference.getProperty('focus-version')` returns it; when the
      presence carries no such property, it is `undefined`. Traces to:
      `conference/focus-version-visibility` — both SC-FOCUS-VERSION-001 and
      SC-FOCUS-VERSION-002. Verify by: repository's local unit test suite
      covering presence-driven `JitsiConference.properties` updates, with
      cases for the property present and absent in presence.
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
      (task 1.1) via a shared fixture or contract test using a sample focus
      presence payload. Evidence owner: `lib-jitsi-meet` (it owns this
      contract test). Traces to: `conference/focus-version-visibility` —
      SC-FOCUS-VERSION-001. Verify by: a contract-level test asserting
      that, given a fixture presence payload shaped like `jitsi-control`'s
      task 1.1 output, `JitsiConference.properties` ends up with the
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

`jitsi-control` and `lib-jitsi-meet` verification is each repository's local
test suite (its existing runner/pattern) plus the cross-repository contract
check owned by `lib-jitsi-meet` (2.3). `jitsi-web` has no component/unit
test harness for `react/features`, so its tasks (3.1-3.4) are verified now
by this repository's existing static checks, with manual UI confirmation —
including the `jitsi-web`-owned contract check (3.4) — deferred to Verify.
No application run happens during Planning or Apply.
