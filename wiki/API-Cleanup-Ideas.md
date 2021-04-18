Notes for **[Extensibility API simplification](https://trello.com/c/iSG75qG4/995-research-extensibility-api-simplification)** research story.

## Commands, Menus, Key bindings

* Use `add()` method name for all three main APIs. Simplify Menu method names to remove unneeded "menuItem" part.
* Expose menu hierarchy directly, e.g. `Menus.viewMenu` as opposed to `Menus.getMenu(Menus.AppMenuBar.VIEW_MENU)`.
* Expose set of commands directly, e.g. `Commands.fileOpen` as opposed to `CommandManager.getCommand(Commands.FILE_OPEN)`

## Documents & Editors

* Allow adding listeners with a direct `.on()` as opposed to `$(...).on()`
* Proxy objects ("CurrentDocument," "ActiveEditor," etc.) that you can attach listeners to once, instead of writing boilerplate code to attach & detach listeners as the current/active Document/Editor changes.
* Events for idle document-change processing (either a debounced delay after typing pauses, or on open/save/refresh as linting works today). Useful for Find in Files results updating, real-time linting, possibly some live development functionality, etc.

## ProjectManager & file structure

## UI: toolbars, icons, panels, etc.

* Generic UI injection-point API -- more robust than current approach of using CSS selectors & jQuery, which easily breaks if we refactor our DOM hierarchy.

## Inline editors

* Inline editor class hierarchy may be confusing
* Inline widget constructor vs. `load()` method confusion (they are always called synchronously back to back right now? -- but the provider is expected to call both itself, so this must be reimplemented by everyone)

## Language extensibility

(See also [Language Extensibility umbrella user story](https://trello.com/c/D2L7SAq5/639-language-extensibility-apis-for-extensions-to-add-new-language-syntax-coloring-mode-rich-editing-support))

## Quick Open search

* QuickOpen plugin format uses confusing method names: `search()` vs. `match()`; `itemFocus()` is not actually related to focus, etc.

## Find, Replace, Find in Files