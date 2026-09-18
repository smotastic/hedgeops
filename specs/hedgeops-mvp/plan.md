# Implementation plan: Hedgeops MVP — Door/window notifications

## Goal

Implement the confirmed specification in `spec.md`: save MQTT messages, discover untrusted devices, maintain approved device readings, and attempt Telegram notifications for opening and closing.

## Technical approach

### Existing constraints

The repository has documentation and repository checks, but no application code, package configuration, or application tests. Follow accepted architecture decisions ADR-0001 through ADR-0005:

- Use strict TypeScript, pnpm workspaces, and Nx for builds and tests.
- Run one plain Node.js worker. Do not add NestJS, an HTTP server, or separate ingestion and automation services.
- Run a separate Hedgeops PostgreSQL database in Docker with persistent storage. Do not change the existing Supabase database or other running containers.
- Store raw messages before interpreting device data. Save common events, current readings, and processing results in one transaction.
- Keep device and automation rules separate from external-system code. Define ports, which are interfaces for external operations, in application code. Adapters implement those ports.

Use Node.js 24 LTS, a supported long-term release. Select and pin compatible pnpm, Nx, TypeScript, and Vitest versions when creating the workspace. Use MQTT.js for MQTT, `pg` for PostgreSQL, Zod for configuration checks, and built-in `fetch` for Telegram. Vitest is the test runner. Use the official PostgreSQL 17 image with a pinned patch version or image digest verified for ARM64, the Pi's processor type. Commit the dependency lockfile.

### Module structure

Create one Nx application, `worker`, and one internal library, `core`.

- `core/ingestion`: accepts transport messages and requests raw storage.
- `core/refining`: validates device data, updates readings, and returns contact changes.
- `core/automation`: matches contact changes and calls a provider-neutral notification port.
- `worker/adapters`: connects those operations to MQTT, PostgreSQL, Shelly message decoding, and Telegram.
- `worker/main.ts`: validates configuration, connects components, and manages startup and shutdown.

Core code must not import MQTT.js, `pg`, Telegram code, or worker files. Modules call published application interfaces rather than another module's internal files. The notification port accepts device name, change, receipt time, and a logical recipient. Telegram destination and text formatting stay in the adapter.

### Processing sequence

1. At the MQTT callback, copy the payload bytes and capture receipt time, an increasing process-local receipt sequence, topic, and delivery flags. Serialize raw inserts in callback order. Do not parse payloads before raw storage succeeds.
2. Insert a raw record and a `pending` processing record together. Assign a database order number. Save every delivery separately, including identical payloads.
3. A single refining loop claims pending records in database order. Store `in_progress` and increment its attempt count before processing. Single-worker operation keeps messages ordered for each device.
4. Decode the stored bytes. Validate the message object, matching topic and `addr`, `service_data`, `encryption: false`, and `BTHome_version: 2`. Normalize valid addresses to uppercase colon-separated form. Separate valid identity detection from individual reading checks.
5. Create an untrusted device for a valid new identity. Do not infer a supported device type from an empty `local_name` or a device address alone. Retained messages can discover an identity, but cannot update readings.
6. Lock the device row while deciding whether the record is eligible. Check status, supported type, approval boundary, and baseline flag. This also orders processing against direct database edits.
7. Check each reading independently. Apply valid fields only. Save a versioned common event for valid approved readings. Save zero events with a reason when no reading is eligible. Save events, readings, baseline changes, and the completed processing result in one transaction.
8. After commit, return any eligible contact change through an in-process interface to automation. Do not call Telegram from the refining module or hold a database transaction during a network request.
9. Automation checks the startup-loaded enable setting. Start the single Telegram request immediately without making the refining loop wait for the result. Use a ten-second request timeout and handle each promise rejection. Log a safe result; never schedule a retry.

### Approval boundaries and recovery

Direct database editing requires database-enforced rules, not only worker validation.

