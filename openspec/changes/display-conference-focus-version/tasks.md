# Implementation Tasks

> **Execution:** Для реализации этого OpenSpec Change вызови установленный
> штатный OpenSpec Apply и следуй актуальным инструкциям схемы. `tasks.md` —
> единственный принятый план; Superpowers executor запускается внутри Apply.
> Создание Tasks не запускает реализацию.

**Change:** `display-conference-focus-version` — surface the Focus
(`jitsi-control`) version serving the current conference in `jitsi-web`'s
conference details UI, for all participants, only when the value is present.

**Accepted inputs:**
- `proposal.md` (scope, optionality, non-goals)
- `specs/conference/focus-version-visibility/spec.md`
  (SC-FOCUS-VERSION-001/002/003)
- `design.md` (Decisions and alternatives, Repository Implementation Map)

**Requirements referenced below:**
- R1 "Conference details UI shows the serving Focus version when known" —
  SC-FOCUS-VERSION-001
- R2 "Conference details UI omits the Focus version row when the value is
  unknown" — SC-FOCUS-VERSION-002
- R3 "Focus version visibility is available to all participants" —
  SC-FOCUS-VERSION-003

**Repository order:** `jitsi-control` → `lib-jitsi-meet` → `jitsi-web`, matching
the value's path through the system. Deploy order is not constrained: each hop
is absent-tolerant, so any ordering reproduces today's behavior (no row) until
all three are live. `jitsi-videobridge` is out of scope; the Store holds no
implementation. Every file anchor below is a current-state anchor to re-confirm
as the first action of the task that uses it.

## 1. `jitsi-control`

**Repository result:** The focus presence published by `JitsiMeetConferenceImpl`
carries `focus-version` as one entry of its existing `ConferenceProperties`,
with the value `CurrentVersionImpl.VERSION.toString()`. The
conference-allocation success response carries no `focus-version` property. No
other `ConferenceProperties` entry, other presence content, or other
conference-allocation response field changes.

**Depends on:** none — this repository produces the value the other two consume.

- [ ] 1.1 Publish `focus-version` in the focus-presence `ConferenceProperties`,
      and keep it off the conference-allocation success response. Required
      result: the focus presence carries a `focus-version` conference property
      equal to `CurrentVersionImpl.VERSION.toString()` when the conference is
      created/joined, following the pattern already used for that object's
      other entries; the conference-allocation success response carries no
      `focus-version` property.
  - **Traces to:** R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002.
  - **Files or anchors:**
    - Modify: `jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java`
      — `joinTheRoom()` builds the initial focus presence extensions and adds
      the result of the `createConferenceProperties()` helper defined in the
      same class.
    - Value source: `jicofo/src/main/java/org/jitsi/jicofo/version/CurrentVersionImpl.java`
      (`CurrentVersionImpl.VERSION`).
    - Modify if needed: `jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt`
      — the `response = ConferenceIq().apply { ... }` block in
      `doHandleConferenceIq`, which attaches optional string properties
      (`authentication`, `externalAuth`, `sipGatewayEnabled`,
      `visitors-supported`, `live`) to the allocation response.
    - Test: the repository's existing test classes covering
      `JitsiMeetConferenceImpl`'s `ConferenceProperties` publication and
      `ConferenceIqHandler`'s allocation response — confirm the exact class
      names at Apply time.
  - **Steps:**
    1. Locate the `ConferenceProperties` construction site in
       `JitsiMeetConferenceImpl` and its existing test coverage; note the
       pattern used for that object's other entries.
    2. Write a failing test asserting the published `ConferenceProperties`
       includes a `focus-version` entry equal to
       `CurrentVersionImpl.VERSION.toString()`, alongside its existing entries
       (so the same test proves no existing entry is removed or altered).
    3. Run that test and confirm it fails because the entry is not published.
    4. Add the `focus-version` entry to `ConferenceProperties` following the
       pattern from step 1, then re-run the test and confirm it passes.
    5. Inspect the `ConferenceIqHandler` allocation-response block and its test
       class. Keep the production response free of `focus-version`, and retain
       or add an explicit negative assertion proving the response omits it.
       Leave every other property and assertion untouched.
    6. Run the `ConferenceIqHandler` test class and confirm the negative
       `focus-version` assertion and all existing response assertions pass.
  - **Verification:**
    - Command: `mvn -pl jicofo -am -Dtest=ConferenceIqHandlerTest -Dsurefire.failIfNoSpecifiedTests=false test`
      (plus the same invocation for the `JitsiMeetConferenceImpl`
      presence/`ConferenceProperties` test class confirmed in step 1)
    - Expected: both runs pass; the presence test asserts the `focus-version`
      entry with the expected value alongside the pre-existing entries, and the
      allocation-response test explicitly asserts that the key is absent.
  - **Review checkpoint:** does the production diff touch only the
    `ConferenceProperties` construction/publication path, while the allocation
    response stays unchanged and its absence assertion remains explicit — with
    no allocation, bridge-selection or routing logic affected?

