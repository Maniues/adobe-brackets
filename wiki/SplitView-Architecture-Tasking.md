# *NOTE:* This is now obsolete.  See the [trello board](https://trello.com/c/2DWV5tEX/1277-splitview-migrate-workingset-management-to-mainviewmanager) for story breakdown and tasking

Here's a proposed guide to breakup the story into smaller arch pieces:

1. Opportunistic Cleanup
   * _openInlineWidget
   * _toggleInlineWidget
   * _showCustomViewer
   * closeInlineWidget
   * getInlineEditors
   * getFocusedInlineWidget

2. Migrate WorkingSet Management to `EditorManager`
   * Code new WorkingSet APIs in EditorManager
   * Move serialization code from DocumentManager
   * add deprecation warnings to old APIs
   * add bridge to new API from old APIs
   * Write issues for extensions still using old APIs
   * Migrate core and default extension code to new WorkingSet API
   * Notify extension authors
   * Update Unit tests

3. Refactor`File.commands` from `DocumentCommandHandlers` into `FileCommandHandlers` 
   (depends on #2)
   * Move FILE.xxx and call EditorManager functions 

4. Deprecate getWorkingSet 
   (depends on #2)
   * Migrate all core instances (7)
   * Migrate all core extension instances
   * Notify extension authors
   * Add deprecation warning
   * Unit tests

5. Migrate DocumentManager.getCurrentDocument 
   (depends on #2)
   * Notify all extension authors 

6. Implement EditorManager code for 1x2 editors
   * implement new Editor and Editor Manager APIs
   * Erect any temporary scaffolding UI to build feature
   * implement EditorLayoutManager
   * implement PaneManagement
   * implement WorkingSetView create
   * implement dynamic editor create
   * implement dynamic WorkingSetView create
   * working set context and gear menus
   * update serialization code
   * close others extension
   * test git extension (notfy zaggino)
   * Unit tests

7. Add support for images in working set
   * implement EditorManager.createCustomViewerForURI
   * implement view factory 
   * create custom viewer for images
   * implement all file based commands to work on non-document based objects
   * Implement all working set commands for non-document based objects
   * getDisplayName() to show in the working set 
   * Unit tests

8. Implement UI for split view
   (depends on 6)
   * implement core changes to direct splits, etc...
