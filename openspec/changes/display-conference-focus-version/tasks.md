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

## 2. `lib-jitsi-meet`

- [ ] 2.1 Parse the optional Focus-version property from the
      conference-allocation response and expose it as a property of the
      current conference, following the existing named-property parsing
      pattern; absence of the property must not raise an error or change
      any other parsed value. Required result: when the response contains
      the property, the current conference's exposed properties include
      it under a stable key; when absent, that key is simply not present.
      Traces to: `conference/focus-version-visibility` — both
      SC-FOCUS-VERSION-001 and SC-FOCUS-VERSION-002. Verify by: repository's
      local unit test suite covering conference-allocation response
      parsing, with cases for property present and property absent.
- [ ] 2.2 Confirm the value is reachable through the conference's existing
      property-read API without adding a new storage mechanism (in-memory
      only, per Proposal Non-Goals) and without modifying
      `_updateProperties`/`getProperty`, which are already key-agnostic.
      Verify by: code inspection confirming `_updateProperties` assigns
      whatever keys `_parseConferenceIq` produces (including
      `focus-version` per 2.1) into `this.properties` unchanged, and that
      `getProperty(key)` reads back from that same map with no key-specific
      branching — no new test is needed for this generic pass-through
      (`JitsiConference.spec.ts` has no existing properties-update pattern
      to extend for this, and none is required since no code changes).
- [ ] 2.3 Cross-repository contract check: verify `lib-jitsi-meet`'s parser
      accepts the exact property name and value format emitted by
      `jitsi-control` in task 1.1 (shared fixture or contract test using a
      sample conference-allocation response payload). Evidence owner:
      `lib-jitsi-meet` (it owns the parsing contract test). Traces to:
      `conference/focus-version-visibility` — SC-FOCUS-VERSION-001. Verify
      by: a contract-level test asserting `lib-jitsi-meet`'s parser, given
      a fixture payload shaped like `jitsi-control`'s task 1.1 output,
      produces the expected exposed property.

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

No application run is included in or required by these Planning tasks.
`jitsi-control` and `lib-jitsi-meet` verification is each repository's
local test suite (its existing runner/pattern) plus the cross-repository
contract check owned by `lib-jitsi-meet` (2.3). `jitsi-web` has no
component/unit test harness for `react/features`, so its tasks (3.1-3.4)
are verified now by this repository's existing static checks, with manual
UI confirmation — including the `jitsi-web`-owned contract check (3.4) —
deferred to Verify; no application run happens during Planning or Apply.
