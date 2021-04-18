Refactoring Brackets to Support Split View with Multiple Documents requires quite a bit of hacking on the plumbing but there are only a few places that need to be heavily refactored to do so.

# Current Implementation
This is an objective view of the system. 

The `.content` area of the application contains many widgets (in addition to the editor) that add to Bracket's user experience. These all start with the `PanelManager` which, for all intents and purposes, only manages the placement of these widgets. 

Here's a 50,000ft view of how it's glued together:
 
`PanelManager` View which manages the placement of all panels (subviews) in the dom node `.content` it does not manage the status-bar.  
`....+->` Replace all Results Panel (subview of mustache-rendered dom node `#search-results`) managed by FindReplace.js   
`....+->` Find In Files Results Panel (subview of mustache-rendered dom node `#replace-all-results`) managed by FindInFiles.js  
`....+->` Problems (jsLint) Panel (subview of of mustache-rendered dom node `#problems-panel`)
Managed by CodeInspection.js  

Various services create, show and manipulate these panels by rendering some HTML using Mustache and adding to the dom by calling `PanelManager.createBottomPanel()` which wraps the HTML snippet and inserts it into the DOM. 

`EditorManager` Manages the creation and destruction of an editor for a document.   
`....+-> Editor` Wraps a code mirror instance, code mirror options and provides a high level api this is basically the view attached to `#editor-holder`  

Currently there is only 1 instance of the visible editor so its dom element is part of the base HTML and it's referenced whenever needed as `$(#editor-holder)` 

The layout of the editor is fluid for the most part, but the height is computed by `PanelManager` and the editor is resized by triggering an Event and passing data with the event to tell the editor how tall it should be. The `EditorManager` then handles this event and sets the `$(#editor-holder)` to the explicit height negotiated by the panel manager.  The event is passed to the `Editor` object which then tells codemirror to recompute its scroll state and other size related data.

# Proposed Implementation 
`EditorLayoutManager` is a helper for `EditorManager` to compute the placement and layout of all full sized editors (inline editors are not managed here) and has various APIs for changing the layout of the editor area and placement `Editor` objects.  The editor objects themselves are managed by the `EditorManager` and the `EditorLayoutManager` is created whenever the layout needs to change to adjust all editor sizes and placement.

`EditorManager` View of `#editor-holder`    
`....+-> Array[1][1]` of `Editor` instances  (`.editor-pane`, id=paneId)  

There cannot be `0,0` (rows, columns) elements so there needs to be logic to enforce that.  Initially it is 1 row 1 column so there will always be at least 1 instance of `EditorManager` to work on.

## EditorPaneManager.setLayoutScheme(_rows_,_columns_)  

This API will change the layout to match _rows_ and _columns_.   

RULES:
* Only 1 row and 2 columns or 2 rows and 1 column are initially supported
* Any Pane created by this API initially will show the Brackets logo interstitial screen until the corresponding `EditorManager` for that pane has been loaded with a document or image.
* When a Pane is  destroyed, all documents in the corresponding working set for that pane are moved to another pane's working set.  Since there is only 2 panes in the initial implementation this is just a matter of collapsing them down to the remaining Pane's working set.

Creating a pane is rendered at runtime using Mustache to generate the HTML and insert it into the DOM.  The `Editor` Instance will generate the HTML when the `EditorManager` asks for it and `EditorPaneManager` will insert it into the DOM in the appropriate place to ensure proper keyboard navigation.  The generation can either be Mustache generated or simple jQuery insertion.

`EditorManager` will handle the `PanelManager`'s resize event and call a function on its `EditorPaneManager` to update its layout. 

The `PanelManager` computes the new size of `#editory-layout-holder` in its `window.resize` event handler and triggers an event to resize that `EditorManager` subscribes to.  Currently `PanelManager` only computes the Height and passes that as data but it needs to also compute Width and pass that as well so that the `EditorPaneManager` can verify that the width doesn't get too lean to handle 2 editor instances.  At the point that it is too lean then it will need to start resizing other columns to get a decent layout.  This algorithm becomes more complex with more columns and rows.  For now it can be a matter of going to something like 50%.  We may want to bump up the shell's minimum width as well to avoid getting too small.

These are proposed APIs that may not be implemented initially.  

## EditorPaneManager.setColumnWidth(_column_, _width_)  
## EditorPaneManager.setRowHeight(_row_, _height_)  

_height_ and _width_ are passed in in pixels and converted to a percentage when affixing the CSS to the columns (e.g. `width: 40%`).  Doing it in a percentage and only applying to all except the rightmost column and bottom most row will yield a fluid layout.  The API will reject setting the width on the rightmost column.  For the initial implementation we may just go with 50% splits all around without the ability to resize. 

`EditorManager` will manage a Working Set for each of its Editor Panes. This may be abstracted and delegated into a pane object if implementation warrants but the API to get the working set will be on EditorManager to make the interface easier to use. Management of the Working Set will move from the `DocumentManager` into `EditorManager` the and the working set will no longer be a collection of `Document` objects.  It will be an array of file names.  Theoretically we could register a view factory to create a view object for each file type (images, html, css, etc...) which would provide an easy way for custom viewers.  This could be extensible in some way but that work is outside the scope of this document.  For the time being, we only manifest CODE and Image viewers.

