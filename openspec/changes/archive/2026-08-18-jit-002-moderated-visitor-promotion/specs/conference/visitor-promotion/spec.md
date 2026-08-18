## Purpose

Позволяет модератору конференции проактивно включить в разговор конкретного
зрителя из полного списка зрителей, без предварительного запроса слова со
стороны самого зрителя.

## ADDED Requirements

### Requirement: Moderator promotes a visitor without a prior request
The system SHALL allow a moderator to invoke an "include in conversation"
action on any visitor in the full visitors list, resulting in that visitor
gaining the ability to send audio, without requiring the visitor to have
sent a promotion request beforehand.

#### Scenario: Moderator selects a visitor who has not requested to speak [SC-VISITOR-PROMOTION-001]
- **WHEN** a moderator invokes the "include in conversation" action on a
  visitor from the full visitors list who has not sent a promotion-request
- **THEN** the system transitions that visitor into a send-capable
  participant, and no prior action from the visitor is required for this to
  happen

### Requirement: Promoted visitor's microphone starts muted
The system SHALL start the promoted visitor's audio-sending capability in a
muted state; the visitor is responsible for unmuting themselves.

#### Scenario: Visitor is promoted by a moderator — microphone state [SC-VISITOR-PROMOTION-002]
- **WHEN** a moderator promotes a visitor into the conversation
- **THEN** the promoted participant's microphone starts muted and no audio
  is sent until the participant unmutes it themselves

### Requirement: Promotion does not enable video
The system SHALL NOT enable the promoted visitor's video as part of the
promotion action.

#### Scenario: Visitor is promoted by a moderator — video state [SC-VISITOR-PROMOTION-003]
- **WHEN** a moderator promotes a visitor into the conversation
- **THEN** the promoted participant's video remains disabled; the action
  grants audio-sending capability only

### Requirement: Only moderators can promote visitors
The system SHALL restrict the "include in conversation" action to
participants holding the moderator role that already manages mute, kick,
and demote actions; no new role or permission is introduced for this
action.

#### Scenario: Non-moderator views the full visitors list [SC-VISITOR-PROMOTION-004]
- **WHEN** a participant without the moderator role views the full visitors
  list
- **THEN** the "include in conversation" action is not available to them

### Requirement: Promotion does not produce a visible rejoin for the promoted visitor
The system SHALL transition the promoted visitor into a send-capable
participant without presenting the user with a visible rejoin, loss of
meeting state, or a "you left the meeting" screen. Internal reconnection
performed by the underlying mechanism is allowed as long as it is not
user-visible.

#### Scenario: Visitor is promoted while actively viewing the conference [SC-VISITOR-PROMOTION-005]
- **WHEN** a moderator promotes a visitor who is currently viewing the
  conference
- **THEN** the visitor's view of the conference continues without an
  interruption screen, a "you left" message, or loss of their current
  meeting state

### Requirement: Existing request/approve flow remains unaffected
The system SHALL continue to support the existing raise-hand →
promotion-request → approve/deny flow unchanged, independently of the
moderator-initiated promotion action; neither flow is a precondition for
the other.

#### Scenario: Visitor raises hand while moderator-initiated promotion is also available [SC-VISITOR-PROMOTION-006]
- **WHEN** a visitor sends a promotion-request through the existing
  raise-hand flow
- **THEN** the request/approve flow behaves as before, unaffected by the
  existence of the moderator-initiated promotion action

### Requirement: Moderator can revert a moderator-promoted visitor using the existing demote action
The system SHALL allow a moderator to return a moderator-promoted
participant back to visitor state using the existing demote action, with no
separate revert mechanism required.

#### Scenario: Moderator demotes a participant who was promoted without a prior request [SC-VISITOR-PROMOTION-007]
- **WHEN** a moderator invokes the existing demote action on a participant
  who was promoted via the moderator-initiated action, not through the
  request/approve flow
- **THEN** the participant returns to visitor state through the existing
  demote behavior

### Requirement: Moderator-promoted visitor is not automatically reverted
The system SHALL NOT automatically revert a moderator-promoted participant
back to visitor state based on elapsed time or inactivity; reverting SHALL
only happen through an explicit moderator action.

#### Scenario: Promoted participant remains inactive for an extended period [SC-VISITOR-PROMOTION-008]
- **WHEN** a participant who was promoted by a moderator remains send-capable
  without speaking or otherwise being active for an extended period
- **THEN** the system does not revert them back to visitor state on its own;
  they remain send-capable until a moderator explicitly demotes them