- Add a device status trigger. On entry to `enabled`, set `enabled_at` using database time, advance an approval generation, and set `needs_contact_baseline = true`. Do not erase saved readings. A name change or worker restart does not reset the baseline.
- Ignore reports received before the current approval boundary for reading updates. This prevents a delayed pending report from becoming eligible merely because the operator approved the device after receipt. Use the same host clock for receipt capture and the database; resolve equal-time cases conservatively as ineligible.
- The first eligible valid contact reading clears the baseline flag without producing a contact change. A report containing only battery or tilt does not clear it.
- Read device status for each processing operation; do not cache approval status across database edits. Unsupported types and blank names cannot pass the database enable constraint.
- Enforce one active worker using a PostgreSQL advisory lock, a database lock owned by the worker connection. Stop processing if that connection is lost.
- At startup, resume pending work and return abandoned `in_progress` work to pending. Keep completed records untouched. A unique `(raw_message_id, event_index)` key prevents duplicate common events on a repeated processing attempt.
- Recovering unfinished ingestion is required by ADR-0002; it is not historical replay. Disable notification dispatch for records from an earlier worker run. Complete their local processing without sending old notifications. Store a worker-run identifier with each raw record to make this distinction explicit.
- An unexpected processing error rolls back event and reading changes and records `failed`, attempt count, and a safe error. Stop later processing rather than pass a failed record and break order. After the fault is corrected, restart can retry failed work with notifications suppressed. Invalid input is a completed rejection, not a failed processing attempt.
- Telegram has no recovery path. A crash after state commit but before sending can lose a notification. Never recover a notification from a common event.

### Configuration and runtime

Use `config/automation.json` for the one automation's enabled flag and Telegram chat destination. Read it once at startup. Keep the logical recipient fixed to `home`; the adapter maps it to the configured chat.

Read `DATABASE_URL`, `MQTT_URL`, optional MQTT credentials, topic subscriptions, and `TELEGRAM_BOT_TOKEN` from environment variables. Require Telegram settings only when automation is enabled. Reject invalid configuration before subscribing. Never log connection strings, passwords, raw Telegram response bodies, or request URLs containing the bot token.

Use an explicit MQTT client ID and a clean session. Request quality of service (QoS) level 1, which permits repeated delivery; the publisher's settings still limit delivery guarantees. Reconnect and resubscribe automatically. Preserve the received retained flag and let the MQTT library handle protocol acknowledgements. This MVP does not promise acknowledgement only after database storage.

On storage failure, log a safe error and stop the worker rather than continue processing unsaved data. Data received before a successful database write can be lost; this is not a durable MQTT delivery design. On normal shutdown, stop intake, finish outstanding database operations, and wait only within the Telegram request timeout. No notification is saved for a later run.

Use Docker Compose only for the new development database. Bind its host port to loopback on a configurable nonconflicting port, with a project-specific volume. Use a separate test Compose project with a disposable database and MQTT broker. Do not connect automated tests to the existing MQTT service or Supabase instance.

## Files to change

All application paths below are new. Names ending in `/` identify directories containing the listed responsibility.

