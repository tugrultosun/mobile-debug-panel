---
title: Documentation
permalink: /docs/
---

An in-game debug panel for Unity 6. Mark up a plain C# object, register it, and its public
members become toggles, number fields, dropdowns, text fields and buttons in an overlay you open
with a triple-tap. Includes a Console tab with the game's logs, a System tab with device facts,
and the same three tabs in an Editor window. Values can be saved across restarts, pinned to a small
on-screen box, and sent off a device as a bug report; logged errors raise an on-screen alert.

It is on in the Editor and in development builds with nothing to set up. Release builds carry
none of it unless `DEBUG_PANEL` is defined, and game code that calls it still compiles without it.

- Unity 6.3 (6000.3 or later); Android and iOS, IL2CPP or Mono; desktop players and the Editor.
- UI Toolkit, built in code. One stylesheet.
- Works with the legacy Input Manager and the Input System package.

## Install

Import it from the Asset Store: *Window › Package Manager › My Assets*, select **Mobile Debug Panel**,
then *Download* and *Import*. Everything goes into `Assets/DebugPanel`. You can move that folder
anywhere under `Assets/`; keep the `.meta` files with it, because the panel finds its own files by
their GUIDs, not by path.

To update, import the new version over the old one. The [CHANGELOG](changelog.md) lists what
changed. Within 1.x, code written against the panel keeps compiling.

That is all: the panel works in the Editor's Play Mode and in every build with *Development Build*
ticked. To also put it into a release build (an internal test build made with release settings),
turn it on in *Project Settings › Debug Panel* (or *Tools › Debug Panel › Enable On All Targets*),
which adds the `DEBUG_PANEL` scripting define for desktop, Android and iOS.

## Quick start

```csharp
using DebugPanel;
using UnityEngine;

[Category("Cheats")]
public class Cheats
{
    public bool GodMode { get; set; }                         // toggle

    [NumberRange(0, 99999), Increment(100)]
    public int Coins { get; set; }                            // number field with - / +

    [Category("Gameplay"), NumberRange(0, 4), Increment(0.25)]
    public float TimeScale                                    // float, clamped
    {
        get => Time.timeScale;
        set => Time.timeScale = value;
    }

    public Difficulty Difficulty { get; set; }                // dropdown
    public string PlayerName { get; set; } = "Player";       // text field

    [DisplayName("Unlock all levels"), Sort(-1)]
    public void UnlockAll() { /* ... */ }                     // button
}

public class CheatsInstaller : MonoBehaviour
{
    readonly Cheats _cheats = new Cheats();

    void OnEnable() => Panel.AddOptions(_cheats);
    void OnDisable() => Panel.RemoveOptions(_cheats);
}
```

Press Play, then press <kbd>`</kbd> in the Game view, or triple-tap the top-left corner on a
device. Open `Assets/DebugPanel/Samples/Cheats/CheatsSample.unity` and press Play: the panel opens with a class that uses every option type. The sample folder can be deleted once you have your own.

## Opening the panel

| Where | How |
| --- | --- |
| Device | Triple-tap a screen corner (top-left by default; change it in *Project Settings › Debug Panel*). The hit area is 10% of the screen's shorter side; taps must be under 0.4 s apart. |
| Editor and desktop players | <kbd>`</kbd> (backquote) toggles the panel. |
| Anywhere | <kbd>Esc</kbd> or Android back closes it. |
| Code | `Panel.Show()`, `Panel.Hide()`, `Panel.Toggle()`, `Panel.IsVisible`. |
| Editor window | *Window › Debug Panel* shows the same tabs, docked, in Play Mode. |

Buttons and text keep the same physical size on every phone. On larger screens the panel grows
with the screen, up to 1.75× on a large tablet or a desktop, so it fills the screen as it does
on a phone; it follows rotation and window resizes. On a phone in portrait, number, text and
dropdown options put their name on one line and the control on a full-width line below.

While it is open the panel covers the screen and blocks uGUI and anything else that goes through
the EventSystem. Input the game reads directly (`Input.GetMouseButtonDown`, `Input.touches`,
Input System actions) still reaches the game, so a tap on a cheat could also act on the game behind
it. `Panel.OnVisibilityChanged` (`Action<bool>`) fires when the panel opens or closes, so the game
can pause its own input:

```csharp
Panel.OnVisibilityChanged += open => inputEnabled = !open;
```

## Options

Any object can be a container, and a game can register several. Each container's public instance
members become options:

| Member | Shown as |
| --- | --- |
| `bool` property or field | toggle |
| `int`, `float`, `double` property or field | number field with - / + (hold to repeat) |
| enum property or field | dropdown |
| `string` property or field | text field |
| public method with no parameters, returning `void` | button |
| get-only property, `readonly` field | read-only row |

