![version](https://img.shields.io/badge/version-20%2B-E23089)
![platform](https://img.shields.io/static/v1?label=platform&message=mac-intel%20|%20mac-arm&color=blue)
[![license](https://img.shields.io/github/license/miyako/4d-plugin-menu)](LICENSE)
![downloads](https://img.shields.io/github/downloads/miyako/4d-plugin-menu/total)

# 4d-plugin-menu

Invoke a standard macOS (AppKit) menu action on the frontmost window of the host 4D application.

## Summary

| Command | Thread-safe | Return |
| --- | --- | --- |
| [`INVOKE MENU ACTION`](#invoke-menu-action) | ✅ | — |

**Platforms:** macOS only — Apple Silicon (arm64) and Intel (x86_64). There is no Windows build.

**Requirements:** macOS 11.5 (Big Sur) or later. Uses the standard AppKit responder-chain actions; behaviour depends on whether the current key window and its focused control support the requested action.

## `INVOKE MENU ACTION`

**Syntax:** `INVOKE MENU ACTION ( menuAction : Longint )`

Executes one of the standard macOS menu-item actions — full screen, tabbed-window handling, toolbar, miniaturize, zoom, key-view navigation — as if the corresponding entry of the application's Window / View menu had been chosen.

### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `menuAction` | Longint | Required. The action to invoke, as a constant value `1`–`17` (a named constant is usually passed, see the [table](#actions) below). |

### Description

`INVOKE MENU ACTION` asks the **key window** (the frontmost key window of the host application) to perform the requested action.

1. The command always runs in the main process/thread. Whatever thread it is called from, it is marshalled to the main thread before doing any work.
2. The action is first offered to the key window's **first responder** (the currently focused view/control). If that responder does not implement the action, the command walks **up the responder chain** until it finds a responder that does.
3. When a matching responder is found, the action is scheduled to run **asynchronously on the main runloop** — the command returns immediately and does not wait for the action to complete.

If no responder in the key window's chain implements the requested action — or there is no key window at all — the command simply does nothing.

**Thread-safety:** the command is thread-safe. Because it always marshals to the main thread, it is safe to call from any context (including a `PROCESS 4D TAGS` process or an `ON EVENT CALL`), but it returns immediately without waiting.

### Actions

Pass the numeric code directly, or — as in the examples — use the named constant provided by the plugin:

| Code | Constant | Standard menu action invoked |
| --- | --- | --- |
| 1 | `Toggle Full Screen` | Toggle the key window's full-screen mode |
| 2 | `Toggle Tab Bar` | Show / hide the window tab bar |
| 3 | `Show All Tabs` | Display all open document windows as tabs |
| 4 | `Merge All Windows` | Merge the application's windows |
| 5 | `Move Tab To New Window` | Detach the current tab into its own window |
| 6 | `Toggle Toolbar Shown` | Show / hide the window toolbar |
| 7 | `Run Toolbar Customization Palette` | Open the toolbar customization palette |
| 8 | `Perform Miniaturize` | Miniaturize (minimize) the key window |
| 9 | `Perform Zoom` | Zoom (toggle) the key window size |
| 10 | `Arrange In Front` | Bring all application windows to the front |
| 11 | `Select Next Key View` | Move focus to the next key view |
| 12 | `Select Previous Key View` | Move focus to the previous key view |
| 13 | `New Tab` | Open a new tab in the key window |
| 14 | `New Window For Tab` | Move the current tab to a new window |
| 15 | `Close Tab` | Close the current tab |
| 16 | `Select Next Tab` | Activate the next tab |
| 17 | `Select Previous Tab` | Activate the previous tab |

### Examples

From the plugin's own test methods:

```4d
INVOKE MENU ACTION(Toggle Full Screen)
```

```4d
INVOKE MENU ACTION(Perform Miniaturize)
```

```4d
INVOKE MENU ACTION(Select Next Key View)
```

Dispatch on a value (the symbolic constants resolve to their numeric codes, so either form works):

```4d
C_LONGINT($action)
$action := 1   // Toggle Full Screen
INVOKE MENU ACTION($action)

INVOKE MENU ACTION(Merge All Windows)   // code 4
```

### Error handling

`INVOKE MENU ACTION` does **not** raise a 4D error and has no return value.

| Situation | Result |
| --- | --- |
| No key window | Nothing happens |
| The key window / responder chain does not implement the action | Nothing happens (silently) |
| Invalid `menuAction` code (outside `1`–`17`) | Nothing happens (silently) |
| Action unsupported on the current macOS version | Nothing happens (silently) |

The action only has an observable effect when the current key window and its focused control genuinely support it. For example, `Toggle Full Screen` does nothing if the key window's view doesn't support full-screen mode, and the tab actions do nothing on a window that isn't tab-capable.

### Quick reference

```4d
// macOS only — one command, one required Longint parameter, no return value
INVOKE MENU ACTION ( menuAction : Longint )

// runs on the main thread; returns immediately; silent no-op when unsupported
```