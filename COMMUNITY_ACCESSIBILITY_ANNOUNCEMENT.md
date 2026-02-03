# Accessibility Announcement for the Blind and Low Vision Community

**Date:** February 2026
**Status:** Under Final Review — Awaiting Merge to Public Release

---

## Summary

We are pleased to share details about upcoming accessibility improvements in Visual Studio Code. These changes are currently under final review by the VS Code team and are expected to be available in a forthcoming release.

---

## What's Coming

### 1. Comprehensive Accessibility Help for All Find and Filter Dialogs

**The Problem We're Solving:**
Until now, when screen reader users opened the Find dialog (Ctrl+F) or other filter inputs throughout VS Code, there was no easy way to learn how these dialogs worked without consulting external documentation.

**The Solution:**
You can now press **Alt+F1** when focused in any find or filter input to receive complete, context-specific help spoken by your screen reader. This works in:

- **Editor Find/Replace** (Ctrl+F, Ctrl+H)
- **Terminal Find** (Ctrl+F in the terminal)
- **Browser/Webview Find** (Ctrl+F in embedded content)
- **Search Across Files** (Ctrl+Shift+F)
- **Output Panel Filter**
- **Problems Panel Filter**
- **Debug Console Filter**
- **Comments Panel Filter**

**What You'll Hear:**
When you press Alt+F1, you receive a complete spoken guide including:
- What dialog you're in and what it does
- Your current search status (what you're searching for, how many matches, your position)
- Keyboard navigation instructions specific to your current focus
- Available options and how to use them
- Relevant settings you can customize
- How focus behaves when you navigate between matches
- How to close the dialog and where focus returns

### 2. Discoverability Hints

**The Problem:**
The Alt+F1 help feature is only useful if users know it exists.

**The Solution:**
When you first focus in a find or filter input, your screen reader will announce: **"Press Alt+F1 for accessibility help."** This hint appears once per session to guide you to the help system without being repetitive.

You can disable this hint by setting `accessibility.verbosity.find` to `false` in Settings once you're familiar with the feature.

## Separate Fix: Arrow Key Navigation in Go To / Quick Input Dialogs

**Status:** Reviewed and Awaiting Merge

**The Problem:**
In the Go To Line dialog (Ctrl+G) and other Quick Input dialogs, screen reader users were unable to use arrow keys to edit text within the input field. Arrow keys would navigate the list instead of moving the cursor within the text you were typing.

**The Solution:**
Arrow key behavior has been corrected. When you're editing text in a Quick Input dialog, left and right arrow keys now move the cursor within your text as expected. You can still navigate the results list using other standard methods.

---

## When Will These Be Available?

These changes are currently:
- **Under final code review** by the VS Code team
- **Awaiting merge** into a public-facing branch
- **Expected** in an upcoming VS Code release (version TBD)

We will provide updates as these changes move through the release process.

---

## How to Test (Once Released)

1. **Update VS Code** to the version containing these changes
2. **Open any file** and press **Ctrl+F** to open Find
3. **Press Alt+F1** to hear the accessibility help
4. **Listen for** the complete spoken guide about using Find
5. **Press Escape** to close the help and verify focus returns to Find
6. **Try in other contexts:** Terminal (Ctrl+`), Search (Ctrl+Shift+F), etc.

---

## Providing Feedback

Your feedback is invaluable. Once these features are released, please share your experience:

- **GitHub Issues:** https://github.com/microsoft/vscode/issues
- **Use the "Accessibility" label** when filing issues
- **Include your screen reader** (NVDA, JAWS, Narrator, VoiceOver) and version

---

## Technical Details for Interested Users

### Settings

- **`accessibility.verbosity.find`** — Controls whether the "Press Alt+F1 for accessibility help" hint is announced (default: true)

### Contexts Where Alt+F1 Help Is Available

| Context | How to Open | Alt+F1 Help ID |
|---------|------------|----------------|
| Editor Find | Ctrl+F | EditorFindHelp |
| Editor Replace | Ctrl+H | EditorFindHelp |
| Terminal Find | Ctrl+F in terminal | TerminalFindHelp |
| Webview Find | Ctrl+F in webview | WebviewFindHelp |
| Search Across Files | Ctrl+Shift+F | SearchHelp |
| Output Filter | Filter icon in Output | OutputHelp |
| Problems Filter | Filter icon in Problems | MarkersHelp |
| Debug Console | Filter in Debug Console | DebugConsoleHelp |

---

## Acknowledgments

These changes were developed collaboratively with members of the blindness community, including contributions from AccessWatch (Jeff Bishop — https://github.com/accesswatch) and BITS, with helpful guidance from the Visual Studio Code development team. Special appreciation to Ken Perry, Taylor Arndt, Michael Doise, Michael Babcock, and others within BITS whose mentorship, encouragement, and partnership have given purpose and drive to make a difference for all Visual Studio Code users.

AccessWatch (Jeff Bishop) remains deeply committed to future projects — stay tuned for more contributions.

We appreciate the community effort and ongoing support that made these improvements possible.

---

**Questions?** Please reach out through GitHub Issues or the VS Code accessibility community channels.

*This announcement will be updated as these changes progress through review and release.*
