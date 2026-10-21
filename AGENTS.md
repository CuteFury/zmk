# AGENTS.md

This repository is a fork of Keychron's ZMK firmware.

Custom development currently targets **Keychron Q3 Ultra 8K ANSI only**. Other keyboards may be supported later, but Q3 is the only required hardware validation target for the initial project.

## Read first

Before making firmware changes, read:

- `docs/custom-firmware/README.md`
- `docs/custom-firmware/ARCHITECTURE.md`
- `docs/custom-firmware/FEATURES.md`
- `docs/custom-firmware/TEST_PLAN.md`
- `docs/custom-firmware/FLASHING.md`
- `docs/custom-firmware/UPSTREAM.md`

## Branch rules

- `rtl8762g` mirrors the Keychron vendor branch. Do not add custom commits to it.
- Custom development targets `custom/main`.
- Create focused feature/fix branches from `custom/main`.
- Do not update the vendor baseline implicitly while implementing an unrelated feature.
- Do not add V0 or other keyboard targets until Q3 is stable unless the task explicitly requires it.

## Engineering rules

- Audit the current Keychron/ZMK implementation before changing it.
- Prefer existing ZMK, Zephyr, and Keychron mechanisms over new subsystems.
- Keep vendor/core changes small and localized.
- Do not copy large upstream subsystems unless there is a demonstrated need.
- Preserve the physical Mac/Windows switch.
- Preserve official Keychron Launcher compatibility where reasonably possible, but do not distort the firmware architecture solely to satisfy unsupported Launcher behavior.
- Do not introduce a custom HID protocol or a new storage format unless existing mechanisms have been audited and shown insufficient.
- Treat USB, DFU, boot, flash layout, persistent storage, receiver firmware, Zephyr, and HAL changes as high risk.
- Do not claim hardware behavior was verified from build results alone.

## Scope discipline

Implement documented requirements, not speculative improvements.

The core should be reusable where natural, but do not over-generalize at the expense of Q3 delivery. Q3-specific matrix/LED/key-position information belongs in board/shield-specific configuration where practical.

## Verification

For each functional change:

1. identify the related requirement IDs in `FEATURES.md`;
2. update or reference tests in `TEST_PLAN.md`;
3. build the Q3 target;
4. report FLASH/RAM/storage impact when material;
5. state which checks require physical hardware;
6. preserve a known recovery path.

## Lower-layer escalation

Prefer changes in this order:

1. ZMK;
2. Keychron Zephyr;
3. Realtek HAL;
4. receiver firmware, if applicable.

Escalate only after showing why the higher layer is insufficient.

## Plans

For large or risky changes, an ExecPlan may be added under `.agent/PLANS.md` or another explicitly documented planning location.
