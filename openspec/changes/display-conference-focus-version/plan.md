# Implementation Plan

> **Execution:** Для реализации этого OpenSpec Change вызови установленный штатный
> OpenSpec Apply и следуй актуальным инструкциям схемы. Superpowers executor
> запускается внутри Apply по этим инструкциям. Создание Plan не запускает реализацию.

> **Correction note:** revised after a confirmed Apply-gap. The
> `jitsi-control` and `lib-jitsi-meet` sections below replace the
> originally accepted plan, which targeted the conference-allocation IQ
> response and a `moderator.js` parser change — that transport does not
> reach `JitsiConference.properties` (see design.md Context, Correction).
> The `jitsi-web` section is unaffected and unchanged. This correction was
> made from confirmed Apply evidence and targeted CodeGraph inspection,
> without running the application. Apply must still verify the corrected
> path with repository tests.

**Goal:** Surface Jicofo's already-public Focus version through its
existing focus-presence `ConferenceProperties`, `lib-jitsi-meet`'s existing
generic presence pass-through into conference properties, and `jitsi-web`'s
conference details UI, visible to all participants only when the value is
present.

**Accepted inputs:**
- `openspec/changes/display-conference-focus-version/proposal.md`
- `openspec/changes/display-conference-focus-version/specs/conference/focus-version-visibility/spec.md`
  (Requirements SC-FOCUS-VERSION-001/002/003)
- `openspec/changes/display-conference-focus-version/design.md`
  (Decisions and alternatives, Repository Implementation Map)
- `openspec/changes/display-conference-focus-version/tasks.md`
  (coarse Tasks 1.1–1.2, 2.1–2.3, 3.1–3.4)

## Repository: `jitsi-control`

**Repository result:** The focus presence Jicofo publishes via
`JitsiMeetConferenceImpl` carries `focus-version` as one entry of its
existing `ConferenceProperties`, with the value
`CurrentVersionImpl.VERSION.toString()`. The `focus-version` property
previously added to the conference-allocation success response (superseded
transport) is removed; no other response field or presence content changes.

**Depends on:** none (first hop; produces the value the other two
repositories consume, now via focus presence instead of the
conference-allocation response).

### Task 1: Remove `focus-version` from the conference-allocation response (correction)

**Traces to:** tasks.md 1.3; design.md Context, Correction (confirmed
Apply-gap: this transport does not reach `JitsiConference.properties`).

**Files:**
- Modify: `jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt:106-124`
  (the `response = ConferenceIq().apply { ... }` block; remove the
  `addProperty(ConferenceIq.Property("focus-version", ...))` call added
  under the original, now-superseded Plan)
- Test: `jicofo/src/test/kotlin/org/jitsi/jicofo/ConferenceIqHandlerTest.kt`
  (remove or update the assertion added for the superseded transport)

- [ ] Step 1: In `ConferenceIqHandlerTest.kt`, find the assertion added
      under the original Plan for the `focus-version` property on the
      conference-allocation response, and remove it.
- [ ] Step 2: In `ConferenceIqHandler.kt`, remove the
      `addProperty(ConferenceIq.Property("focus-version", ...))` line added
      under the original Plan; remove the now-unused `CurrentVersionImpl`
      import from this file only if nothing else in the file still uses it.
- [ ] Step 3: Run the class's test suite and confirm it passes with no
      `focus-version` assertion remaining, and that all other existing
      assertions are unaffected.
      Command: `mvn -pl jicofo -am -Dtest=ConferenceIqHandlerTest -Dsurefire.failIfNoSpecifiedTests=false test`
      Expected: all tests pass; no `focus-version` property assertion
      remains in the class.
- [ ] Step 4: Commit.
      ```bash
      git add jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt \
              jicofo/src/test/kotlin/org/jitsi/jicofo/ConferenceIqHandlerTest.kt
      git commit -m "fix(jicofo): remove focus-version from conference-allocation response (superseded transport)"
      ```

**Review checkpoint:** confirm the diff only removes the property added
under the superseded transport, with no other response field touched.

### Task 2: Add `focus-version` to the focus-presence `ConferenceProperties`