Note: To abstract the working set's pane location, each editor pane is addressed by paneId rather than row,col.  This is a change from the previous draft which had row, col addressable panes.  Valid paneId values cannot be `false, 0, null, undefined or ""` so that they can be used in `truthy` tests.

 
This allows for:   
1) Panes to be found in the DOM by ID  
2) Easier to move panes around when changing layouts in the future and not break API  
3) less data that callers need to understand about the implementation details  

## Constant Pane IDs
These are shortcuts paneIds for APIs to avoid having to maintain a reference to the pane in which a file belongs.  This also allows us to 1 API rather than 2 different APIs. One for `All` and one for `Focused` panes.

```text
------------------------+-----------------------------------------------------------------------------------------------------
Constant                | Usage
------------------------+-----------------------------------------------------------------------------------------------------
ALL_PANES               | Perform the operation on all panes (e.g. search for fullpath in all working sets)
FOCUSED_PANE            | Perform the operation on the currently focused pane (can also use EditorManager.getFocusedPane())
------------------------+-----------------------------------------------------------------------------------------------------
```

# EditorManager.getFocusedEditor()
Returns the editor object with the current focus
  
## EditorManager.getWorkingSet(paneId)  
## EditorManager.addToWorkingSet(paneId, _file_, _open_)  
## EditorManager.removeFromWorkingSet(paneId, _file_)  

## EditorManager.getAllWorkingSets()   
Returns an array of working set arrays.

### Working Sets

Working sets will move from being a property of the `DocumentManager` and become a property of the `EditorManager`

The plan for migrating the working set:

# Deprecate `DocumentManager.getWorkingSet()` 
A deprecation warning will be added and the function will return a unique list of all documents (not files) from all working sets by calling the various functions above, flattening and filtering the list as necessary to maintain backwards compatibility.

All core extensions and functions that use `DocumentManager.getWorkingSet()` will call `getAllWorkingSets` and return a list of working set entries who have open document objects.  This will exclude things like images which is the behavior that is currently implemented.

```text
3rd Party Extensions
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
Close-Others               | Adds Close commands the Working Set Menu| No change Needed. The extension is  
                           |                                         | packaged to run only on brackets  
                           |                                         | <= 0.32.0   
---------------------------+-----------------------------------------+-----------------------------------  
Brackets-Typescript        | Subclasses Working The Set and exposes  | This is a very complicated case.  
                           | back to the rest of the extension.  It  | We should probably contact the   
                           | also handles various DocumentManager    | extension author propose he   
                           | events that are triggered when the      | use the new api.  This is tricky  
                           | working set is changed.                 | because it's also responding to                                                              
                           |                                         | editorManager.activeEditorChange  
                           |                                         | should probably be ok but we   
                           |                                         | can use this to validate the old API  
---------------------------+-----------------------------------------+-----------------------------------  
Brackets-autosave-files-on-| Saves all files in working set on       | OK using deprecated API  
window-blur-1.0.4          | window.blur()                           | 
---------------------------+-----------------------------------------+-----------------------------------  
Brackets-autosave-files-on-| Saves all files in working set on       | OK using deprecated API  
window-blur-1.0.4          | window.blur()                           |    
---------------------------+-----------------------------------------+-----------------------------------  
Brackets-editor-nav-master | Adds a command for QuickOpen to search  | OK using deprecated API but   
                           | the open document list                  | Peter Flynn wrote it and can fix it  
---------------------------+-----------------------------------------+-----------------------------------  
Brackets-editor-nav-master | Adds a command for QuickOpen to search  | OK using deprecated API but   
                           | the open document list                  | Peter Flynn wrote it and can fix it  
---------------------------+-----------------------------------------+-----------------------------------  
zaggino.brackets.git       | Iterates over the working set and closes| OK using deprecated API   
                           | any document that has been removed      | Doesn't file watchers do this?  
---------------------------+-----------------------------------------+-----------------------------------  
brackets-wdminmap-master   | Hides a widget when the working set is  | OK using deprecated API  
                           | EMPTY                                   |
---------------------------+-----------------------------------------+-----------------------------------  
zaggino.brackets.git       | Iterates over the working set and closes| OK using deprecated API   
                           | any document that has been removed      | **Doesn't file watchers do this?**  
---------------------------+-----------------------------------------+-----------------------------------  
zaggino.brackets.git       | Adds a command to close unmodified files| OK using deprecated API   
                           | Iterates over the working set and closes|   
                           | any document that hasn't been modified  |   
---------------------------+-----------------------------------------+-----------------------------------  


Core Usage
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
Close-Others               | Adds Close commands the Working Set Menu| Uses New API for working set
                           |                                         | Working set info is passed in 
                           |                                         | Event data
---------------------------+-----------------------------------------+-----------------------------------  
DocumentCommnadHandlers    | Save and Close All Command Handlers     | Move to EditorCommandHandlers
                           |                                         |   HandleSave is passed on as
                           |                                         | DoCommand("Document.Save")
---------------------------+-----------------------------------------+-----------------------------------  
LiveDevelopment            | Initial Document                        | Replace with 
                           |                                         | DocumentManager
						   |                                         |   .getAllOpenDocuments()
---------------------------+-----------------------------------------+-----------------------------------  
FileSyncManager            | Scans documents in workingset that have | Replace with 
						   | yet to be opened                        |   EditorManager
                           |                                         |      .getAllWorkingSets()
                           |                                         | and iterate over all working sets
---------------------------+-----------------------------------------+-----------------------------------  
ProjectManager             | `getAllFiles` not sure what this does   | Replace with EditorManager
                           |                                         |      .getAllWorkingSets()
                           |                                         | and iterate over all working sets
---------------------------+-----------------------------------------+-----------------------------------  
WorkingSetView             | View of the working set. Constructed    | Replace with 
                           | with the event:                         |    EditorManager
                           |  `EditorManager.editorPaneCreated`      |      .getWorkingSet(paneId)
                           | and destroyed with the event:           | 
                           |  `EditorManager.editorPaneDestroyed`    | paneId is passed with Event Data
---------------------------+-----------------------------------------+-----------------------------------  
FindInFiles                | Search includes files opened that are   | Replace with 
                           | open in the working set but are not     |   DocumentManager
                           | part of the currently opened project    |       .getAllOpenDocuments()
---------------------------+-----------------------------------------+-----------------------------------  
```

