# Implementation Plan

> **Execution:** Для реализации этого OpenSpec Change вызови установленный штатный
> OpenSpec Apply и следуй актуальным инструкциям схемы. Superpowers executor
> запускается внутри Apply по этим инструкциям. Создание Plan не запускает реализацию.

**Goal:** Surface Jicofo's already-public Focus version through the existing
conference-allocation response, `lib-jitsi-meet`'s conference properties, and
`jitsi-web`'s conference details UI, visible to all participants only when
the value is present.

**Accepted inputs:**
- `openspec/changes/display-conference-focus-version/proposal.md`
- `openspec/changes/display-conference-focus-version/specs/conference/focus-version-visibility/spec.md`
  (Requirements SC-FOCUS-VERSION-001/002/003)
- `openspec/changes/display-conference-focus-version/design.md`
  (Decisions and alternatives, Repository Implementation Map)
- `openspec/changes/display-conference-focus-version/tasks.md`
  (coarse Tasks 1.1–1.2, 2.1–2.3, 3.1–3.4)

## Repository: `jitsi-control`

**Repository result:** The conference-allocation success response built by
`ConferenceIqHandler.doHandleConferenceIq` carries an optional
`focus-version` property whose value is `CurrentVersionImpl.VERSION.toString()`,
with no other response field changed.

**Depends on:** none (first hop; produces the value the other two
repositories consume).

### Task 1: Emit `focus-version` on the conference-allocation response

**Traces to:** tasks.md 1.1, 1.2; specs SC-FOCUS-VERSION-001,
SC-FOCUS-VERSION-002 (the property must be present when known and simply
absent when not — no separate "unknown" branch is needed since Jicofo
always knows its own version).

**Files:**
- Modify: `jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt:106-124`
  (the `response = ConferenceIq().apply { ... }` block that already calls
  `addProperty(ConferenceIq.Property("authentication", ...))` etc.)
- Test: confirmed existing test location for `ConferenceIqHandler.kt` is
  `jicofo/src/test/kotlin/org/jitsi/jicofo/ConferenceIqHandlerTest.kt`,
  package/class `org.jitsi.jicofo.ConferenceIqHandlerTest` — this test
  package is `org.jitsi.jicofo`, not `org.jitsi.jicofo.xmpp`, even though
  the class under test lives in the `xmpp` subpackage; `ConferenceIqHandlerTest.kt`
  already exists at that path — use it, do not create a new test class.

- [ ] Step 1: In `ConferenceIqHandlerTest.kt`, write a failing test asserting
      that the `ConferenceIq` response produced for a request includes a
      `focus-version` property equal to `CurrentVersionImpl.VERSION.toString()`,
      alongside the existing `authentication` property (to confirm no
      existing property is removed or altered).
- [ ] Step 2: Run the new test and confirm it fails because the property is
      not yet emitted.
      Command: `./gradlew :jicofo:test --tests "org.jitsi.jicofo.ConferenceIqHandlerTest"`
      Expected: test run reports a failure for the new assertion (property
      `focus-version` missing from the response), all other assertions in
      the class unaffected.
- [ ] Step 3: In `ConferenceIqHandler.kt`, inside the `response = ConferenceIq().apply { ... }`
      block (same block that sets `focusJid` and the `authentication`
      property), add:
      `addProperty(ConferenceIq.Property("focus-version", CurrentVersionImpl.VERSION.toString()))`
      Add the corresponding import for `CurrentVersionImpl`
      (`org.jitsi.jicofo.version.CurrentVersionImpl`) if not already
      imported in this file.
- [ ] Step 4: Re-run the test and confirm it passes, and that the full
      pre-existing test suite for this class still passes (no other
      property assertions broke).
      Command: `./gradlew :jicofo:test --tests "org.jitsi.jicofo.ConferenceIqHandlerTest"`
      Expected: all tests in the class pass, including the new one.
- [ ] Step 5: Run the module's full local test suite as a regression check.
      Command: `./gradlew :jicofo:test`
      Expected: build succeeds, no new failures.
