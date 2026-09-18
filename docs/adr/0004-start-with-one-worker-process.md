---
number: "0004"
title: "Start with one worker process"
status: "Accepted"
date: "2026-09-18"
supersedes: null
superseded_by: null
---

# ADR-0004: Start with one worker process

## Context

HedgeOps must receive device messages, maintain current device
state, and run automations on a Raspberry Pi.

The initial automation detects a window opening and sends a
Telegram notification.

These responsibilities need clear code boundaries. They do not
yet need separate running applications.

## Decision

Run the initial application as one plain Node.js worker.
A worker processes messages and tasks without serving web requests.

Use separate modules for:

- MQTT ingestion. MQTT is the messaging protocol used to
  receive device messages.
- Event refining and current device state. Refining converts
  device messages into a common form.
- Automation criteria and actions.

Event processing maintains device state. It does not send
notifications or own automation rules.

The automation module evaluates criteria against state changes.
When criteria match, it runs the configured actions.
Notification delivery is one such action.

For the first window automation, trigger on a transition from
closed to open. Repeated reports of the open state must not
trigger repeated notifications.

An initial observation of an open window is not a known
closed-to-open transition. Startup and recovery behaviour need
separate requirements.

Use Telegram as the first notification provider.

Communicate between modules through explicit in-process
interfaces. Do not add an API, another message queue, or
separate services yet.

Use one Nx application project to start the worker.
Nx manages repository projects and their tasks.
Code can live in separate Nx library projects.
This decision does not require one Nx project per module.

An external notification failure must not stop local ingestion,
state updates, or automation evaluation.

This decision does not guarantee durable action delivery.
Define retries, recovery, and delivery guarantees separately.

Consider separate services only when there is a demonstrated
need, such as independent deployment, different resource needs,
or protection from failures in another module.

## Consequences

### Positive

- Initial deployment uses one application process.
- Event processing and automation have separate responsibilities.
- Notifications are actions, not device-state rules.
- Module communication does not need another message queue.

### Negative

- Modules share process resources and deployment.
- A process failure stops all modules.
- Slow external actions need limits so they do not delay
  local processing.
- Moving modules into separate services will require new
  communication and failure-handling decisions.

## Alternatives considered

### Separate event and automation workers

Not selected because the initial application does not yet need
independent deployment or process isolation.

### Notification delivery inside event processing

Not selected because event processing should describe device
state. Automation decides which actions follow from that state.

### Add an API or another message queue now

Not selected because no current requirement justifies them.

## Links

- [ADR-0001](0001-use-typescript-and-nestjs-in-an-nx-monorepo.md)
- [ADR-0005](0005-use-hexagonal-architecture.md)