# Working Set Context Menus
This currently works by listening to `contextmenu` events on the `#open_files_container`.  This will change to listen to `contextmenu` events on an `.open_files_container` and the `WorkingSetView` who attached to the `.open_files_container` will be maintain the paneId from the `EventData` it was passed during the create event so that callers (Extensions) will be able to determine which `EditorPane` has focus when the menu is invoked.

This will also trigger a focus action on the DOM node causing the `Editor` to gain focus.  The Default extension, `CloseOthers`, will be retooled to work on the working set for the editor pane associated with the `WorkingSetView` that manages it and ask that `EditorManager` instance for the working set.

# Working Set Management
The Implementation of these functions will move from `DocumentManager` to `EditorManager` and a deprecation warning will be displayed.

## EditorManager.findInWorkingSet
Used by (pflynn.brackets.editor.nav) which has a few other working set API calls.   
* Deprecation warning is displayed
* pflynn updates his extension.

** Changed to return {paneId: _paneId_, index: _index_) or undefined if not found **

## EditorManager.findInWorkingSetAddedOrder  
** DocumentManager.findInWorkingSetAddedOrder is NOT USED BY EXTENSIONS **  
No Deprecation warning. API goes away internal use is changed to `EditorManager.findInWOrkingSetAddedOrder`

```text
Core Usage (covers both findInWorkingSet and findInWOrkingSetAddedOrder)
---------------------------+-----------------------------------------+-----------------------------------
 Name                      | Usage                                   | Proposed Change                   
---------------------------+-----------------------------------------+-----------------------------------
SaveAs Command Handler     | Removes the document from the working   | Move to EditorCommandHandlers
                           | set and re-adds the saved as document   | findInWorkingSet Semantecs Change
                           | with the new name.                      | so that the paneId is returned
                           |                                         | instead of true/false then 
                           |                                         | removeFromW...Set / addToW...Set
                           |                                         | can be called with paneId 
---------------------------+-----------------------------------------+-----------------------------------
Close Others Extension     | uses the current document to figure     | will need to be re-worked to 
                           | out which document in the working set   | allow for images and other files
                           | the file was clicked on to dispatch     | by finding the item clicked on
                           | the close command selected              | and issuing the command on that 
                           |                                         | file's pane's working set can't 
                           |                                         | trust getCurrentDocument() 
---------------------------+-----------------------------------------+-----------------------------------
FileViewController         | Determines if the file is open in the   | changed to use EditorManager.
                           | working set or not.                     | findInWorkingSet() !== undefined
---------------------------+-----------------------------------------+-----------------------------------
WorkingSetView             | drag and drop to reorder items in the   | changed to use EditorManager.
                           | working set.                            | findInWorkingSet().index
---------------------------+-----------------------------------------+-----------------------------------
DragAndDrop                | used to determine if a file is open in  | changed to use EditorManager.
                           | the working set when dropping one or    | findInWorkingSet() !== undefined
                           | more files on Brackets                  | 
---------------------------+-----------------------------------------+-----------------------------------
WorkingSetSort             | "Sort by added" uses                    | changed to use EditorManager.
                           | findInWorkingSetAddedOrder to determine | findInWorkingSetAddedOrder().index
                           | sort order                              |
---------------------------+-----------------------------------------+-----------------------------------
```
## EditorManager.addToWorkingSet
** DocumentManger.addToWorkingSet will be Deprecated.  A deprecation warning is written to the console and the old API will call `EditorManager.addToWorkingSet(getFocusedPane(), path, ...)`

