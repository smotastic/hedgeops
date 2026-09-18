---
number: "0002"
title: "Use PostgreSQL for storage and event processing"
status: "Accepted"
date: "2026-09-18"
supersedes: null
superseded_by: null
---

# ADR-0002: Use PostgreSQL for storage and event processing

## Context

HedgeOps collects smart-home device messages through MQTT,
a messaging protocol. It converts device-specific data into
events that other parts of the system can use.

Collection and processing must work locally on a Raspberry Pi
without internet access.

Messages can contain invalid or unsupported data. Processing
can also stop because of a failure or restart. The system must
preserve stored messages and identify unfinished work.

The minimum viable product (MVP) needs raw data for fault
investigation and later processing changes.

## Decision

Use PostgreSQL to store raw messages, processed events, and
current device states.

Deployment is defined in ADR-0003. Backups and disaster recovery
are outside this decision.

### Store before processing

Store each received MQTT message before interpreting its
device data or using it to update device state.

Preserve the original payload as bytes, including invalid data.
Also record a unique record ID, topic, receipt time, and
available delivery information.

Do not change the stored payload during processing.

Database storage alone does not guarantee delivery from MQTT.
A separate decision must define MQTT delivery settings,
acknowledgements, and behaviour when storage is unavailable.

### Convert messages into common events

Process stored messages into a common event format.

The format must identify:

- The source raw message.
- The device and event type.
- The time HedgeOps received the message.
- The device-reported event time, when available.
- The event data and format version.

One raw message can produce zero, one, or multiple events.
Record a reason when a message produces no event.

Keep device-specific interpretation separate from the common
event format. Define exact event fields during implementation.

### Track processing and recovery

Store processing status in PostgreSQL. Distinguish messages
that are pending, in progress, complete, or failed.

Record attempt counts and failure details. Invalid or
unsupported messages must remain visible for investigation.

Use a database transaction, a group of changes that either
all succeed or all fail, to save events and mark their source
message complete together.

Any device-state changes made in that processing step must
be part of the same transaction.

After a restart, resume pending work and recover abandoned
in-progress work. A message must not remain stuck merely
because its processor stopped.

Repeated processing attempts must not create duplicate events
for the same source message. This does not remove duplicate
messages delivered separately through MQTT.

Define retry timing and duplicate-delivery handling separately.
This decision does not guarantee that each device action
occurs exactly once.

### Keep raw messages for the MVP

Keep all raw messages during the MVP, including failed and
successfully processed messages.

Do not automatically delete raw messages. Define deletion
rules in a later ADR.

Monitor storage use.

## Consequences

### Positive

- One database supports raw data, events, and device states.
- Stored messages remain available for investigation.
- Processing can resume after a restart.
- Transactions prevent partially saved processing results.
- Common events reduce dependence on device-specific formats.
- Local operation does not require a hosted database.

### Negative

- PostgreSQL needs deployment and updates.
- Raw message storage will grow until deletion rules exist.
- Database writes add work for the Raspberry Pi and its storage.
- Processing claims, retries, and recovery need implementation
  and tests.
- Keeping raw data requires access controls because messages
  can reveal activity in the home.

## Alternatives considered

### SQLite

SQLite needs less database administration. PostgreSQL is
preferred for shared access by collectors, processors, and
future applications.

### Process messages before storing them

This reduces raw storage needs. It was not selected because
processing failures could discard useful source data.

### Keep unfinished work only in memory

This is simpler but loses processing status after a restart.

### Add a separate message queue immediately

A queue could distribute processing work. It was not selected
for the MVP because stored work in PostgreSQL avoids another
service to operate.

### Delete raw messages after processing

This limits storage growth. It was not selected because raw
messages are needed for investigation and processing changes
during the MVP.

## Links

- [Project foundation](../project-foundation.md)
- [ADR-0001: Backend stack](0001-use-typescript-and-nestjs-in-an-nx-monorepo.md)