| File | Change | Reason |
| --- | --- | --- |
| `package.json`, `pnpm-workspace.yaml`, `pnpm-lock.yaml`, `.node-version` | Pin tools and define workspace scripts | Reproducible setup |
| `nx.json`, `tsconfig.base.json`, `vitest.config.ts` | Define build, type-check, and test targets | Shared strict checks |
| `.gitignore`, `.env.example` | Ignore secrets and build output; document variable names | Safe local configuration |
| `apps/worker/package.json`, `apps/worker/project.json`, `apps/worker/tsconfig.json` | Define the single worker project | Nx build and execution |
| `libs/core/package.json`, `libs/core/project.json`, `libs/core/tsconfig.json` | Define the internal core library | Enforce dependency direction |
| `libs/core/src/ingestion/` | Add raw-storage input operation and port | Store before interpretation |
| `libs/core/src/refining/` | Add device types, reading rules, processing operation, and storage/decoder ports | Device approval and state |
| `libs/core/src/automation/` | Add contact-change rule, notification action, and notification port | Provider-independent automation |
| `libs/core/src/index.ts` | Export module application interfaces | Explicit module boundaries |
| `apps/worker/src/main.ts`, `apps/worker/src/config.ts` | Wire modules, validate settings, handle lifecycle | One plain Node.js process |
| `apps/worker/src/adapters/mqtt.ts` | Subscribe and capture bytes plus delivery details | Existing publisher input |
| `apps/worker/src/adapters/shelly.ts` | Decode observed format and validate fields | Isolate device-specific interpretation |
| `apps/worker/src/adapters/postgres/` | Implement raw storage, processing, device locks, and recovery | Transactional state updates |
| `apps/worker/src/adapters/telegram.ts`, `apps/worker/src/adapters/logger.ts` | Send once with timeout and safe logs | Notifications without retries |
| `config/automation.json` | Add disabled-by-default example rule and destination placeholder | Version-controlled automation |
| `db/migrations/001_initial.sql`, `tools/migrate.ts` | Create tables, checks, approval trigger, and migration runner | Repeatable empty-database setup |
| `compose.yaml` | Add isolated PostgreSQL service and persistent volume | Local database per ADR-0003 |
| `compose.test.yaml`, `tests/support/mosquitto.conf` | Add disposable test database and broker | Safe integration testing |
| `tests/fixtures/shelly/` | Add sanitized captured examples and derived invalid/partial cases | Verified input contract |
| `libs/core/src/**/*.test.ts`, `apps/worker/src/**/*.test.ts` | Add unit and adapter tests | Fast behavioral checks |
| `tests/integration/`, `tests/acceptance/`, `tests/support/` | Add database, MQTT, restart, and fake Telegram checks | Full-path verification |
| `README.md`, `docs/operations/hedgeops-mvp.md` | Document setup, SQL approval, limits, and manual acceptance | Operator workflow |
| `.github/workflows/validate.yml` | Keep current repository checks; add application checks | Automated validation |

## Data and interfaces

### PostgreSQL records

Use parameterized SQL and forward-only numbered migrations. Use separate runtime and migration credentials. The runtime role must not update or delete raw records. The operator uses a trusted role to edit device configuration.

| Table | Main fields and constraints |
| --- | --- |
| `raw_messages` | Unique ID, increasing order number, worker-run ID, topic, payload `bytea`, receipt time, retained flag, received QoS, duplicate flag where available. No payload-based uniqueness. |
| `message_processing` | Raw ID, `pending/in_progress/complete/failed`, attempt count, outcome code, field errors, safe failure details, start and completion times. |
| `devices` | Unique normalized address, nullable name and type before approval, status, discovery time, enabled time, approval generation, baseline flag. Enabling requires a supported type and trimmed nonempty name. |
| `device_readings` | Device ID, `unknown/open/closed` contact state, nullable battery/tilt/light/signal readings, separate receipt time for each reading, source raw ID for each reading. |
| `device_events` | Raw ID, event index, device ID, event type `readings_observed`, format version 1, receipt time, nullable device event time, valid readings, optional previous/current contact state. Unique raw ID plus event index. |
| `schema_migrations` | Applied migration number and time. |

Events for this publisher have no device event time. Use explicit `null`; do not copy receipt time into that field. Store packet identifiers and other unused source fields in the raw payload, not as event identity.

Use a receipt order number to break equal timestamp ties. Do not interpret `pid` wraparound, decreasing values, or a repeated payload as proof of an old event. Do not convert missing readings to zero. Units are battery percent, rotation degrees, illuminance lux, and signal strength dBm, the radio power unit used by the publisher. Preserve original numerical values; confirm these units against the publisher contract in adapter tests and documentation.

### Application interfaces

- `ReceiveMessage`: accepts bytes and transport metadata; returns the saved raw record identity.
- `DecodeDeviceMessage`: returns a validated identity, valid reading fields, and field-level errors, or a whole-message rejection.
- `ProcessNextMessage`: commits processing results and returns zero or one contact change for this message. Storage manages row locks and approval checks.
- `EvaluateContactChange`: applies the startup-loaded automation setting and invokes the notification port when applicable.
- `NotifyContactChange`: returns success or a safe failure result. It does not retry. The core contains no Telegram chat IDs or bot tokens.