- [ ] Step 6: Commit.
      ```bash
      git add jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt \
              jicofo/src/test/kotlin/org/jitsi/jicofo/ConferenceIqHandlerTest.kt
      git commit -m "feat(jicofo): emit focus-version on conference-allocation response"
      ```

**Review checkpoint:** confirm the diff touches only the response-building
block in `ConferenceIqHandler.kt` (no allocation, bridge-selection, or
routing logic changed), matching design.md's "purely additive, optional
data" risk assessment.

## Repository: `lib-jitsi-meet`

**Repository result:** `JitsiConference.getProperty('focus-version')`
returns the value from the conference-allocation response when
`jitsi-control` sent it, and `undefined` when it did not — with no other
parsed property affected either way.

**Depends on:** `jitsi-control` Task 1 (the property name and that its
value is a plain string, not a boolean flag like sibling properties).

### Task 1: Parse `focus-version` out of the conference-allocation response

**Traces to:** tasks.md 2.1; specs SC-FOCUS-VERSION-001, SC-FOCUS-VERSION-002.

**Files:**
- Modify: `modules/xmpp/moderator.js:261-291` (`_parseConferenceIq`, the
  method that already reads `property[name="authentication"][value="true"]`
  etc. via the file's existing `exists`/`findFirst`/`getAttribute` helpers)
- Test: `modules/xmpp/moderator.spec.js` (existing test file covering
  `Moderator`, confirmed by CodeGraph as the test for this class)

- [ ] Step 1: In `modules/xmpp/moderator.spec.js`, write a failing test
      calling `_parseConferenceIq` (or the public method that wraps it, per
      the existing tests' convention in this file) with a fixture IQ result
      containing `<property name="focus-version" value="1.1-123-abc1234"/>`,
      asserting the returned `properties.focus-version` equals
      `'1.1-123-abc1234'`. Add a second case with no such `<property>`
      element, asserting `properties['focus-version']` is `undefined`.
- [ ] Step 2: Run the native test suite (which includes
      `moderator.spec.js`) and confirm both new cases fail (parser does not
      yet read this property). `package.json`'s `test:native` script is
      `karma start karma.conf.js`; there is no existing per-file filtering
      to narrow this to a single spec file.
      Command: `npm run test:native`
      Expected: 2 new failing assertions in `moderator.spec.js`, existing
      tests in the file and suite still pass.
- [ ] Step 3: In `_parseConferenceIq`, add (mirroring the existing
      `sipGatewayEnabled` check's use of `findFirst`/`getAttribute`, since
      `focus-version` carries a variable string value rather than a fixed
      `"true"` literal):
      ```javascript
      const focusVersionProperty = findFirst(
          resultIq, ':scope>conference>property[name="focus-version"]');

      if (focusVersionProperty) {
          conferenceRequest.properties['focus-version']
              = getAttribute(focusVersionProperty, 'value');
      }
      ```
      placed alongside the other `conferenceRequest.properties.*` checks in
      the same method, before the `return conferenceRequest;` line.
- [ ] Step 4: Re-run the native test suite and confirm all cases pass.
      Command: `npm run test:native`
      Expected: all tests pass, including the 2 new cases in
      `moderator.spec.js`.
- [ ] Step 5: Commit.
      ```bash
      git add modules/xmpp/moderator.js modules/xmpp/moderator.spec.js
      git commit -m "feat(xmpp): parse focus-version conference property"
      ```

### Task 2: Confirm end-to-end exposure via `JitsiConference.getProperty`

**Traces to:** tasks.md 2.2; specs SC-FOCUS-VERSION-001, SC-FOCUS-VERSION-002.

**Files:**
- Reference (no change expected): `JitsiConference.ts:2126-2184`
  (`_updateProperties`, the single place that assigns `this.properties` and
  emits `PROPERTIES_CHANGED`) and `JitsiConference.ts:4710-4712`
  (`getProperty`)

`JitsiConference.spec.ts` has no existing properties-update test pattern to
extend for this, and `_updateProperties`/`getProperty` are already
key-agnostic (they operate on whatever keys `_parseConferenceIq` produces,
with no per-key branching) — so this confirmation is code inspection, not a
new test. Task 1's fixture-based parser test already exercises
`focus-version` flowing into `conferenceRequest.properties`.

- [ ] Step 1: Read `_updateProperties` (`JitsiConference.ts:2126-2184`) and
      `getProperty` (`JitsiConference.ts:4710-4712`) and confirm neither
      contains key-specific logic that would need to special-case
      `focus-version` — both should already treat every key in the parsed
      properties object generically.
- [ ] Step 2: If Step 1 finds key-specific logic that would reject or drop
      `focus-version` (unexpected per design.md), fix it and add a
      regression test at that point, then commit alongside that fix.
      Otherwise, no production code or test change is needed for this task.

**Review checkpoint:** confirm no new storage mechanism was introduced
(Proposal Non-Goal) — the value must live only in the existing
`properties` map — and confirm `_updateProperties`/`getProperty` were left
unchanged (Step 1 found them already generic).

## Repository: `jitsi-web`

**Repository result:** The conference details header (`ConferenceInfo`)
shows a "Focus version" row for every participant, built from
`lib-jitsi-meet`'s `focus-version` conference property, and shows nothing
(no row, no placeholder, no error) when that property is absent.

**Depends on:** `lib-jitsi-meet` Task 1–2 (the exact property key read via
`getProperty('focus-version')`).

### Task 1: Add a Focus-version label following the existing conditional-label pattern

**Traces to:** tasks.md 3.1, 3.2, 3.3, 3.4; specs SC-FOCUS-VERSION-001,
SC-FOCUS-VERSION-002, SC-FOCUS-VERSION-003.

**Files:**
- Create: `react/features/conference/components/web/FocusVersionLabel.tsx`
  (sibling to the other label components already imported by
  `ConferenceInfo.tsx` from this same directory: `InsecureRoomNameLabel`,
  `RaisedHandsCountLabel`, `SpeakerStatsLabel`, `SubjectText`,
  `ToggleTopPanelLabel`; no `.web` suffix, following the established local
  pattern in this web directory for this kind of component — e.g. the
  earlier `ConferenceRegionLabel` — rather than the per-platform
  `.web.tsx` convention used elsewhere in the codebase)
- Modify: `react/features/conference/components/web/ConferenceInfo.tsx`
  (add `FocusVersionLabel` to the `COMPONENTS` array, e.g.
  `{ Component: FocusVersionLabel, id: 'focus-version' }`, and add the
  corresponding import alongside the other sibling-label imports)
- Modify: `react/features/conference/components/constants.ts` (register
  `'focus-version'` in the default `autoHide` array — the array
  `getConferenceInfo` in `functions.any.ts` falls back to when
  `config.conferenceInfo` does not override it — following its existing
  entries; `autoHide` is the correct list, not `alwaysVisible`, since this
  row must disappear when the value is absent rather than persist
  unconditionally). Without this change, `ConferenceInfo` does not render
  the new component by default regardless of the value's presence.
- Modify: `lang/main.json` (add the translation key `info.focusVersion`
  with value `"Focus version: {{version}}"`, following this file's existing
  `info.*` key entries, so the new label has a string to render instead of
  hard-coded text)

There is no component/unit test harness for `react/features` in this
repository: no test file exists for `ConferenceInfo.tsx` or its sibling
label components in this directory, and this repository's `npm test` is
WebdriverIO end-to-end, not a component-test runner. This task is therefore
verified by this repository's existing static checks plus manual
confirmation deferred to Verify, not by a new Jest/component-test file.

- [ ] Step 1: Create `FocusVersionLabel.tsx` implementing a component
      that reads the current conference's `focus-version` property (via
      the same Redux-connected access pattern this feature area already
      uses to reach the current `JitsiConference`, e.g. as
      `VisitorsCountLabel`/`E2EELabel` do for their own conference-derived
      values) and:
      - returns `null` when the value is `undefined`
      - otherwise renders it as a text row inside this directory's
        existing label presentation wrapper (matching the visual pattern
        of sibling labels such as `SpeakerStatsLabel`), using the
        `info.focusVersion` translation key added in Step 2 below
      No moderator-role check or other participant-role gating is added,
      per SC-FOCUS-VERSION-003.
- [ ] Step 2: Add the `info.focusVersion` key to `lang/main.json` with
      value `"Focus version: {{version}}"`, alongside the file's existing
      `info.*` entries.
- [ ] Step 3: Add `FocusVersionLabel` to the `COMPONENTS` array in
      `ConferenceInfo.tsx` with `id: 'focus-version'`, and import it
      alongside the file's other sibling-label imports.
- [ ] Step 4: Register `'focus-version'` in the default `autoHide` array in
      `constants.ts`, alongside its existing entries.
- [ ] Step 5: Run this repository's existing static checks (its configured
      TypeScript type-check and ESLint commands — confirm the exact command
      names from `package.json` scripts rather than assuming one), plus
      `npm run lint:lang` to validate the new `lang/main.json` key, over the
      new and changed files. Expected: no new type, lint, or lang-lint
      errors.
