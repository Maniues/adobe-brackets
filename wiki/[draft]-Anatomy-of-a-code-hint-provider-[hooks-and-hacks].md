# Code hint provider in Brackets
## How to write a code hint providers for Brackets?
> IntelliSense is the heart of any modern code editor and code hints are integral part of this. Brackets, being a modern code editor has a robust hinting framework which can be extended very easily by writing hint plugins. To write a hint plugin, there are few points which needs to be answered first - 
* Which language/mode is being catered using this plugin?
* How to populate the hint list?
* When to show the hint list?
* How to show the hint list?
* What information can be shown as part of the hint list?

| Name | Description |
| ---- | ----------- |
| `hasHints*` | `CodeHintManager` uses this mandatory hook to query whether this particular hint provider can provide hint in the current context.|
| `getHints*` | `CodeHintManager` uses this mandatory hook to fetch hint list in the current context if this provider is selected to be the code hint provider for the current hinting session.|
| `insertHint*` | `CodeHintManager` uses this mandatory hook to let the active hint provider handle the selected token insertion.|