# Implementation Plan

> **Execution:** Для реализации этого OpenSpec Change вызови установленный штатный
> OpenSpec Apply и следуй актуальным инструкциям схемы. Superpowers executor
> запускается внутри Apply по этим инструкциям. Создание Plan не запускает реализацию.

**Change:** `display-conference-focus-version` (capability
`conference/focus-version-visibility`).

**Goal:** Surface Jicofo's already-public Focus version through its existing
focus-presence `ConferenceProperties`, `lib-jitsi-meet`'s existing generic
presence pass-through into conference properties, and `jitsi-web`'s conference
details UI, visible to all participants only when the value is present.

**Accepted inputs:**
- `openspec/changes/display-conference-focus-version/proposal.md`
- `openspec/changes/display-conference-focus-version/specs/conference/focus-version-visibility/spec.md`
  (SC-FOCUS-VERSION-001/002/003)
- `openspec/changes/display-conference-focus-version/design.md`
  (Decisions and alternatives, Repository Implementation Map)
- `openspec/changes/display-conference-focus-version/tasks.md`
  (coarse Tasks 1.1–1.2, 2.1–2.3, 3.1–3.4)

**Requirements referenced below:**
- R1 "Conference details UI shows the serving Focus version when known" —
  SC-FOCUS-VERSION-001
- R2 "Conference details UI omits the Focus version row when the value is
  unknown" — SC-FOCUS-VERSION-002
- R3 "Focus version visibility is available to all participants" —
  SC-FOCUS-VERSION-003

**Repository boundaries:** work is limited to `jitsi-control`,
`lib-jitsi-meet` and `jitsi-web`. `jitsi-videobridge` is out of scope; the
Store repository holds no implementation. Each repository's tasks are executed
and verified inside that repository only.

**No application run:** no step in this Plan starts, builds or manually
exercises the running application. Apply does not launch the application;
manual UI confirmation is deferred to Verify.

## Repository: `jitsi-control`

**Repository result:** The focus presence Jicofo publishes via
`JitsiMeetConferenceImpl` carries `focus-version` as one entry of its existing
`ConferenceProperties`, with the value `CurrentVersionImpl.VERSION.toString()`.
The conference-allocation success response carries no `focus-version`
property. No other `ConferenceProperties` entry, other presence content, or
other conference-allocation response field changes.

**Depends on:** none (first hop; produces the value the other two repositories
consume via focus presence).

### Task 1: Publish `focus-version` in the focus-presence `ConferenceProperties`

**Traces to:** tasks.md 1.1; R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002.

**Files:**
- Modify: `jicofo/src/main/java/org/jitsi/jicofo/conference/JitsiMeetConferenceImpl.java`
  — `joinTheRoom()` builds the initial focus presence extensions and adds
  `createConferenceProperties()` at lines 642-663; the helper is defined at
  lines 722-727. Confirm these anchors are still current at Apply time.
- Value source: `jicofo/src/main/java/org/jitsi/jicofo/version/CurrentVersionImpl.java`
  (`CurrentVersionImpl.VERSION`).
- Test: repository's existing local test coverage for `ConferenceProperties`
  and/or `JitsiMeetConferenceImpl`'s presence publication — confirm the exact
  test class at Apply time alongside the anchors above.

- [ ] Step 1: Locate the exact construction site of `ConferenceProperties` in
      `JitsiMeetConferenceImpl.java` and its existing test coverage; confirm
      the pattern used for that object's other entries.
- [ ] Step 2: Write a failing test asserting the published
      `ConferenceProperties` includes a `focus-version` entry equal to
      `CurrentVersionImpl.VERSION.toString()`, alongside its existing entries
      (so the test also proves none of them is removed or altered).
- [ ] Step 3: Run that test and confirm it fails because the entry is not yet
      published.
- [ ] Step 4: Add the `focus-version` entry to `ConferenceProperties`,
      following the pattern confirmed in Step 1.
- [ ] Step 5: Re-run the test and confirm it passes, and that the repository's
      existing local test suite for this class/area still passes.
- [ ] Step 6: Commit.
      ```bash
      git commit -m "feat(jicofo): publish focus-version on focus presence ConferenceProperties"
      ```
      (stage the exact files touched in Steps 1-4, confirmed at Apply time)

**Review checkpoint:** confirm the diff touches only the
`ConferenceProperties` construction/publication path — no allocation,
bridge-selection or routing logic — matching design.md's "purely additive,
optional data" risk assessment.

### Task 2: Keep `focus-version` off the conference-allocation response

