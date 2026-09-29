# ESP32 DNS Filtering Appliance

An ESP-IDF firmware-learning project for a small LAN DNS filtering appliance, written primarily in C. The goal is to learn DNS packet handling, FreeRTOS concurrency, bounded memory management, networking recovery, and persistent configuration through incremental implementation and testing.

## Status

**Planning stage.** The architecture and milestones are documented locally, but firmware, ESP-IDF build configuration, and automated tests are not implemented. The board/module and ESP-IDF release have not been selected. No hardware behavior or performance has been verified.

## Planned Behavior

1. Receive DNS queries from LAN clients.
2. Check the requested domain against a blocklist.
3. Return a local blocked response or forward the request upstream.
4. Cache eligible valid responses while respecting their TTLs.
5. Collect basic request and health statistics.
6. Expose a small LAN configuration and status interface.

The initial design uses IPv4 UDP transport, exact-name blocking with REFUSED responses, a small compiled blocklist, and conservative positive A/AAAA caching. TCP support and explicit EDNS/large-response handling are required before broader deployment. Querying AAAA records does not require IPv6 transport.

## Architecture

```text
Client -> validate -> blocklist -> cache -> upstream resolver
             |           |          |             |
             +-----------+----------+-------------+-> response
```

One DNS task owns sockets, pending requests, active filtering policy, cache, and counters. Management communicates through bounded queues and copied snapshots. Settings use NVS; the cache and frequently updated counters remain in RAM. Long operations and flash writes stay outside DNS processing.

The appliance forwards to an upstream recursive resolver. Recursive resolution, DHCP service, DNSSEC validation, encrypted DNS, automatic blocklist downloads, and OTA are deferred.

## Repository Layout

| Path | Intended role |
| --- | --- |
| `main/` | Application startup and wiring. |
| `components/` | DNS, networking, policy, cache, and management modules. |
| `tests/` | Host tests, target tests, and integration tools. |
| `hardware/datasheets/` | Hardware reference documents. |
| `docs/` | Local design and planning notes. |

Implementation directories are currently scaffolding; empty directories are not preserved by Git.

## Development Roadmap

1. Select the board and pin ESP-IDF; establish build, flash, and monitor steps.
2. Implement network lifecycle and a tested DNS protocol core.
3. Add asynchronous UDP forwarding and exact-name filtering.
4. Add conservative caching, then TCP and DNS compatibility handling.
5. Add management, persistence, and overload/recovery testing.

No working build or test command is available yet. Add reproducible commands when the project configuration exists. Start with one manually configured test client before changing router-wide DNS settings.

## Local Planning Documents

The local `docs/` folder contains `requirements.md`, `architecture.md`, `decisions.md`, and `milestones.md`. The milestones document includes the session handoff and next decisions.

`docs/`, `AGENTS.md`, and `.gitignore` are intentionally excluded from Git in this workspace. They may be absent from a fresh clone and require separate backup.

## Limitations

Clients using other resolvers or encrypted DNS can bypass filtering. Domain filtering cannot distinguish ads from desired content served by the same domain. Hardware capacity and performance targets remain to be measured.