```text
3rd Party Extensions
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
alexsalsas.brackets-jekyll | Create a git hub page...                | OK to use deprecated API
                           | Adds "GemFile" to the workingset & sets | 
                           | the current document to <curProjectDir>/| 
                           | "GemFile"                               | 
---------------------------+-----------------------------------------+-----------------------------------  
brackets-reopener          | Keeps track of recently opened files    | Problematic at best with many
                           | Reopens the last one closed             | working sets.  reopened files 
                           | (many working set apis and events used) | would reopen in the current pane
                           |                                         | Probably OK to use deprecated APIs
---------------------------+-----------------------------------------+-----------------------------------  

Core Usage
---------------------------+-----------------------------------------+-----------------------------------
 Name                      | Usage                                   | Proposed Change                   
---------------------------+-----------------------------------------+-----------------------------------
Live Development           | Opens Initial Document                  | findInWorkingSet Semantecs Change
                           |                                         | so that the paneId is returned
                           |                                         | instead of true/false then 
                           |                                         | removeFromW...Set / addToW...Set
                           |                                         | can be called with paneId 
---------------------------+-----------------------------------------+-----------------------------------
DocumentManager            | sets the current document               | Replace with EditorManager
.setCurrentDocument        |                                         |  .addToWorkingSet({
                           |                                         |     paneId: FOCUSED_PANE, ...
                           |                                         |   }, doc.file)
---------------------------+-----------------------------------------+-----------------------------------
DocumentManager            | Adds the document to the working set    | same as above
 .on("_dirtyFlagChange",...|  if it was opened but not added to the  |                                                      
                           |   working set                           | 
---------------------------+-----------------------------------------+-----------------------------------


In addition to those two extensions, several other extensions make use of `FileViewController.addToWorkingSetAndSelect()`.  

3rd Party Extension usage of FileViewController.addToWorkingSetAndSelect()
---------------------------+-----------------------------------------+-----------------------------------
 Name                      | Usage                                   | Proposed Change                   
---------------------------+-----------------------------------------+-----------------------------------
brackets-Reopener          | Opens Document and Selects it           | No Changes
---------------------------+-----------------------------------------+-----------------------------------
brackets.swatcher          | Same 
---------------------------+-----------------------------------------+-----------------------------------
Zaggino.brackets-git       | Same
---------------------------+-----------------------------------------+-----------------------------------
jhatwich.                  |
    brackets-related-files | ** Extension doesn't work**
---------------------------+-----------------------------------------+-----------------------------------


Core usage of FileViewController.addToWorkingSetAndSelect()
---------------------------+-----------------------------------------+-----------------------------------
 Name                      | Usage                                   | Proposed Change                   
---------------------------+-----------------------------------------+-----------------------------------
ProjectManager             | Opens Document and Selectes it          | No Changes
---------------------------+-----------------------------------------+-----------------------------------
FindInFiles                | Same as above
---------------------------+-----------------------------------------+-----------------------------------


```

## EditorManager.addListToWorkingSet

```text
3rd Party Extensions
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
brackets-reopener          | Keeps track of recently opened files    | Problematic at best with many
                           | Reopens the last one closed             | working sets.  reopened files 
                           | (many working set apis and events used) | would reopen in the current pane
                           |                                         | Probably OK to use deprecated APIs
---------------------------+-----------------------------------------+-----------------------------------  

Core usage
---------------------------+-----------------------------------------+-----------------------------------
 Name                      | Usage                                   | Proposed Change                   
---------------------------+-----------------------------------------+-----------------------------------
DocumentManager.           | Called to present the file open dialog  | EditorManager 
  _doOpenWithOPtionalPath  | when the user opens more than 1 file    |   .addToWorkingSet
                           |                                         |       (getFocusedPane(), filelist)
---------------------------+-----------------------------------------+-----------------------------------

```

## EditorManager.removeFromWorkingSet

```text
3rd Party Extensions
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
Close-Others               | Adds Close commands the Working Set Menu| No change Needed. The extension is  
                           |                                         | packaged to run only on brackets  
                           |                                         | <= 0.32.0   
---------------------------+-----------------------------------------+----------------------------------- 

Core Usage
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
DocumentCommandHandler     | Removes the document from the working   | Will need to call EditorManager
    ._doSaveAs()           | set and readds it with the new name     |   .findDocumentInWorkingSet() 
						   | to determine which pane to reopen       | the new document in, etc...
---------------------------+-----------------------------------------+-----------------------------------  
DocumentCommandHandler     | Closes document on error during open    | EditorManager.removeFromWorkingSet
    .doOpen                |                                         | can be used with the paneId const
                           |                                         | ALL_PANES 
						   |                                         |   EditorManager
						   |                                         |     .removeFromWorkingSet(
						   |                                         |               ALL_PANES, _path_);
---------------------------+-----------------------------------------+----------------------------------- 

```

## EditorManager.removeListFromWorkingSet
Used by brackets-vim

```text
Core Usage
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                     
---------------------------+-----------------------------------------+-----------------------------------  
DocumentCommandHandlers.   | Implements:                             | EditorManager
   handleFileClose         | Commands.FILE_CLOSE_ALL,                |  .removeListFromWorkingSet(
						   | Commands.FILE_CLOSE_LIST                |     commandData.panedId || FOCUSED_PANE, 
						   |                                         |     fileList
						   |                                         | );
---------------------------+-----------------------------------------+----------------------------------- 

```


## EditorManager.swapWorkingSetIndexes
** NOT USED BY EXTENSIONS **

```text
Core Usage
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
WorkingSetView             | Swap indices of items during drag and   | No Deprecation warning. 
                           |    drop                                 |  calls its `EditorManager
                           |                                         |      .swapWorkingSetIndexes()`
                           |                                         | 
                           |                                         | 
---------------------------+-----------------------------------------+----------------------------------- 
_.partial can be used to bind the paneId to the context of the call for command handlers since this is a context menu.
```

## EditorManager.sortWorkingSet
** NOT USED BY EXTENSIONS **

```text
Core Usage
---------------------------+-----------------------------------------+-----------------------------------  
 Name                      | Usage                                   | Proposed Change                    
---------------------------+-----------------------------------------+-----------------------------------  
WorkingSetView             | Sort is one of the commands on the      | No Deprecation warning. 
                           |    Working Set Menu                     | calls its `EditorManager
                           |                                         |      .sortWorkingSet()`
                           |                                         | 
                           |                                         | 
---------------------------+-----------------------------------------+----------------------------------- 
_.partial can be used to bind the paneId to the context of the call for command handlers since this is a context menu.

```



