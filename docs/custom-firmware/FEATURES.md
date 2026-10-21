# Feature Specification v1

## Status

This document is the functional source of truth for the initial Q3 custom firmware project.

Implementation details belong in `ARCHITECTURE.md` and feature-specific plans.

# E0 — Stock baseline and recovery

## STOCK-001 — Reproducible stock build
Build Q3 Ultra 8K ANSI from the project fork using the selected Keychron baseline.

## STOCK-002 — Physical stock validation
Verify on the physical Q3:
- USB;
- Bluetooth;
- 2.4 GHz;
- physical Mac/Windows switch;
- RGB;
- encoder;
- keymap;
- sleep/wake;
- reconnect.

## STOCK-003 — Official Launcher baseline
Before custom functionality, verify stock Launcher behavior including:
- polling/report rate;
- debounce/bounce configuration;
- stock RGB controls;
- dynamic keymap editing;
- other relevant stock Q3 settings.

## STOCK-004 — Recovery
Document and physically verify return to official stock firmware before risky changes.

## STOCK-005 — Known-good baseline
Tag the first fully verified stock-equivalent build.

# E1 — Resources and runtime

## RES-001 — Flash budget
Track firmware size against the approximately 800 KiB application partition.

## RES-002 — RAM budget
Record stock RAM usage and changes at significant milestones.

## RES-003 — Storage budget
Track persistent-storage use against the platform storage partition and existing Keychron use.

## RES-004 — Feature deltas
Significant PRs should report approximate FLASH/RAM/storage impact.

## PERF-001 — Responsiveness
Custom functionality must not materially degrade input latency or reliability.

## PERF-002 — High-rate operation
Do not compromise stable stock high-rate USB operation.

## PERF-003 — Time-critical path isolation
RGB, persistence and diagnostics must not unnecessarily block input/report processing.

## PERF-004 — Event-driven processing
Prefer recomputation on relevant state changes over continuous expensive work.

# E2 — OS families and profiles

## OS-001 — Physical selector
The physical Mac/Windows switch remains authoritative for the active OS family.

## PROFILE-001 — Generic profiles
A profile is generic and is not inherently "gaming" or another special core type.

## PROFILE-002 — Profiles per OS family
Each OS family may expose one or more profiles. Mac and Windows do not need the same number of profiles.

## PROFILE-003 — Persistent last profile per OS
Each OS family independently restores its last active profile after switching back or rebooting.

## PROFILE-004 — Explicit profile selection
Profile selection uses explicit selection actions, not a toggle. Selecting the already active profile is a no-op.

## PROFILE-005 — Arbitrary selector placement
Profile selectors are keymap bindings, not hard-coded physical positions.

## PROFILE-006 — Functional-layer association
Every profile identifies an associated functional layer.

## PROFILE-007 — Shared functional layers
Multiple profiles may reference the same functional layer.

## PROFILE-008 — Dedicated functional layers
A profile may instead use a dedicated functional layer. Subsystems must not assume either topology.

# E3 — Persistence

## PERSIST-001 — Existing infrastructure first
Audit and reuse existing Keychron/Zephyr settings/NVS mechanisms where appropriate.

## PERSIST-002 — Active profile persistence
Persist last active profile independently for Mac and Windows.

## PERSIST-003 — F-row persistence
Persist F-row state per applicable profile.

## PERSIST-004 — Profile RGB persistence
Architecture shall support persistent RGB configuration per profile.

## PERSIST-005 — Indicator policy persistence
Architecture shall allow profile-specific indicator policy to persist where needed.

## PERSIST-006 — Namespace isolation
Custom settings must not collide with Keychron Launcher/settings namespaces.

## PERSIST-007 — Write minimization
Do not write persistent storage for high-frequency transient state.

## PERSIST-008 — Reset semantics
The relevant factory/config reset restores compiled defaults for custom persistent state.

# E4 — Functional layers and F-row

## LAYER-001 — Momentary Fn
The functional layer is normally active only while Fn is held.

## LAYER-002 — Profile controls Fn mapping
The active profile determines which functional layer Fn activates.

## FROW-001 — Primary/secondary F-key behavior
Support switching the configured F-key positions between primary and secondary bindings.

## FROW-002 — Q3 F-key scope
Initial Q3 scope is F1 through F12 only.

## FROW-003 — Reusable switching behavior
Expose F-row switching as a reusable action/behavior rather than hard-coding the Escape position.

## FROW-004 — Initial Q3 binding
Initial Q3 configuration may bind the action to Fn+Esc.

## FROW-005 — Binding-controlled availability
A layer exposes F-row switching only if the action is bound there.

## FROW-006 — State belongs to profile
F-row state belongs to the profile, not the functional layer.

## FROW-007 — Shared-layer correctness
Profiles sharing one functional layer may have independent F-row state.

## FROW-008 — Dedicated-layer correctness
Profiles using dedicated functional layers must also work.

## FROW-009 — Persistence
F-row state survives reboot and power loss.

