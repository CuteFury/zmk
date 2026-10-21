# Custom Keychron Firmware

## Purpose

This project extends Keychron's ZMK firmware for the **Keychron Q3 Ultra 8K ANSI** while preserving as much stock behavior and official Keychron Launcher compatibility as is reasonable.

The first goal is not to add features. The first goal is to reproduce and verify a completely stock-equivalent Q3 build from this fork.

## Current target

Primary and only required initial hardware target:

- Keychron Q3 Ultra 8K ANSI

Potential later targets:

- Keychron V0 Ultra 8K
- other compatible Keychron Q/V Ultra boards

Those later targets must not delay Q3 development.

## Current milestone

### M0 — Stock Reproduction

Before custom functionality is introduced:

1. build Q3 from this fork;
2. flash the physical keyboard;
3. verify stock-equivalent behavior;
4. verify official Keychron Launcher compatibility;
5. verify and document recovery to official stock firmware;
6. capture FLASH/RAM/storage baseline;
7. freeze the resolved west dependency revisions.

No custom firmware feature should be considered started until M0 is complete.

## Design principles

- Audit first; do not assume earlier code analysis is still current.
- Prefer existing ZMK/Keychron/Zephyr mechanisms.
- Keep changes small and individually testable.
- Preserve the physical Mac/Windows switch.
- Treat build success and hardware success as separate things.
- Keep recovery documented before risky work.
- Keep custom protocol/storage work out of scope until existing mechanisms are proven insufficient.
- Design reusable core mechanisms where natural, but validate only on Q3 initially.

## Documentation

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — system model and subsystem boundaries
- [`FEATURES.md`](FEATURES.md) — functional and non-functional requirements
- [`TEST_PLAN.md`](TEST_PLAN.md) — automated and physical verification
- [`FLASHING.md`](FLASHING.md) — build/flash/recovery procedures and safety notes
- [`UPSTREAM.md`](UPSTREAM.md) — remotes, baseline, dependencies, upstream policy

## Explicit non-goals for v1

- custom WebHID launcher;
- custom HID protocol unless required by a proven blocker;
- full V0 support;
- wholesale migration to current upstream ZMK/Zephyr;
- replacing the Keychron RGB stack when extension is sufficient.