- [ ] Step 6: Commit.
      ```bash
      git add react/features/conference/components/web/FocusVersionLabel.tsx \
              react/features/conference/components/web/ConferenceInfo.tsx \
              react/features/conference/components/constants.ts \
              lang/main.json
      git commit -m "feat(conference-info): show Focus version when known"
      ```

**Review checkpoint:** confirm the row is absent (not empty/placeholder)
when `getProperty('focus-version')` is `undefined`, that no moderator/role
check was added, that `'focus-version'` is present in `constants.ts`'s
default `autoHide` array, and that `lang/main.json`'s `info.focusVersion`
key is present and passes `npm run lint:lang` — these directly verify the
Delta Spec's requirements (SC-FOCUS-VERSION-001 through 003) and the
rendering precondition confirmed during Design. Manual confirmation in the
running app (all three requirements, including the present/absent row and
non-moderator visibility) is deferred to Verify; no application run happens
during Apply for this repository.

## Cross-repository convergence

**Rollout ordering:** not strictly required. Per design.md, each hop
(`jitsi-control` emitting, `lib-jitsi-meet` parsing, `jitsi-web` rendering)
is independently absent-tolerant, so any deployment order reproduces
today's behavior (no row) until all three are live. The natural production
sequence is still `jitsi-control` → `lib-jitsi-meet` → `jitsi-web`, matching
the dependency order of the sections above.

