The following make use of working sets so these should be tested with the working set refactoring code

* git integration (ziggino.brackets-git)
* WD Minimap (websiteduck.wdminimap)
* Copy File Info to clipboard (vmalone.fileinfo-toclipboard)
* Extension Highlighter (tjeffree.extensionhighlighter) --> Fails.  The event repub mechanism subscribes to the 'paneViewList` events and repubs them as 'workingSet' events but the event listener is added before extensions are loaded. This used to work because extensions are loaded later and their event listeners are added *after* brackets' internal event listeners are added. The workingsetview listened to changes to the workingset and updated the view accordingly.  The extension highlighter extension listens for this event and reconfigures the view.  Unfortunately the pub/sub arch doesn't guarantee load order. The solution was to register deprecated events in appInit.ready rather than at DocumentManager module load.  Also the workingsetsort event was not one of the events that we repub'd so that was fixed as well.
* recognizer
* Editor Nav (pflynn.brackets.editor.nav)
* ivogabe.icons
* typescript for brackets (fdecampredon.brackets-typescript)
* csscomb (csscomb-brackets)
* outliner (crabcode.outline)
* swatcher (brackets.swatcher)
* xunit (brackets.xunit)
* typescript Quick Edit (brackets-typescript-code-intel) -- this extension does not work with recent codemirror update (https://github.com/tomsdev/brackets-typescript-code-intel/issues/11) 
* reopener (brackets-reopener)
* jekyll (alexsalas.brackets-jekyll) -- unable to get this extension to work correctly.
* vim (brackets-vim)   
=== all extensions work with the 2 exceptions noted above that could not be tested ===

The following extensions listen for "editorAreaResize" which has moved and will be deprecated.

* BFxOS for Brackets (buildingfirefoxos.bfxos-brackets)  
* DevDocs Viewer  (gruehle.dev-docs-viewer)  
* Markdown Preview (gruehle.markdown-preview)  
* AsciiDoc Preview (nerk.asciidoc-preview)  
=== the extensions were not tested and the event was not forwarded so they will fail in the SplitView-Rebased branch ===

TODO: Which extensions should be tested when set current document has been deprecated   
TODO: Which extensions should be tested when Editor Manager is refactored  
TODO: Which extensions should be tested when we have multiple panes / working set views   
TODO: Which extensions should be tested when working sets have things that are not documents  