**Traces to:** tasks.md 1.4; specs SC-FOCUS-VERSION-001, SC-FOCUS-VERSION-002.

**Files:**
- Modify: `jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java`
  — `joinTheRoom()` builds the initial focus presence extensions and adds
  `createConferenceProperties()` at lines 642-663; the helper is defined at
  lines 722-727. Confirm these anchors are still current at Apply time.
- Test: repository's existing test coverage for `ConferenceProperties`
  and/or `JitsiMeetConferenceImpl`'s presence publication — exact test
  class to confirm at Apply time alongside the file/line anchor above.

- [ ] Step 1: Locate the exact construction site of `ConferenceProperties`
      in `JitsiMeetConferenceImpl.java` and its existing test coverage;
      confirm the pattern used for its other entries.
- [ ] Step 2: Write a failing test asserting the published
      `ConferenceProperties` includes a `focus-version` entry equal to
      `CurrentVersionImpl.VERSION.toString()`, alongside its existing
      entries (to confirm none is removed or altered).
- [ ] Step 3: Run that test and confirm it fails because the entry is not
      yet published.
- [ ] Step 4: Add the `focus-version` entry to `ConferenceProperties`
      following the pattern confirmed in Step 1.
- [ ] Step 5: Re-run the test and confirm it passes, and that the
      repository's existing test suite for this class/area still passes.
- [ ] Step 6: Run the repository's CI-equivalent full verification as a
      regression check.
      Command: `mvn verify -B -Pcoverage`
      Expected: build succeeds, no new failures.
- [ ] Step 7: Commit.
      ```bash
      git commit -m "feat(jicofo): publish focus-version on focus presence ConferenceProperties"
      ```
      (stage the exact files touched in Steps 1-4, confirmed at Apply time)

**Review checkpoint:** confirm the diff touches only the
`ConferenceProperties` construction/publication path (no allocation,
bridge-selection, or routing logic changed), matching design.md's "purely
additive, optional data" risk assessment.

## Repository: `lib-jitsi-meet`

**Repository result:** `JitsiConference.getProperty('focus-version')`
returns the value published in the focus presence by `jitsi-control` when
present, and `undefined` when it is not — via the existing generic MUC
presence `conference-properties` pass-through, with no parser or storage
change and no other parsed/relayed property affected either way.

**Depends on:** `jitsi-control` Task 2 (the property name and that its
value is a plain string, published in the focus presence).

### Task 1: Regression-test the existing generic presence pass-through carries `focus-version`

**Traces to:** tasks.md 2.1, 2.2; specs SC-FOCUS-VERSION-001, SC-FOCUS-VERSION-002.
Supersedes the original Task 1, which targeted parsing the
conference-allocation response in `modules/xmpp/moderator.js` — that
transport does not reach `JitsiConference.properties` (see design.md
Context, Correction). Any in-progress local change made against
`moderator.js` for this change should not be carried forward; it belongs to
the superseded transport, not this corrected Plan.

**Files:**
- Reference (no change expected): `modules/xmpp/ChatRoom.ts:1123-1135`
  parses `conference-properties` and emits `CONFERENCE_PROPERTIES_CHANGED`;
  `JitsiConference.ts:673` binds that event directly to `_updateProperties`;
  `JitsiConference.ts:2126-2184` implements `_updateProperties`, and
  `JitsiConference.ts:4710-4712` implements `getProperty`.
- Test: new or existing test file covering presence-driven
  `JitsiConference.properties` updates — exact file to confirm at Apply
  time (this Plan does not assume `moderator.spec.js`, since that file
  covers the superseded IQ-parsing transport, not presence).

- [ ] Step 1: Locate the existing test coverage (if any) for MUC presence
      `conference-properties` being relayed into `JitsiConference.properties`
      via `ChatRoom` / `JitsiConference` / `_updateProperties`.
- [ ] Step 2: Write a failing (or new, if no such coverage exists) test
      asserting that when a fixture focus presence carries a
      `focus-version` conference property, `JitsiConference.getProperty('focus-version')`
      returns its value; and a case with no such property in presence,
      asserting the same call returns `undefined`.
