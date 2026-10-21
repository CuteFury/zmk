# Upstream and Dependency Policy

## Repository roles

### Development fork

```text
origin = https://github.com/CuteFury/zmk
```

Primary custom branch:

```text
custom/main
```

### Primary vendor upstream

```text
keychron = https://github.com/Keychron/zmk
branch   = rtl8762g
```

This is the production firmware base for Q3.

### Reference ZMK upstream

```text
zmk = https://github.com/zmkfirmware/zmk
branch = main
```

Use it as a reference source for bug fixes, newer behaviors, architecture ideas and selective backports.

Do not merge `zmk/main` wholesale into the custom firmware without a dedicated migration project.

# Current bootstrap baseline

At Phase A completion:

```text
Keychron/zmk:rtl8762g
7f8d16732a3782e237915bff30a1f67de09268b7

CuteFury/zmk:rtl8762g
7f8d16732a3782e237915bff30a1f67de09268b7

CuteFury/zmk:custom/main
7f8d16732a3782e237915bff30a1f67de09268b7
```

Thus `custom/main` initially contains no custom firmware changes.

# Vendor dependency preservation

Preservation forks:

```text
CuteFury/zephyr
CuteFury/hal_realtek
```

Relevant branches currently preserved:

```text
CuteFury/zephyr:rtl87x2g
2c11634850f7d6b04e55a253d4506a6e7c8bd673

CuteFury/hal_realtek:2.4G
3c8e0bf787d80bc98c0fe4041f660ee1bf091206
```

These exist primarily as supply-chain/recovery preservation copies.

The normal M0 build should still use the dependencies declared by the unmodified Keychron manifests unless there is a reason to change them.

Do not redirect `west.yml` to preservation forks just because they exist.

# West manifest freeze

Because vendor branches may move, every verified baseline must store a frozen manifest.

First baseline location:

```text
docs/custom-firmware/baselines/q3-stock-v1/west-frozen.yml
```

Generate using:

```bash
west manifest --freeze
```

Record:
- ZMK SHA;
- Zephyr SHA;
- Realtek HAL SHA;
- all resolved west project SHAs;
- build image/toolchain version;
- firmware artifact hash if practical.

# Branch policy

## `rtl8762g`

Purpose: mirror/reference of the Keychron vendor branch.

Rules:
- no custom documentation commits;
- no custom firmware commits;
- no opportunistic fixes.

If Keychron advances by fast-forward, update intentionally.

If the update is non-fast-forward or history is rewritten:
1. stop;
2. record the previous known SHA;
3. compare history/tree changes;
4. decide explicitly how the mirror should move.

Never force-update blindly.

## `custom/main`

Purpose: integration branch for this project.

Rules:
- starts from a known Keychron baseline;
- receives docs and custom firmware work;
- should remain buildable;
- feature/fix branches are based from it.

# Recommended local remotes

ZMK checkout:

```text
origin    → CuteFury/zmk
keychron  → Keychron/zmk
zmk       → zmkfirmware/zmk
```

For the west-managed Zephyr checkout, reference remotes may be added separately if useful:

```text
keychron → Keychron/zephyr
zephyr   → zephyrproject-rtos/zephyr
```

Zephyr is a separate repository and should not be added as a Git remote inside ZMK.

# Updating the Keychron baseline

Do not mix vendor updates with unrelated feature work.

Recommended flow:
1. fetch `keychron/rtl8762g`;
2. compare to the current tracked vendor baseline;
3. audit relevant changes;
4. build the stock vendor target;
5. run physical smoke tests;
6. integrate intentionally into `custom/main`;
7. rerun custom regressions.

A vendor update is its own integration task.

# Selective backports

Newer upstream ZMK/Zephyr may contain useful fixes.

Use them as research/reference sources.

Selective backports require:
- identifying the original fix;
- understanding prerequisites;
- minimizing copied code;
- documenting why the backport is needed;
- physical regression testing.

Avoid accidental partial migration to a newer ZMK/Zephyr generation.

# Lower-layer forks

Do not create active custom branches in Zephyr or HAL preemptively.

Only modify them if an audited requirement cannot be solved correctly in ZMK.

If lower-layer modification becomes necessary:
1. create a dedicated branch in the relevant preservation fork;
2. document why ZMK is insufficient;
3. update the manifest intentionally;
4. add lower-layer build/recovery tests;
5. treat future upstream integration as additional maintenance burden.

# Offline preservation

GitHub forks are not a complete backup strategy.

After the first verified baseline, keep mirror clones of at least:
- ZMK;
- Keychron Zephyr;
- Keychron Realtek HAL.

Example:

```bash
git clone --mirror <repo-url>
```

Store mirrors outside the primary GitHub account when practical.
