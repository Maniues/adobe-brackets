Brackets has a few different APIs for opening and managing files.  This document describes best practices for opening files and examines the differences between the APIs used for opening and managing files in Brackets.

## Best Practices
The best way to open a document in Brackets is to execute the command `CMD_ADD_TO_WORKINGSET_AND_OPEN`.  This command will open the file in the specified pane, add it to the working files list and select it in the working file list.  This is the preferred way to open a file.

Here's an example:
```javascript
CommandManager.execute(Commands.CMD_ADD_TO_WORKINGSET_AND_OPEN, {fullPath: "./view/WorkingSetView.js", paneId: "first-pane"});
```  


## API Details

| COMMAND | Function | Notes | 
| ------- | -------- | ----- |
| FILE_ADD_TO_WORKING_SET | Opens a Document to the Working Files List | Resolves to a Document Object. Deprecated. |
| CMD_ADD_TO_WORKINGSET_AND_OPEN | Opens a File to the Working Files List | Resolves to a File Object |
| FILE_OPEN | Opens a Document (not to the Working Files List) | Resolves to a Document Object. Deprecated. |
| CMD_OPEN | Opens a File (not to the Working Files List) | Resolves to a File Object  |


| API     | Function | Notes | 
| ------- | -------- | ----- |
| MainViewManager .addToWorkingSet | Adds a File to the Working Files List (does not open the file)| For Internal Use only |
| MainViewManager .addListToWorkingSet | Adds an array of Files to the Working Files List (does not open anything) | For Internal Use only |
| DocumentManager .setCurrentDocument | Sets the currently view document to the document specified  | Deprecated |
| FileViewController .openAndSelectDocument | opens the specified file and selects it | For Internal Use only |
| FileViewController .openFileAndAddToWorkingSet | opens the specified document and selects it | Deprecated. For Internal Use only. |



## File Open Command Data

When opening a file, you generally construct a call to `CommandManager.execute` along with its `commandData`. It's the `commandData` which provides various options when opening the file:

| Member | Meaning | Notes |
| ------ | ------- | ------- |
| fullPath | Path to the file to open | Required. Must be canonical. |
| index | Index in the working set to put the file | Optional. For internal use. |
| silent | Suppress error messages | Optional |
| forceRedraw | Forces the working set view to redraw | Optional. |
| paneId | Pane in which to open the file | Optional or undefined. May MainViewManger.ACTIVE_PANE or a valid pane id. |
| options | Options to use when creating the view of the file | Optional. |

## Options
Options to use when creating the view of the file

| Member | Meaning | Notes |
| ------ | ------- | ------- |
| noPaneActivate | Open the file in the specified pane but don't activate it | |
| noPaneRedundancyCheck | Improves performance by not checking to see if the document is open in another pane | For internal use. Used when the file has already been checked for redundancy. Will be deprecated. |

```javascript
CommandManager.execute(Commands.CMD_ADD_TO_WORKINGSET_AND_OPEN, {fullPath: "./view/WorkingSetView.js", paneId: "first-pane", options: {noPaneActivate: true}})
```  

## Pane IDs
PaneId strings which can be passed to any function requiring a `paneId`. These are not undocumented but, because the number of panes may change over time, it is not a finite list so it wasn't documented.  You can still get the list of paneIds by calling `MainViewManager.getPaneIdList()`  

| PaneId | Placement |
| ------ | ------- |
| "first-pane" | top/left |
| "second-pane" | bottom/right |