# Supporting Images

Working set so far has be devoid of image files.  To support images in split view, we will need to change the working set rules to allow for images to be in the working set. This means that callers of the new working set APIs will need to check to make sure they can operate on a file by getting its file type from the language manager or checking the extension.  Also, File Command handlers implemented in `DocumentManager` will move to another layer to allow for files or documents to be targeted.  Mirrored commands (e.g. `Document.open` mirrors `File.open`) for just documents will be created to make the transition easier.

The working set will basically just be a list of files that may or may not have a Document object owned by the Document Manager.  The deprecated `DocumentManager.getWorkingSet()` will filter out any image files without a Document object.

# Events


```text
-----------------------------+-----------------------------------------+-----------------------------------  
 Name                        | Usage                                   | Proposed Change                    
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetSort               | Fires when the order of the working set | Moves from DocumentManager to 
                             | changes                                 | EditorManager.  Data about which                  
                             | ** NOT USED BY EXTENSIONS **            | working set is added to event data
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetAdd                | Fires when an item was added to the     | Moves from DocumentManager to 
                             | working set                             | EditorManager Data about which                  
                             | * TypeScript Extension to maintain      | working set is added to event data
                             |   integrity between its working set list|
                             |   and Brackets working set list         | The related files extension may
                             | * Reopener Extension (see above)        | be a problem but it appears to be
                             | * Related Files Extension (josh hatwich)| broken. There is at least 1 
                             |   appears he's using this event to know | deprecation warning and it looks 
                             |   when a file is opened in the editor   | like some callback data has 
                             |                                         | changed in the 8 months since it 
                             |                                         | was last touched. issue added.
                             |                                         | Related files works quite hevily
                             |                                         | with the working set apis, views
                             |                                         | and events which may stop working
                             |                                         | after changes are made to handle
                             |                                         | multiple working sets and views
                             |                                         | 
                             |                                         | The other two should be ok with
                             |                                         | changes to the event object
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetAddList            | List added to working set               | (same as above)
                             | Oddly, the reopener extension does not  | 
                             | subscribe to this event.                | 
                             | Related Files and Typescript extensions | 
                             | subscribe to this event                 | 
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetRemove             | Item removed from working set           | (same as above)
                             | Repopener, Typescript subscribe         | 
                             |  wdminimap - hides the minimap on event | 
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetRemoveList         | List removed from working set           | (same as above)
                             | ** NOT USED BY EXTENSIONS **            | 
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetSort               | List sorted                             | (same as above)
                             | ** NOT USED BY EXTENSIONS **            | 
-----------------------------+-----------------------------------------+-----------------------------------  
workingSetDisableAutoSorting | Sorting disabled due to swapping item   | (same as above)
                             | indices (manual reorder during drag)    |
                             | ** NOT USED BY EXTENSIONS **            | 
-----------------------------+-----------------------------------------+-----------------------------------  
```

# New Events

## EditorManager.editorPaneCreated

Sent when an editor pane is created. Event data about the pane is sent with the event.

## EditorManager.editorPaneDestroyed

Sent when an editor pane is destroyed. Event data about the pane is sent with the event.

## EditorManager.activePaneChanged

Sent when the focused pane changes

## EditorManager.activeEditorChanged
## EditorManager.fullEditorChanged

## EditorManager.currentlyViewedFileChange 
will not be deprecated but will now provide details about the the file, pane, document, etc... in the event data

Sent when the editor focus switches

## DocumentManager.openDocumentListChanged

Sent when the open document list has been modified.  Listeners should call `DocumentManager.getAllOpenDocuments()` and resync to the list of open documents.  

## DocumentManager.currentDocumentChange

Deprecated. Use EditorManager.fullEditorChanged instead. 

DocumentManager will listen for fullEditorChanged and dispatch a currentDocumentChange event to maintain backwards compatibility.  A deprecation warning will be written to the console when the event is fired.