# E5 — Per-profile RGB

## RGB-PROFILE-001 — Profile-scoped RGB
Each profile may own persistent RGB settings.

Candidate settings include:
- enabled;
- effect;
- hue/color;
- saturation;
- brightness;
- animation speed;
- effect-specific stock parameters.

## RGB-PROFILE-002 — Preserve stock effects
Preserve existing Keychron effects where practical.

## RGB-PROFILE-003 — Initial simplification allowed
Initial implementation may use identical RGB values across profiles while the model remains profile-scoped.

## RGB-PROFILE-004 — Restore on profile switch
Switching profile restores that profile's RGB state immediately.

## RGB-PROFILE-005 — Launcher target behavior
If official Launcher RGB controls remain compatible, changes made while a profile is active should affect that profile.
If this is impractical in v1, rebuild-based custom configuration is acceptable temporarily.

# E6 — RGB composition

## RGB-COMP-001 — Ordered composition
Target order:
1. profile base RGB;
2. lock/status overlays;
3. functional-layer semantic overrides;
4. disabled/no-action masks;
5. host/power master gate.

## RGB-COMP-002 — Deterministic priority
Overlapping sources resolve predictably.

## RGB-COMP-003 — Local override
A higher-priority overlay overrides only LEDs it owns.

## RGB-COMP-004 — Power gate
Host/power blackout has absolute priority and forces all RGB OFF.

## RGB-COMP-005 — Board-specific physical mapping
Generic logic must not hard-code Q3 LED indices where board/shield configuration can provide them.

# E7 — Functional-layer semantic RGB

## RGB-FN-001 — Active only while Fn is held
Semantic Fn colors are shown only while the associated functional layer is active.
A media action moved to the base layer by F-row switching uses ordinary profile RGB when Fn is not held.

## RGB-FN-002 — Inherit base RGB
Keys without semantic Fn overrides keep the profile base RGB by default.

## RGB-FN-003 — Future unmapped modes
Architecture may later support inherit/dim/off. v1 only requires inherit.

## RGB-FN-004 — Binding-based classification
Semantic color follows the effective binding/action, not physical position.

## RGB-FN-005 — Semantic categories
Initial candidates:
- function;
- media transport;
- audio;
- display/brightness;
- RGB control;
- wireless/connectivity;
- battery/power;
- system/navigation;
- profile selection.

## RGB-FN-006 — Semantic roles
A category may have related role variants. Example: Volume Up/Down same color, Mute a lighter/darker related variant.

## RGB-FN-007 — Firmware-defined palette
v1 uses predefined firmware colors.

## RGB-FN-008 — Unclassified fallback
Unknown actions use a deterministic fallback presentation.

## RGB-FN-009 — F-row awareness
After F-row switching, the Fn color reflects the action currently exposed by Fn.

# E8 — Profile selector presentation

## RGB-PROFILESEL-001 — Selector identification
While the functional layer containing profile selectors is active, selector keys are visually identifiable.

## RGB-PROFILESEL-002 — Active selector
The selector for the active profile blinks white.

## RGB-PROFILESEL-003 — Inactive selectors
Inactive profile selectors use the predefined profile-selector color.

## RGB-PROFILESEL-004 — Binding driven
Visualization follows the selector binding and is not tied to fixed physical positions.

# E9 — Lock/status indicators

## IND-001 — Overlay model
Lock indicators may use any visible region rather than only the LED under the physical lock key.

## IND-002 — Caps Lock
Caps Lock ON may show a profile-appropriate overlay. Caps Lock OFF removes it.

## IND-003 — Scroll Lock
Scroll Lock ON may show its own overlay.

## IND-004 — Per-profile policy
Each profile may enable/disable individual indicators.

## IND-005 — Effect-aware presentation
Exact presentation may depend on the active RGB effect.

## IND-006 — True OFF
Custom indicator/mask logic must be able to set LEDs to true zero/off.

## IND-007 — Fn overlap
If Fn semantic RGB and a lock overlay target the same LED, Fn wins on that LED while the rest of the lock overlay remains visible.

# E10 — Disabled/no-action keys

## KEYMASK-001 — No-action bindings
Profiles may intentionally bind a key to no action.

## KEYMASK-002 — Disabled-key presentation
The RGB engine may present such keys as disabled.
Initial Q3 Gaming use case:
- Windows key = no action;
- Windows-key LED = OFF.

## KEYMASK-003 — Generic implementation
Do not hard-code this as a special Gaming/Windows-key rule.

# E11 — Initial Windows Gaming profile

## GAME-001 — Ordinary profile
Windows Gaming uses the generic profile system.

## GAME-002 — Dedicated base bindings
Gaming may differ from Windows Normal through ordinary bindings.

## GAME-003 — Windows key
Initial desired Gaming configuration disables the Windows key.

## GAME-004 — Ctrl binding
Initial Gaming configuration binds Ctrl to the special Ctrl+End behavior.

## GAME-005 — No crouch-jump
Space+Duck/crouch-jump is out of scope.