- [ ] 1.2 Regression-confirm that no other `jitsi-control` behavior changed.
      Required result: the existing conference-allocation request/response path
      and the existing focus-presence/`ConferenceProperties` content behave as
      before for everything other than the `focus-version` entry.
  - **Traces to:** R1/SC-FOCUS-VERSION-001 (no regression in the surfaces it
    extends); proposal non-goal "no change to conference allocation, bridge
    selection, or media routing".
  - **Depends on:** task 1.1 complete in this repository.
  - **Files or anchors:** no production change expected — the existing test
    suites for the conference-allocation request/response path and for
    focus-presence content, as identified in task 1.1.
  - **Steps:**
    1. Run the existing local test suite for the conference-allocation
       request/response path and confirm every assertion unrelated to
       `focus-version` passes unmodified.
    2. Run the existing local test suite for focus-presence /
       `ConferenceProperties` content and confirm every entry other than
       `focus-version` passes unmodified.
    3. Run the repository's CI-equivalent full verification as a regression
       check.
  - **Verification:**
    - Command: `mvn verify -B -Pcoverage`
    - Expected: build succeeds with no new failures relative to the branch
      point.
  - **Review checkpoint:** was any assertion outside the `focus-version` entry
    and the `focus-version` response property modified in order to make the
    suite pass?

## 2. `lib-jitsi-meet`

**Repository result:** `JitsiConference.getProperty('focus-version')` returns
the value `jitsi-control` publishes in the focus presence when present, and
`undefined` when it is not — via the existing generic MUC presence
`conference-properties` pass-through, with no parser change, no new storage, and
no other relayed property affected either way.

**Depends on:** `jitsi-control` task 1.1 — the property name `focus-version` and
its plain-string value published in the focus presence.

- [x] 2.1 Add regression coverage proving the existing generic presence
      pass-through carries `focus-version`. Required result: a test shows that
      when the focus presence carries a `focus-version` conference property,
      `JitsiConference.getProperty('focus-version')` returns it, and when the
      presence carries no such property the same call returns `undefined` — with
      no parser change.
  - **Traces to:** R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002.
  - **Files or anchors:**
    - Reference, no change expected: `modules/xmpp/ChatRoom.ts` — the handler
      that reads every property under `conference-properties` and emits
      `CONFERENCE_PROPERTIES_CHANGED`; `JitsiConference.ts` — the subscription
      binding that event to `_updateProperties`, plus `_updateProperties` and
      `getProperty` themselves.
    - Test: the test file covering presence-driven `JitsiConference.properties`
      updates — confirm the exact file (existing or new) at Apply time.
  - **Steps:**
    1. Locate existing test coverage, if any, for MUC presence
       `conference-properties` being relayed into `JitsiConference.properties`
       through `ChatRoom` → `_updateProperties`.
    2. Write the test: a fixture focus presence carrying a `focus-version`
       conference property, asserting `getProperty('focus-version')` returns its
       value; plus a case with no such property in presence, asserting the same
       call returns `undefined`.
    3. Run both cases against the unmodified production code and confirm they
       pass — `_updateProperties`/`getProperty` are already key-agnostic. If
       either case fails, that contradicts the accepted Design: stop and reopen
       Design rather than patching the parser.
  - **Verification:**
    - Command: `npm run test:native` (confirm the exact script covering the
      chosen test file in `package.json` at Apply time)
    - Expected: pass, including both the property-present and property-absent
      cases.
  - **Review checkpoint:** did any production code change in this repository for
    this task, and does the fixture presence payload use the exact key and value
    format `jitsi-control` publishes in task 1.1?