```text
3rd Party Extension usage
-----------------------------+-----------------------------------------+---------------------------------------------
 Name                        | Usage                                   | Disposition
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-auto-pairs          | registers/reregisters key handlers      | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-code-overview       | Reloads the code overview editor        | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-flake8-0.0.1        | Hides flake8 if there is no dcocument   | Should work with deprecated event 
                             | or language manager support for the doc |
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-fontsize-sync       | Shows/Hides indent guide overlay if     | Should work with deprecated event 
                             |  enabled                                |
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-latex               | Shows/Hides latex toolbar based on if   | Should work with deprecated event 
                             | a text file is open in the editor       |
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-oncopy              | listens to copy and cut to dome events  | Should work with deprecated event 
                             | to show the number of chars copied in   |
                             | the statusbar                           |
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-pep8                | Same as flake8 except req.python files  | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-quick-markup        | Resets internal state                   | Extension handles codemirror events directly
                             |                                         | to format markup as you type. 
                             |                                         | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-robotframework      | Installs robot cm extension when doc is | Should work with deprecated event 
                             | a robot filetype                        |                                   
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-ruler               | Shows/Hides ruler if there is a document| Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-smooth-scroll       | Installs scroll event listeners when    | Should work with deprecated event 
                             | document changes                        | 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-various-improvements| Adds info to project header when        | Should work with deprecated event 
                             | document changes                        | 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-xunit               | runs xunit tests on document            | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
busykai.indent-right         | builds indentation information about    | Should work with deprecated event 
                             | current document                        |  
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-htmlescaper         | Escapes document                        | Should work with deprecated event 
                             |                                         |  
-----------------------------+-----------------------------------------+---------------------------------------------
cezarywojcik.charlimit       | Not sure                                | Should work with deprecated event 
                             |                                         |  
-----------------------------+-----------------------------------------+---------------------------------------------
edge-code-web-fonts          | enableds ewf toolbar when supported     | Redeploy?
                             | document is opened                      |  
-----------------------------+-----------------------------------------+---------------------------------------------
edge-inspect                 | resets skylab obj to open new document  | Redeploy?
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-FileTreeSync        | Syncs to the file in the file tree      | This will work as-is but only for documents
                             |                                         | Notify extension author to update extension
-----------------------------+-----------------------------------------+---------------------------------------------
MarkdownPreview              | shows md preview div on doc change      | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
jhatwich.brackets-related-   | **extension no longer works **          | 
files                        |                                         |   
-----------------------------+-----------------------------------------+---------------------------------------------
mikaeljorhult.brackets-todo  | syncs todo panel with current doc       | This will work as-is but only for documents
                             | by filename                             | Notify extension author to update extension
-----------------------------+-----------------------------------------+---------------------------------------------
pflynn.svg.preview           | toggles svg preview pane when an svg    | Should work with deprecated event 
                             | document is opened                      |
-----------------------------+-----------------------------------------+---------------------------------------------
pockata.striptrailingspaces  | removes trailing spaces from documents  | Should work with deprecated event 
                             | if they were changed                    |
-----------------------------+-----------------------------------------+---------------------------------------------
soimon.                      | outlines document if it is HTML         | Should work with deprecated event  
brackets-document-outliner   |                                         |
-----------------------------+-----------------------------------------+---------------------------------------------
Brackets-Select-Lines        |                                         | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
brackets-bookmarks           | Sync's bookmarks to current doc         | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
websiteduck.wdminimap        | Show/Hide wdMinimap                     | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------
zaggino.brackets-git         | Show/Hide wdMinimap                     | Should work with deprecated event 
-----------------------------+-----------------------------------------+---------------------------------------------

```

# Commands
Most "file.*" commands are handled by DocumentManager.  These will move to the EditorManager so that a more generic way of dealing with Images and Images in the Working Set can be operated on without the need for specialized testing.

```text
-----------------------------+-----------------------------------------+-----------------------------------  
 Name                        | Usage                                   | Proposed Change                    
-----------------------------+-----------------------------------------+-----------------------------------  
file.addToWorkingSet         | Used to open a document in the editor   | Moves from DocumentManager to 
                             |    FileViewControl.                     | EditorManager.  Data about which                  
                             |       addToWorkingSetAndSelect          | pane to open the document is added
                             |    (initial Sample Project Open)        | to the command data. 
                             |    QuickOpen                            | Currently focused pane is used by 
                             |    DragAndDrop                          | default if no 
-----------------------------+-----------------------------------------+-----------------------------------  
file.open					 | Opens a file                            | Moves from DocumentManager to 
							 |                                         | EditorManager.  A corresponding 
							 |                                         | Document.Open command is created
							 |                                         | to handle opening a document and
							 |                                         | all of the innerworkings of 
							 |                                         | DocumentManager.handleFileOpen.
							 |                                         | All image handling code in 
							 |                                         | DocumentManager.handleFileOpen 
							 |                                         | is handled by 
							 |                                         | EditorManager.handleFileOpen
-----------------------------+-----------------------------------------+-----------------------------------  
file.rename					 | Checks that the thing you are renaming  | Moves from DocumentManager to 
file.delete               	 | or deleteing is in an customViewer      | EditorManager. 
-----------------------------+-----------------------------------------+-----------------------------------  
file.close        			 | discreet file closing                   | Moves from DocumentManager to
file.closeAll                |                                         | EditorManager.
file.closeList  			 |                                         | 
-----------------------------+-----------------------------------------+-----------------------------------  
NextDoc         			 |                                         | Moves from DocumentManager to 
PrevDoc						 |                                         | EditorManager
-----------------------------+-----------------------------------------+-----------------------------------  
ShowInTree  				 |                                         | Opportunistic refactor -- move 
                             |                                         | to EditorCommands 
							 |                                         | and rename to File.showInTree
-----------------------------+-----------------------------------------+-----------------------------------  

```

# Additional Changes

## DocumentManager.getCurrentDocument()
Deprecated.  
* Callers will get the deprecation warning and the function will map to `EditorManager.getFocusedPane().getEditor().getDocument()`  
* Probably implemented more cleanly as `EditorManager.getFocusedEditor().getDocument()`
* Extensions will get the deprecation warning but they will continue to function.  



