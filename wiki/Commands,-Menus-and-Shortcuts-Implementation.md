This page describes how commands, menus and keyboard shortcuts are implemented in Brackets. 

### Populating commands
CommandManager module is responsible for creating commands. When a module or extension registers a command with CommandManager, CommandManager creates a new command and adds the new command instance to its command map. All Brackets core command IDs are listed in Commands.js as string constants. So to add a new command in Brackets core, add the new command ID in Commands.js first. Then, call `CommandManager.register(name, id, commandFn)` from the module where the command function is implemented.

#### CommandManager module
- register(name, id, commandFn)
- registerInternal(id, commandFn)
- get(id)
- getAll()
- execute(id)

- Command class
    - Command(name, id, commandFn)    // Constructor

    - getID()
    - execute()

    - getChecked()
    - getEnabled()
    - getName()

    - setChecked(checked)   // Also trigger menu update
    - setEnabled(enabled)   // Also trigger menu update
    - setName(name)         // Also trigger menu update

### Creating key bindings
Key bindings for Brackets core commands are defined in base-config/keyboard.json which is keyed by command IDs. When a command is registered with CommandManager, it also triggers `CommandRegistered` event to add the corresponding key bindings. The triggered event is then handled by `_handleCommandRegistered(event, command)` in KeyBindingManager module.

So to create key bindings for a new command, add your key bindings to base-config/keyboard.json in the following sample formats. If `platform` key is not specified, then `Ctrl` modifier key is used for cross-platforms. It means `Command` modifier key on Mac and `Ctrl` key on Windows. 

```
    "edit.undo": [
        "Ctrl-Z"
    ]
```    
If you want to use `Ctrl` for `control` modifier key on Mac, then you also need to explicitly specify it with `"platform": "mac"` as in the following example.
```
"edit.selectLine":  [
    "edit.selectLine":  [
        {
            "key": "Ctrl-L"
        },
        {
            "key": "Ctrl-L",
            "platform": "mac"
        }
    ]
```

In a similar way `Alt` is used for `option` modifier key on Mac and `Alt` key on Windows, and you do not have to specify the platform. However, you should use `Opt` for mac-only key binding.


### Populating menus
Brackets menus are populated in DefaultMenus module which explicitly calls `addMenu(name, id, position, relativeID)` for each menu and `addMenuItem(command, keyBindings, position, relativeID)` for each menu item. Calling `addMenuItem` also retreives the corresponding key binding from KeyBindingManager for the provided command so that the shortcut is added next to the menu item label.

### Native vs. HTML menus
Brackets provides native menus on Windows and Mac, but only HTML menus on Linux since native menus are not yet implemented on Linux. All context menus are still HTML menus on all platforms. In addition, in-browser version also uses HTML menus. So be sure to test on HTML menus when fixing issues for menus or key bindings. To test HTML menus on Windows or Mac, replace `var hasNativeMenus = params.get("hasNativeMenus");` with `var hasNativeMenus = "false";` in utils/Globals.js file and restart Brackets.

### Keydown events handling in KeyBindingManager
KeyBindingManager module not only handles key bindings, but it also listens to all key down events and handles them in `_handleKeyEvent(event)` function. However, if a keyboard shortcut is specified for a native menu item, then the corresponding command is executed directly via `executeCommand` in ShellAPI module and not through the keydown event handler in KeyBindingManager. `_handleKeyEvent` is invoked only when Brackets is using HTML menu or the invoked shortcut is not one of those listed in the native menus. For example, `Alt-E` shortcut (used with right Alt key) on a German keyboard layout for inserting a Euro symbol. So the following functions are added to handle AltGr shortcuts for non-US keyboard layouts.

- _quitAltGrMode()
- _onCtrlUp()
- _detectAltGrKeyDown(e)    // distinguish between right and left Alt keys

In addition, in order to prevent some keydown events being handled by KeyBindingManager as keyboard shortcuts, Brackets also provides keydown hooks with the following functions. 

- addGlobalKeydownHook(hook)
- removeGlobalKeydownHook(hook)

Dialogs module and CodeHintList module are using the keydown hooks to process some specific keys not defined as key bindings. Only keydown events not handled by global keydown hooks are passed to KeyBindingManager.