These steps are designed to walk you through almost every piece of the Brackets UI. This is especially valuable when working on localization since it lets you see every string in the context it's used in.

### Tip on Mac/Unix (Optional)
Using Symbolic links, you can test your localization directly in the APP. (Sprint version 26.)
```bash
# In Terminal.
# Assuming you have Brackets Sprint installed into your Applications folder
$ cd /Applications/Brackets\ Sprint\ 26.app/Contents/www/nls/
# ln -s source_file/directory target_file/directory
# Change "lang" with your localization.
$ ln -s lang ~/path/to/forked/project/brackets/src/nls/lang/
# Default (root) - Fallback for language, you may change the strings.js file.
# Add your language to the LANGUAGE_xx: "Language" dict.
$ ln -s root ~/path/to/forked/project/brackets/src/nls/root/
# string.js - With your localization specified.
$ ln -s strings.js ~/path/to/forked/project/brackets/src/nls/lang/strings.js
```
**NOTE 1:** Does file already exist in the `/Applications/Brackets Sprint 26.app/Contents/www/nls/` folder, you will have to delete it first, before writing your symbolic link. (ln -s) You can also use `ls -la` to see were your symbolic links are pointed to.

**NOTE 2:** This isn't meant to overtake the "Notes for testing," only meant as another way to watch how your language looks like in the Brackets APP. This way you can simply CMD+R (if you use Brackets to make your localization) every time you save the `lang/strings.js` file. To see your stuff in action as is.

### Notes for testing

* Clone the brackets git repository to your local machine. The repo contains test used for these localization tests.
* ``citrus completed`` test project: https://github.com/adobe/brackets/tree/master/test/smokes/citrus%20completed
* ``server-smokes`` test project: https://github.com/adobe/brackets/tree/master/test/smokes/server-smokes
* To setup each test, load the ``citrus completed`` project via ``File > Open folder...``
* Placeholders for screenshots are written as ``<image_filename_placeholder>``
* To open files from the project panel (i.e. to add them to the working set), double-click on the file.

# General UI

### Menus

**Note: Debug menu is not tested**

1. Top Level
2. File
3. Edit
4. Find
5. View
6. Navigate
7. Help

### Toolbar
1. Lightning bolt icon tooltip ``<tooltip_live_preview>``
2. Extension Manager

### About
1. On Mac, ``Brackets > About Brackets``. On Windows, ``Help > About Brackets``
2. Confirm about dialog text ``<dialog_about>``

### Sidebar

1. Click ``View > Hide Sidebar`` menu
2. Open the ``View`` menu again
3. Confirm the label has changed to ``View > Show Sidebar`` ``<menu_view_show_sidebar>``
4. Click ``View > Show Sidebar``
5. Open ``index.html``.  In the Sidebar, click the gear icon in the Working Files header.  Confirm the sort methods.
6. In the Sidebar, click the Split icon at the top. Confirm the Split View options.

# File Operations

### Open

1. Click ``File > Open...`` menu
2. Confirm file open dialog ``<dialog_file_open>``
3. Choose ``Cancel``
4. Click ``File > Open Folder...`` menu
5. Confirm open folder dialog ``<dialog_folder_open>``

### Quick Open

1. Click ``Navigate > Quick Open``
2. Confirm the quick open UI appears at the top of the editor ``<dialog_quick_open>``

### Error opening file (Mac only)

_On Windows, we have a backlog item to address the lack of an error message https://trello.com/c/lSdnZmBc_

1. In the project tree, expand the ``images`` folder.
2. Click on ``events.jpg``
3. Confirm error message ``<dialog_error_opening_file>``

### Invalid file name

1. Click ``File > New`` menu
2. Confirm ``untitled.js`` editable file name <project_panel_untitled_file>
3. Rename ``untitled.js`` to ``foo:js``
4. Confirm error message ``<dialog_error_invalid_file_name>``
5. Select ``index.html``
6. Click ``File > New`` menu
7. Rename ``untitled.js`` to ``index.html``
8. Confirm error message ``<dialog_error_file_already_exists>``

### Save

