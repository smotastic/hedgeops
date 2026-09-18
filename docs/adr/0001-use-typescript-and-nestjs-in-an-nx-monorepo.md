---
number: "0001"
title: "Use TypeScript and NestJS in an Nx monorepo"
status: "Accepted"
date: "2026-09-18"
supersedes: null
superseded_by: null
---

# ADR-0001: Use TypeScript and NestJS in an Nx monorepo

## Context

HedgeOps collects smart-home device data, stores raw messages,
maintains current device states, and controls devices.

The deployment target is a Raspberry Pi. Data collection,
processing, and local device control must work without internet
access. External services must be optional.

Backend applications and the future frontend will share one
repository, called a monorepo. A common language will reduce
the number of languages developers must use.

The backend needs a common application structure. Initial work
includes ingestion, refining, and automation. An API will follow.
These responsibilities do not determine service boundaries.

The hedgehome project supplies device knowledge and proven
behaviour. HedgeOps does not need to preserve its code or API.

## Decision

Use the following backend stack:

- TypeScript with strict type checks.
- Node.js, the program that executes backend application code,
  on a supported Long-Term Support release.
- NestJS as the common backend framework.
- pnpm for dependency installation and workspace packages.
- Nx for repository task management and project dependencies.
- Official Nx support for NestJS applications.

Use TypeScript for the future frontend as well. This decision
does not select a frontend framework.

Use pnpm workspaces to link packages in the repository.
Use Nx to run builds, tests, and checks in dependency order.
Nx can reuse saved task results and identify projects affected
by a change.

Nx Cloud is optional. Local development must not require it.

Support both workers and API applications with NestJS.
A worker is a backend program that processes messages or
performs tasks without waiting for web requests.

Workers may run without a web server. Add web endpoints only
when needed, for example for health checks.

Use shared NestJS conventions for modules and dependency
injection, which supplies components with the objects they need.

Separate ADRs will define:

- Service boundaries and communication.
- Message delivery, retries, and recovery.
- Database technology and data retention.
- Frontend framework and tools.
- Deployment details.

This decision does not require a separate service for each
responsibility. It also does not guarantee reliable message
delivery merely by selecting NestJS.

## Consequences

### Positive

- Backend and frontend developers use one language.
- Backend applications follow a common structure.
- Workers and API applications use the same framework.
- pnpm links local packages within one repository.
- Nx coordinates tasks across applications and shared packages.
- Local operation does not depend on hosted services.

### Negative

- NestJS adds framework concepts and setup code.
- Nx adds configuration and maintenance work.
- Compatible Node.js, NestJS, Nx, and pnpm versions must be
  maintained together.
- Application memory and CPU use must be checked on the Pi.
- Shared packages can make applications depend too closely on
  each other. Their boundaries need review.
- TypeScript checks do not validate incoming device data while
  the application runs. Separate input checks are still needed.

## Alternatives considered

### Python backend with a TypeScript frontend

This follows the broad approach used in hedgehome.
It was not selected because HedgeOps wants one application
language across backend and frontend.

### TypeScript with smaller libraries instead of NestJS

This can reduce framework setup and give each application
more freedom. It was not selected because a common backend
structure is preferred over separate application conventions.

### pnpm workspaces without Nx

This requires fewer tools and may be enough for a small
repository. It was not selected because coordinated tasks,
project dependency tracking, and reuse of task results are
desired from the start.

### One full-stack framework for backend and frontend

This could combine web application development under one
framework. It was not selected because backend workers must
not depend on a frontend framework choice.

## Links

- [Project foundation](../project-foundation.md)
