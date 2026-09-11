## 1. `jitsi-control`

- [ ] 1.1 Add an optional `focus-region` property to the conference creation
      response, populated with the configured region only when it is
      non-empty (Scenario [SC-CONFERENCE-REGION-DIAGNOSTICS-001]). Verify: an
      automated test asserts the response contains `focus-region` equal to
      the configured value when a non-empty region is configured.
- [ ] 1.2 Ensure the property is omitted entirely (not sent as an empty
      string) when no region is configured or the configured value is empty
      (Scenario [SC-CONFERENCE-REGION-DIAGNOSTICS-002]). Verify: an automated
      test asserts the response does not contain the `focus-region` property
      in that case.

## 2. `jitsi-web`

- [ ] 2.1 Adapt the client's lib-jitsi-meet integration to read
      `focus-region` from the initial conference creation response and
      expose the parsed value (or its absence) to the UI layer (underlies
      Scenario [SC-CONFERENCE-REGION-DIAGNOSTICS-003] and
      [SC-CONFERENCE-REGION-DIAGNOSTICS-004]). Verify: a unit test confirms the
      exposed value for both a populated and an omitted response property.
- [ ] 2.2 Capture the region value once from the initial conference creation
      response only; do not re-read or update it on later events such as a
      focus migration (Scenario [SC-CONFERENCE-REGION-DIAGNOSTICS-005]).
      Verify: a test confirms that a later conference-info update does not
      change the previously captured region value.
- [ ] 2.3 Render a new localized "Conference region: <value>" item in the
      existing web conference info bar when a non-empty value was parsed
      (Scenario [SC-CONFERENCE-REGION-DIAGNOSTICS-003]). Verify: a component
      test asserts the item renders with the localized caption and the
      value shown verbatim as plain, non-interactive text.
- [ ] 2.4 Hide the item, without surfacing an error to the participant, when
      the value is absent, empty, or could not be parsed (Scenario
      [SC-CONFERENCE-REGION-DIAGNOSTICS-004]). Verify: a component test
      asserts no region item and no error UI is rendered in each of the
      three cases.
- [ ] 2.5 Add the new label's caption string as a key in the client's
      existing localization resources, sourced through the existing
      localization system rather than hardcoded (Scenario
      [SC-CONFERENCE-REGION-DIAGNOSTICS-006]). Verify: a test or existing
      localization-resource check confirms the new key resolves through the
      client's localization system.
- [ ] 2.6 Confirm the change is scoped to the web conference info bar only,
      with no changes to mobile client code paths (Scenario
      [SC-CONFERENCE-REGION-DIAGNOSTICS-007]). Verify: the PR diff touches no
      mobile-client files, and the existing mobile test suite remains
      unaffected (still passing).

## 3. Cross-repository verification

Owner: `jitsi-web` (sole consumer of the new wire contract; responsible for
end-to-end evidence spanning both repositories).

- [ ] 3.1 Prepare end-to-end evidence for PR Review, IFT and QA covering both
      the "region configured on the focus service" and "region not
      configured" cases on a shared Snapshot of `jitsi-control` and
      `jitsi-web` (Scenario [SC-CONFERENCE-REGION-DIAGNOSTICS-001],
      [SC-CONFERENCE-REGION-DIAGNOSTICS-002], [SC-CONFERENCE-REGION-DIAGNOSTICS-003],
      [SC-CONFERENCE-REGION-DIAGNOSTICS-004]). Do not mark this task done until
      the evidence has actually been produced from a real run; preparing the
      plan is not the same as completing the verification.
- [ ] 3.2 Record the observable success signals and rollback condition for
      this user-facing scenario before Gate 3, per the project's release
      process. Do not mark this task done ahead of that confirmation.
