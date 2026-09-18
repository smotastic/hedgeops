# Feature: Hedgeops MVP — Door/window notifications

## Summary

Hedgeops receives MQTT messages from an existing publisher, saves the raw messages, and maintains the latest known readings for approved Shelly BLU Door/Window devices. MQTT is the messaging protocol used by the publisher. One configurable automation sends a Telegram notification when an enabled device changes from open to closed or closed to open.

The operator manages devices directly in the database. This minimum viable product (MVP) has no screen, external API, or management command-line tool.

## User stories

### US1 — Preserve received messages (P1)

As an operator, I want incoming messages saved without changes, so that I can inspect the source data and processing decisions.

**Acceptance criteria**
- Given a configured MQTT topic, when Hedgeops receives a message, then it saves the original payload, topic, receipt time, and retained-message flag.
- Given a repeated, malformed, unsupported, unknown-device, or disabled-device message, when it arrives, then Hedgeops still saves the raw message.
- Given a message that cannot update readings, when processing finishes, then its processing record identifies the reason.

### US2 — Approve discovered devices (P1)

As an operator, I want new devices recorded as untrusted, so that I can approve them before they affect readings or notifications.

**Acceptance criteria**
- Given a valid message containing a new device address, when Hedgeops receives it, then it creates one device record with status `untrusted`.
- Given further messages from that address, when they arrive, then they refer to the same device record.
- Given an untrusted device, when I set a supported type, a nonempty name, and status `enabled` in the database, then subsequent eligible messages can update its readings.
- Given an untrusted or disabled device, when messages arrive, then raw messages remain available but processed readings and notifications do not change.
- Given a device that becomes enabled, when its first eligible contact report arrives, then it establishes a starting contact state without a notification.

### US3 — Maintain current device readings (P1)

As an operator, I want the latest known device readings stored in the database, so that I can inspect contact state and supporting measurements.

**Acceptance criteria**
- Given an enabled device, when a valid live report arrives, then Hedgeops updates the valid readings present in that report.
- Given a report with no contact value, when it contains a valid tilt angle, then the tilt angle updates without changing contact state or sending a notification.
- Given a report with valid contact data and invalid battery data, when it is processed, then contact state updates, the previous battery value remains, and the battery error is recorded.
- Given saved readings, when Hedgeops restarts, then the readings remain available.

### US4 — Notify on contact changes (P1)

As an operator, I want one Telegram message for each observed opening or closing, so that I know when a door or window changes state.

**Acceptance criteria**
- Given an enabled device with known state `closed` and an enabled automation, when an eligible report changes state to `open`, then Hedgeops attempts a notification saying the named device opened.
- Given the same conditions with state `open`, when an eligible report changes state to `closed`, then Hedgeops attempts a notification saying the named device closed.
- Given an unknown starting state or an unchanged contact state, when a report arrives, then Hedgeops sends no notification.
- Given a Telegram request that fails, when the failure is detected, then Hedgeops logs the error and makes no retry.

### US5 — Control the automation through configuration (P1)

As an operator, I want one file-based automation, so that I can enable or disable notifications without a rule editor.

**Acceptance criteria**
- Given a changed configuration file, when Hedgeops restarts, then it uses the new configuration.
- Given a disabled automation, when device reports arrive, then raw storage and reading updates continue without notification requests.
- Given changes observed while the automation was disabled, when it becomes enabled again, then Hedgeops does not send notifications for those past changes.

## Requirements

### Input and raw storage

- FR-001: The system shall receive messages from the existing MQTT service using configured connection settings and topic subscriptions.
- FR-002: The system shall save every received message on those subscriptions, including repeated and rejected messages. Each raw record shall preserve the original payload bytes, topic, receipt time, and MQTT retained-message flag.
- FR-003: The system shall preserve raw records without automatic deletion or replacement by processed data.
- FR-004: The system shall record processing outcomes and reasons for rejected messages or readings, linked to the corresponding raw record.
- FR-005: The system shall process the observed publisher format: a device address as the topic, an `addr` field, an optional `rssi` field, and a `service_data` object containing decoded BTHome version 2 readings. BTHome is the device data format already decoded by the publisher.
- FR-006: The system shall reject reading updates for malformed messages, conflicting topic and payload addresses, unsupported formats, and encrypted data that the publisher has not decoded. It shall still preserve the raw messages.

### Device approval

- FR-007: The system shall keep one device record per normalized device address. Address letter case shall not create separate devices.
- FR-008: The system shall create an `untrusted` device record when a valid message identifies a previously unknown device. Invalid or ambiguous identities shall not create device records.
- FR-009: The system shall support `untrusted`, `enabled`, and `disabled` device statuses. The operator shall be able to change device name, type, and status directly in the database.
- FR-010: The system shall require a nonempty name and a supported device type before a device can become enabled for processing. Shelly BLU Door/Window shall be the only supported device type in this MVP.
- FR-011: The system shall update processed readings and evaluate notifications only for enabled devices. Approval shall affect future eligible messages only; it shall not process earlier raw messages again.
- FR-012: The system shall treat the first eligible contact report after initial approval or re-enabling as a new starting state. It shall not notify for that report or infer a change across a disabled period.

### Current readings