**Cross-repository contract checks** (from tasks.md; run after the
repository-local tasks above are complete):

- **2.3, owned by `lib-jitsi-meet`:** a contract-level test asserting that
  `lib-jitsi-meet`'s `_parseConferenceIq` (Task 1 above), given a fixture
  IQ payload shaped exactly like `jitsi-control`'s `ConferenceIqHandler`
  output from Task 1 (property name `focus-version`, value format
  `CurrentVersionImpl.VERSION.toString()`, e.g. `'1.1-123-abc1234'`),
  produces the expected `properties['focus-version']`. Add this fixture
  next to the existing fixtures already used in `modules/xmpp/moderator.spec.js`.
  There is no existing per-test filtering for the `test:native` Karma
  script, so this case runs as part of the same full-suite command as
  Task 1's cases.
  Command: `npm run test:native`
  Expected: pass, including this fixture case.
- **3.4, owned by `jitsi-web`:** code review confirming the literal key
  string `'focus-version'` passed to `getProperty` inside
  `FocusVersionLabel` (Task 1 above) matches the key `lib-jitsi-meet`
  exposes per the `lib-jitsi-meet` contract check above, plus manual
  confirmation in the running app (present and absent cases) deferred to
  Verify — no component/unit test harness exists in this repository to
  automate this check (see Task 1 above).

No step in this Plan starts, builds, or manually exercises the running
application. `jitsi-control` and `lib-jitsi-meet` verification is each
repository's existing local test suite (via its existing runner/pattern)
plus the `lib-jitsi-meet`-owned contract check (2.3). `jitsi-web` has no
component/unit test harness for `react/features`, so its verification here
is limited to this repository's existing static checks (type-check, lint);
manual UI confirmation of all `jitsi-web` requirements — including the
`jitsi-web`-owned contract check (3.4) — is deferred to Verify, per
tasks.md.