1. Open ``index.html``
2. Make any change
3. Click ``File > Close`` menu
4. Confirm dialog ``<dialog_save_changes_one_file>``
5. Choose ``Cancel``
6. From the project tree, open ``desktop.css``
7. Make any change
8. Click ``File > Quit`` menu
9. Confirm dialog ``<dialog_save_changes_multiple_files>``
10. Choose ``Cancel``
11. In the operating system, delete ``desktop.css``
12. Return to Brackets
13. Confirm the external changes dialog ``<dialog_external_changes_deleted>``
14. Choose ``Close (Don't Save)``
15. In the operating system, restore ``desktop.css``

# File Errors

**Note: These tests assume that Brackets does not update the project tree when files are added and removed. Since these tests require changes to the file system, it is recommended to work with a separate copy of the ``citrus completed`` project and to close and restart Brackets for each test.**

### Directory Permissions

1. In the operating system, remove **write** permissions from the ``css`` directory. On Mac, use ``chmod 444 css``. On Windows, open the file properties dialog. Under "Security", click "Edit...". Choose your user account, then check the "Deny" checkbox for the "Write" permission. Save permission changes when finished.
2. In Brackets, select the ``css`` directory in the project tree.
3. Click ``File > New`` menu
4. Hit Enter to accept the default name
5. Confirm error creating file dialog ``<dialog_error_creating_file_no_modifications>``
6. In the operating system, restore the original permissions to the ``css`` directory

### Write Permissions

1. In the operating system, remove **write** permissions from ``index.html``. On Mac, use ``chmod 444 index.html``. On Windows, open the file properties dialog. Under "Security", click "Edit...". Choose your user account, then check the "Deny" checkbox for the "Write" permission. Save permission changes when finished.
2. In Brackets, open ``index.html``
3. Make any edit to the file
4. Click ``File > Save`` menu
5. Confirm error saving file dialog ``<dialog_error_saving_file_no_modifications>``
6. In the operating system, restore the original file permissions to ``index.html``

### Read Permissions

1. In the operating system, remove **read** permissions from ``index.html``. On Mac, use ``chmod 0 index.html``. On Windows, open the file properties dialog. Under "Security", click "Edit...". Choose your user account, then check the "Deny" checkbox for the "Read" permission. Save permission changes when finished.
2. In Brackets, open ``index.html``
3. Confirm error opening file dialog ``<dialog_error_opening_file_not_readable>``
4. In the operating system, restore the original file permissions to ``index.html``

### Read Permissions on Reload

1. In Brackets, open ``index.html``
2. Make any edit, do not save. A dirty dot should appear in the working set next to the file and in the toolbar next to the file name.
3. Open ``index.html`` in a different program.
4. Dirty the file so that the last modified time metadata is updated
5. Return to Brackets
6. Confirm dialog ``<dialog_external_changes_reload>``
7. In the operating system, remove **read** permissions from ``index.html``. On Mac, use ``chmod 0 index.html``. On Windows, open the file properties dialog. Under "Security", click "Edit...". Choose your user account, then check the "Deny" checkbox for the "Read" permission. Save permission changes when finished.
8. Return to Brackets
9. Choose ``Reload from Disk``
10. Confirm dialog ``<dialog_error_reloading_changes_from_disk>``
11. Choose ``OK`` and confirm the file in Brackets is still dirty
12. In the operating system, restore the original file permissions to index.html

### File Encoding Error (Mac only)

1. Open ``index.html`` in a text editor that allows saving with a different encoding (e.g. Sublime Text 2)
2. In the text editor save ``index.html`` with encoding: UTF-16 BE with BOM
3. Return to Brackets
4. Open ``index.html``
5. Confirm dialog ``<dialog_error_reloading_changes_from_disk_generic_error_code_2>``
6. Choose ``OK``
7. In the text editor save ``index.html`` restore the original UTF-8 encoding

### File not found

1. In Brackets, close ``index.html`` if open
2. In the operating system, delete ``index.html``
3. In Brackets, ``index.html`` is still listed.
4. Attempt to open ``index.html``
5. Confirm error opening file dialog ``<dialog_error_opening_file_file_not_found>``
6. In the operating system, restore ``index.html``

### Directory not found

1. In Brackets, collapse all folders (only ``css``, ``images`` and ``index.html`` should be visible in the tree)
2. Quit and restart Brackets
3. In the operating system, delete the ``images`` directory
4. Attempt to open the ``images`` directory
5. Confirm error loading project dialog ``<dialog_error_loading_project_directory_contents>``
6. In the operating system, restore the ``images`` directory

### Directory not found at startup

1. With ``citrus completed`` as the current project, quit Brackets
2. In the operating system, rename the ``citrus completed`` directory to ``foo``
3. Open Brackets
4. Confirm the error loading project dialog ``<dialog_error_loading_project_request_nfs>``
5. In the operating system, restore the original directory name to ``citrus completed``

# Editor

### Find/Replace
1. Open ``index.html``
2. Click ``Find > Find`` menu
3. Confirm find UI appears at the top of the editor ``<dialog_find>``
4. Press escape
5. Click ``Find > Replace`` menu
6. Confirm replace UI appears at the top of the editor ``<dialog_replace_1>``
7. Type any character(s) into the text box, press Enter
8. Confirm replace UI appears at the top of the editor ``<dialog_replace_2>``
9. Type a search string that actually exists in the open document.  Click ``Batch..``
10. Confirm Replace matches UI appears at the bottom of the editor.  Click the 'X' to close the UI.

### Find/Replace in Files

1. Click ``Find > Find in Files``
2. Confirm find UI appears at the top of the editor.
3. Click ``No Files Excluded`` button.  Confirm exclusion set menu dropdown.
4. From that menu, click ``New Exclusion Set...``.  Confirm Edit Exclusion Set dialog appears.
5. Click ``Cancel`` to close the dialog.
6. Click ``Find > Replace in Files`` menu.
7. Confirm replace in files UI appears at the top of the editor.
8. Type a search string that actually exists in the open document.  Click ``Replace...``
9. Confirm Replace in files matches UI appears at the bottom of the editor.  Click the 'X' to close the UI.
10. Press escape to close the Replace in Files bar.

### Find/Replace in Selected File/Folder

1. Click ``Find > Find in Select File/Folder``
2. Confirm Find in selected location UI appears at the top of the editor.
3. Press escape to close the Find bar.
4. Click ``Find > Replace in Select File/Folder``
5. Confirm Replace in selected location UI appears at the top of the editor.
6. Press escape to close the Replace bar.

### Collapse/Expand Current Block of Code

1. Place cursor randomly inside a file (like html,js,json,c,etc.).
2. Click ``View > Collapse Current``
3. Confirm that the innermost block (that the current location belongs to) gets collapsed.
4. Click ``View > Expand Current``
5. Confirm that the innermost block (that the current location belongs to) gets expanded.

### Collapse/Expand All

1. Click ``View > Collapse All``
2. Confirm that all blocks of code in the file get collapsed. Inner blocks will be hidden inside outer ones.
3. Click ``View > Expand All``
4. Confirm that all blocks of code get expanded.

### JSLint

1. Enable ``View > Enable JSLint`` (a checkmark will appear when enabled)
2. Confirm JSLint panel title ``<panel_jslint>``

# Inline Editors

## Inline Color Editor

1. Open ``desktop.css``
2. Place the cursor on any hex color, e.g. #000. Press ``CMD+E`` or ``Ctrl+E``
3. In the saturation/luminosity, select a new contrasting color
4. Hover over the current color swatch (the new chosen color). Confirm tooltip ``<current_color>``.
5. Hover over the original color swatch (the original color). Confirm tooltip ``<original_color>``.
6. Hover over each button in the button bar that contains (RGBa, HEX, HSLa). Confirm tooltips ``<rgba_format>``, ``<hex_format>`` and ``<hsla_format>``.
7. Hover over one of the colors listed on the right. Confirm tooltop ``<color_used_n_times>``.

### Inline Cubic Bezier Editor

1. At the end of ``desktop.css``, add the following line: ``#div2 {transition-timing-function: cubic-bezier();}``.
2. Place the cursor on the word `timing` in the new line.  Press ``CMD+E`` or ``Ctrl+E``.  Confirm labels on grid, and information text on right.

### Inline Step Editor

1. At the end of ``desktop.css``, add the following line: ``#div3 {transition-timing-function: steps();}``.
2. Place the cursor on the word `timing` in the new line.  Press ``CMD+E`` or ``Ctrl+E``.  Confirm labels on grid, and information text on the right.

### New Rule Editor

1. Within the first ``<body>`` tag of ``index.html``, start a new line and type ``<h10``.
2. Press ``CMD+E`` or ``Ctrl+E``.  Confirm New Rule inline editor appears.
3. Click ``New Rule``.  Confirm inline editor changes to show the new rule definition.
4. Click ``New Rule`` button again.  Confirm a list of references to the new rule appears on the right-side of the inline editor.
5. Press escape to close the inline editor.

### Inline Editor Errors

1. Open ``index.html``, place cursor at end of any line.
2. Press ``CMD+E`` or ``Ctrl+E``.  Confirm popover message appears with "No Quick Edit available for current cursor position", and messages fades away after 5 seconds.
3. Place cursor in ``href`` attribute of any ``<a>`` tag.
4. Press ``CMD+E`` or ``Ctrl+E``.  Confirm popover message appears with "CSS Quick Edit: place cursor in tag, class, or id".

# Brackets Health Report

1. Click ``Help > Health Report``.
2. Confirm that the Brackets Health Report dialog comes up.
3. Confirm that it shows the option to enable/disable sharing data.
4. Also confirm that it shows the data that would be sent if sharing is enabled.

**NOTE: Health Data pop up appears for the very first launch of Brackets (v1.3 onwards).**

# Quick Docs

1. Open ``desktop.css``, place cursor in any CSS Property (e.g. "font-family").
2. Press ``CMD+K`` or ``Ctrl+K``.  Confirm Quick Docs Viewer is opened with information for the chosen CSS Property.
3. Press ``Esc`` to close.
4. Place cursor at end of any line.
5. Press ``CMD+K`` or ``Ctrl+K``.  Confirm popover message appears with "No Quick Docs available for current cursor position", and messages fades away after 5 seconds.

# Live Preview

**Requires Google Chrome to be installed http://www.google.com/chrome**

### Connect

1. Exit Chrome if it is running.
2. Open ``index.html``
3. Click ``File > Live Preview`` menu
4. [First launch only] Confirm ``Welcome to Live Preview!`` dialog appears.  Click OK.
5. In Brackets, hover over the lightning bold icon
6. Confirm tooltip ``<tooltip_disconnect_live_file_preview>``
7. Quit Chrome

### Connection Error (Mac only)
1. Start Chrome
2. Click ``File > Live Preview`` menu
3. Confirm live preview error dialog ``<dialog_live_development_connection_error>``
4. Click ``Cancel``

### Wrong File Type
1. Open ``css\desktop.css``
2. Click ``File > Live Preview`` menu
3. If there is a file named ``index.html`` (or ``index.htm``), in same folder or parent folder within project, it will be opened.
4. Otherwise, confirm live preview error dialog ``<dialog_live_development_file_type_error>``

### Chrome Not Found
1. Close Chrome if running
2. Rename the Chrome executable
3. Open ``index.html``
4. Click ``File > Live Preview`` menu
5. Confirm live preview error dialog ``<dialog_live_development_chrome_not_found>``
6. Restore the Chrome executable name

### Base URL
1. Open the ``server-smokes`` project
2. ``File > Project Settings...``
3. Confirm project settings dialog ``<project_settings_dialog>``
4. Open ``server.php``
5. ``File > Live Preview``
6. Confirm project settings dialog appears with warning message ``<project_settings_specify_base_url>``
7. Enter ``ftp://path/to/foo`` (does not have to be a functioning path)
8. Confirm error message ``<project_settings_error_invalid_protocol>``
9. Enter ``http://path/to?foo``
10. Confirm error message ``<project_settings_error_search_disallowed>``
11. Enter ``http://path/to#foo``
12. Confirm error message ``<project_settings_error_hash_disallowed>``
13. Enter ``http://path to foo``
14. Confirm error message ``<project_settings_error_invalid_char>``

# Update Notification

The following tests configure fake updates for testing ``Help > Check for Updates``. Update information is normally pulled from http://brackets.io.

### Manual Check for Updates
1. Click ``Debug > Show Developer Tools`` to open dev tools window (if Debug is not available, open Chrome, go to http://localhost:9234, then choose the last link)
2. Click the ``Sources`` tab at the top of the dev tools window
3. If the script console is hidden, press ``esc`` to open the console
4. In the console window, enter:
``require("utils/UpdateNotification").checkForUpdate(true, {_buildNumber: 0, _lastNotifiedBuildNumber: 0})``
5. Confirm update information dialog appears ``<dialog_update_notification_up_to_date>``
6. In the console window, enter:
```
require("utils/UpdateNotification").checkForUpdate(true, {_buildNumber: 0, _lastNotifiedBuildNumber: 0, _versionInfoURL: "https://raw.github.com/adobe/brackets/master/test/spec/UpdateNotification-test-files/versionInfo.json"})
```
7. Confirm update information dialog appears ``<dialog_update_notification_update_available>`` (Note that the actual notes of each release are not translated in this test file)
8. Press Cancel
9. Hover over the update icon (a gift box) in the toolbar
10. Confirm update information tooltip appears ``<tooltip_update_notification>``

# Extension Installation

1. ``File > Extension Manager...``  Confirm Extension Manager dialog appears.
2. From the Available tab, click ``Install from URL...``.  Confirm Install Extension dialog appears.
3. Paste this URL https://github.com/adobe/brackets/raw/master/test/spec/extension-test-files/incompatible-version.zip and press Install
3. Confirm error message appears in dialog ``<install_extension_incompatible_version>``.  Click ``Close``.
4. Click Installed tab.  Confirm either ``No extensions installed yet.`` message appears in the dialog, or, if you've previously installed one or more extensions, that they are listed.
5. Click ``Close``

# Themes

1. ``View > Themes...``  Confirm Themes dialog appears
2. Change Current Theme, confirm colors in Editor change
3. Click Cancel, confirm editor colors revert to original colors

# Split View

1. ``View > Split Vertically``
2. Edit 2 files in side by side panes. There are now 2 working sets named **Left** and **Right**
3. ``View > Split Horizontally``
4. Edit 2 files one above the other. The working sets are named **Top** and **Bottom**
5. Drag separator to change pane sizes.
6. In Sidebar, drag a file from one working set to the other to switch panes.
7. ``View > No Split`` - to return to editing a single file

# User Key Map

### Corrupted Key Map File Error
1. `Debug > Open User Key Map`.
2. Delete `}` on the last line and save the file.
3. Confirm the title and the error message in the dialog.

### Error on Loading Key Map File
1. `Debug > Open User Key Map`.
2. Right click on keymap.json in the working set and select Show in Finder/Explorer from context menu.
3. Copy an image file in Finder/Explorer and replace keymap.json. i.e rename your image file to keymap.json after copying.
4. Back in Brackets keymap.json may no longer be showing in the workint set. So `Debug > Open User Key Map` again.
5. Verify the title and the error message in the dialog.
6. Delete the fake keymap.json in Finder/Explorer.

### Restricted Commands Error
1. `Debug > Open User Key Map`.
2. Paste `"cmd-w": "edit.copy"` or `"Ctrl-w": "edit.copy"` to the empty line inside the `"overrides"` block.
3. Save the file and verify that you get the correct error message.

### Restricted Shortcuts Error
1. `Debug > Open User Key Map`.
2. Paste `"cmd-c": "edit.selectLine"` or `"Ctrl-C": "edit.selectLine"` to the empty line inside the `"overrides"` block.
3. Save the file and verify that you get the correct error message.
4. On Mac, you can also test with `"Cmd-Q"`, `"Cmd-H"` or `"Cmd-M"` in step 2.

### Multiple Shortcuts to the Same Command Error
```
    "ctrl-W": "edit.selectLine",
    "Ctrl-o": "edit.selectLine"
```    
1. `Debug > Open User Key Map`.
2. Paste the above key bindings and replace the content inside the `"overrides"` block.
3. Save the file and verify that you get the correct error message.

### Duplicate Shortcuts Error
```
    "ctrl-W": "edit.selectLine",
    "Ctrl-w": "edit.lineComment",
    "ctrl-w": "edit.selectLine"
```    
1. `Debug > Open User Key Map`.
2. Paste the above key bindings and replace the content inside the `"overrides"` block.
3. Save the file and verify that you get the correct error message.

### Invalid Shortcuts Error
```
    "Command-W": "edit.selectLine",
    "Control-w": "edit.lineComment",
    "ctrl-pageup": "edit.blockComment"
```    
1. `Debug > Open User Key Map`.
2. Paste the above key bindings and replace the content inside the `"overrides"` block.
3. Save the file and verify that you get the correct error message.

### Non-existent Commands Error
```
    "ctrl-o": "edit.NotACommand",
    "Ctrl-w": "edit.noComment",
    "ctrl-i": "test.blockComment"
```    
1. `Debug > Open User Key Map`.
2. Paste the above key bindings and replace the content inside the `"overrides"` block.
3. Save the file and verify that you get the correct error message.
