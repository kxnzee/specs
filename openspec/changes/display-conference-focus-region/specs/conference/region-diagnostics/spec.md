## Purpose

Отображает участникам и эксплуатации настроенный регион focus-сервиса (Jicofo),
полученный при создании конференции, как диагностическое поле в информационной
строке веб-клиента конференции.

## ADDED Requirements

### Requirement: Conference creation response includes the configured focus region
The system SHALL include a `focus-region` property in the conference creation
response when the focus service is configured with a non-empty region value.
The system SHALL NOT include this property when no region is configured or the
configured value is empty.

#### Scenario: Region configured on the focus service [SC-CONFERENCE-REGION-DIAGNOSTICS-001]
- **WHEN** the focus service is configured with a non-empty region value and a
  client creates a conference
- **THEN** the conference creation response includes `focus-region` set to that
  configured value

#### Scenario: Region not configured on the focus service [SC-CONFERENCE-REGION-DIAGNOSTICS-002]
- **WHEN** the focus service has no configured region, or the configured value
  is empty, and a client creates a conference
- **THEN** the conference creation response does not include the `focus-region`
  property

### Requirement: Web conference info bar displays the configured region
The system SHALL display a "Conference region: <value>" item in the web
client's conference info bar when the conference creation response contained a
non-empty, successfully parsed `focus-region` value. The system SHALL NOT
display this item, and SHALL NOT surface an error to the participant, when the
value is absent, empty, or could not be parsed.

#### Scenario: Region value received [SC-CONFERENCE-REGION-DIAGNOSTICS-003]
- **WHEN** the conference creation response contains a non-empty `focus-region`
  value
- **THEN** the web client's conference info bar shows "Conference region:
  <value>" alongside the existing items of that bar

#### Scenario: Region value absent, empty, or unparseable [SC-CONFERENCE-REGION-DIAGNOSTICS-004]
- **WHEN** the conference creation response does not contain a `focus-region`
  value, contains an empty value, or the value cannot be parsed by the client
- **THEN** the web client's conference info bar does not show a region item,
  and no error is presented to the participant

### Requirement: Displayed region value stays fixed for the conference lifetime
The system SHALL capture the `focus-region` value only from the initial
conference creation response. The system SHALL NOT update the displayed value,
or its absence, if the focus service handling the conference changes after
creation.

#### Scenario: Focus migrates after conference creation [SC-CONFERENCE-REGION-DIAGNOSTICS-005]
- **WHEN** the focus service handling an active conference migrates to a
  different region after the conference was created
- **THEN** the conference info bar continues to show the region item (or its
  absence) exactly as determined from the initial conference creation response

### Requirement: Region label is localized and rendered as safe plain text
The system SHALL source the "Conference region" caption text from the web
client's existing localization system. The system SHALL render the region
value exactly as received, as plain, non-interactive text, without
interpreting it as markup.

#### Scenario: Label uses localization and renders value verbatim [SC-CONFERENCE-REGION-DIAGNOSTICS-006]
- **WHEN** the web client displays the "Conference region" item
- **THEN** its caption text comes from the localization system and the region
  value is shown exactly as received, with no markup interpretation and no
  interactive behavior

### Requirement: Region display applies only to the web conference info bar
The system SHALL introduce the "Conference region" item only in the existing
web client's conference info bar. This capability SHALL NOT change mobile
client behavior.

#### Scenario: Mobile client is unaffected [SC-CONFERENCE-REGION-DIAGNOSTICS-007]
- **WHEN** a participant uses a mobile client to join or create a conference
- **THEN** no "Conference region" item or related behavior from this
  capability is introduced for that client
