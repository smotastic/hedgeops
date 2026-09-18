---
number: "0003"
title: "Run PostgreSQL in Docker"
status: "Accepted"
date: "2026-09-18"
supersedes: null
superseded_by: null
---

# ADR-0003: Run PostgreSQL in Docker

## Context

HedgeOps uses PostgreSQL for storage and event processing.

The system runs locally on a Raspberry Pi. Normal operation
must not require internet access.

For the minimum viable product (MVP), it is acceptable for
collection and database access to stop when the Pi stops.

Database installation and settings should be defined in
repository files rather than through manual installation.

## Decision

Run PostgreSQL in Docker on the same Raspberry Pi as the
HedgeOps application.

Docker runs software in a container, a packaged process
with its required software.

Use the official PostgreSQL container image, the package
Docker uses to create the container. Select a version that
supports the Pi's processor and operating system.

Record the selected image version and container settings
in the repository. Do not use the moving `latest` tag.
Keep database passwords outside version control.

Store database files outside the container's writable
layer, in storage that remains when the container is
replaced. Routine container replacement must not delete
database files.

After installation, database operation must not require
internet access.

This decision covers PostgreSQL only. It does not require
application components to run in Docker.

Backups, disaster recovery, and recovery time targets are
out of scope. Restart handling for unfinished messages
remains part of ADR-0002.

## Consequences

### Positive

- Database installation is defined in repository files.
- PostgreSQL does not need direct installation into the
  Pi's operating system.
- Container replacement preserves database files.
- Application access does not depend on another machine.
- Normal operation remains local.

### Negative

- Docker adds software to install and maintain.
- The application and database share the Pi's resources.
- A Pi failure stops both application and database access.
- Persistent files do not protect against storage failure.
- PostgreSQL version changes still need compatibility
  checks. Replacing a container does not automatically
  upgrade existing database files.

## Alternatives considered

### Install PostgreSQL directly on the Pi

This avoids Docker. It was not selected because installation
and settings should be defined through container files
in the repository.

### Run PostgreSQL on another local machine

This separates database resources from the application.
It was not selected because it adds another machine and
a network dependency for the MVP.

### Use a hosted PostgreSQL service

This was not selected because local operation must not
depend on internet access.

## Links

- [ADR-0002: Storage and event processing](0002-use-postgresql-for-storage-and-event-processing.md)
