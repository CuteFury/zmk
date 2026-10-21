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
- RAM usage;
- persistent-storage capacity, existing Keychron consumption where measurable, and available headroom;
- representative input/report reliability and responsiveness evidence with stock RGB active, including stable high-rate USB operation, for later comparisons.

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

## BASELINE-M0-001
After all M0 build, physical, Launcher and recovery checks pass, tag the fully verified stock-equivalent source revision (STOCK-005).
Record the known-good tag with the source/dependency revisions, frozen west manifest and firmware artifact hash where practical.

# Resource regression

## RES-TEST-001
Capture stock FLASH usage.

## RES-TEST-002
Capture stock RAM usage.

## RES-TEST-003
For significant PRs report approximate FLASH/RAM/persistent-storage deltas against the current known-good `custom/main`, including remaining storage headroom (RES-003, RES-004).

# Performance regression

## PERF-TEST-001
After feature changes, compare representative input/report reliability and responsiveness with the M0 evidence under comparable host, transport, report-rate and test conditions (PERF-001 through PERF-003).
Exercise typing, held keys/modifiers and rapid press/release while RGB effects are active, semantic RGB recomputation occurs once implemented, and persistent settings are changed. Include stable high-rate USB operation.
Check for missed/duplicated reports, stuck keys and material responsiveness regressions. Record observations and timing measurements where available; do not assume numerical latency thresholds before measurement and intentional selection.

## PERF-TEST-002
Audit or instrument the implemented RGB, persistence and diagnostic paths as appropriate (PERF-003, PERF-004).
Verify that expensive state/keymap recomputation follows relevant events where practical rather than unnecessary continuous rescanning, and that storage/diagnostic work does not unnecessarily block input/report processing.

# Profiles and persistence

## PROFILE-TEST-001
Switch between explicit profiles. Selecting the active profile is a no-op.

## PROFILE-TEST-002
Set Windows Gaming active, switch to Mac, return to Windows.
Expected: Windows Gaming restored.

## PERSIST-TEST-001
Reboot and power-cycle.
Expected: active profile per OS restored.

## PERSIST-TEST-002
Once the storage mechanism is selected, audit custom setting names/keys or equivalent isolation boundaries for collisions with existing Keychron/Launcher state.
Save/reload representative stock and custom settings and verify neither corrupts the other (PERSIST-006).

## PERSIST-TEST-003
With persistent configuration unchanged and pending legitimate writes settled, repeatedly exercise Fn press/release, lock-state changes and host loss/restoration.
Use audits or instrumentation appropriate to the selected mechanism to verify these transient events do not cause unnecessary custom persistent writes (PERSIST-007). Distinguish legitimate configuration changes and unrelated stock writes.

## PERSIST-TEST-004
When persistent profile indicator policy is implemented, configure different policies for two profiles, switch profiles, reboot and power-cycle.
Expected: each profile's indicator policy is restored independently (PERSIST-005).

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
Set different F-row states in two profiles sharing one functional layer, switch between them and verify the states remain independent.
Hold/release Fn in each active profile and verify momentary activation of its associated functional layer.

## FROW-TEST-006
Verify a profile using a dedicated functional layer: hold/release Fn and verify momentary activation routes to the active profile's associated layer, while its F-row state remains independent of other profiles.
Both FROW-TEST-005 and FROW-TEST-006 are required (PROFILE-006 through PROFILE-008, LAYER-001, LAYER-002, FROW-006 through FROW-008). Use a temporary test configuration/fixture if the shipping keymap does not exercise both topologies; a dedicated shipping profile is not required.

## FROW-TEST-007
Move the F-row switching binding away from its initial position and verify the action works there. Remove all switching bindings from that layer and verify it no longer exposes F-row switching.
The original key must follow its current binding (FROW-003 through FROW-005).

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
Expected: semantic color follows the binding; both the new and original positions reflect their current effective bindings.

## RGB-FN-TEST-004
Verify semantic F-row colors after F-row switching.

## RGB-FN-TEST-005
Place a supported but unclassified binding on the functional layer and activate that layer repeatedly.
Expected: deterministic fallback presentation for the same logical state (RGB-FN-008). The exact fallback color remains a configuration decision.

## RGB-PROFILESEL-TEST-001
Hold the layer containing profile selectors.
Expected:
- inactive selectors use profile-selector color;
- active profile selector blinks white.

## RGB-PROFILESEL-TEST-002
Relocate a profile-selector binding and verify explicit profile selection and selector presentation follow the new position while its functional layer is active.
The original position must follow its current binding (PROFILE-005, RGB-PROFILESEL-004).

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

## MASK-TEST-002
In a test configuration, apply no-action/disabled presentation to a different key, including in a profile other than Gaming.
Verify no action is emitted and the configured disabled presentation follows that key, while the original key follows its current binding (KEYMASK-001 through KEYMASK-003).

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

Before each active-host loss/restoration case below, set and record a non-default active profile, F-row state, profile RGB and indicator policy.
Verify host loss does not overwrite these values and restoration preserves all four (POWER-008), using observable binding/indicator behavior and state inspection where available rather than RGB appearance alone.

## POWER-USB-001
Cable + active host → RGB normal.

## POWER-USB-002
Host unavailable/suspended while VBUS remains → target RGB OFF.

## POWER-USB-003
Resume/reconnect → previous RGB presentation and all recorded POWER-008 state restored.

## POWER-USB-004
Disconnect USB upstream while powered hub remains → RGB OFF.
Reconnect to active host → RGB restored.

## POWER-BT-001
With Bluetooth as the active transport, exercise host disconnect/reconnect and applicable suspend/resume.
Verify RGB OFF while no usable host is available and restoration of RGB and all recorded POWER-008 state. Record any host/stack limitation affecting detection or verification.

## POWER-24G-001
2.4 GHz + active host → RGB normal.

## POWER-24G-002
Host shuts down while receiver remains powered → target RGB OFF.

## POWER-24G-003
Stock reference observation:
when host is unavailable and RGB is unexpectedly ON, unplugging the receiver causes RGB to turn OFF.

## POWER-24G-004
Restore the downstream host and reconnect if needed; verify RGB presentation and all recorded POWER-008 state are restored.
For 2.4 GHz loss/restoration cases, record any unmet target, supporting evidence and unresolved receiver/protocol limitation. A radio link to a powered receiver alone is not proof of host availability; receiver changes remain a separately approved scope (POWER-005 through POWER-007).

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

Repeat applicable binding-relocation/removal cases above through runtime/dynamic-keymap edits where Launcher supports those bindings.
Verify actions and RGB presentation follow the effective bindings without stale old positions, including after reboot where persistence is supported (LAUNCHER-004). Record unsupported cases explicitly; custom Launcher UI is not required.

# Q3 v1 release gate

A Q3 v1 candidate must have:
- clean build;
- acceptable FLASH/RAM/storage footprint;
- verified recovery;
- profile persistence;
- F-row persistence;
- intended RGB behavior;
- no stuck-key regressions;
- cable power behavior tested;
- 2.4 GHz behavior documented;
- Launcher compatibility status documented;
- known deviations recorded.
