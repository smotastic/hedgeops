---
number: "0001"
title: "Use TypeScript and Nx for backend applications"
status: "Accepted"
date: "2026-09-18"
supersedes: null
superseded_by: null
---

# ADR-0001: Use TypeScript and Nx for backend applications

## Context

HedgeOps collects smart-home device data, stores raw messages,
maintains current device states, and controls devices.

The deployment target is a Raspberry Pi. Data collection,
processing, and local device control must work without internet
access. External services must be optional.

Backend applications and the future frontend will share one
repository, called a monorepo. A common language will reduce
the number of languages developers must use.

Initial backend work includes ingestion, refining, and automation.
Workers do not need an API framework. An API will follow and
will use common framework conventions.
These responsibilities do not determine service boundaries.

The hedgehome project supplies device knowledge and proven
behaviour. HedgeOps does not need to preserve its code or API.

## Decision

Use the following backend stack:

- TypeScript with strict type checks.
- Node.js, the program that executes backend application code,
  on a supported Long-Term Support release.
- Plain Node.js for workers and NestJS for API applications.
- pnpm for dependency installation and workspace packages.
- Nx for repository task management and project dependencies.
- Official Nx support for NestJS API applications.

Use TypeScript for the future frontend as well. This decision
does not select a frontend framework.

Use pnpm workspaces to link packages in the repository.
Use Nx to run builds, tests, and checks in dependency order.
Nx can reuse saved task results and identify projects affected
by a change.

Nx Cloud is optional. Local development must not require it.

Use plain Node.js for workers. Use NestJS for API applications.
An API is an interface for other applications.

A worker processes messages and tasks without serving web requests.
Worker code does not require NestJS.

Connect worker components through explicit startup code.
Use NestJS modules and dependency injection for API applications.
Dependency injection supplies components with the objects they need.

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
- API applications follow common NestJS conventions.
- Workers do not need an API framework.
- pnpm links local packages within one repository.
- Nx coordinates tasks across applications and shared packages.
- Local operation does not depend on hosted services.

### Negative

- NestJS adds framework concepts and setup code to APIs.
- Worker startup code must connect components explicitly.
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

### NestJS for all backend applications

This would give workers and APIs the same framework conventions.
It was not selected because workers do not need NestJS.
Explicit startup code can connect their components.

### TypeScript with smaller libraries for APIs

This can reduce framework setup and give each API application
more freedom. It was not selected because common NestJS
conventions are preferred for APIs.

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
