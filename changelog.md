---
title: Changelog
permalink: /changelog/
---

## [1.0.2] - 2026-11-07

### Changed
- The product is now called **Mobile Debug Panel** (the Package Manager's display name and the
  README title), the name it is sold under on the Asset Store; "Debug Panel" was taken there. The
  package id, the `DebugPanel` namespace and the API are unchanged, and the Editor menus and window
  still say "Debug Panel".

## [1.0.1] - 2026-11-07

### Fixed
- The red error alert in the pinned box ended further right than the pinned lines under it. Its
  text now lines up with them, and the red background still reaches a little past them.

## [1.0.0] - 2026-11-06

The public API is now stable. Within 1.x, releases only add to it and fix bugs; anything that
would break code written against it (renaming, removing, or changing the signature of a public
member) waits for 2.0. The public API is:

- `Panel`: `IsAvailable`, `AddOptions`, `RemoveOptions`, `IsVisible`, `OnVisibilityChanged`,
  `Show`, `Hide`, `Toggle`, `ErrorAlertEnabled`, `ResetSavedOptions`, `SaveReport`, `OnReportSaved`.
- `OptionContainer`: the constructor, `Category`, `Count`, `AddButton`, `AddToggle`, `AddInt`,
  `AddFloat`, `AddText`, `AddChoice`, `Remove`, `Clear`.
- The attributes `Category`, `DisplayName`, `Sort`, `NumberRange`, `Increment` and `Persist`.
- Editor only: `DebugPanelDefines` and `TriggerCorner`.

All of it compiles in every build. In a release build without the panel every member does nothing.

### Changed
- `OptionRegistry`, `OptionCategory`, `OptionDescriptor` and `OptionKind` are now internal. They
  were public by accident, and since they do not exist in release builds without the panel, game
  code that used them would not compile there.

### Added
- Documentation comments on every public member, shown by IDEs.
- README: turning the panel on from a build script with `DebugPanelDefines`.

## [0.6.0] - 2026-11-06

### Added
- The package works from any folder: under `Packages/` as before, or unpacked anywhere under
  `Assets/` (as a `.unitypackage` installs). The panel's settings and stylesheet are found by their
  GUIDs instead of a fixed `Packages/com.tugrultosun.debugpanel/...` path; before, a copy under
  `Assets/` ran with no styles and no settings. Checked in a fresh Unity 6.3 project with the
  package under `Assets/DebugPanel`.

## [0.5.5] - 2026-11-06

Found by checking Android tablets and a foldable in the Unity Device Simulator.

