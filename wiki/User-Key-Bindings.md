This page describes how user can update key bindings in Brackets.

Key bindings for default commands or extensions can be overridden or removed.

Currently, only key bindings for commands can be updated -- not editor bindings currently handled by the CodeMirror keymap. Key bindings cannot be modified for special commands: cut, copy, paste, undo, redo, or select all.

### Quick Start
Use `Debug > Open User Key Map` to open the `keymap.json` in Brackets so it can be edited. If the file does not already exist, then it will be automatically created.

Changes to key bindings are applied immediately on File Save.

### Location
The `keymap.json` file is located in the the user data folder. The user data folder is located at:

* Mac: ```~/Library/Application Support/Brackets```
* Win XP: ```C:\Documents and Settings\<username>\Application Data\Brackets``` -- (aka ```%appdata%\Brackets```)
* Win Vista/7/8: ```C:\Users\<username>\AppData\Roaming\Brackets``` -- (aka ```%appdata%\Brackets```)
* Linux: ``~/.config/brackets`` -- (aka ```$XDG_CONFIG_HOME/brackets```)

### JSON Data Format

The JSON  data format is:

    {
        "overrides": {
            "<key>": "<command-id>"
            [, "<key>": "<command-id>"]
        }
    }

Where:

`<key>` is a unique key.
`<command-id>` is the command id string to assign, or `null` to remove a key binding.

Modifier keys are operating system specific, so on Mac use `Cmd` versus `Ctrl` on Windows. The following table lists all the valid modifier keys and other supported key names.

| Windows/Linux | Mac | Notes | 
|:--------------:|:---------------:|:--------:| 
|Ctrl|Cmd|| 
|Alt|Opt|| 
|Shift|Shift|| 
||Ctrl|Control key only available on Mac| 
|Up|Up|displayed as ↑ on mac|
|Down|Down|displayed as ↓ on mac|
|Left|Left|displayed as ← on mac|
|Right|Right|displayed as → on mac|
|Backspace|Backspace||
|Enter|Enter||
|Tab|Tab||
|Space| Space||
|PageUp|PageUp||
|PageDown|PageDown||
|Home|Home||
|End|End||
|Insert|Insert||
|Delete|Delete|||

Command ID can be found in [this file](https://github.com/adobe/brackets/blob/master/src/command/Commands.js), but it does not have any command ID from the extensions. So it is better to install "Display Shortcuts" extension from Extension Manager. See https://github.com/redmunds/brackets-display-shortcuts for how to use the extension to show the key bindings.

### Exceptions

#### Special Commands
Currently, the shortcuts to these commands cannot be updated or re-assigned:
* key: `Ctrl/Cmd-A`, command: `"edit.selectAll"`
* key: `Ctrl/Cmd-C`, command: `"edit.copy"`
* key: `Ctrl/Cmd-V`, command: `"edit.paste"`
* key: `Ctrl/Cmd-X`, command: `"edit.cut"`
* key: `Ctrl/Cmd-Z`, command: `"edit.undo"`
* key: `Ctrl/Cmd-Y, Cmd-Shift-Z`, command: `"edit.redo"`

#### Mac Reserved Shortcuts
These shortcuts cannot be overridden on Mac:
* `"Cmd-,"`
* `"Cmd-H"`
* `"Cmd-Alt-H"`
* `"Cmd-M"`
* `"Cmd-Q"`

### Import Mac key bindings from Sublime Text

```
    "overrides": {
        "Cmd-Shift-L":     "edit.splitSelIntoLines",
        "Ctrl-Shift-Up":   "edit.addCursorToPrevLine",
        "Ctrl-Shift-Down": "edit.addCursorToNextLine",
        "Cmd-Shift-D":     "edit.duplicate",
        "Ctrl-Shift-K":    "edit.deletelines",
        "Cmd-L":           "edit.selectLine",
        "Cmd-D":           "cmd.addNextMatch",
        "Cmd-Opt-1":       "cmd.splitViewNone",
        "Cmd-Opt-2":       "cmd.splitViewVertical",
        "Cmd-Opt-Shift-2": "cmd.splitViewHorizontal",
        "Ctrl-G":          "navigate.gotoLine",
        "Cmd-Opt-Right":   "navigate.nextDoc",
        "Cmd-Opt-Left":    "navigate.prevDoc",
        "Cmd-R":           "navigate.gotoDefinition",
        "Cmd-P":           "navigate.quickOpen"
    }
```

### Import Windows key bindings from Sublime Text

```
    "overrides": {
        "Ctrl-Shift-L":  "edit.splitSelIntoLines",
        "Ctrl-Alt-Up":   "edit.addCursorToPrevLine",
        "Ctrl-Alt-Down": "edit.addCursorToNextLine",
        "Ctrl-Shift-D":  "edit.duplicate",
        "Ctrl-Shift-K":  "edit.deletelines",
        "Ctrl-D":        "cmd.addNextMatch",
        "Alt-Shift-1":   "cmd.splitViewNone",
        "Alt-Shift-2":   "cmd.splitViewVertical",
        "Alt-Shift-8":   "cmd.splitViewHorizontal",
        "Ctrl-R":        "navigate.gotoDefinition",
        "Ctrl-P":        "navigate.quickOpen"
    }
```
