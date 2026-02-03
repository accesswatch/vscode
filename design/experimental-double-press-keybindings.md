# Experimental: Double-Press Keybindings (single vs double-tap) (opt-in)

## Executive summary
Introduce an experimental, opt-in subsystem that maps a single physical keybinding to two related actions: single-press and quick double-press. This subsystem follows announcement, verbosity, and gating principles and is disabled by default.

## Problem
Users want a faster ergonomic pattern (tap vs double-tap) for related commands. Chords exist but are different in feel and can conflict.

## Proposed solution
- Add experimental `keyboard.doublePress.*` settings and per-binding configuration.
- Implement `DoublePressService` that handles timing and dispatch logic.
- Only a second press of the same configured key within the timeout counts as a double-press. Any different keypress during the timeout cancels the pending single action and the other key is dispatched normally.

## Prototype settings (example)
```jsonc
{
  "keyboard.doublePress.enabled": false,
  "keyboard.doublePress.timeout": 400,
  "keyboard.doublePress.bindings": [
    {
      "key": "ctrl+shift+c",
      "when": "inChat",
      "singleCommand": "workbench.action.chat.copyItem",
      "doubleCommand": "workbench.action.chat.copyAll",
      "ariaAnnouncement": {
        "single": { "normal": "Copied item", "verbose": "Copied selected chat item to clipboard" },
        "double": { "normal": "Copied conversation", "verbose": "Copied full chat conversation to clipboard" }
      },
      "announceOnSuccess": { "single": true, "double": true },
      "ignoreRepeat": true
    }
  ]
}
```

## Key rules (important)
- Double only if the second keypress is the same configured key within `timeout`.
- If any other key is pressed before timeout (including modifiers or different keys), cancel the double-press timer and do NOT execute the single action. The other keypress is handled normally.
- If the same key is pressed twice within `timeout`, execute `doubleCommand` immediately.
- If timeout expires with no second same-key press, execute `singleCommand`.

## Implementation notes
- `DoublePressService`:
  - Intercepts the configured key at dispatch time (respect `when`).
  - On first press: start timer.
  - On second same-key press before timeout: cancel timer and run `doubleCommand`.
  - On any other key press before timeout: cancel timer, do not run `singleCommand`; allow other key to dispatch normally.
  - On timeout expiry: run `singleCommand`.
  - Respect `ignoreRepeat` to avoid treating held-key repeats as presses.
- Announcements: after successful execution, post confirmations via the announcements mechanism (verbosity-aware) if enabled.

## Keybindings Editor & UX
- Show experimental double-press bindings, with an “Experimental” badge.
- Warn about chord conflicts and collisions.
- Provide UI to edit per-action announcements and `announceOnSuccess` toggles.

## Accessibility & UX
- Default disabled; opt-in per user.
- Adjustable `timeout` to accommodate motor-accessibility.
- Per-action announce opt-out and verbosity-aware messages.
- Dedupe with visible notifications.

## Acceptance criteria
- `keyboard.doublePress.*` settings and JSON schema exist.
- `DoublePressService` timing respects same-key-only rule and cancels on other keypresses.
- `ignoreRepeat` behavior implemented.
- Announcements fire per-binding/verbosity when enabled.
- Keybindings Editor shows experimental bindings and conflict warnings.
- Unit + integration tests for timing, repeat handling, non-same-key cancellation, context evaluation, and announcements.

## Labels
`experimental`, `feature-request`, `keybindings`, `accessibility`, `area-workbench`

---
Related issue: https://github.com/microsoft/vscode/issues/292578
