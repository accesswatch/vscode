# User-Contributed Accessibility Help System

## Executive Summary
Create a universal, settings-driven mechanism that allows users to define custom accessibility help text for any context in VS Code. This removes the technical barrier for product teams, designers, and end-users who want to provide or customize Alt+F1 accessibility help content without writing specialized TypeScript code.

## Problem Statement

### Current Architecture Limitations
Today, accessibility help in VS Code requires:
1. **Specialized TypeScript code** for each scenario (Find, Terminal, Search, etc.)
2. **Product teams/designers write code** in `provideContent()` methods within `IAccessibleViewContentProvider` implementations
3. **Technical barriers** prevent non-developers from contributing or customizing help text
4. **Duplicated patterns** across many implementations (e.g., `EditorFindAccessibilityHelp`, `TerminalFindAccessibilityHelpProvider`, `SearchAccessibilityHelpProvider`, etc.)
5. **No user customization** - end users cannot tailor accessibility help to their needs

### Impact
- Users with specialized accessibility needs cannot customize help content
- Adding accessibility help for new features requires developer involvement
- Product teams must write nearly identical boilerplate code for each new context
- ARIA alert behavior and verbosity settings are wired up inconsistently per-implementation

## Proposed Solution

### Core Concept
Create a **universal accessibility help content provider** that:
1. Reads user-defined help text from settings
2. Automatically honors ARIA alert policies and verbosity settings
3. Provides a fallback to built-in defaults when no user customization exists
4. Supports context-aware content selection based on current focus/context

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    User Settings (JSON)                         │
│  accessibility.help.customContent: {                            │
│    "editor": "Custom editor help...",                           │
│    "terminal": "Custom terminal help...",                       │
│    "search": { "normal": "...", "verbose": "..." }             │
│  }                                                              │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│           Universal Accessibility Help Service                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • Resolves content by context ID                        │    │
│  │ • Applies verbosity settings (off/normal/verbose)       │    │
│  │ • Honors ARIA alert policy (polite/assertive/off)       │    │
│  │ • Falls back to built-in defaults                       │    │
│  │ • Deduplicates with visible notifications               │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              Accessible View Display                            │
│  Shows resolved content with proper announcements               │
└─────────────────────────────────────────────────────────────────┘
```

### Schema Design

#### User Settings Schema

```jsonc
{
  // Master enable for user-contributed content
  "accessibility.help.experimental.enableUserContent": true,
  
  // Global verbosity level (applies to all help content)
  "accessibility.help.verbosity": "normal", // "off" | "normal" | "verbose"
  
  // ARIA announcement policy
  "accessibility.help.ariaPolicy": "polite", // "off" | "polite" | "assertive"
  
  // User-contributed help content by context
  "accessibility.help.customContent": {
    // Simple string for single verbosity level
    "editor": "You are in the code editor. Press Tab to indent, Escape to exit.",
    
    // Verbosity-aware object
    "terminal": {
      "normal": "Terminal ready. Type commands here.",
      "verbose": "You are in the integrated terminal. This terminal supports shell integration for enhanced accessibility. Type commands and press Enter to execute. Use Ctrl+` to toggle visibility."
    },
    
    // Override specific context with appended content
    "editor.find": {
      "append": true,  // Append to built-in help, don't replace
      "content": "My custom find tips: Use regex for complex patterns."
    },
    
    // Context with keybinding placeholders
    "search": "Search across files. Press {keybinding:search.action.focusNextSearchResult} to go to next result."
  }
}
```

#### Context Identifiers (Discoverable by Users)
Users can define help for any registered `AccessibleViewProviderId` plus additional contexts:
- `editor` - Main code editor
- `editor.find` - Find/Replace dialog in editor
- `terminal` - Integrated terminal
- `terminal.find` - Terminal find widget
- `search` - Search across files view
- `problems` - Problems panel
- `output` - Output panel
- `debug` - Debug console
- `chat` - Copilot chat panel
- `notebook` - Notebook editor
- `scm` - Source control view
- Custom extension-provided contexts

### Implementation Components

#### 1. `IUserAccessibilityHelpService`

```typescript
export interface IUserAccessibilityHelpService {
  readonly _serviceBrand: undefined;
  
  /**
   * Get resolved help content for a context, applying user customizations and verbosity.
   * @param contextId The accessibility help context identifier
   * @returns Resolved content string, or undefined if no content available
   */
  getHelpContent(contextId: string): string | undefined;
  
  /**
   * Check if user has custom content for a context.
   */
  hasUserContent(contextId: string): boolean;
  
  /**
   * Get current verbosity level.
   */
  getVerbosityLevel(): 'off' | 'normal' | 'verbose';
  
  /**
   * Announce content via ARIA, respecting policy settings.
   */
  announceHelp(content: string): void;
  
