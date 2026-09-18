# Tasks: Hedgeops MVP — Door/window notifications

## Dependencies

The specification and plan are approved. Application files named below do not exist yet. Create them unless the task explicitly updates an existing file.

Ports are application-owned interfaces. Adapters connect those interfaces to PostgreSQL, MQTT, or Telegram. PostgreSQL-specific queries, locks, and triggers must stay outside the application core.

- T001 before T002–T003.
- T002 before T004–T006.
- T003 and T006 before T007; T007 before T008.
- T004 before T009; T005 and T009 before T010.
- T004 and T005 before T011; T005 and T011 before T012.
- T007 and T011 before T013; T008 and T013 before T014.
- T004 and T011 before T015; T013 and T015 before T016.
- T008, T012, T014, and T016 before T017; T017 before T018.
- T004 before T019; T019 before T020.
- T004 before T021; T021 before T022.
- T019 and T021 before T023; T023 before T024.
- T010, T018, T020, T022, and T024 before T025; T025 before T026.
- T008 and T022 before T027; T026 and T027 before T028–T030.
- T028–T030 before T031–T032.
- T031 and T032 before T033; T033 before T034.

Tasks marked `[P]` can run alongside other independent tasks after their prerequisites are complete. Shared files still require coordinated edits. Tests are required by the specification and plan.

## Tasks

### Workspace and database setup

- [ ] T001 Create the workspace in `package.json`, `pnpm-workspace.yaml`, `pnpm-lock.yaml`, and `.node-version`. Pin Node.js 24 LTS and compatible pnpm, Nx, TypeScript, Vitest, MQTT.js, `pg`, and Zod versions. Define build, type-check, test, migration, and worker-start scripts.
- [ ] T002 Define the `worker` application and `core` library in `nx.json`, `tsconfig.base.json`, `vitest.config.ts`, `apps/worker/package.json`, `apps/worker/project.json`, `apps/worker/tsconfig.json`, `libs/core/package.json`, `libs/core/project.json`, and `libs/core/tsconfig.json`. Enable strict type checks and dependency-ordered builds. Keep container tests outside the cached unit-test target.
- [ ] T003 [P] Define the isolated development database in `compose.yaml`, `.env.example`, and `.gitignore`. Pin an official PostgreSQL 17 image verified for ARM64, the Pi's processor type. Add a health check, persistent named volume, configurable loopback port, and secret variables without real credentials. Do not reuse existing container names, volumes, or the Supabase database.
- [ ] T004 [P] Define core data types and ports in `libs/core/src/ingestion/ports.ts`, `libs/core/src/refining/ports.ts`, `libs/core/src/refining/types.ts`, `libs/core/src/automation/ports.ts`, and `libs/core/src/index.ts`. Cover raw storage, decoding, transactional processing, clock/identity inputs, contact changes, and provider-neutral notification results. Expose module application interfaces without database or provider types.
- [ ] T005 [P] Add sanitized captured messages in `tests/fixtures/shelly/open.json`, `tests/fixtures/shelly/closed.json`, and `tests/fixtures/shelly/README.md`. Record sample origin and verified field units. Add derived partial, malformed, encrypted, conflicting-identity, unsupported-device, and invalid-reading cases. Do not copy credentials or production database dumps.
- [ ] T006 [P] Implement database provisioning in `db/init/001_roles.sh` and wire it into `compose.yaml`. On an empty volume, create the Hedgeops database and separate migration, runtime, and trusted operator login roles. Make the migration role the schema owner; do not grant runtime or operator superuser rights. Read passwords from external settings, quote values safely, and do not print them. Document that initialization does not rerun on an existing volume.
- [ ] T007 Implement numbered schema setup in `db/migrations/001_initial.sql` and `tools/migrate.ts`. Create raw, processing, device, reading, common-event, and migration tables from the plan. Add keys, indexes, reading checks, supported-type approval checks, and the approval trigger. Grant runtime insert/read access to raw records but no update/delete access. Limit device updates to fields each role needs. Apply migrations with migration credentials, serialize migration runs, and leave already-applied migrations unchanged.
- [ ] T008 Verify provisioning in `compose.test.yaml`, `tests/support/mosquitto.conf`, `tests/support/database.ts`, and `tests/integration/provisioning.test.ts`. Use a separate disposable database and MQTT broker. Test role permissions, fresh migrations, repeated migration execution, schema constraints, and persistence after container replacement. Reject non-test database targets before destructive test operations.

