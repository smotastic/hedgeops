# HedgeOps Project Foundation

## Purpose and scope

HedgeOps is a smart-home system. Its code lives in one repository, including device integration, data storage, an API, and a future user interface.

The system mainly uses Shelly devices. It collects device data and lets software change device settings through the HedgeOps API, the interface used by other software.

This document describes the project scope and direction. It is not a detailed specification or an Architecture Decision Record (ADR), which records an individual design decision.

## Device data

HedgeOps receives device data through MQTT, a messaging protocol, or other protocols.

A database stores all raw data received. HedgeOps processes this data to maintain the latest known state of each device. The system supports both historical data and current device states.

Device examples include:

- Shelly BLU Door/Window sensors.
- Shelly TRV radiator valves.

## Device control

Software uses the HedgeOps API to change device settings. For example, it can set the target temperature of a Shelly TRV radiator valve.

## Future user interface

A user interface will become part of HedgeOps later in the project. It will use the HedgeOps API.

## Possible scheduled tasks

Scheduled tasks could also use the API to change device settings. For example, a task could set the heating each day at 17:00 during winter months.

Scheduled tasks are a possible future feature, not a confirmed requirement.

## Decisions for future ADRs

Future ADRs will define:

- The database choice.
- Data retention: how long stored data is kept.

This document does not select either policy or technology.
