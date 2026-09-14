## Purpose

Gives participants and support agents a way to read which Focus
(`jitsi-control`) version is serving the current conference directly from
the conference details UI, without needing server log access. This removes
a log-access dependency from routine, Focus-version-specific support
investigations (for example, confirming whether a conference is served by a
release that already contains a known fix).

## ADDED Requirements

### Requirement: Conference details UI shows the serving Focus version when known

The system SHALL show a "Focus version" row in the conference details UI,
built from the Focus version value associated with the current conference,
whenever that value is available.

#### Scenario: Current conference has a known Focus version [SC-FOCUS-VERSION-001]

- **WHEN** a participant views the conference details UI for a conference
  whose Focus version value is available
- **THEN** the conference details UI shows a "Focus version" row containing
  that value

### Requirement: Conference details UI omits the Focus version row when the value is unknown

The system SHALL NOT show a "Focus version" row, a placeholder value, or an
error state in the conference details UI when the Focus version value is
not available for the current conference.

#### Scenario: Current conference has no known Focus version [SC-FOCUS-VERSION-002]

- **WHEN** a participant views the conference details UI for a conference
  whose Focus version value is not available
- **THEN** the conference details UI shows no "Focus version" row, no
  placeholder text, and no error state

### Requirement: Focus version visibility is available to all participants

The system SHALL make the "Focus version" row, when shown, visible to all
participants viewing the conference details UI, without restricting it to
moderators or support roles.

#### Scenario: Non-moderator participant views conference details with a known Focus version [SC-FOCUS-VERSION-003]

- **WHEN** a participant without the moderator role views the conference
  details UI for a conference whose Focus version value is available
- **THEN** the "Focus version" row is shown to them the same as it would be
  to a moderator
