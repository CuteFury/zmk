# Building, Flashing and Recovery

## Status

This document intentionally separates **verified procedure** from **planned procedure**.

Do not turn assumptions into instructions. Update this file only after the relevant step has been tested on the physical Q3.

# Safety rules

Before flashing experimental firmware:
- confirm the exact target is Keychron Q3 Ultra 8K ANSI;
- preserve a known official stock firmware source/image;
- know how to enter the documented recovery/DFU path;
- do not change boot/DFU/partition layout during ordinary feature work;
- do not experiment with Zephyr/HAL/receiver firmware without a separate recovery plan.

Build success does not prove an artifact is safe to flash.

# Baseline source

Initial ZMK baseline:

```text
Keychron/zmk:rtl8762g
7f8d16732a3782e237915bff30a1f67de09268b7
```

Custom development branch starts from the same commit:

```text
CuteFury/zmk:custom/main
7f8d16732a3782e237915bff30a1f67de09268b7
```

Preservation copies currently contain:

```text
CuteFury/zephyr:rtl87x2g
2c11634850f7d6b04e55a253d4506a6e7c8bd673

CuteFury/hal_realtek:2.4G
3c8e0bf787d80bc98c0fe4041f660ee1bf091206
```

These are preservation references. M0 must still record the exact revisions resolved by west.

# Build environment

Preferred initial strategy:
- use the same generation of ARM build environment as the Keychron vendor branch;
- initialize west from `app`;
- run `west update`;
- freeze the resulting manifest;
- build Q3 with no custom functional changes.

Expected general shape:

```bash
west init -l app
west update
west zephyr-export
west build -s app -p -b <board> -- -DSHIELD=<q3-shield>
```

**TODO M0:** replace placeholders with the exact Q3 board/shield command verified against the current Keychron workflow.

# Dependency freeze

After the first successful workspace resolution:

```bash
west manifest --freeze
```

Store the result as:

```text
docs/custom-firmware/baselines/q3-stock-v1/west-frozen.yml
```

Also record:
- ZMK SHA;
- Zephyr SHA;
- Realtek HAL SHA;
- build image/toolchain version.

# Stock-equivalent flash

**TODO M0 — physically verify before documenting as final.**

Document:
1. exact artifact;
2. exact Q3 mode required for flashing;
3. host-side tool/procedure;
4. expected success indications;
5. expected reboot behavior;
6. post-flash smoke test.

# Recovery to official stock firmware

**TODO M0 — must be physically verified before feature implementation.**

Document:
1. official stock firmware source;
2. exact method to enter recovery/DFU;
3. flashing method;
4. expected result;
5. verification that USB, OS switch and Launcher work after recovery.

Recovery is not considered verified until executed successfully at least once.

# Persistent-state reset

Identify which Keychron factory/config reset should clear:
- custom active-profile state;
- F-row state;
- per-profile RGB;
- indicator policy.

**TODO:** verify interaction with Keychron dynamic-keymap/settings reset before implementation.

# High-risk changes

Create a separate plan before changes involving:
- flash partitions;
- bootloader/DFU;
- recovery-critical USB descriptors;
- Zephyr platform code;
- Realtek HAL;
- 2.4 GHz receiver firmware.