**Traces to:** tasks.md 1.1 (required result: the conference-allocation
success response carries no `focus-version` property).

**Files:**
- Modify: `jicofo/src/main/kotlin/org/jitsi/jicofo/xmpp/ConferenceIqHandler.kt:106-124`
  (the `response = ConferenceIq().apply { ... }` block in
  `doHandleConferenceIq`) — it must contain no
  `addProperty(ConferenceIq.Property("focus-version", ...))` call.
- Test: repository's local test suite for `ConferenceIqHandler` (confirm the
  exact test class at Apply time) — it must carry no `focus-version`
  assertion on the conference-allocation response.

- [ ] Step 1: Inspect the `ConferenceIqHandler` test class; if it asserts a
      `focus-version` property on the conference-allocation response, remove
      that assertion. Leave every other assertion untouched.
- [ ] Step 2: Inspect `ConferenceIqHandler.kt:106-124`; if it adds a
      `focus-version` property to the response, remove that call. Remove the
      `CurrentVersionImpl` import from this file only if nothing else in the
      file still uses it.
- [ ] Step 3: Run the class's test suite and confirm it passes with no
      `focus-version` assertion remaining and all other assertions unaffected.
      Command: `mvn -pl jicofo -am -Dtest=ConferenceIqHandlerTest -Dsurefire.failIfNoSpecifiedTests=false test`
      Expected: all tests pass; no `focus-version` property assertion remains.
- [ ] Step 4: Commit (skip if Steps 1-2 found nothing to change).
      ```bash
      git commit -m "fix(jicofo): keep focus-version off the conference-allocation response"
      ```
      (stage the exact files touched in Steps 1-2, confirmed at Apply time)

**Review checkpoint:** confirm the diff only affects the `focus-version`
property on the conference-allocation response; no other response field
(`ready`, `focusJid`, existing properties, error handling) is touched.

### Task 3: Regression-confirm no other behavior changed

**Traces to:** tasks.md 1.2.

- [ ] Step 1: Run the repository's existing local test suite for the
      conference-allocation request/response path and confirm it passes
      unmodified in every assertion unrelated to `focus-version`.
- [ ] Step 2: Run the repository's existing local test suite for
      focus-presence / `ConferenceProperties` content and confirm it passes
      unmodified for every entry other than `focus-version`.
- [ ] Step 3: Run the repository's CI-equivalent full verification as a
      regression check.
      Command: `mvn verify -B -Pcoverage`
      Expected: build succeeds, no new failures.

**Review checkpoint:** confirm no assertion outside the `focus-version` entry
and the `focus-version` response property was modified to make the suite pass.

## Repository: `lib-jitsi-meet`

**Repository result:** `JitsiConference.getProperty('focus-version')` returns
the value `jitsi-control` publishes in the focus presence when present, and
`undefined` when it is not — via the existing generic MUC presence
`conference-properties` pass-through, with no parser change, no new storage,
and no other relayed property affected either way.

**Depends on:** `jitsi-control` Task 1 (the property name, and that its value
is a plain string published in the focus presence).

### Task 1: Regression-test that the generic presence pass-through carries `focus-version`

**Traces to:** tasks.md 2.1; R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002.

**Files:**
- Reference (no change expected): `modules/xmpp/ChatRoom.ts:1123-1135` parses
  `conference-properties` and emits `CONFERENCE_PROPERTIES_CHANGED`;
  `JitsiConference.ts:673` binds that event directly to `_updateProperties`;
  `JitsiConference.ts:2126-2184` implements `_updateProperties`;
  `JitsiConference.ts:4710-4712` implements `getProperty`.
- Test: new or existing test file covering presence-driven
  `JitsiConference.properties` updates — confirm the exact file at Apply time.

- [ ] Step 1: Locate the existing test coverage (if any) for MUC presence
      `conference-properties` being relayed into `JitsiConference.properties`
      via `ChatRoom` / `JitsiConference` / `_updateProperties`.
- [ ] Step 2: Write a test (failing first if coverage already exists, new
      otherwise) asserting that when a fixture focus presence carries a
      `focus-version` conference property,
      `JitsiConference.getProperty('focus-version')` returns its value; plus a
      case with no such property in presence, asserting the same call returns
      `undefined`.
- [ ] Step 3: Run the test and confirm both cases pass against the existing
      generic pass-through with no production code change — per design.md,
      `_updateProperties`/`getProperty` are already key-agnostic. If either
      case fails, that contradicts design.md and is a new current-state
      finding: stop and re-open Design rather than patching silently.
