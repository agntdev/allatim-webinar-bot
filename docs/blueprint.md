# Webinar Registration Bot — Bot specification

**Archetype:** booking

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Registers attendees for Alla Tim's buyer-training webinar, collects contact details, manages reminders, and provides organizers with attendee lists and notifications. Supports both new users and existing students with duplicate detection and data export capabilities.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- New potential attendees discovering the webinar
- Existing students/subscribers of Alla Tim

## Success criteria

- Successful registration confirmation with calendar options
- Automated 24h and 1h reminders sent
- Admin receives real-time registration notifications and daily exports
- CSV export available on admin request

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with webinar details and registration buttons
- **Register** (button, actor: user, callback: registration:start) — Initiate registration flow
  - inputs: Name, Email, Phone
  - outputs: Registration confirmation message
- **Details** (button, actor: user, callback: webinar:details) — Show webinar title, date/time, and description
- **/attendees** (command, actor: admin, command: /attendees) — Export current attendee list as CSV
- **/count** (command, actor: admin, command: /count) — Show current registration count

## Flows

### Registration
_Trigger:_ registration:start

1. Show webinar details
2. Request name
3. Validate email format
4. Validate phone length
5. Check for existing registration
6. Display summary with confirmation button
7. Send calendar ICS file

_Data touched:_ Attendee, Registration

### Reminder
_Trigger:_ scheduled:24h & 1h

1. Check registered users without sent reminders
2. Send Telegram message with event details

_Data touched:_ Registration

### Admin Export
_Trigger:_ /attendees

1. Generate CSV with all attendee data
2. Send file to admin

_Data touched:_ Attendee

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Webinar** _(retention: persistent)_ — Webinar metadata and scheduling
  - fields: title, date_time, description
- **Attendee** _(retention: persistent)_ — Registered user information
  - fields: name, email, phone, telegram_id, registration_time, reminder_24h_sent, reminder_1h_sent
- **Registration** _(retention: persistent)_ — Registration status and history
  - fields: status, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure admin chat ID for notifications
- Edit webinar details via admin commands
- Manage registration status (confirm/cancel)

## Notifications

- Registration confirmation with calendar options
- 24h and 1h pre-event reminders
- Real-time new-registration alerts to admin chat
- Daily attendee list export to admin

## Permissions & privacy

- Implicit opt-in for notifications
- Minimal data collection (name, email, phone)
- Data retention until owner deletion

## Edge cases

- Duplicate registration detection by Telegram ID/email
- Failed reminder delivery handling
- Invalid phone/email format validation
- Concurrent registration updates

## Required tests

- End-to-end registration flow with validation and confirmation
- Admin CSV export functionality
- Reminder scheduling at 24h/1h intervals
- Duplicate registration prevention

## Assumptions

- Webinar details provided by owner before launch
- Admin chat ID configured by owner
- ICS calendar format used for compatibility