  /**
   * Register a built-in help provider as fallback.
   */
  registerBuiltInProvider(contextId: string, provider: () => string): IDisposable;
}
```

#### 2. Universal Help Content Provider

```typescript
export class UniversalAccessibilityHelpProvider implements IAccessibleViewContentProvider {
  id = AccessibleViewProviderId.Universal;
  verbositySettingKey = 'accessibility.help.verbosity';
  
  constructor(
    private readonly contextId: string,
    @IUserAccessibilityHelpService private readonly userHelpService: IUserAccessibilityHelpService,
  ) {}
  
  provideContent(): string {
    return this.userHelpService.getHelpContent(this.contextId) ?? 
           this.getDefaultContent();
  }
  
  private getDefaultContent(): string {
    return localize('noHelpAvailable', 
      'No accessibility help available for this context. You can add custom help in Settings under accessibility.help.customContent.{0}', 
      this.contextId);
  }
}
```

#### 3. Settings Configuration

New settings to add to `accessibilityConfiguration.ts`:

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `accessibility.help.experimental.enableUserContent` | boolean | false | Enable user-contributed accessibility help content |
| `accessibility.help.verbosity` | enum | "normal" | Global verbosity level for all accessibility help |
| `accessibility.help.ariaPolicy` | enum | "polite" | ARIA announcement policy (off/polite/assertive) |
| `accessibility.help.customContent` | object | {} | User-defined help content per context |
| `accessibility.help.announceDelay` | number | 100 | Delay in ms before ARIA announcement |

#### 4. Migration Path for Existing Providers

Existing providers continue to work but can optionally:
1. Register as built-in fallbacks via `registerBuiltInProvider()`
2. Check for user content before providing built-in content
3. Support the `append` mode for user additions

### UX Considerations

#### Discoverability
1. **Command Palette**: Add "Accessibility: Configure Help for Current Context" command
2. **Settings UI**: Provide schema-driven autocomplete for context IDs
3. **Context Menu**: "Customize Accessibility Help" in accessible view header

#### Documentation
1. List all available context IDs in Settings description
2. Provide examples in accessibility documentation
3. Show current context ID in accessible view footer

#### Error Handling
1. Invalid context IDs: Warn in Problems panel, fall back to built-in
2. Invalid verbosity values: Default to "normal"
3. Missing content: Show helpful message with setup instructions

### ARIA Alert Integration

The universal service ensures consistent ARIA behavior:

```typescript
announceHelp(content: string): void {
  const policy = this.configurationService.getValue('accessibility.help.ariaPolicy');
  if (policy === 'off') return;
  
  const delay = this.configurationService.getValue('accessibility.help.announceDelay');
  
  // Dedupe: Skip if notification is already visible with same content
  if (this.notificationService.hasVisibleMessage(content)) return;
  
  setTimeout(() => {
    this.ariaService.alert(content, policy === 'assertive' ? 'assertive' : 'polite');
  }, delay);
}
```

### Keybinding Placeholder Support

User content can include keybinding placeholders that get resolved at runtime:

```
"Press {keybinding:editor.action.find} to open Find"
→ "Press Ctrl+F to open Find"
```

### Localization Considerations

1. User-contributed content is not localized (user's language)
2. Built-in fallbacks remain localized via NLS
3. Documentation recommends users provide content in their preferred language

## Acceptance Criteria

### Functional
- [ ] User can define custom help text via settings for any context
- [ ] Verbosity levels (off/normal/verbose) are respected globally
- [ ] ARIA announcement policy works consistently
- [ ] Keybinding placeholders resolve correctly
- [ ] Append mode allows additions to built-in content
- [ ] Fallback to built-in when no user content exists
- [ ] "Configure Help for Current Context" command works

### Quality
- [ ] Unit tests for content resolution and verbosity selection
- [ ] Unit tests for ARIA policy enforcement
- [ ] Integration tests for settings schema validation
- [ ] Accessibility smoke tests with screen reader

### Documentation
- [ ] Settings descriptions include available context IDs
- [ ] Examples provided in settings schema
- [ ] Accessibility documentation updated

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| User content overwrites critical help | Medium | High | Provide "restore defaults" command; show warning for complex contexts |
| Performance impact from settings lookups | Low | Medium | Cache resolved content; only invalidate on settings change |
| Confusion over context IDs | High | Medium | Provide autocomplete, show current context in accessible view |
| Incompatibility with extensions | Medium | Medium | Extensions can register via API; backward compatible |

## Future Enhancements

1. **Extension API**: Allow extensions to contribute help content via `package.json`
2. **Community Sharing**: Marketplace for sharing accessibility help configurations
3. **AI-Generated Help**: Suggest help content based on current context
4. **Help Templates**: Pre-built help templates for common scenarios
5. **Multi-language Support**: Per-language user content definitions

## Labels
`accessibility`, `feature-request`, `settings`, `user-customization`, `area-workbench`

---

## Related Work
- [Experimental Keybinding Announcements](./experimental-keybinding-announcements.md) - ARIA announcement system for keybindings
- Current `IAccessibleViewService` and `AccessibleViewRegistry` implementations