These are internal interfaces, not a public library or external API. No database or API compatibility with the reference application is required.

## Validation

### Commands to provide during implementation

These commands are planned; they cannot run until the workspace and tests exist.

```bash
pnpm install --frozen-lockfile
pnpm nx run-many -t build typecheck test

docker compose -p hedgeops-mvp-test -f compose.test.yaml up -d --wait
pnpm test:integration
pnpm test:acceptance
docker compose -p hedgeops-mvp-test -f compose.test.yaml down -v

git diff --check
```

Configure `test:integration` and `test:acceptance` to use test-only connection settings and reject production database names. Disable Nx test caching for tests that use containers. Run migrations against a new test database and verify that a second migration run does not change existing records. Test volume persistence by replacing the test database container without deleting its volume.

### Requirement coverage

| Requirements | Implementation and checks |
| --- | --- |
| FR-001–FR-004 | MQTT adapter and raw/processing tables. Check exact bytes, invalid bytes, repeated deliveries, subscribed topics, delivery flags, rejection reasons, and no automatic deletion. |
| FR-005–FR-006 | Shelly decoder. Test captured open/closed examples, malformed data, address mismatch, encrypted data, unsupported version, and wrong types. |
| FR-007–FR-010 | Device table, discovery, checks, and SQL edits. Test normalized identity, one record per device, all statuses, and rejected approval with missing name or unsupported type. |
| FR-011–FR-012 | Approval trigger and processing transaction. Test pending pre-approval messages, disable/re-enable between reports and while the worker is stopped, baseline handling, and no processing of completed historical records. |
| FR-013–FR-018 | Device/readings separation and independent field validation. Test every field, allowed bounds, missing/null/invalid values, partial updates, initial unknown values, and per-field receipt times. |
| FR-019–FR-021 | Ordered processing, retained exclusion, and persistent readings. Test rapid mixed-device messages, timestamp ties, packet identifier reuse, retained discovery, restart, and abandoned-work recovery. |
| FR-022–FR-025 | Startup configuration and core automation. Test both transitions, unchanged state, first state, disabled automation with continued readings, restart configuration loading, and no missed-change notifications. Test secret redaction. |
| FR-026–FR-028 | Telegram adapter and fake local HTTP receiver. Check text, receipt time, request start within five seconds, timeout, rejected HTTP response, Telegram error response, and exactly one attempt. A stalled receiver must not block ingestion. |
| FR-029 | Automated acceptance path combines isolated MQTT, PostgreSQL, the worker, and fake Telegram. Inject failures between raw insert, processing claim, transaction commit, and notification dispatch. Verify no duplicate events or notification recovery. |
| FR-030 | Operator runbook covers controlled discovery, SQL approval, first report, opening, closing, and database/Telegram inspection. |

Run the manual check only with explicit operator approval and a dedicated Hedgeops database. Use a separate MQTT client to subscribe without changing the publisher, existing consumers, or broker configuration. The check must not write to the reference Supabase database.

## Risks and decisions

- ADR-0002 requires stored processing recovery. Keep that separate from notification recovery, which is explicitly excluded. Pending raw records can recover local state; earlier-run records cannot send Telegram notifications.
- Current-state accuracy follows receipt order, not physical event order. Delayed live messages can still cause misleading changes. This is an accepted specification limit.
- Direct database approval can race with pending processing. Database row locks, an approval boundary, and a baseline trigger prevent old reports from silently becoming fresh transitions.
- Raw messages grow without a retention limit. Document a storage-size query and operator checks. Do not add automatic deletion.
- Fast incoming traffic can exceed storage or Telegram capacity. Normal-load acceptance must meet the five-second request-start limit. Do not add a durable notification queue to mask this limit.
- A network timeout does not prove Telegram rejected a message. Log the uncertainty and do not retry.
- MQTT acknowledgement is not tied to successful storage in this MVP. Do not claim guaranteed input delivery during database, process, or host failures.
- The reference system is evidence, not a code or schema dependency. Do not copy its separate service layout, management commands, or notification retry system.
- No architecture decision changes are planned. The implementation uses the accepted worker, storage, and dependency rules.
