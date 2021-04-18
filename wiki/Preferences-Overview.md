This is an overview of all Brackets preference, state, and caching management as of January 2015.

### Persistence

This section describes where Brackets stores data.

#### Client

Brackets Client is the code in [brackets repo](https://github.com/adobe/brackets).
Data stored in client is classified as "preferences" and "states"
where preferences are settings user would explicity specify such as Tab Size,
and state information is settings implicitly set by app such as last file edited.

Client information is stored in JSON files:

| **Type** | **Location**
|----------|-------------
| Global user preferences | *user-dir*/brackets.json
| Project preferences | *project-dir*/.brackets.json
| State information | *user-dir*/state.json

Where:
* *user-dir* is the operating system specific user folder. See [this article](https://github.com/adobe/brackets/wiki/Cache-Folder).
* *project-dir* is the folder currently opened in Brackets using File > Open Folder..., or by dragging a folder and dropping it on Brackets.

#### Native

Brackets Native code is in the [brackets-shell](https://github.com/adobe/brackets-shell repo).
On Windows, brackets-shell stores data such as app window size and position
in the Windows Registry at `HKEY_CURRENT_USER/Software/Brackets`.
This is automatically handled by Mac OS and not yet implemented on Linux.

#### CEF cache

Chromium Embedded Framework caches data in *user-dir*/cef_data/. Files in this folder should never be edited manually. Folder can be deleted to clear cache.

#### Live Preview

Live Preview uses a custom Chrome browser profile which caches data in *user-dir*/live-dev-profile/. Files in this folder should never be edited manually. Folder can be deleted to clear cache.

#### Extension Manager

The Extension Manager currently adds files to and removes files from the *user-dir*/extensions/user folder. An extensions/disabled folder is also provided so users can manually move extensions from user folder for disabling, but this is not yet supported in Extension Manager.


### Preferences GUI

Currently, there is no *global* GUI for where every preference can be accessed.
On Mac, App > Preferences menu item is disabled.
But there are some places in UI where prefs can be set:

- Status bar
  - Tabs vs Spaces, spaceUnits
  - Show/hide lint errors
  - File type
  - Insert vs Overwrite mode
  - Error Indicator
    - Enabled with Debug > Show Errors in Status Bar
- Menu
  - File
    - Live Preview
    - Enable Experimental Live Preview
    - Project Settings... dialog
  - Edit
    - Auto Close Braces
  - View
    - almost everything...
  - Debug
    - Show Errors in Status Bar

### State

State info is stored for things such as:
- Previously opened projects
- Current project
- Working Set files
- MRU file list
- Project file tree folders: open/close state
- Scroll and cursor positions for previously edited files


### Updating Other Preferences

Preferences that cannot be set via GUI have to be set manually.
See [Preferences section of How To Use Brackets](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#preferences)
wiki page for more info on editing brackets.json file and a list of preferences defined in Brackets.

Saving preference file to disk automatically updates in Brackets,
even when edited with external editor.

For quick access to brackets.json use: Debug > Open Preferences File


### Extensions

Extensions have full access to Preference System so can get or set Brackets preferences/states and define their own preferences/states.


### Preference System Code

For detailed info see [Preference System documentation](https://github.com/adobe/brackets/wiki/Preferences-System) in wiki. Also see
[PreferencesBase module](http://brackets.io/docs/current/modules/preferences/PreferencesBase.html)
in API Docs.


