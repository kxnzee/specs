## Context

See `proposal.md` — Why, for motivation. Relevant current state:

- The conference creation response already carries several optional
  properties beyond its fixed fields; this pattern is what the new
  `focus-region` property will follow.
- The focus service already has the configured region value available
  internally (it is already used for its own bridge-selection logic); this
  Change only exposes it externally in the conference creation response.
- jitsi-web integrates with a pinned lib-jitsi-meet release. The executable web
  artifact is built by the lib-jitsi-meet repository, so parsing the new wire
  property belongs in that repository rather than in a patch of generated code.
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
  tests already planned in the affected repositories — this is a UI-only diagnostic
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

- **lib-jitsi-meet owns parsing and snapshot semantics.** It parses
  `focus-region` and exposes a conference getter; jitsi-web only consumes that
  public API. This avoids patching a generated minified dependency artifact.

## Repository Implementation Map

Implementation order follows the contract: `jitsi-control` produces the optional
property, lib-jitsi-meet parses and exposes it, and jitsi-web consumes the getter.
`jitsi-control` can still ship independently because older clients ignore unknown
optional properties.

| Repository | Responsibility | Contracts and dependencies |
| --- | --- | --- |
| `jitsi-control` | Add the optional `focus-region` property to the conference creation response, populated with the configured region only when non-empty; cover both the presence and the absence case with backend tests. | Reads the already-available region configuration internally; extends the existing conference creation response contract with one new optional property; introduces no new inbound or outbound contract. |
| `lib-jitsi-meet` | Parse `focus-region`, capture its value or absence once from the initial response, and expose it through the conference API. | Depends on the extended conference creation response contract from `jitsi-control`; supplies the value to `jitsi-web`. |
| `jitsi-web` | Read the region through the lib-jitsi-meet conference API; render it as a new, localized item in the existing web conference info bar; hide the item without an error when the value is absent, empty, or unparseable. | Depends on the new lib-jitsi-meet getter; renders through the client's existing localization system; scoped to the web client only. |

## Risks / Trade-offs

- [Risk] The displayed region can become stale after a focus migration,
  since the value is captured once at conference creation → [Mitigation]
  Documented as an explicit, accepted design choice; if live accuracy is
  needed later, that is a separate follow-up Change, not a defect of this
  one.
- [Risk] jitsi-web can consume the new getter only after a lib-jitsi-meet build
  containing it is available → [Mitigation] Preserve the current UI when the
  getter or value is absent and verify the repositories together before release.
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
- Rollout: `lib-jitsi-meet` must be released before jitsi-web updates its pinned
  version. `jitsi-control` can ship independently: older clients ignore the
  property, while the updated client hides the label when the property is absent.
- Rollback: jitsi-web and `jitsi-control` remain independently rollback-safe;
  rolling back the lib-jitsi-meet API requires also using a jitsi-web revision
  that does not require the getter.

## Open Questions

Нет.