```text
Core Usage  
---------------------------+-----------------------------------------+-----------------------------------
 Name                      | Usage                                   | Proposed Change                   
---------------------------+-----------------------------------------+-----------------------------------
DocumentCommandHandlers    | Called when the document is dirtied     | DocumentManager will provide 
 .updateTitle              | or the name of the file has changed     | details about the document that
 .updateDocumentTitle      |                                         | changed in the event data for 
 .handleDirtyChange        |                                         |   "dirtyFlagChange"
                           |                                         |   "filenameChange"
						   |                                         | Handlers will use this data but 
						   |                                         | fallback to:
						   |                                         |  EditorManager
						   |                                         |   .getFocusedEditor()
						   |                                         |   .getDocument()
						   |                                         | Setting the title on the title bar
						   |                                         |  moves to EditorManager
---------------------------+-----------------------------------------+-----------------------------------
DocumentCommandHandlers    | handles opening & uses currentDocument  | Uses 
  .doOpen                  | to determine if the file actually       | EditorManager
                           | changed and aborts the open with a      |  .getFocusedEditor()
                           | successful promise resolve if the file  |  .getDocument()
                           | is the same as the currently viewed     |  
                           | file                                    |  
---------------------------+-----------------------------------------+-----------------------------------
DocumentCommandHandlers    | Uses current document if no data is     | Uses 
  .handleFileClose         | passed to the command                   | EditorManager
                           |                                         |  .getFocusedEditor()
                           |                                         |  .getDocument()
---------------------------+-----------------------------------------+- ----------------------------------
DocumentCommandHandlers    | Uses current document to determine if   | FILE commands really need to move
  .handleFileClose         | an image is open and calls              |  to EditorCommandHandler
DocumentCommandHandlers    | EditorManager._closeCustomViewer        | So it can determine how to proceed
  .handleFileCloseAll      |                                         |  and close imageViewers, etc...
DocumentCommandHandlers    |                                         | 
  .handleFileCloseAll      |                                         | 
---------------------------+-----------------------------------------+-----------------------------------
DocumentCommandHandlers    | uses current document if nothing is     | Same as above to allow handling 
  .handleFileRename        | selected in the file tree.              | of image files in the working set
                           |                                         | 
---------------------------+-----------------------------------------+-----------------------------------
ProjectManager             | finds the current document in the       | changed to use EditorManager
  .handleShowInTree        | project tree and selects it             |     .getFocusedEditor()
                           |                                         |     .getFilename()
---------------------------+-----------------------------------------+-----------------------------------
QuickOpenCSS extension     | builds the list of selectors from the   | changed to use EditorManager
                           | currently opened document               |     .getFocusedEditor()
                           |                                         |     .getDocument()
---------------------------+-----------------------------------------+-----------------------------------
QuickOpenHTML extension    | builds the list of ids from the         | changed to use EditorManager
                           | currently opened document               |     .getFocusedEditor()
                           |                                         |     .getDocument()
---------------------------+-----------------------------------------+-----------------------------------
QuickOpenJavaScript        | builds the list of functions from the   | changed to use EditorManager
 Extension                 | currently opened document               |     .getFocusedEditor()
                           |                                         |     .getDocument()
---------------------------+-----------------------------------------+-----------------------------------
CodeInspection             | builds a list of lint providers for     | changed to use EditorManager
 Extension                 | the currently opened file               |     .getFocusedEditor()
                           |                                         |     .getFilename()
---------------------------+-----------------------------------------+-----------------------------------
LiveDevelopment            | Implements its own _getCurrentDocument  | changed to use EditorManager
                           | which calls                             |     .getFocusedEditor()
                           |  DocumentManager.getCurrentDocument     |     .getDocument()
---------------------------+-----------------------------------------+-----------------------------------
FileViewController         | Tracks changes to the current document  | changed to use EditorManager
.on(                       |   to track where the file is            |     .getFocusedEditor()   
"currentlyViewedFileChange"|      WORKING_SET_view                   |     .getFilename()   
)                          |      PROJECT_MANAGER                    |     
---------------------------+-----------------------------------------+-----------------------------------
FileViewController         | opens the file if it's already          | changed to use EditorManager
 .openAndSelectDocument    |   the current document then it does     |     .getFocusedEditor()   
                           |      nothing. otherwise dispatches      |     .getFilename()   
                           |    "file.open"                          | Probably further optimization by 
						   |                                         |  checking if it's in a working set
						   |                                         |  and just opening the file    
---------------------------+-----------------------------------------+-----------------------------------
ProjectManager             | if there is nothing selected then uses  | changed to use EditorManager
 .getSelectedItem          | the current document and returns the    |    .getFocusedEditor()
                           | filename of the selected item.          |    .getFilename()   
---------------------------+-----------------------------------------+-----------------------------------
WorkingSetView             |                                         | changed to use this
._scrollSelectedDocIntoView| impementation details which may change  |    ._editor
._updateListSelection      | (see section below on WorkingSetView)   |    .getFilename()   
._createNewListItem        |                                         | Each workingSetView caches its 
						   |                                         | editor instance
---------------------------+-----------------------------------------+-----------------------------------
QuickOpen                  | Gets the language Id from the current   | changed to use EditorManager
 ._filterCallback          |  document so that it can favor plugins  |    .getFocusedEditor()
                           |  for that language                      |    .getDocument()   
---------------------------+-----------------------------------------+-----------------------------------
QuickOpen                  | Uses the current document's selection   | changed to use EditorManager
 .doDefinitionSearch       |  as search data.                        |    .getFocusedEditor()
                           |                                         |    .getDocument()
---------------------------+-----------------------------------------+-----------------------------------
ViewCommandHandlers        | Enables/Disables comamnds if a document | changed to use EditorManager
 ._updateUI                | is open or not.                         |    .getFocusedEditor()
                           |                                         |    .getDocument()
---------------------------+-----------------------------------------+-----------------------------------

```

## EditorManager.getCurrentlyViewedPath
Deprecated.  Callers will get the deprecation warning and the function maps to `EditorManager.getFocusedPane().getCurrentlyViewedPath()`

## EditorManager.createCustomViewerForFile
Replaces EditorManager._showCustomViewer to create a Read Only viewer for a file.

