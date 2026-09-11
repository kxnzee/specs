## Context

See `proposal.md` — Why, for motivation. Relevant current state:

- The conference creation response already carries several optional
  properties beyond its fixed fields; this pattern is what the new
  `focus-region` property will follow.
- The focus service already has the configured region value available
  internally (it is already used for its own bridge-selection logic); this
  Change only exposes it externally in the conference creation response.
- jitsi-web already integrates with lib-jitsi-meet as a pinned dependency, and
  the repository already has a convention for maintaining local patches on
  top of pinned third-party dependencies for other libraries. No such patch
  currently exists for lib-jitsi-meet specifically — this Change is expected
  to introduce the first one, following that existing convention (see
  Decisions and Open Questions).
- The web conference info bar already renders a small set of short,
  localized label items side by side; this Change adds one more, following
  that established pattern.

## Goals / Non-Goals

**Goals:**
- Define the exact wire contract of the new `focus-region` property and the
  client-side rules for when the corresponding label is shown or hidden.
- Keep the change additive and independently deployable in either
  repository, with no coordinated rollout requirement.

**Non-Goals:**
- Live-tracking the region if the focus service migrates after conference
  creation (explicitly out of scope — see Decisions).
- Mapping the region value to a human-readable name, or mobile client parity
  (already excluded in `proposal.md`).
- New backend logging or metrics for the region value beyond the automated
  tests already planned in both repositories — this is a UI-only diagnostic
  aid, not a monitored operational signal, so no additional observability
  work is introduced.

## Decisions

- **Wire property name is `focus-region`.** Chosen to match the naming style
  of the conference creation response's other optional properties, rather
  than reusing the backend configuration key name verbatim. Alternative
  considered: name the property after the configuration key directly —
  rejected to keep the public wire contract decoupled from the backend's
  internal configuration naming.

- **The property is present only when the configured region is non-empty;
  otherwise it is omitted entirely (not sent as an empty string).** This
  means "field present" is the only signal jitsi-web needs to decide whether
  to show the label — no separate empty-string handling is needed on the
  client. Alternative considered: always send the field, possibly empty —
  rejected because it would push "is this meaningful" logic onto every
  consumer of the response.

- **The displayed value is a one-time snapshot from the initial conference
  creation response; it is not updated if the focus service migrates
  afterward.** This keeps the client simple (no need to react to a later
  change in a value it already consumed) and matches the feature's purpose
  as basic diagnostics rather than a live monitoring signal. Trade-off:
  accepted and tracked under Risks below.

- **Absent, empty, and unparseable values are all treated identically on the
  client: the label is hidden, with no error surfaced to the participant.**
  Alternative considered: distinguish "could not parse" from "not
  configured" with separate client-side handling — rejected for this Change
  to keep the behavior simple and testable; if operators need to
  distinguish these cases, that is a separate follow-up using existing
  client-side diagnostics, not a user-facing error.

- **The label's caption goes through the client's existing localization
  system; the region value itself is inserted as plain, non-interactive
  text and is never interpreted as markup.** This matches how the info bar
  already renders its other short label items and keeps the region value
  (backend-configuration-controlled but still externally supplied display
  data) safe to render without a trust assumption on its content.

- **Scope is limited to the web client's conference info bar; mobile clients
  are not changed by this Change.** Matches the explicit scope decision
  already recorded in `proposal.md`; mobile parity, if wanted later, is a
  separate decision.

- **jitsi-web's lib-jitsi-meet integration is adapted to surface
  `focus-region`, following the repository's existing convention for
  adapting pinned dependencies (already used for other libraries) — this
  will be the first such adaptation applied to lib-jitsi-meet itself.** The
  exact low-level mechanism is intentionally left to Tasks/Apply — see Open
  Questions — since it does not change the observable behavior, the chosen
  approach, or the task breakdown at this level.

## Repository Implementation Map

Implementation order: either repository can be implemented and shipped
first (see Migration Plan) — `jitsi-control`'s addition does not require a
prior `jitsi-web` change, and vice versa, since the response already
tolerates unrecognized optional properties and `jitsi-web` already treats
an absent property as the hide-label case.

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-control` | Add the optional `focus-region` property to the conference creation response, populated with the configured region only when non-empty; cover both the presence and the absence case with backend tests. | Reads the already-available region configuration internally; extends the existing conference creation response contract with one new optional property; introduces no new inbound or outbound contract. |
| `jitsi-web` | Read `focus-region` from the initial conference creation response through its lib-jitsi-meet integration, captured once and not updated on later focus migration; render it as a new, localized item in the existing web conference info bar; hide the item without an error when the value is absent, empty, or unparseable. | Depends on the extended conference creation response contract from `jitsi-control`; renders through the client's existing localization system; scoped to the web client only. |

## Risks / Trade-offs

- [Risk] The displayed region can become stale after a focus migration,
  since the value is captured once at conference creation → [Mitigation]
  Documented as an explicit, accepted design choice; if live accuracy is
  needed later, that is a separate follow-up Change, not a defect of this
  one.
- [Risk] jitsi-web currently has no existing patch for lib-jitsi-meet;
  introducing the first one carries some maintenance risk on future
  dependency upgrades → [Mitigation] Keep the adaptation minimal (read one
  additional property) to reduce patch surface and upgrade friction,
  following the same convention already used for the repository's other
  patched dependencies.
- [Risk] Treating "unparseable value" the same as "absent" could hide a
  genuine backend/contract defect from operators, since no error is shown
  to end users → [Mitigation] Accepted for this Change per the explicit
  decision recorded in Proposal/Intake; automated tests in both
  repositories still assert the expected property format, so a contract
  regression is caught in testing rather than only in production.

## Migration Plan

- No data migration is required: the change is purely additive on the
  response contract (one new optional property) and purely additive on the
  UI (one new, conditionally shown label).
- Rollout: `jitsi-control` and `jitsi-web` changes can ship independently in
  either order. A `jitsi-web` build without this change simply ignores the
  unrecognized property, consistent with the response already carrying
  other optional properties today. A `jitsi-control` build without this
  change simply keeps the property absent, which `jitsi-web` already
  treats as the hide-label case.
- Rollback: either repository can be rolled back independently, for the
  same reason — no coordinated rollback step is required.

## Open Questions

- The exact mechanism for adapting jitsi-web's lib-jitsi-meet integration
  (for example, whether a new dependency patch must be introduced, or the
  value can be surfaced without one) is left to be confirmed during
  implementation. It does not change the specs, the chosen approach, or the
  task breakdown at this level of granularity.