### US1 — Preserve received messages

- [ ] T009 [US1] Implement store-before-parse ingestion in `libs/core/src/ingestion/receive-message.ts` and `apps/worker/src/adapters/postgres/raw-store.ts`. Save copied bytes and receipt/delivery metadata with a pending processing record in one transaction. Save repeated payloads as separate records. Preserve worker-run identity and deterministic receipt order.
- [ ] T010 [US1] Verify raw ingestion in `libs/core/src/ingestion/receive-message.test.ts` and `tests/integration/raw-store.test.ts`. Check unchanged bytes, invalid text bytes, identical deliveries, topic, receipt time, retained flag, delivery flags, and rollback when the pending processing record cannot be saved. Prove that decoding cannot run before storage succeeds.

### US2 — Approve discovered devices

- [ ] T011 [US2] Implement publisher decoding in `apps/worker/src/adapters/shelly.ts`. Validate JSON structure, topic/address agreement, address normalization, supported BTHome version, and decoded unencrypted data. Return valid identity independently of individual reading errors. Do not infer a supported device type from an address or empty name.
- [ ] T012 [US2] Verify decoder behavior in `apps/worker/src/adapters/shelly.test.ts`. Use captured fixtures and check case normalization, invalid identity, wrong types, malformed bytes, encrypted messages, unsupported formats, and independent reading errors.
- [ ] T013 [US2] Implement device storage in `apps/worker/src/adapters/postgres/device-store.ts`. Create one untrusted record per valid identity. Read approval status under a row lock for every processing operation. Apply the database-controlled approval generation, receipt-time boundary, and contact baseline flag without caching device approval.
- [ ] T014 [US2] Verify direct database approval in `tests/integration/device-approval.test.ts`. Cover repeated discovery, invalid approval, unknown and disabled devices, approval after message receipt, equal-time exclusion, disabled/re-enabled devices, changes while the worker is stopped, and concurrent status edits. Confirm that name-only edits do not reset contact baseline.

### US3 — Maintain current device readings

- [ ] T015 [US3] Implement reading and contact rules in `libs/core/src/refining/readings.ts` and `libs/core/src/refining/readings.test.ts`. Cover unknown/open/closed state, battery bounds, finite numeric fields, missing and invalid readings, per-field receipt times, initial baseline, and known-to-known contact changes. A tilt-only or battery-only report must not clear the contact baseline.
- [ ] T016 [US3] Implement stored-message processing in `libs/core/src/refining/process-next-message.ts` and `apps/worker/src/adapters/postgres/processing-store.ts`. Claim records in order, count attempts, and commit events, readings, baseline changes, and outcomes together. Record field errors and zero-event reasons. Permit retained-message discovery but exclude retained reading updates. Return contact changes only after commit and never call Telegram from this module.
- [ ] T017 [US3] Verify processing transactions in `tests/integration/processing.test.ts`. Cover valid and rejected messages, unknown/disabled devices, partial updates, preserved values, per-field times, event format/version, absent source time, packet identifier reuse, equal receipt timestamps, retained messages, and mixed-device ordering. Inject a transaction failure and confirm that no partial event or state update remains.
- [ ] T018 [US3] Implement and verify unfinished-work recovery in `apps/worker/src/adapters/postgres/recovery.ts` and `tests/integration/recovery.test.ts`. Acquire the single-worker database lock, recover pending and abandoned work, and stop on lock loss. Record unexpected failures without letting later records pass them. Retry corrected failed work only on restart. Preserve completed records and readings, prevent duplicate common events, and suppress notifications for earlier worker-run records.

### US5 — Control the automation through configuration