## EditorManager.getCurrentFullEditor
Reimplemented as EditorManager.getFocusedEditor().getCurrentFullEditor()


# Opportunistic Cleanup

The following `EditorManager` functions will move to `Editor`

```text
-----------------------------------+-----------------------------------------+-----------------------------------
 Name                              | Usage                                   | Disposition                  
-----------------------------------+-----------------------------------------+-----------------------------------
_openInlineWidget                  | Internal Inline Widget Management       | Moves to Editor without inpunity
-----------------------------------+-----------------------------------------+-----------------------------------
_toggleInlineWidget                | Command Handler                         | Current Implementation moves to 
                                   |                                         | Editor.
                                   |                                         | Command handler will  call
					        	   |                                         |   _focusedEditor
								   |                                         |    .toggleInlineWidget()
-----------------------------------+-----------------------------------------+-----------------------------------
_showCustomViewer                  | API                                     | Illegal usage from 
                                   |                                         |   DocumentCommandHandlers
								   |                                         | Command handler moves to 
								   |                                         |  EditorCommandHandlers
								   |                                         | Function is renamed to 
								   |                                         |  EditorManager
								   |                                         |    createViewerForFile
-----------------------------------+-----------------------------------------+-----------------------------------
closeInlineWidget                  |                                         | Moves to Editor
                                   |                                         | InlineWidget.close will call
								   |                                         | this.hostEditor.closeInlineWidget()
-----------------------------------+-----------------------------------------+-----------------------------------
getInlineEditors             	   |                                         | Moves to Editor
                             	   |                                         | InlineWidget
								   |                                         |    ._syncGutterWidths(
								   |                                         |                hostEditor
								   |                                         | ) 
								   |                                         | calls hostEditor.getInlineEditors()
-----------------------------------+-----------------------------------------+-----------------------------------
getFocusedInlineWidget             |                                         | Doesn't move but passes through to
                                   |                                         |    getFocusedEditor()
								   |                                         |      .getFocusedInlineWidget();
-----------------------------------+-----------------------------------------+-----------------------------------
```



# Implementing the Layout Manager

The initial implementation will be 2 columns x 1 row or 2 rows x 1 column.  However, implementing an arbitrary number of rows and columns could be trivial using this code:
https://github.com/FriendCode/codebox/blob/master/client/views/grid.js which is Apache-licensed.
However, this is a fairly integral piece to codebox and depends on other codebox libraries in order to work.

The initial implementation will be mostly handled by `EditorManager` but `EditorLayoutManager` will be created just to help handle the layout.  Since it's 1x1, 1x2 or 2x1 initially, it should be fairly easy to implement without the need for advanced layout mechanics.

# WorkingSetViews
 
WorkingSetView objects are created when the event `EditorPaneCreated` is handled.  `SideBarView` will handle this event and create a `WorkingSetView` object which is bound to the working set created for the pane and passed in as event data.

`#open-files-container` Is a container which contains one or more `.working-set-container` divs in the DOM. Several 3rd Party Extensions rely or use the `#open-files-container` div. The Extensions which Style The elements will continue to work. The following extensions may no longer work:

```text
3rd Party Extension usage
---------------------------+-----------------------------------------+---------------------------------------------
 Name                      | Usage                                   | Disposition
---------------------------+-----------------------------------------+---------------------------------------------
brackets-legibility        | Style Changes                           | Continues to work without issue
---------------------------+-----------------------------------------+---------------------------------------------
jhatwich.                  |
    brackets-related-files | ** Extension doesn't work**
---------------------------+-----------------------------------------+---------------------------------------------
brackets-tabs              | Converts the working set view into tabs | Extension will no longer work. 
                           |                                         | Contact Sean Davies directly:
						   |                                         |    seandavies@live.com
						   |                                         | A git repo is not provided in package.json 
---------------------------+-----------------------------------------+---------------------------------------------
Brackets-Themes            | Style Changes                           | Continues to work without issue
---------------------------+-----------------------------------------+---------------------------------------------
themesforbrackets          | Style Changes                           | Continues to work without issue
---------------------------+-----------------------------------------+---------------------------------------------
zaggino.brackets-git       | listens to open-files-container events  | Events on child divs should continue to 
                           |                                         | propogate down to parent node unless some
						   |                                         | event handler stops propogation so this
						   |                                         | extension *should* continue to work but
						   |                                         | file an issue in git repo to change this 
						   |                                         | to $(".working-set-container").on("...")
---------------------------+-----------------------------------------+---------------------------------------------
zaggino.brackets-git       | iterates over child li elements         | continues to work because he's using 
                           |                                         | $("#open-files-container").find("li")
						   |                                         | contact zaggino anyway, changes to working-
						   |                                         | set may not work correctly.
---------------------------+-----------------------------------------+---------------------------------------------
```

## Menus 

```javascript
DefaultMenus:
        $("#open-files-container").on("contextmenu", function (e) {
            working_set_cmenu.open(e);
        });

Becomes:
        $(".working-set-container").on("contextmenu", function (e) {
            e.workingSetPaneId = this._paneId;
            working_set_cmenu.open(e);
        });

```
When Working Set Views are constructed, information about the Pane they are associated with are passed to the constructor via event data.

# Performance Testing

Do early research on typing and scrolling performance.

# Tasking
Moved to [SplitView-Architecture-Tasking](SplitView-Architecture-Tasking)