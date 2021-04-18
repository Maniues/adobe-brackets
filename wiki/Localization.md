## TL;DR

### Using Localized Strings

The snippets below show how to use localized strings in core Brackets code.

In JavaScript
```javascript
var Strings = require("strings"); // load the Strings module
...
$("<span/>").text(Strings.CMD_ABOUT); // insert a localized string
```

In an HTML template
```html
<!-- templateContent.html -->
<span>{{CMD_ABOUT}}</span>
```
```javascript
/* JavaScript */
var Strings = require("strings"),
    templateContent = require("text!templateContent.html"); // load text content of template file

var html = Mustache.render(templateContent, Strings); // use Mustache to insert translated strings
```

### String concatenation
Because word order varies between languages, don't assume you can append separate strings together in a fixed order (e.g. using `+` in JavaScript). Instead, make a single _parameterized_ string:

1. Let's say you need to make a message that looks like "Current position: item I out of N."
2. Figure out which parts vary dynamically: "I" and "N" in this case.
3. Add parameterized string in strings.js: `CURRENT_POSITION: "Current position: item {0} out of {1}",`
4. Dynamically substitute values in JavaScript: `StringUtils.format(Strings.CURRENT_POSITION, currentIndex, items.length)`

TODO: explain how to do this in HTML/Mustache strings...

## Implementation Details

Brackets uses the [i18n plugin for RequireJS](https://github.com/requirejs/i18n) to load translations. The locale is determined by ``brackets.app.language`` (``navigator.language`` isn't used due to a CEF3 bug). Our main ``strings`` module re-exports the root bundle (i.e. ``require("i18n!nls/strings.js")``). Client code should only use the main strings module (i.e. ``require("strings")``).

As of Sprint 13, brackets-shell hardcodes strings for both English and French, primarily for the limited set of native menus. Once [native menus](https://trello.com/c/Zc2LP82u) are implemented, there will be no translated strings in brackets-shell.

### Creating or Updating/Improving Translations
[Creating new translations - README](https://github.com/adobe/brackets/blob/master/src/nls/README.md)

### Localizing Extensions
[How to Localize Your Extension - README](https://github.com/adobe/brackets/tree/master/src/extensions/samples/LocalizationExample/README.MD) (part of the [sample localized extension](https://github.com/adobe/brackets/tree/master/src/extensions/samples/LocalizationExample) included in the Brackets source)