- [ ] T019 [P] [US5] Implement startup configuration in `apps/worker/src/config.ts`, `config/automation.json`, and `.env.example`. Read the one automation file once, default the example to disabled, and validate database/MQTT settings. Require a Telegram destination and external bot token only when enabled. Keep the logical recipient `home` separate from the Telegram destination.
- [ ] T020 [US5] Verify configuration in `apps/worker/src/config.test.ts`. Cover invalid settings, optional Telegram credentials when disabled, required credentials when enabled, rejection before subscription, file edits taking effect only after reload, and no credential values in validation errors.

### US4 — Notify on contact changes

- [ ] T021 [P] [US4] Implement criteria and action logic in `libs/core/src/automation/evaluate-contact-change.ts` and `libs/core/src/automation/evaluate-contact-change.test.ts`. Match both known-to-known contact changes across enabled devices. Suppress initial, unchanged, disabled-automation, and recovery changes. Call a fake notification port in core tests without importing Telegram code.
- [ ] T022 [US4] Implement one-attempt delivery in `apps/worker/src/adapters/telegram.ts`, `apps/worker/src/adapters/logger.ts`, `apps/worker/src/adapters/telegram.test.ts`, and `apps/worker/src/adapters/logger.test.ts`. Format device name, opened/closed text, and receipt time. Use a ten-second timeout. Handle network, HTTP, and Telegram response errors without retrying. Redact secrets, connection strings, token-bearing URLs, and unsafe response bodies.

### US1 and US5 — Connect the input service

- [ ] T023 [US1] [US5] Implement the MQTT subscriber in `apps/worker/src/adapters/mqtt.ts`. Use a configured client ID, clean session, subscriptions, and requested QoS level 1. Capture bytes, receipt time, sequence, and delivery flags at the callback. Serialize raw inserts, reconnect/resubscribe, and report storage failures to worker shutdown rather than process unsaved input.
- [ ] T024 [US1] [US5] Verify the subscriber in `apps/worker/src/adapters/mqtt.test.ts` and `tests/integration/mqtt.test.ts`. Cover subscriptions, reconnect, receipt order, retained flags, repeated delivery, raw metadata, and failure reporting. Keep the tests on the isolated test broker.

### Connect and verify the complete worker

- [ ] T025 Connect the modules in `apps/worker/src/main.ts` and `apps/worker/src/runtime.ts`. Load configuration, acquire the worker lock, recover unfinished records, and start ingestion plus one refining loop. Dispatch contact changes after commit without waiting for Telegram completion. Handle rejected promises, storage failure, lock loss, and bounded shutdown. Add no HTTP server, notification queue, or public API.
- [ ] T026 Verify worker lifecycle and dependency boundaries in `apps/worker/src/runtime.test.ts` and `tests/integration/architecture.test.ts`. Check startup order, no subscription with invalid settings, one active worker, state persistence, shutdown, and local processing during stalled Telegram requests. Reject core imports of worker adapters, `pg`, MQTT.js, and provider code. Exercise the same storage-port behavior through PostgreSQL and a test implementation, so storage rules are not tied to SQL types.
- [ ] T027 Create isolated full-path test support in `tests/support/fake-telegram.ts`, `tests/support/worker.ts`, `tests/support/test-environment.ts`, and `package.json`. Provide fake HTTP responses, request timestamps, process restart/fault controls, test-only connection checks, and the `test:integration` and `test:acceptance` scripts. Never send real Telegram requests in automated tests.
- [ ] T028 [US1] [US2] [US3] [US4] Verify the full message path in `tests/acceptance/door-window.test.ts`. Publish captured reports, discover and approve a device, establish its baseline, open and close it, and inspect raw records, readings, common events, and two fake Telegram requests. Check notification text and request start within five seconds under normal test load.
- [ ] T029 [US1] [US2] [US3] [US5] Verify exclusions and configuration changes in `tests/acceptance/exclusions.test.ts`. Cover retained reports, repeated states, unknown/disabled devices, re-enabling, malformed and partial readings, disabled automation with continued storage, and restart-loaded configuration without missed-change notifications.
- [ ] T030 [US3] [US4] Verify failure and restart limits in `tests/acceptance/recovery.test.ts`. Stop the worker after raw storage, after claim, and after processing commit. Check local recovery without duplicate events or recovered notifications. Test HTTP failure, Telegram error response, timeout, and stalled delivery with continued local processing. Assert one delivery attempt and safe logs.