- FR-013: The system shall store general device identity, name, type, and approval status separately from the meaning of device-specific readings.
- FR-014: The system shall maintain contact state as `unknown`, `open`, or `closed`. For supported messages, `service_data.window` value `0` shall mean `closed`, and value `1` shall mean `open`. Other values shall not update contact state.
- FR-015: The system shall maintain battery level from `battery`, tilt angle from `rotation`, light level from `illuminance`, and received signal strength from `rssi`, when present and valid.
- FR-016: The system shall accept battery values as integer percentages from 0 through 100, light level as a finite nonnegative number, and tilt angle and signal strength as finite numbers. It shall reject values of the wrong type rather than convert them silently.
- FR-017: The system shall update valid fields independently. Missing or invalid fields shall not erase or replace earlier valid readings. A reading with no valid value yet shall remain unknown.
- FR-018: The system shall retain the receipt time of the latest valid update for each reading. Receipt time shall not be represented as a device-provided event time.
- FR-019: The system shall process eligible messages in receipt order per device. It shall not use the packet identifier `pid` as a permanent unique event identifier.
- FR-020: The system shall save MQTT retained messages as raw data without updating current readings or triggering notifications. A retained message is a previous value stored by the MQTT service for later subscribers.
- FR-021: The system shall preserve current readings across restarts. A restart alone shall not reset an enabled device's contact state or create a notification.

### Automation and Telegram

- FR-022: The system shall support one file-configured automation covering both opening and closing for all enabled, supported devices. Configuration shall include an enable setting and one Telegram chat destination. File changes shall take effect after restart.
- FR-023: The system shall keep Telegram credentials separate from version-controlled automation configuration and shall not include credentials in notification error logs.
- FR-024: The system shall trigger the automation only for observed `open` to `closed` or `closed` to `open` changes. Starting states, repeated states, and changes to other readings shall not trigger it.
- FR-025: The system shall continue raw storage and reading updates while the automation is disabled. Enabling it shall not send notifications for past changes.
- FR-026: Each notification shall include the device name, whether it opened or closed, and the receipt time of the report that caused the change.
- FR-027: Under normal operation, the system shall start the Telegram request within five seconds after receiving the report that causes a qualifying change. Normal operation means the worker and database are available and no service outage or processing backlog prevents progress. Telegram delivery time is not part of this limit.
- FR-028: The system shall attempt each triggered notification once. It shall log request failures without retrying them or storing a pending notification queue. Notification failure shall not undo saved raw data or current readings.

### Acceptance verification

- FR-029: Automated tests shall use captured publisher message examples and a fake Telegram receiver. They shall cover both contact changes, starting states, repeated states, approval, disabling and re-enabling, retained messages, invalid and partial readings, configuration changes, restart state persistence, and logged notification failures without retries.
- FR-030: A manual acceptance check shall use the existing publisher and a Telegram chat to discover a device, approve it, establish its starting state, open it, and close it. The check shall verify raw records, updated readings, and both notifications.

## Edge cases

- An unknown device sends repeated messages: create only one untrusted device record and save every raw message.
- A message identifies an unsupported device: preserve the raw data and any valid discovered identity, but do not enable device-specific processing for an unsupported type.
- A message has conflicting device addresses: preserve it and record the identity error without updating readings or creating an ambiguous device.
- A device first reports `open`: establish `open` without claiming that Hedgeops observed an opening.
- A device changes while disabled: its first eligible contact report after re-enabling establishes a new starting state without notification.
- A report changes only tilt, battery, light level, or signal strength: update valid readings without a notification.
- A repeated message reports the current contact state: save it without creating another contact-change notification.
- An old report arrives after a newer report: receipt order determines current state. This MVP cannot reliably identify all delayed reports and can therefore report a delayed change.
- A retained message arrives after a restart or reconnect: save it without changing readings or notifying. Later live reports use the saved contact state.
- A device stops reporting: keep the latest known readings and their times. Do not infer a new contact state or send an offline alert.
- Telegram fails or Hedgeops stops during notification processing: a notification may be lost. Do not retry or replay it.
- A device has no valid value for a measurement: represent that reading as unknown, not zero.

## Out of scope

- Sensor pairing, publisher installation, publisher changes, and MQTT service setup.
- Screens, external APIs, a public database-access library, and device management commands.
- Processing other device types, including plugs and temperature sensors.
- Device control, scheduled actions, offline alerts, and automations based on measurements other than contact state.
- A general rule editor, multiple destinations, custom message templates, and database-managed automation rules.
- Telegram retries, saved pending notifications, delivery recovery, and guaranteed delivery.
- Historical replay, rebuilding readings from old raw messages, automatic data deletion, and data migration.
- Changes to the existing running system or reuse of its records as production data for this MVP.
- Reliable source-time ordering or complete detection of delayed and repeated transport deliveries.

## Assumptions

- The existing publisher and MQTT service remain available as external inputs. Hedgeops receives already-decoded device data rather than decoding radio packets.
- The inspected database is a read-only reference. Samples came from `supabase-db`, table `public.device_events_bronze`.
- Observed door/window messages contain `addr`, `rssi`, `local_name`, and `service_data`. Observed service fields are `encryption`, `BTHome_version`, `pid`, `battery`, `illuminance`, `window`, and `rotation`.
- The observed messages contain no source event timestamp. The saved `message_timestamp` is empty in the inspected samples, so receipt time defines processing order.
- Approval confirms the device type. The observed empty `local_name` does not prove the exact hardware model.
- The operator has trusted database access and supplies device names, approval decisions, MQTT settings, and Telegram credentials.
- Automation configuration is version-controlled; secrets are supplied separately.
- Storage capacity must be monitored because this MVP does not delete raw messages automatically.
- The linked [Shelly BLU Door/Window ZB documentation](https://www.shelly.com/blogs/documentation/shelly-blu-door-window-zb) provides device context. Captured publisher messages define the input examples used for acceptance tests.
- Technical structure, database layout, and libraries belong in `plan.md`, not this specification.
