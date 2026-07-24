# Restaurant Reservation Bot — Bot specification

**Archetype:** booking

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Guests book available restaurant slots by date/time/party size. The bot enforces real-time availability based on table inventory, opening hours, and sitting duration. Confirmations include reference codes, with reminders and rescheduling options. Owner receives admin notifications and a centralized view.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant guests
- restaurant owner/admin

## Success criteria

- Guests receive booking confirmations with 6-character reference codes
- Owner receives instant notifications for new bookings/cancellations
- No overlapping table allocations in any time slot

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open booking calendar and party size selection
- **Reschedule** (button, actor: user, callback: booking:reschedule) — Initiate rescheduling flow from confirmation screen
  - inputs: selected booking ID
  - outputs: availability calendar
- **Cancel** (button, actor: user, callback: booking:cancel) — Cancel booking with one-tap confirmation
  - inputs: booking ID
  - outputs: cancellation confirmation

## Flows

### Booking flow
_Trigger:_ /start

1. Date selection
2. Party size selection
3. Available time slots
4. Guest info entry
5. Confirmation screen

_Data touched:_ bookings, tables, opening_hours

### Admin notifications
_Trigger:_ booking_created

1. Generate summary message
2. Send to admin chat

_Data touched:_ bookings, tables

### Reminder flow
_Trigger:_ 2h_before_booking

1. Send reminder message with reference code

_Data touched:_ bookings

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Table types** _(retention: persistent)_ — Configured table inventory with counts and seat capacities
  - fields: table_type, count, seats_per_table
- **Bookings** _(retention: persistent)_ — Confirmed reservations with status tracking
  - fields: booking_id, date_time, party_size, tables_used, reference_code, status
- **Opening hours** _(retention: persistent)_ — Daily schedule and service shifts
  - fields: day_of_week, start_time, end_time

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure table types and capacities
- Set opening hours and sitting duration
- View admin notifications and booking summaries

## Notifications

- New booking confirmation
- Cancellation alert
- Reschedule request
- No-show flag

## Permissions & privacy

- Guest contact info stored only with explicit consent
- Admin view shows booking summaries without raw contact data
- Reference codes are the only shared identifiers

## Edge cases

- Guest selects party size larger than available tables
- Multiple simultaneous booking requests for same slot
- Invalid date/time inputs during rescheduling

## Required tests

- End-to-end booking flow with confirmation and admin notification
- Double-booking prevention test
- Reminder message delivery timing validation

## Assumptions

- Default 2-hour reminder before booking
- 15-minute slot granularity
- 90-minute sitting duration default