- [x] 2.2 Confirm the pass-through stays key-agnostic and storage-free. Required
      result: the `focus-version` value lives only in the existing in-memory
      conference `properties` map, with no key-specific branching and no new
      state container.
  - **Traces to:** R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002; proposal
    non-goal "no new storage, logging, or analytics for the value".
  - **Depends on:** task 2.1 (its regression test is the only artifact this
    task's conclusion rests on).
  - **Files or anchors:** `JitsiConference.ts` — `_updateProperties` and
    `getProperty`; no change expected in either.
  - **Steps:**
    1. Read `_updateProperties` and confirm it assigns whatever keys the
       presence `conference-properties` element carries — including
       `focus-version` — into `this.properties` unchanged.
    2. Read `getProperty` and confirm it reads back from that same map with no
       key-specific branching.
    3. Confirm no new storage mechanism was added: the value is held only in the
       existing in-memory `properties` map. No production or test change is
       needed for this task beyond task 2.1's regression test.
  - **Verification:**
    - Command: `git diff --stat -- modules/xmpp/ChatRoom.ts JitsiConference.ts`
    - Expected: empty output — neither file was modified by this change.
  - **Review checkpoint:** are `_updateProperties` and `getProperty` byte-for-byte
    unchanged, and was no new state container introduced anywhere in this
    repository's diff?

- [x] 2.3 Cross-repository contract check owned by this repository: prove the
      presence pass-through accepts the exact property name and value format
      `jitsi-control` publishes. Required result: given a fixture focus presence
      payload shaped like task 1.1's output, `JitsiConference.properties` ends up
      with the expected `focus-version` entry.
  - **Traces to:** R1/SC-FOCUS-VERSION-001.
  - **Depends on:** `jitsi-control` task 1.1 (the published payload shape) and
    task 2.1 (the test file the fixture lives beside).
  - **Files or anchors:** the presence-fixture test file confirmed in task 2.1,
    step 1; payload shaped like `jitsi-control`'s published
    `ConferenceProperties` — property name `focus-version`, value format
    `CurrentVersionImpl.VERSION.toString()` (for example `1.1-123-abc1234`).
  - **Steps:**
    1. Add the fixture presence payload next to the task 2.1 test, using the
       exact property name and value format `jitsi-control` publishes.
    2. Assert that after that presence is processed,
       `JitsiConference.properties` contains the expected `focus-version` entry.
    3. Run the suite and confirm the fixture case passes.
  - **Verification:**
    - Command: `npm run test:native` (same script confirmed in task 2.1)
    - Expected: pass, including the fixture contract case.
  - **Review checkpoint:** does the fixture reproduce `jitsi-control`'s actual
    published shape rather than a payload written to match the consumer?

## 3. `jitsi-web`

**Repository result:** The conference details header (`ConferenceInfo`) shows a
"Focus version" row to every participant, built from `lib-jitsi-meet`'s
`focus-version` conference property, and shows nothing — no row, no placeholder,
no error state — when that property is absent.

**Depends on:** `lib-jitsi-meet` tasks 2.1–2.2 — the exact property key carried
by the generic `JitsiConference.properties` update and emitted through the
existing properties-changed event consumed by `jitsi-web`.

**Test harness note (applies to 3.1–3.4):** this repository has no
component/unit test harness for `react/features` — no test file exists for
`ConferenceInfo.tsx` or its sibling label components, and `npm test` here is
WebdriverIO end-to-end. These tasks are therefore verified by the repository's
existing static checks plus code review; manual confirmation in the running app
belongs to Verify. No task in this plan starts, builds or exercises the running
application.

- [ ] 3.1 Add the "Focus version" row to the conference details UI. Required
      result: with only the default configuration, the row renders with the value
      for all participants, using a new translation key.
  - **Traces to:** R1/SC-FOCUS-VERSION-001, R3/SC-FOCUS-VERSION-003.
  - **Files or anchors:**
    - Create: `react/features/conference/components/web/FocusVersionLabel.tsx`
      — sibling to the label components `ConferenceInfo.tsx` already imports
      from this directory (`InsecureRoomNameLabel`, `RaisedHandsCountLabel`,
      `SpeakerStatsLabel`, `SubjectText`, `ToggleTopPanelLabel`); no `.web`
      suffix, matching this directory's local convention.
      Select the optional value from
      `state['features/base/conference'].properties`, the existing Redux map
      populated from `JitsiConferenceEvents.PROPERTIES_CHANGED`; do not call
      `getProperty` on the local `IJitsiConference` interface, which does not
      expose that method.
    - Modify: `react/features/conference/components/web/ConferenceInfo.tsx` —
      add `{ Component: FocusVersionLabel, id: 'focus-version' }` to the
      `COMPONENTS` array plus the matching import.
    - Modify: `react/features/conference/components/constants.ts` — register
      `'focus-version'` in the default `autoHide` array, the array
      `getConferenceInfo` in `functions.any.ts` falls back to when
      `config.conferenceInfo` does not override it. `autoHide`, not
      `alwaysVisible`, because the row must disappear when the value is absent.
      Without this registration `ConferenceInfo` never renders the component by
      default, regardless of the value.
    - Modify: `lang/main.json` — add `info.focusVersion` with value
      `"Focus version: {{version}}"`, alongside the existing `info.*` entries.
  - **Steps:**
    1. Create `FocusVersionLabel.tsx`: use `useSelector` with `IReduxState` to
       read `focus-version` from the existing
       `features/base/conference.properties` map. Guard the generic object with
       a property-membership and string-value check; do not add a new reducer,
       state field, selector module or `IJitsiConference.getProperty` typing.
    2. Render the value as a text row inside this directory's existing label
       presentation wrapper, matching the visual pattern of sibling labels such
       as `SpeakerStatsLabel`, using the `info.focusVersion` key. Add no
       moderator-role or other participant-role gating.
    3. Add `info.focusVersion` to `lang/main.json` with the value above.
    4. Add `FocusVersionLabel` to the `COMPONENTS` array in
       `ConferenceInfo.tsx` with `id: 'focus-version'` and import it alongside
       the other sibling-label imports.
    5. Register `'focus-version'` in the default `autoHide` array in
       `constants.ts`, alongside its existing entries.
  - **Verification:**
    - Command: `npm run lint:lang`, plus this repository's configured
      TypeScript type-check and ESLint scripts over the new and changed files
      (confirm the exact script names in `package.json` rather than assuming
      them)
    - Expected: no new lang-lint, type or lint errors on the new and changed
      files.
  - **Review checkpoint:** is `'focus-version'` present in the `constants.ts`
    default `autoHide` array and in `ConferenceInfo.tsx`'s `COMPONENTS` array
    with matching ids, and does the label render translated text from
    `info.focusVersion` rather than hard-coded copy?

- [ ] 3.2 Omit the row entirely when the value is absent. Required result:
      rendering the conference details UI with no `focus-version` property
      produces no trace of the row — no empty label, no placeholder, no "unknown"
      text, no error state.
  - **Traces to:** R2/SC-FOCUS-VERSION-002.
  - **Depends on:** task 3.1 (the component under review).
  - **Files or anchors:**
    `react/features/conference/components/web/FocusVersionLabel.tsx` — its early
    return; sibling labels in the same directory as the reference "render
    nothing when the underlying value is absent" pattern.
  - **Steps:**
    1. Confirm `FocusVersionLabel` returns `null` when the Redux properties map
       does not contain a non-empty string at `focus-version`, before any
       wrapper or text is rendered.
    2. Confirm the absent path emits no placeholder string, no empty label
       element and no error state — compare against a sibling label that already
       renders nothing when its value is absent.
    3. Confirm nothing outside the component compensates for the absent value
       (no default string injected at the `COMPONENTS` or translation layer).
  - **Verification:**
    - Command: `grep -n "return null" react/features/conference/components/web/FocusVersionLabel.tsx`
    - Expected: the early return exists and is reached whenever the property is
      `undefined`; no fallback/placeholder literal appears in the file. Manual
      confirmation in the running app belongs to Verify.
  - **Review checkpoint:** in the absent case, is there any rendered output at
    all from this component — including whitespace or an empty wrapper element?

- [ ] 3.3 Keep the row ungated by participant role. Required result: the row's
      visibility depends only on the presence of the value, with no moderator or
      other role/permission check anywhere on its path.
  - **Traces to:** R3/SC-FOCUS-VERSION-003.
  - **Depends on:** task 3.1 (the component under review).
  - **Files or anchors:**
    `react/features/conference/components/web/FocusVersionLabel.tsx` and its
    entry in `ConferenceInfo.tsx`'s `COMPONENTS` array.
  - **Steps:**
    1. Confirm the component reads only the `focus-version` entry from the
       existing base-conference Redux properties map and derives its visibility
       from that value alone.
    2. Confirm no moderator/role/permission selector is imported or consulted in
       the component or in its `COMPONENTS` registration.
  - **Verification:**
    - Command: `grep -niE "moderator|isLocalParticipantModerator|permission|role" react/features/conference/components/web/FocusVersionLabel.tsx`
    - Expected: no matches. Manual confirmation as a non-moderator participant
      belongs to Verify.
  - **Review checkpoint:** does any indirect path — a shared wrapper, a
    connected selector, or the label container — reintroduce role gating that
    the component itself does not perform?

- [ ] 3.4 Cross-repository contract check owned by this repository: prove the UI
      reads the exact key `lib-jitsi-meet` emits through
      `JitsiConferenceEvents.PROPERTIES_CHANGED`, for both the present and absent
      cases. Required result: the literal key used by the component matches the
      key relayed into the base-conference Redux properties map.
  - **Traces to:** R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002.
  - **Depends on:** `lib-jitsi-meet` tasks 2.1–2.2 (the exposed key) and task 3.1
    (the consuming component).
  - **Files or anchors:**
    `react/features/conference/components/web/FocusVersionLabel.tsx` — the Redux
    properties lookup; compared against the key asserted in `lib-jitsi-meet`
    task 2.1 and the existing `PROPERTIES_CHANGED` event bridge in
    `react/features/base/conference/actions.any.ts`.
  - **Steps:**
    1. Extract the literal key string used to read the base-conference Redux
       properties map in `FocusVersionLabel.tsx`.
    2. Compare it character-for-character with the key asserted in
       `lib-jitsi-meet` task 2.1 and published by `jitsi-control` task 1.1.
    3. Confirm the component treats a missing, non-string or empty Redux value
       as absent, without adding a fallback value.
  - **Verification:**
    - Command: `grep -n "focus-version" react/features/conference/components/web/FocusVersionLabel.tsx`
    - Expected: one property lookup with the literal `'focus-version'` —
      identical to the key in `lib-jitsi-meet` task 2.1's assertions. Manual
      confirmation of both cases in the running app belongs to Verify.
  - **Review checkpoint:** is the key a shared literal used consistently across
    all three repositories, with no repository-local alias, casing variant or
    constant that could drift?

## Cross-repository convergence

**Contract check ownership:** each compatibility check sits on the task in the
repository that owns the evidence — `lib-jitsi-meet` task 2.3 (the published
payload is accepted by the presence pass-through) and `jitsi-web` task 3.4 (the
UI reads the exposed key). No separate convergence checkbox exists for either
result. Both run after their repository's local tasks are complete.

**Wire contract:** property name `focus-version`, plain string value in
`CurrentVersionImpl.VERSION.toString()` format (for example `1.1-123-abc1234`),
carried in the focus presence `ConferenceProperties`, relayed unchanged into
`JitsiConference.properties`, emitted through the existing properties-changed
event, stored in the base-conference Redux properties map and selected there by
the UI label.

**Ordering and absent/old-version behavior:** deployment order is unconstrained.
Every hop treats the value as optional, so an older `jitsi-control` that omits
the property, or an older `lib-jitsi-meet`/`jitsi-web` against a newer
`jitsi-control`, reproduces today's behavior end-to-end: no row, no placeholder,
no error state. Rollback of any single repository returns that hop to "value
absent", which the other hops already treat as the normal case.

**Verification summary:** `jitsi-control` and `lib-jitsi-meet` are verified by
their own local test suites plus the `lib-jitsi-meet`-owned contract check
(2.3). `jitsi-web` has no component/unit test harness for `react/features`, so
its tasks are verified by that repository's static checks and code review, with
manual UI confirmation — including the `jitsi-web`-owned contract check (3.4) —
belonging to Verify. Human Feature Acceptance belongs to Verify, not to these
tasks.
