---
number: "0005"
title: "Use hexagonal architecture"
status: "Accepted"
date: "2026-09-18"
supersedes: null
superseded_by: null
---

# ADR-0005: Use hexagonal architecture

## Context

Device rules and automation must not depend on MQTT clients,
notification providers, or an application framework.

Telegram must be replaceable with another provider without
changing automation criteria.

These rules must remain useful if the application later runs
as several services.

## Decision

Use hexagonal architecture for backend applications.
This means keeping application rules separate from the code
that connects them to external systems.

Structure each module around:

- Domain logic: concepts and rules.
- Application logic: operations that coordinate those rules.
- Ports: interfaces through which application logic receives
  work or requests external operations.
- Adapters: implementations that connect ports to specific
  technologies.

### Driving adapters

Driving adapters start application operations through input
ports.

An MQTT subscriber is a driving adapter. It translates received
messages into application input.

A future API controller can also be a driving adapter.

### Driven adapters

Driven adapters implement output ports called by application
logic.

A Telegram sender is a driven adapter for a notification port.
A future Discord sender can implement the same port.

Automation criteria decide when to run an action.
A notification action calls the notification port.
Neither criteria nor action logic calls Telegram directly.

### Dependency rules

Define ports in the application core: domain and application
code.

Adapters depend on the core. The core must not import adapter
implementations, provider libraries, or NestJS.

Connect ports to adapters in startup code outside the core.
For APIs, NestJS can perform this setup.

Call other modules through their published application
interfaces. Do not access their internal state or adapters.

Do not require an interface for every class. Use ports at
application boundaries.

### Exchangeable notifications

The notification port describes application needs, such as
a logical recipient and message.

Keep provider-specific destinations, payloads, and formatting
inside adapters and their configuration.

Select the adapter during application setup. Replacing
Telegram with Discord must not require changes to automation
criteria or notification action logic.

Adapters must follow the same success and failure contract.
Define its exact types during implementation.

This decision does not require runtime provider switching or
delivery through several providers at once.

## Consequences

### Positive

- Device and automation rules can be tested without external
  services or NestJS.
- Notification providers can change without changing rules.
- Framework and provider code have explicit boundaries.
- These boundaries can remain when deployment changes.

### Negative

- Ports, translation code, and startup setup add work.
- Provider-specific features may not fit the common contract.
- Developers must maintain dependency rules.
- Moving code into separate services still requires decisions
  about communication, delivery, and recovery.

## Alternatives considered

### Direct external calls from application rules

Not selected because provider changes would affect application
logic and make isolated testing harder.

### NestJS throughout the application core

Not selected because workers do not need NestJS.
API framework choices must not determine device rules.

### Interfaces for every class

Not selected because internal code does not always need an
exchangeable implementation.

## Links

- [ADR-0004](0004-start-with-one-worker-process.md)