- [ ] Step 4: Commit.
      ```bash
      git commit -m "test(xmpp): regression-test focus-version over presence pass-through"
      ```
      (stage the exact test file(s) touched in Steps 1-2, confirmed at Apply
      time)

**Review checkpoint:** confirm no production code changed in this repository
for this task, and that the fixture presence payload matches the exact key and
value format `jitsi-control` publishes in its Task 1.

### Task 2: Confirm the pass-through stays key-agnostic and storage-free

**Traces to:** tasks.md 2.2; R1/SC-FOCUS-VERSION-001, R2/SC-FOCUS-VERSION-002.

**Files:**
- Reference (no change expected): `JitsiConference.ts:2126-2184`
  (`_updateProperties`) and `JitsiConference.ts:4710-4712` (`getProperty`).

- [ ] Step 1: Read `_updateProperties` and confirm it assigns whatever keys
      the presence `conference-properties` element carries (including
      `focus-version` per Task 1) into `this.properties` unchanged.
- [ ] Step 2: Read `getProperty` and confirm it reads back from that same map
      with no key-specific branching.
- [ ] Step 3: Confirm no new storage mechanism was introduced — the value
      lives only in the existing in-memory `properties` map (Proposal
      Non-Goal). No production code or test change is needed for this task
      beyond Task 1's regression test.

**Review checkpoint:** confirm `_updateProperties` and `getProperty` were left
unchanged and that no new state container was added.

## Repository: `jitsi-web`

**Repository result:** The conference details header (`ConferenceInfo`) shows
a "Focus version" row for every participant, built from `lib-jitsi-meet`'s
`focus-version` conference property, and shows nothing — no row, no
placeholder, no error state — when that property is absent.

**Depends on:** `lib-jitsi-meet` Tasks 1–2 (the exact property key read via
`getProperty('focus-version')`).

### Task 1: Add a Focus-version label following the existing conditional-label pattern

**Traces to:** tasks.md 3.1, 3.2, 3.3; R1/SC-FOCUS-VERSION-001,
R2/SC-FOCUS-VERSION-002, R3/SC-FOCUS-VERSION-003.

**Files:**
- Create: `react/features/conference/components/web/FocusVersionLabel.tsx`
  (sibling to the other label components already imported by
  `ConferenceInfo.tsx` from this same directory: `InsecureRoomNameLabel`,
  `RaisedHandsCountLabel`, `SpeakerStatsLabel`, `SubjectText`,
  `ToggleTopPanelLabel`; no `.web` suffix, following the established local
  pattern in this directory rather than the per-platform `.web.tsx`
  convention used elsewhere in the codebase)
- Modify: `react/features/conference/components/web/ConferenceInfo.tsx` (add
  `FocusVersionLabel` to the `COMPONENTS` array, e.g.
  `{ Component: FocusVersionLabel, id: 'focus-version' }`, plus the
  corresponding import alongside the other sibling-label imports)
- Modify: `react/features/conference/components/constants.ts` (register
  `'focus-version'` in the default `autoHide` array — the array
  `getConferenceInfo` in `functions.any.ts` falls back to when
  `config.conferenceInfo` does not override it — following its existing
  entries. `autoHide` is the correct list, not `alwaysVisible`, since the row
  must disappear when the value is absent. Without this registration,
  `ConferenceInfo` never renders the new component by default, regardless of
  the value.)
- Modify: `lang/main.json` (add the translation key `info.focusVersion` with
  value `"Focus version: {{version}}"`, following this file's existing
  `info.*` entries, so the label renders a translated string instead of
  hard-coded text)

There is no component/unit test harness for `react/features` in this
repository: no test file exists for `ConferenceInfo.tsx` or its sibling label
components, and this repository's `npm test` is WebdriverIO end-to-end, not a
component-test runner. This task is therefore verified by this repository's
existing static checks plus manual confirmation deferred to Verify, not by a
new component-test file.

- [ ] Step 1: Create `FocusVersionLabel.tsx` implementing a component that
      reads the current conference's `focus-version` property (via the same
      Redux-connected access pattern this feature area already uses to reach
      the current `JitsiConference`, e.g. as `VisitorsCountLabel`/`E2EELabel`
      do for their own conference-derived values) and:
      - returns `null` when the value is `undefined`
      - otherwise renders it as a text row inside this directory's existing
        label presentation wrapper (matching the visual pattern of sibling
        labels such as `SpeakerStatsLabel`), using the `info.focusVersion`
        translation key added in Step 2
      No moderator-role check or other participant-role gating is added, per
      SC-FOCUS-VERSION-003.