Anything else (other types, indexers, methods with parameters or a return value, generic
methods) is skipped, with one warning per container type listing what was skipped. Static members
are ignored. Structs cannot be containers. On a `MonoBehaviour` or `ScriptableObject`, only the
members your own classes declare are used, never Unity's (`enabled`, `name`, ...).

### Attributes

| Attribute | On | Effect |
| --- | --- | --- |
| `[Category("Name")]` | member or class | Groups options. On a class, it is the default for its members. |
| `[DisplayName("Label")]` | member | Label instead of the member name. |
| `[Sort(n)]` | member | Order inside the category; lower first. Default 0. |
| `[NumberRange(min, max)]` | number | Clamps typed values and - / + steps. |
| `[Increment(step)]` | number | Step for - / +. Default 1 for `int`, 0.1 for `float` and `double`. |
| `[Persist]` | member or class | Saves the value when you change it in the panel and brings it back next run. See [Saving values](#saving-values). |

Without `[DisplayName]`, member names are made readable: `m_godMode` → "God Mode",
`AddCoins100` → "Add Coins 100". Without any `[Category]`, the container's class name is the
category.

**Order.** Categories are ordered by the lowest `[Sort]` among their options, then by name. Inside
a category: by `[Sort]`, then by the order containers were registered, then by declaration order
(fields, then properties, then methods; base class members first).

**`System.ComponentModel` clash.** That namespace also has `CategoryAttribute` and
`DisplayNameAttribute`, and classes that implement `INotifyPropertyChanged` usually import it.
Then `[Category]` is ambiguous. Either alias ours, keeping the `Attribute` suffix:

```csharp
using System.ComponentModel;
using DebugPanel;
using CategoryAttribute = DebugPanel.CategoryAttribute;
using DisplayNameAttribute = DebugPanel.DisplayNameAttribute;
```

or use `System.ComponentModel`'s `[Category]` and `[DisplayName]`, which the panel also reads.

### Keeping values current

If a container implements `INotifyPropertyChanged`, rows update as soon as it raises
`PropertyChanged` (a null or empty name refreshes every row of that container). Otherwise the
rows on screen are re-read every 0.5 s. Only visible rows are refreshed, and the list is
virtualized, so hundreds of options cost the same as a dozen.

Exceptions thrown by your getters, setters or methods are logged and do not break the panel.

### Saving values

Mark an option `[Persist]` and the value you set in the panel survives a restart: God Mode stays
on, the chosen difficulty stays chosen.

```csharp
[Persist] public bool GodMode { get; set; }
[Persist] public Difficulty Difficulty { get; set; }

[Persist]                      // on a class: every writable option it declares
public class TutorialCheats { public bool SkipTutorial { get; set; } }
```

- The value is saved when it is changed **in the panel**, not when the game changes it. It is
  applied to the container when it is registered with `Panel.AddOptions`, through the setter,
  so `[NumberRange]` still clamps it.
- Stored in PlayerPrefs under `DebugPanel.<container type's full name>.<member name>`. Enums are
  stored by name, so reordering an enum keeps the choice; a saved value that no longer fits (a
  removed enum value, say) is ignored. `[Persist("key")]` uses your own key, for a member you renamed.
- Every registered container of a type gets the same saved values.
- Read-only members and buttons are never saved. Saved options show a small *saved* tag.
- Written to disk when the panel closes and when the app pauses or quits.
- *Reset saved* (next to the search box) or `Panel.ResetSavedOptions()` gives the options the
  values they had before a saved one was applied and deletes every saved value.

### Options built in code

For options that come from data rather than from a class, such as a button per level or a
dropdown of themes, build an `OptionContainer` and register it like any container:

```csharp
var levels = new OptionContainer("Levels");                 // the category
for (int i = 1; i <= 30; i++)
{
    int n = i;
    levels.AddButton($"Play level {n}", () => LevelLoader.Load(n));
}
levels.AddChoice("Theme", themeNames, () => currentTheme, i => currentTheme = i);
levels.AddInt("Lives", () => lives, v => lives = v, min: 0, max: 5);
Panel.AddOptions(levels);
```

| Method | Shown as |
| --- | --- |
| `AddButton(name, action)` | button |
| `AddToggle(name, get, set)` | toggle |
| `AddInt(name, get, set, min, max, step)` / `AddFloat(...)` | number field with - / + |
| `AddText(name, get, set)` | text field |
| `AddChoice(name, choices, getIndex, setIndex)` | dropdown of the given texts |

- Leave out the setter for a read-only row. Every method takes an optional `sort`.
- Adding a name that exists replaces it in place; `Remove(name)` and `Clear()` take options away.
  Changes show up at once, also while the container is registered.
- No reflection, so nothing to keep from code stripping. Options built in code are not `[Persist]`.
- In a release build the class still compiles and every method does nothing.

## Console

Captures every `Debug.Log*` message (from any thread) from start-up on, in development builds
too when nobody opens the panel. Keeps the newest 1000 messages; messages and stack traces over
16 KB are cut.

- Info / Warn / Error filters with counts (Error includes asserts and exceptions).
- Collapse identical messages into one line with a count.
- Tap a line for its full message and stack trace; copy it, or copy the whole visible list.
- Clear.

**Error alert.** While the panel is closed, an error, assert or exception puts a red *N errors*
line at the top of the screen (with the pinned values, if any), so a failure on a test device does
not go unnoticed. Opening the panel clears it. Turn it off with `Panel.ErrorAlertEnabled = false`.

## System

Application (version, identifier, Unity version, scripting backend, uptime, FPS, quality level),
device (model, OS, CPU, battery), memory (system RAM, Unity and managed heap usage), graphics and
display (resolution, refresh rate, DPI, safe area, orientation). Live values refresh every second
while the tab is open. *Copy* puts the whole report on the clipboard.

## Pinned values

*Pin* on an option row puts that value in a small box at the top of the screen that stays while
the panel is closed, so you can watch it while you play. *Pin FPS* on the System tab adds the
frame rate. The box sits in the top corner away from the one that opens the panel, inside the safe
area, and never takes a tap: everything under it still reaches the game.

Pins are saved, so they come back after a restart. A pinned option whose container is not
registered right now is simply not shown. Buttons cannot be pinned.

## Bug reports

*Report* in the panel's tab bar asks *What went wrong?* (optional), then saves a folder with:

- `screenshot.png`: the game without the panel or the pinned box.
- `report.txt`: time and scenes, the description, every option's value, the System report, and
  the whole Console log with stack traces.

From code: `Panel.SaveReport()` or `Panel.SaveReport("what went wrong")`, without asking.

The text is also copied to the clipboard, so you can paste it straight into a chat or a ticket.
`Panel.OnReportSaved` (`Action<string>`) gives the folder, if the game wants to share it with a
share-sheet plugin.

The folders are under `Application.persistentDataPath/DebugPanel/Reports/<date-time>`:

| Platform | Getting the files |
| --- | --- |
| Android | `adb pull /sdcard/Android/data/<package>/files/DebugPanel/Reports` |
| iOS | Finder or the Files app, if the app has *File Sharing* (`UIFileSharingEnabled`) on |
| Editor and desktop | `Application.persistentDataPath` (on Windows, under `AppData/LocalLow`) |

## Release builds

| Where | Panel code | Panel assets | `Panel` calls |
| --- | --- | --- | --- |
| Editor (Play Mode) | yes | yes | work |
| Development build | yes | yes | work |
| Release build with `DEBUG_PANEL` defined | yes | yes | work |
| Release build without it | no | no | compile, do nothing |

Without the define, `DebugPanel.Runtime` contains only the attributes and an empty `Panel`
class: `AddOptions` and `Show` do nothing, `IsVisible` is always false, and `Panel.IsAvailable`
is `false`. The panel's PanelSettings, theme and stylesheet are not in a `Resources` folder; the
package's Editor code adds them to a build only when that build compiles the panel.

## IL2CPP and code stripping

The panel binds properties and methods with `Delegate.CreateDelegate`, and reads fields and enum
properties by reflection. It never uses `Expression.Compile`, so it runs under IL2CPP.

Your cheat members are only reached by reflection, so managed code stripping would normally
remove them. The package prevents that on its own: for every build that compiles the panel it
writes a `link.xml` that keeps whole each class using one of its attributes (`[Category]`,
`[Persist]`, `[Sort]` and the rest, on the class, a base class or any member), with its base
classes. You need no `link.xml` of your own for them, and a release build without the panel
keeps none of them.

A cheat class that uses no panel attribute at all is not found: put `[Category("...")]` on it.
Options built in code (`OptionContainer`) use no reflection and need nothing.

## Settings

*Project Settings › Debug Panel*:

- **In release builds too** per build target: adds or removes `DEBUG_PANEL`. The Editor and
  development builds always have the panel.
- **Triple-tap corner**: stored as a define (`DEBUG_PANEL_CORNER_TOP_RIGHT`,
  `..._BOTTOM_LEFT`, `..._BOTTOM_RIGHT`; none means top-left), so nothing is stored in the build.

Changing a setting edits the scripting defines and recompiles scripts. A build script can do the
same with `DebugPanelDefines` (Editor only), for example to put the panel into one QA build made
with release settings:

```csharp
DebugPanelDefines.SetEnabled(NamedBuildTarget.Android, true);
```

## Known Unity messages

- **"Hiding input field is not supported when using Game Activity."** A warning logged on Android
  when you type in the panel (the search box, a text option, the report description). It comes from
  Unity, not the panel: under GameActivity, Unity 6's default Android entry point, Unity reports the
  keyboard's input field as always hidden, every UI Toolkit text field then asks to hide it, and
  Unity's GameActivity code warns about that request. Any UI Toolkit text field in your game logs the
  same. Typing works normally.

## License

Licensed under the Standard Unity Asset Store EULA. For support, email ttosun350@gmail.com.