- [ ] Step 3: Run the test and confirm the new cases pass against the
      existing generic pass-through with no production code change — per
      design.md, `_updateProperties`/`getProperty` are already key-agnostic.
      If either case fails, that contradicts design.md's Correction and
      must be treated as a new current-state finding, not patched silently:
      stop and re-open Design before proceeding.
- [ ] Step 4: Commit.
      ```bash
      git commit -m "test(xmpp): regression-test focus-version over existing presence pass-through"
      ```
      (stage the exact test file(s) touched in Steps 1-2, confirmed at
      Apply time)

**Review checkpoint:** confirm no production code changed in this
repository for this task (Proposal Non-Goal: no new storage; Design
Correction: no parser change), and that the new test fixture's presence
payload matches the exact key and value format `jitsi-control` publishes in
Task 2 above.

### Task 2: Confirm end-to-end exposure via `JitsiConference.getProperty`

**Traces to:** tasks.md 2.2; specs SC-FOCUS-VERSION-001, SC-FOCUS-VERSION-002.

**Files:**
- Reference (no change expected): `JitsiConference.ts:2126-2184`
  (`_updateProperties`) and `JitsiConference.ts:4710-4712` (`getProperty`)

- [ ] Step 1: Read `_updateProperties` (`JitsiConference.ts:2126-2184`) and
      `getProperty` (`JitsiConference.ts:4710-4712`) and confirm neither
      contains key-specific logic that would need to special-case
      `focus-version` — both should already treat every key in the
      presence-sourced properties object generically.
- [ ] Step 2: If Step 1 finds key-specific logic that would reject or drop
      `focus-version` (unexpected per design.md), fix it and add a
      regression test at that point, then commit alongside that fix.
      Otherwise, no production code or test change is needed for this task
      beyond Task 1's regression test.

**Review checkpoint:** confirm no new storage mechanism was introduced
(Proposal Non-Goal) — the value must live only in the existing
`properties` map — and confirm `_updateProperties`/`getProperty` were left
unchanged.

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
  `lib-jitsi-meet`'s existing generic presence pass-through (lib-jitsi-meet
  Task 1 above), given a fixture focus presence payload shaped exactly like
  `jitsi-control`'s `ConferenceProperties` output from jitsi-control Task 2
  (property name `focus-version`, value format
  `CurrentVersionImpl.VERSION.toString()`, e.g. `'1.1-123-abc1234'`),
  produces the expected `JitsiConference.getProperty('focus-version')`. Add
  this fixture next to whichever existing presence-fixture test file is
  confirmed in lib-jitsi-meet Task 1 Step 1 — not `modules/xmpp/moderator.spec.js`,
  which covers the superseded IQ-parsing transport.
  Command: this repository's native test command (confirm the exact script
  covering the chosen test file at Apply time; historically
  `npm run test:native`).
  Expected: pass, including this fixture case.
- **3.4, owned by `jitsi-web`:** code review confirming the literal key
  string `'focus-version'` passed to `getProperty` inside
  `FocusVersionLabel` (jitsi-web Task 1 above) matches the key
  `lib-jitsi-meet` exposes per the `lib-jitsi-meet` contract check above,
  plus manual confirmation in the running app (present and absent cases)
  deferred to Verify — no component/unit test harness exists in this
  repository to automate this check (see jitsi-web Task 1 above).

No step in this Plan starts, builds, or manually exercises the running
application — including this correction, which was made from confirmed
Apply evidence without running the application or performing new code
exploration. `jitsi-control` and `lib-jitsi-meet` verification is each
repository's existing local test suite (via its existing runner/pattern)
plus the `lib-jitsi-meet`-owned contract check (2.3). `jitsi-web` has no
component/unit test harness for `react/features`, so its verification here
is limited to this repository's existing static checks (type-check, lint);
manual UI confirmation of all `jitsi-web` requirements — including the
`jitsi-web`-owned contract check (3.4) — is deferred to Verify, per
tasks.md. `jitsi-control` Task 1 and `lib-jitsi-meet` Task 1 exact
file/line anchors for the corrected presence-based transport remain to be
confirmed as the first step of their respective Apply work.
