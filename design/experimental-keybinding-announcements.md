# Experimental: Keybinding Action Confirmation & ARIA Announcement System (opt-in)

## Executive summary
Add an experimental, opt-in mechanism to attach ARIA/live-region spoken confirmations to keybindings so screen-reader and keyboard-first users get reliable feedback when shortcuts run. This feature is experimental, disabled by default.

## Problem
Keybindings often perform “silent” actions with little or no feedback. Screen-reader users and fast keyboard users lack consistent confirmation that commands executed.

## Proposed solution
- Extend keybinding entries to allow optional, localized announcements and per-binding announce toggles.
- Provide global experimental gating and verbosity controls.
- Implement an announcement service that respects global/per-binding settings, verbosity, delay, localization, and deduplication with visible notifications.

## Schema examples

Simple:
```jsonc
{
  "key": "ctrl+shift+c",
  "command": "workbench.action.chat.copyItem",
  "when": "chatIsVisible",
  "ariaAnnouncement": "Copied item to clipboard",
  "announceOnSuccess": true
}
```

Verbosity-aware (optional):
```jsonc
{
  "key": "ctrl+shift+c",
  "command": "workbench.action.chat.copyItem",
  "when": "chatIsVisible",
  "ariaAnnouncement": {
    "normal": "Copied item",
    "verbose": "Copied selected chat item to clipboard"
  },
  "announceOnSuccess": true
}
```

### New/extended properties
- `ariaAnnouncement`: string | { off?, normal?, verbose? } — optional; object keys map to verbosity levels. Runtime selects message by `accessibility.keybindings.verbosity` (fallback: verbose → normal → string → none).
- `announceOnSuccess`: boolean — optional per-binding toggle (default true when `ariaAnnouncement` present).

### Global (experimental) settings
- `accessibility.keybindings.experimental.enable` (boolean, default: false) — master gate.
- `accessibility.keybindings.announceDelay` (number, ms).
- `accessibility.keybindings.verbosity` (enum: `off` | `normal` | `verbose`).

## Implementation notes
- Add `IKeybindingAnnouncementService` (or extend existing accessibility service) that:
  - Observes command execution hooks and resolves the keybinding metadata and `when` context.
  - Applies delay, verbosity selection, localization, and dedupe rules (skip when a visible notification already informs the user).
  - Honors global/per-binding opt-outs.
- Keybindings Editor (`Ctrl+K Ctrl+S`) should surface `ariaAnnouncement` and show experimental guidance.

## Accessibility & UX rules
- Keep messages concise and localized.
- Provide master and per-binding opt-outs.
- Announce only on successful execution by default.
- Fallback selection uses verbose → normal → string rules.

## Acceptance criteria
- Schema supports `ariaAnnouncement` and `announceOnSuccess`.
- Experimental master setting exists and gates runtime behavior.
- Announcement service announces correct message per verbosity and dedupe rules.
- Keybindings Editor surfaces and edits announcement metadata.
- Unit tests for selection/fallback/deduping and accessibility smoke tests.

## Risks & mitigations
- Noise: opt‑in experimental default + verbosity and per-binding opt‑out.
- Localization: use NLS and document extension guidance.
- Double-announcements: dedupe logic.

## Labels
`experimental`, `feature-request`, `accessibility`, `keybindings`, `area-workbench`