### Fixed
- On screens narrower than about 360 points (a folded phone's cover screen, such as the Galaxy Z
  Fold2's, is 320) the Options search field kept its full width and pushed *Reset saved* off the
  right edge of the panel. The field now gives up width first, and typed text no longer widens it.
  Its placeholder is now "Search" (was "Search options").

## [0.5.4] - 2026-11-06

Found by running the sample on an Android phone (Redmi 9T, Android 12, IL2CPP).

### Fixed
- Option names were cut off on phones in portrait ("Damage Mul...", "Player...", and Difficulty's
  name hidden entirely next to its *saved* tag): a number control or dropdown left the name about
  90 points. Below 520 points wide, number, text and dropdown options now stack: Pin, name and
  *saved* on one line, the control at full width below (the list uses variable-height rows there).
  Toggles, buttons and headers stay one line; wider screens and the Editor window are unchanged.
- Rebuilding the options list left the old row elements subscribed to value and pin changes and
  listed as on screen.
- Scrolling the options list on a phone opened the keyboard or a dropdown menu whenever the finger
  started on a number box, text box or dropdown: UI Toolkit acts on the press. These now act only on
  a tap (released within 12 points); a drag that starts on them just scrolls.

### Added
- README: *Known Unity messages*, explaining the "Hiding input field is not supported when using
  Game Activity" warning that Unity logs when typing in any UI Toolkit text field on Android.

## [0.5.3] - 2026-11-06

### Fixed
- On tablets everything looked small: sizes are physical, so the panel kept its phone size on a
  much larger screen. It now grows with the screen's short side, from 1× on phones (up to 3.6 in)
  to 1.75× on large tablets and desktops, and follows rotation and window resizes.

## [0.5.2] - 2026-11-06

### Fixed
- The report sheet's *Save report* button did not fit on a phone in portrait: the title, Cancel
  and Save shared one row. The title now has its own line, and Cancel and Save sit side by side
  at full width below it, still above the text box and clear of the keyboard.

## [0.5.1] - 2026-11-06

### Fixed
- The tab bar's buttons ran together into one line of text ("OptionsConsoleSystem") and the tab
  names were cut off on phones. Each tab, Report and Close is now its own rounded button with a gap
  around it; the selected tab is filled and outlined. On a narrow screen (a phone in portrait),
  Report and Close wrap to a second row together, so the tab names always fit.

## [0.5.0] - 2026-11-06

### Changed
- The panel is always compiled into the Editor, so it works in Play Mode with no setup.
  `DEBUG_PANEL` is now needed only to put it into a release (non-development) build.
  The Editor never defines `DEVELOPMENT_BUILD`, so before this a game without the define could not
  open the panel in Play Mode.

### Fixed
- Cheats vanishing from stripped builds. For every build that compiles the panel, the package now
  writes a `link.xml` that keeps whole each class using one of its attributes, with its base
  classes. Games no longer need their own `link.xml` for cheat classes. A class with no panel
  attribute at all still needs `[Category]`.

### Removed
- The Cheats sample's `link.xml` and `[Preserve]` attributes, which the package now covers.

## [0.4.0] - 2026-11-06

### Added
- Error alert: while the panel is closed, errors, asserts and exceptions show a red "N errors" line
  at the top of the screen, also with nothing pinned. Opening the panel clears it.
  `Panel.ErrorAlertEnabled` turns it off.
- `OptionContainer`: options built in code from delegates (button, toggle, int, float, text, and a
  dropdown of any list of texts), for options that come from data such as one button per level.
  Options added or removed after registering show up at once. Compiles to no-ops in release builds.
- Bug reports ask *What went wrong?* first; the answer is saved at the top of report.txt.
  `Panel.SaveReport(description)` takes it from code.

## [0.3.0] - 2026-11-06

### Added
- `[Persist]` on a member or class: values changed in the panel are saved to PlayerPrefs and applied
  when a container of the type is registered again, also after a restart. Enums are stored by name.
  Saved options show a *saved* tag; *Reset saved* and `Panel.ResetSavedOptions()` restore the
  values from before and delete every saved one.
- Pinned values: *Pin* on an option row and *Pin FPS* on the System tab show those values in a
  small box at the top of the screen while the panel is closed. The box ignores taps, stays in the
  safe area and away from the trigger corner. Pins are saved across restarts.
- Bug reports: *Report* in the tab bar, or `Panel.SaveReport()`, saves a screenshot without the
  panel and a report.txt (options, system info, the whole log with stack traces) to
  persistentDataPath/DebugPanel/Reports, copies the text to the clipboard and raises
  `Panel.OnReportSaved` with the folder.
- `OptionDescriptor.Key` and `OptionDescriptor.IsPersistent`.
- README: what the open panel does and does not block of the game's input.

## [0.2.0] - 2026-11-06

### Changed
- Requires Unity 6.3 (6000.3) or later; 0.1.0 targeted 6000.0. The development project is on
  6000.3.25f1, where every EditMode and PlayMode test passes. Unity 6.0 is no longer tested.
- The virtualization tests for the Console and Options lists now allow the rows that fit in the
  list's viewport, instead of a fixed 60 that failed on a tall Game view.

## [0.1.0] - 2026-11-06

### Added
- Option attributes: `[Category]`, `[DisplayName]`, `[Sort]`, `[NumberRange]`, `[Increment]`.
- `OptionRegistry` and `OptionDescriptor`: reflect a container once, bind AOT-safe accessors.
- `Panel` facade with a no-op stub when the runtime is stripped.
- Options tab (UI Toolkit, built in code, one USS): collapsible categories, search, virtualized
  ListView; toggle, number field with hold-to-repeat +/-, dropdown, text field and button rows.
  Visible rows refresh on INotifyPropertyChanged, or by polling every 0.5 s.
- Triple-tap corner trigger (default top-left), ` key in the Editor and desktop players,
  Escape / Android back to close. Works with the legacy Input Manager and the Input System.
- Project Settings > Debug Panel: enable per build target (DEBUG_PANEL) and pick the trigger corner.
- Console tab: captures Application.logMessageReceivedThreaded from start-up into a thread-safe
  ring buffer (1000 messages); Info / Warn / Error filters with counts, collapse duplicates,
  tap a line for its stack trace, copy one line or the whole list, clear.
- System tab: application, device, memory, graphics and display facts; live values (memory,
  FPS, uptime, resolution, safe area, battery) refresh once a second; copy as text.
- Window > Debug Panel: the same Options, Console and System tabs in a dockable Editor window,
  live in Play Mode, in a compact desktop layout.
- Release builds without the panel carry none of its assets: the PanelSettings, theme and
  stylesheet are not in a Resources folder and are added to a build only when it compiles the
  runtime (DEBUG_PANEL, or a development build).
- "Cheats" sample showing every option type, with a link.xml for managed code stripping.
- Verified: release IL2CPP builds for iOS (Simulator, High stripping) and Android (ARM64);
  install by git URL into a clean Unity 6 project; compiles with DEBUG_PANEL on and off.
