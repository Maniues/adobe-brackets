Brackets stores some data in a cache folder managed by CEF -- roughly equivalent to a browser cache, except specific to Brackets. At the moment, this folder stores both throwaway data **_and some user preferences_**.

If you're having trouble starting Brackets after upgrading from a previous build, try deleting this folder (see below). You may lose some stored preferences (especially any custom extension-specific preferences).

## Cache location

* Mac: ```~/Library/Application Support/Brackets/cef_data```
* Win XP: ```C:\Documents and Settings\<username>\Application Data\Brackets\cef_data``` -- (aka ```%appdata%\Brackets\cef_data```)
* Win Vista/7/8/10: ```C:\Users\<username>\AppData\Roaming\Brackets\cef_data``` -- (aka ```%appdata%\Brackets\cef_data```)
* Linux: ``~/.config/brackets/cef_data`` -- (aka ```$XDG_CONFIG_HOME/brackets/cef_data```)

Note: for **Adobe Edge Code**, replace `Brackets` in the path with `Adobe/Edge Code`.

## Resetting cache & preferences

1. Quit Brackets entirely
2. Delete the `cef_data` folder located above
3. In the parent of this folder, delete `brackets.json` and `state.json` if present

This will _reset all of your preferences_ and clear all cached data.  (It's not yet possible to do clear one of those separately from the other).  To ensure a full 'factory reset,' also uninstall Brackets and reinstall from a freshly downloaded installer.