- [ ] Step 2: Add the `info.focusVersion` key to `lang/main.json` with value
      `"Focus version: {{version}}"`, alongside the file's existing `info.*`
      entries.
- [ ] Step 3: Add `FocusVersionLabel` to the `COMPONENTS` array in
      `ConferenceInfo.tsx` with `id: 'focus-version'`, and import it alongside
      the file's other sibling-label imports.
- [ ] Step 4: Register `'focus-version'` in the default `autoHide` array in
      `constants.ts`, alongside its existing entries.
- [ ] Step 5: Run this repository's existing static checks (its configured
      TypeScript type-check and ESLint commands — confirm the exact script
      names from `package.json` rather than assuming one), plus
      `npm run lint:lang` to validate the new `lang/main.json` key, over the
      new and changed files.
      Expected: no new type, lint or lang-lint errors.
- [ ] Step 6: Commit.
      ```bash
      git add react/features/conference/components/web/FocusVersionLabel.tsx \
              react/features/conference/components/web/ConferenceInfo.tsx \
              react/features/conference/components/constants.ts \
              lang/main.json
      git commit -m "feat(conference-info): show Focus version when known"
      ```

**Review checkpoint:** confirm the component returns `null` — no row, no empty
label, no "unknown" text — when `getProperty('focus-version')` is `undefined`
(tasks.md 3.2); that no moderator or other role check was added (tasks.md
3.3); that `'focus-version'` is present in `constants.ts`'s default `autoHide`
array; and that `lang/main.json`'s `info.focusVersion` key is present and
passes `npm run lint:lang`. Manual confirmation in the running app — the
present row, the absent row, and non-moderator visibility — is deferred to
Verify. Apply does not launch the application for this repository.

## Cross-repository convergence

**Rollout ordering:** not strictly required. Per design.md, each hop
(`jitsi-control` publishing, `lib-jitsi-meet` relaying, `jitsi-web` rendering)
is independently absent-tolerant, so any deployment order reproduces today's
behavior (no row) until all three are live. The natural production sequence is
still `jitsi-control` → `lib-jitsi-meet` → `jitsi-web`, matching the dependency
order of the sections above.

**Cross-repository contract checks** (run after the repository-local tasks
above are complete):

- **tasks.md 2.3, owned by `lib-jitsi-meet`** — traces to
  R1/SC-FOCUS-VERSION-001. A contract-level test asserting that the existing
  generic presence pass-through (`lib-jitsi-meet` Task 1 above), given a
  fixture focus presence payload shaped exactly like `jitsi-control`'s
  `ConferenceProperties` output from its Task 1 (property name
  `focus-version`, value format `CurrentVersionImpl.VERSION.toString()`, e.g.
  `'1.1-123-abc1234'`), leaves `JitsiConference.properties` with the expected
  `focus-version` entry. Add the fixture next to whichever presence-fixture
  test file is confirmed in `lib-jitsi-meet` Task 1 Step 1.
  Command: this repository's native test command (confirm the exact script
  covering the chosen test file at Apply time; historically
  `npm run test:native`).
  Expected: pass, including this fixture case.
- **tasks.md 3.4, owned by `jitsi-web`** — traces to R1/SC-FOCUS-VERSION-001,
  R2/SC-FOCUS-VERSION-002. Code review confirming the literal key string
  `'focus-version'` passed to `getProperty` inside `FocusVersionLabel`
  (`jitsi-web` Task 1 above) matches the key `lib-jitsi-meet` exposes per its
  Task 1/2, for both the present and absent cases. Manual confirmation in the
  running app (present and absent cases) is deferred to Verify — no
  component/unit test harness exists in this repository to automate it.

**Verification summary:** `jitsi-control` and `lib-jitsi-meet` are verified by
each repository's existing local test suite (via its existing runner/pattern)
plus the `lib-jitsi-meet`-owned contract check (2.3). `jitsi-web` has no
component/unit test harness for `react/features`, so its verification here is
limited to that repository's existing static checks (type-check, lint,
`lint:lang`); manual UI confirmation of all `jitsi-web` requirements —
including the `jitsi-web`-owned contract check (3.4) — is deferred to Verify.
No step in this Plan starts, builds or manually exercises the running
application: Apply does not launch it. The `jitsi-control` and
`lib-jitsi-meet` file/line anchors above are to be re-confirmed as the first
step of their respective Apply work.