# E12 — Ctrl + End behavior

## CTRLEND-001 — Press semantics
On physical press:
1. Left Ctrl DOWN;
2. short deterministic technical delay if required;
3. End DOWN;
4. End UP;
5. Left Ctrl remains DOWN.

## CTRLEND-002 — Release semantics
On physical release, Left Ctrl goes UP.

## CTRLEND-003 — One End tap
End is emitted exactly once per physical press.

## CTRLEND-004 — Existing mechanisms first
Audit existing ZMK macro/behavior facilities before adding custom C behavior.

## CTRLEND-005 — Deterministic timing
Technical delays may be configurable constants chosen for reliable HID behavior.
Randomized timing intended to disguise automation is out of scope.

## CTRLEND-006 — HID state guarantee
The behavior must not leave Ctrl or End stuck under normal edge cases.

# E13 — Host availability and power-aware RGB

## POWER-001 — Host availability abstraction
RGB is controlled by whether the active transport has a usable host, not merely by electrical power.

## POWER-002 — Cable behavior
- active host → normal RGB;
- host suspended/unavailable → RGB OFF;
- resume/reconnect → restore previous logical presentation.

## POWER-003 — Native USB mechanisms first
Audit native disconnect/suspend/resume behavior before considering a SOF watchdog.

## POWER-004 — Active transport authority
An inactive transport must not override the active transport's host state.

## POWER-005 — 2.4 GHz behavior
A powered radio-connected dongle is not automatically proof of an active downstream host.
Target:
- dongle + active PC → normal RGB;
- dongle remains powered after PC shutdown → RGB OFF.

## POWER-006 — 2.4 GHz audit
Determine whether keyboard/receiver protocol already exposes downstream host availability.

## POWER-007 — Receiver escalation
Receiver firmware changes require a separate approved scope with its own build/flash/recovery plan.

## POWER-008 — Preserve logical state
Power blackout does not overwrite profile RGB, active profile, F-row state, or indicator policy.

# E14 — Official Keychron Launcher compatibility

## LAUNCHER-001 — Best effort
Preserve official Launcher compatibility where reasonable.

## LAUNCHER-002 — Stock functions
Initially attempt to retain:
- polling/report rate;
- debounce;
- standard keymap editing;
- stock RGB controls;
- other compatible stock settings.

## LAUNCHER-003 — Custom UI not required
The official Launcher need not understand custom profiles, semantic RGB, custom indicators, or custom F-row behavior.

## LAUNCHER-004 — Dynamic-keymap awareness
Account for the Launcher modifying and persisting runtime key bindings.

## LAUNCHER-005 — Temporary rebuild configuration
If custom configuration cannot cleanly coexist with Launcher in v1, rebuild-based configuration is acceptable.

## LAUNCHER-006 — Firmware update
Official Firmware Update may replace custom firmware with stock firmware.

# E15 — Reset, safety and lower-layer escalation

## SAFE-001 — Recoverable custom state
Custom persistence has documented reset semantics.

## SAFE-002 — High-risk areas
Changes to boot/DFU, partitioning, recovery-critical USB descriptors, storage layout, Zephyr, HAL or receiver firmware require explicit review.

## SAFE-003 — Escalation order
Prefer:
1. ZMK;
2. Keychron Zephyr;
3. Realtek HAL;
4. receiver firmware if required.

# E16 — Future V0 extension

Not required for Q3 v1.

## V0-001 — Optional second target
V0 Ultra 8K may later become the second physical target.

## V0-002 — Reuse shared infrastructure
Where applicable reuse profiles, persistence, RGB composition, semantic Fn presentation, host/power handling and indicators.

## V0-003 — Num Lock OFF indicator
Preferred V0 behavior:
- Num Lock ON → normal presentation;
- Num Lock OFF → visible warning overlay.

Exact presentation remains TBD.

# Cross-cutting requirements

## GEN-001 — Q3 first
Other keyboards must not delay Q3 v1.

## GEN-002 — Generic where natural
Reusable mechanisms should not unnecessarily encode Q3 physical positions.

## GEN-003 — Audit existing code first
Audit current Keychron/ZMK/Zephyr mechanisms before implementing replacements.

## GEN-004 — Small isolated changes
Keep changes independently testable and minimize vendor divergence.

## GEN-005 — Physical validation
Build success is not hardware success.

## GEN-006 — Custom launcher excluded
A custom WebHID launcher is outside v1.

## GEN-007 — Custom HID excluded by default
A new custom HID protocol is outside v1 unless a proven requirement makes it necessary.

# Open configuration decisions

These do not block the architecture:
- exact Q3 profile-selector keys;
- number of optional initial profiles;
- exact semantic RGB palette;
- category-role color variants;
- exact Caps/Scroll overlay regions;
- exact effect-aware lock-indicator rendering;
- default indicator policy for each initial profile;
- unknown-action fallback color;
- whether V0 joins the first public release;
- exact solution for 2.4 GHz downstream-host detection.
