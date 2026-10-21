# Architecture

## 1. Architectural intent

The project extends Keychron's `rtl8762g` ZMK branch without turning the fork into an unrelated firmware distribution.

The architecture should support later reuse across other compatible Keychron boards, but Q3 Ultra 8K ANSI is the only required initial target.

Board-specific facts such as matrix positions, LED indices, lock keys, and available transports should remain configurable instead of being embedded unnecessarily in generic logic.

## 2. Dependency stack

```text
Custom firmware changes
        ↓
Keychron ZMK
        ↓
Keychron Zephyr
        ↓
Realtek HAL / platform support
        ↓
RTL8762G hardware
```

Default escalation order:

1. solve in ZMK if possible;
2. change Keychron Zephyr only if ZMK lacks the required platform behavior;
3. change the Realtek HAL only if the problem is demonstrably below Zephyr;
4. treat 2.4 GHz receiver firmware as a separate scope expansion.

## 3. State model

```text
Physical Mac/Windows switch
        ↓
OS family
        ↓
active profile
   ├── base layer
   ├── associated functional layer
   ├── persistent F-row state
   ├── persistent RGB settings
   └── indicator policy

Transient state
   ├── Fn held
   ├── lock states
   └── active key/modifier state

Transport state
   ├── Cable / Bluetooth / 2.4 GHz
   └── active-host availability
```

The physical Mac/Windows switch remains authoritative.

Mac and Windows each maintain their own last active profile.

Profiles are generic. "Gaming" is not a special core mode.

A profile may define different bindings, RGB configuration, indicator policy, and F-row state.

Examples:

```text
Mac
└── Default
    └── Mac Fn

Windows
├── Normal ─────┐
├── Gaming ─────┼── Windows Fn
└── Profile 2 ──┘
```

or:

```text
Windows
├── Normal ───────► Windows Fn
└── Gaming ───────► Gaming Fn
```

Both shared and dedicated functional layers must be supported.

## 4. F-row model

The firmware exposes a reusable F-row switching behavior.

For Q3 it is expected to be bound to `Fn+Esc`, but the behavior must not be hard-coded to the Escape position.

The configured F-key positions are initially F1 through F12 only.

The logical state belongs to the active profile so that:

- two profiles sharing one Fn layer may still have different F-row orientation;
- a profile with a dedicated Fn layer also works correctly.

Typical states:

```text
Primary F-row:
base F9 = F9
Fn+F9   = Play/Pause

Secondary F-row:
base F9 = Play/Pause
Fn+F9   = F9
```

The exact implementation is intentionally not fixed yet. Candidate approaches include persistent state with runtime resolution or controlled runtime keymap materialization.

The effective runtime binding remains the source of truth for semantic RGB whenever practical.

## 5. Persistence model

Use existing Keychron/Zephyr settings/NVS infrastructure where possible.

Persistent state includes at least:

- last active profile per OS family;
- F-row state per applicable profile;
- eventually per-profile RGB settings;
- profile indicator policy where needed.

Persistent state must use a custom namespace that does not collide with Keychron Launcher data.

Writes should be minimized and should not occur for high-frequency transient events.

Factory/config reset behavior must restore custom state to compiled defaults.

## 6. RGB architecture

RGB is a presentation system driven by logical state.

The intended composition order is:

```text
profile base RGB
      ↓
lock/status overlays
      ↓
functional-layer semantic overrides
      ↓
disabled/no-action masks
      ↓
host/power master gate
      ↓
physical LEDs
```

Higher-priority stages override only LEDs they actually own, except the host/power gate which may force all LEDs OFF.

### Profile RGB

Each profile may eventually own persistent RGB parameters such as:

- enabled state;
- effect;
- hue/color;
- saturation;
- brightness;
- animation speed;
- effect-specific parameters supported by the Keychron stack.

Initial implementation may use identical values across profiles while keeping the architecture profile-scoped.

### Functional-layer RGB

Semantic colors appear only while the functional layer is active.

Keys without semantic overrides inherit the base profile RGB by default.

Classification follows the effective binding, not the physical key position.

Possible semantic categories:

- function keys;
- media transport;
- audio;
- display/brightness;
- RGB controls;
- wireless/connectivity;
- battery/power;
- system/navigation;
- profile selection.

Categories may contain related role variants, for example Mute using a lighter/darker variant of the Volume color.

### Profile selector presentation

When a functional layer exposes profile selectors:

- inactive selectors use the profile-selector semantic color;
- the selector for the currently active profile blinks white.

This visualization exists only while that functional layer is active.

### Lock indicators

Lock indicators are overlays, not LEDs tied to the physical lock-key position.

Indicator enablement is profile-specific.

For example, a Normal profile may show Caps Lock while a Gaming profile disables that indicator entirely.

Exact overlay regions and colors are configuration decisions and may depend on the base RGB effect.

### Disabled keys

Disabled/no-action presentation should derive from bindings/configuration where practical, not from a hard-coded "Gaming" mode.

Initial Q3 Gaming profile use case:

- Windows key binding = no action;
- corresponding LED = OFF.

## 7. Host and power architecture

The target abstraction is **active host availability**, not simple VBUS presence.

```text
active transport has usable host
    → RGB may operate normally

active transport has no usable host
    → RGB OFF
```

An inactive transport must not override the active one.

Example:

```text
active transport = 2.4 GHz
USB cable = charging only
USB host = suspended
```

RGB must remain available if the 2.4 GHz path has a usable host.

### Cable

Audit native disconnect/suspend/resume behavior first.

A SOF watchdog is a fallback only if native events cannot represent the required state.

### 2.4 GHz

A radio connection to a powered dongle does not necessarily mean the dongle has an active downstream USB host.

The project must determine whether Keychron's keyboard/receiver protocol exposes downstream host availability.

If receiver firmware must change, create a separate approved plan with its own build, flashing, recovery, and test strategy.

## 8. Launcher compatibility boundary

Official Keychron Launcher compatibility is best-effort.

The project should preserve ordinary stock capabilities such as polling/report rate, debounce, dynamic keymap and stock RGB controls where practical.

The Launcher is not required to understand custom profiles, semantic Fn RGB, custom indicators, or other custom behaviors.

If per-profile RGB cannot cleanly coexist with the stock Launcher in v1, firmware correctness takes priority and rebuild-based configuration is acceptable temporarily.

Firmware Update in the official Launcher may replace custom firmware with stock firmware. This path remains unverified until successful physical validation during M0.

## 9. Ctrl + End behavior

The special behavior is just another binding that may be used by any profile.

Required semantics:

```text
physical press
  → Left Ctrl DOWN
  → short deterministic technical delay if needed
  → End tap exactly once
  → Left Ctrl remains DOWN

physical release
  → Left Ctrl UP
```

Existing ZMK macro/behavior facilities must be evaluated first.

The implementation must guarantee a valid HID state and must never leave Ctrl or End permanently pressed.

Randomized timing intended to disguise automation is not part of the project.

## 10. Performance model

Prefer event-driven updates.

Examples of events that may trigger presentation recomputation:

- active profile changed;
- F-row state changed;
- Fn pressed/released;
- lock state changed;
- host availability changed;
- relevant dynamic keymap binding changed.

Avoid repeatedly scanning/classifying the entire keymap every RGB frame if the result can be cached.

RGB, storage, and diagnostics must not block time-sensitive keyboard input/report processing.