### Documentation and release checks

- [ ] T031 Document operation in `README.md` and `docs/operations/hedgeops-mvp.md`. Give exact commands for container creation, initial roles, migrations, worker start, and safe restart. Include SQL examples for approval/name/status edits, secret setup, log inspection, storage-size checks, container persistence, and known input/notification loss limits. Explain that PostgreSQL is an exchangeable storage adapter, but a replacement must implement equivalent transaction and approval behavior. Include the manual acceptance procedure and explicit approval gate.
- [ ] T032 Extend `.github/workflows/validate.yml` with pinned Node/pnpm setup, frozen dependency installation, build/type checks, unit tests, and isolated integration/acceptance tests. Preserve existing repository checks. Exclude generated/dependency directories from repository text scans after installation. Clean up only the test Compose project even when tests fail, and prevent container-test result caching.
- [ ] T033 Run the validation commands below and record commands, results, and any blockers in `specs/hedgeops-mvp/validation.md`. Do not mark failed or unexecuted checks as passed. Confirm that the existing containers and reference database remain unchanged.
- [ ] T034 [US1] [US2] [US3] [US4] Run the operator-approved manual acceptance procedure from `docs/operations/hedgeops-mvp.md` and record its result in `specs/hedgeops-mvp/validation.md`. Use a dedicated Hedgeops database and a separate subscriber to the existing publisher. Discover, approve, establish state, open, and close a physical device; verify both Telegram messages and database readings. Leave this task unchecked until the operator authorizes live access and completes the physical actions.

## Validation

These commands become available during implementation. No application tests were run when this task list was written.

```bash
pnpm install --frozen-lockfile
pnpm nx run-many -t build typecheck test

docker compose -p hedgeops-mvp-test -f compose.test.yaml up -d --wait
pnpm test:integration
pnpm test:acceptance
docker compose -p hedgeops-mvp-test -f compose.test.yaml down -v

git diff --check
```

- Expected result: installation, compilation, type checks, and tests succeed. Repository checks report no errors.
- Use only test connection settings for automated integration and acceptance checks. The cleanup command intentionally deletes only disposable test volumes.
- Verify initialization on an empty volume, migration reruns, role restrictions, and persistence without volume removal.
- Verify that the runtime user cannot change or delete raw data and cannot grant itself approval permissions.
- Verify adapter separation with core tests that need no PostgreSQL, MQTT service, or Telegram access.
- Require explicit operator authorization for live MQTT subscription and Telegram messages. Never write to the reference Supabase database.

### Requirement coverage

| Requirements | Tasks |
| --- | --- |
| FR-001 | T019–T020, T023–T025 |
| FR-002–FR-004 | T007, T009–T010, T016–T018, T028–T030 |
| FR-005–FR-006 | T005, T011–T012, T029 |
| FR-007–FR-010 | T007, T011–T014, T028–T029 |
| FR-011–FR-012 | T013–T018, T029 |
| FR-013–FR-018 | T004, T007, T011–T012, T015–T017, T028–T029 |
| FR-019–FR-021 | T009–T010, T016–T018, T023–T026, T029–T030 |
| FR-022–FR-023 | T019–T022, T029–T031 |
| FR-024–FR-025 | T015, T021, T025, T028–T030 |
| FR-026–FR-028 | T022, T025–T028, T030 |
| FR-029 | T005, T008, T010, T012, T014–T015, T017–T018, T020–T022, T024, T026–T030, T032–T033 |
| FR-030 | T031, T034 |

T006 makes the approved database provisioning requirement explicit. T026 verifies the storage adapter boundary. Neither task adds another database implementation to production or expands the product scope.
