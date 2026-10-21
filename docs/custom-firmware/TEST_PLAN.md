# Test Plan

## Purpose

A successful build is not evidence of correct physical behavior.

The initial required physical target is Keychron Q3 Ultra 8K ANSI.

## Test classes

- **A — Build/static:** compile, linker, footprint, configuration validation.
- **B — Physical Q3:** keys, RGB, encoder, OS switch, persistence.
- **C — Host/transport:** USB/BT/2.4 GHz topology and power state.
- **D — Launcher:** official Keychron Launcher behavior.
- **E — Recovery/destructive:** requires a verified recovery path.

# M0 stock baseline

## BUILD-M0-001
Build stock-equivalent Q3 from `custom/main`.

Record:
- ZMK SHA;
- Zephyr SHA;
- Realtek HAL SHA;
- frozen west manifest;
- build image/toolchain;
- artifact names/hashes;
- FLASH usage;
- RAM usage.

## HW-M0-001 — USB
Verify cable boot, typing, encoder and normal stock HID behavior.

## HW-M0-002 — Physical OS switch
Verify Mac and Windows positions behave like stock.

## HW-M0-003 — Bluetooth
Verify pairing, reconnect, typing and basic sleep/wake.

## HW-M0-004 — 2.4 GHz
Verify connect, typing, reconnect and stock RGB behavior.

## HW-M0-005 — RGB
Verify stock enable/disable, brightness, effects and Fn RGB controls.

## LAUNCHER-M0-001
Verify official Launcher detects the Q3.

## LAUNCHER-M0-002
Verify representative settings:
- polling/report rate;
- debounce/bounce;
- stock RGB controls;
- dynamic keymap edit;
- persistence where stock supports it.

## RECOVERY-M0-001
Physically verify return to official stock firmware.

Only after this passes should risky custom flashing work begin.

# Resource regression

## RES-TEST-001
Capture stock FLASH usage.

## RES-TEST-002
Capture stock RAM usage.

## RES-TEST-003
For significant PRs compare resource use to the current known-good `custom/main`.

# Profiles and persistence

## PROFILE-TEST-001
Switch between explicit profiles. Selecting the active profile is a no-op.

## PROFILE-TEST-002
Set Windows Gaming active, switch to Mac, return to Windows.
Expected: Windows Gaming restored.

## PERSIST-TEST-001
Reboot and power-cycle.
Expected: active profile per OS restored.

## RESET-TEST-001
Perform documented factory/config reset.
Expected: custom persistent state returns to compiled defaults.

# F-row

## FROW-TEST-001
Verify primary F1–F12 and Fn secondary actions.

## FROW-TEST-002
Trigger the F-row switching action.
Expected: F1–F12 primary/secondary behavior reverses.

## FROW-TEST-003
Reboot/power-cycle.
Expected: F-row state preserved.

## FROW-TEST-004
Verify Escape/Print Screen/Delete do not change unintentionally.

## FROW-TEST-005
Verify independent F-row state for two profiles sharing one functional layer.

## FROW-TEST-006
Verify dedicated functional-layer case if/when configured.

# Per-profile RGB

## RGB-PROFILE-TEST-001
Use visibly different RGB states for two profiles and switch between them.
Expected: each profile restores its own state.

## RGB-PROFILE-TEST-002
Reboot.
Expected: profile RGB persists once implemented.

## RGB-LAUNCHER-TEST-001
If Launcher RGB remains supported, change RGB while one profile is active and verify the intended scope.
If unsupported by design, document the limitation explicitly.

# Fn semantic RGB

## RGB-FN-TEST-001
Without Fn held, base-layer media/system actions use ordinary profile RGB.

## RGB-FN-TEST-002
Hold Fn.
Expected:
- ordinary keys inherit base RGB;
- classified actions use semantic colors.

## RGB-FN-TEST-003
Move a classified action to another physical key.
Expected: semantic color follows the binding.

## RGB-FN-TEST-004
Verify semantic F-row colors after F-row switching.

## RGB-PROFILESEL-TEST-001
Hold the layer containing profile selectors.
Expected:
- inactive selectors use profile-selector color;
- active profile selector blinks white.

# Indicators and masks

## IND-TEST-001
Enable Caps indicator in one profile and verify ON/OFF overlay behavior.

## IND-TEST-002
Disable Caps indicator in another profile and verify no custom overlay appears.

## IND-TEST-003
Overlap a Caps overlay with an Fn semantic key.
Expected: Fn wins only on the overlapping LED.

## MASK-TEST-001
In the initial Gaming profile:
- Windows key produces no action;
- its LED is OFF.
Switch to Normal and verify normal behavior returns.

# Ctrl + End

## CTRLEND-TEST-001
Press and hold configured key.
Expected:
- Left Ctrl DOWN;
- End tap exactly once;
- Left Ctrl remains DOWN.

## CTRLEND-TEST-002
Release physical key.
Expected: Left Ctrl UP.

## CTRLEND-TEST-003
Repeat normal presses.
Expected: one End tap per press; no looping.

## CTRLEND-TEST-004
Exercise fast taps/profile transitions.
Expected: no stuck Ctrl or End.

# Host/power

Use the real topology:

```text
Q3 / receiver
→ powered USB hub
→ monitor/KVM
→ PC/laptop
```

## POWER-USB-001
Cable + active host → RGB normal.

## POWER-USB-002
Host unavailable/suspended while VBUS remains → target RGB OFF.

## POWER-USB-003
Resume/reconnect → previous logical RGB restored.

## POWER-USB-004
Disconnect USB upstream while powered hub remains → RGB OFF.
Reconnect to active host → RGB restored.

## POWER-24G-001
2.4 GHz + active host → RGB normal.

## POWER-24G-002
Host shuts down while receiver remains powered → target RGB OFF.

## POWER-24G-003
Stock reference observation:
when host is unavailable and RGB is unexpectedly ON, unplugging the receiver causes RGB to turn OFF.

## POWER-MIXED-001
Active 2.4 GHz or BT + USB charging only + inactive USB data host.
Expected: inactive USB must not force RGB OFF.

# Launcher regression smoke test

After milestones touching keymap, RGB, persistence, HID, USB descriptors or settings:
- detect keyboard;
- read representative settings;
- write representative settings;
- reboot;
- verify firmware operation.

# Q3 v1 release gate

A Q3 v1 candidate must have:
- clean build;
- acceptable FLASH/RAM footprint;
- verified recovery;
- profile persistence;
- F-row persistence;
- intended RGB behavior;
- no stuck-key regressions;
- cable power behavior tested;
- 2.4 GHz behavior documented;
- Launcher compatibility status documented;
- known deviations recorded.
