## Purpose

Gives participants and support agents a way to read the build identifier of
the Focus (`jitsi-control`) serving the current conference directly from
Conference info, so a reported issue can be matched to the specific server
build without needing server log access.

## ADDED Requirements

### Requirement: Conference info shows the Focus build id when known

The system SHALL show a "Focus build id" row in Conference info whenever the
focus-build-id value associated with the current conference contains at least
one non-whitespace character. The system SHALL show that value without changing
its content.

#### Scenario: Current conference has a known Focus build id — display-conference-focus-build-id-001

- **WHEN** a participant views Conference info for a conference whose
  focus-build-id value is `build-2026.09`
- **THEN** Conference info shows a "Focus build id" row containing that
  exact value without changing its content

#### Scenario: Non-blank value retains surrounding whitespace — display-conference-focus-build-id-006

- **WHEN** the current conference provides the non-blank value `"  build-2026.09  "`
  and a participant views Conference info
- **THEN** the row uses that original string, without trimming it for display

### Requirement: Long Focus build ids use the existing presentation

The system SHALL display long non-blank Focus build ids using the existing
Conference info presentation, without introducing custom truncation for this field.

#### Scenario: A participant views a long Focus build id — display-conference-focus-build-id-007

- **WHEN** the current conference provides a long non-blank Focus build id and
  a participant views Conference info
- **THEN** the row uses the existing Conference info presentation and does not
  add custom truncation of the provided value

### Requirement: Conference info omits the Focus build id row when the value is not provided

The system SHALL NOT show a "Focus build id" row, a placeholder value, or an
error state in Conference info when the focus-build-id value has not been
provided, is empty, or contains only whitespace; the rest of Conference info
SHALL remain available.

#### Scenario: Current conference has no focus-build-id value — display-conference-focus-build-id-002

- **WHEN** a participant views Conference info for a conference whose
  focus-build-id value has not been provided
- **THEN** Conference info shows no "Focus build id" row, no placeholder
  text, and no error state, and the rest of Conference info remains
  available

#### Scenario: Current conference has a blank focus-build-id value — display-conference-focus-build-id-005

- **WHEN** a participant views Conference info for a conference whose
  focus-build-id value is empty or contains only whitespace
- **THEN** Conference info shows no "Focus build id" row, no placeholder
  text, and no error state, and the rest of Conference info remains
  available

### Requirement: Focus build id visibility is compatible with a Focus that does not provide the value

The system SHALL treat a conference served by a Focus that does not provide
a focus-build-id value, including an older or unmodified Focus, the same as
the case where the value is not provided, without changing any other
Conference info row or behavior.

#### Scenario: Conference is served by a Focus without focus-build-id support — display-conference-focus-build-id-003

- **WHEN** a participant views Conference info for a conference served by a
  Focus that does not provide a focus-build-id value
- **THEN** Conference info behaves the same as when the value is not
  provided, and no other Conference info row or behavior changes

### Requirement: Focus build id visibility is available to all participants

The system SHALL make the "Focus build id" row, when shown, visible to all
participants viewing Conference info, without restricting it to moderators
or support roles.

#### Scenario: Non-moderator participant views Conference info with a known Focus build id — display-conference-focus-build-id-004

- **WHEN** a participant without the moderator role views Conference info
  for a conference whose focus-build-id value contains a non-whitespace character
- **THEN** the "Focus build id" row is shown to them the same as it would be
  to a moderator
