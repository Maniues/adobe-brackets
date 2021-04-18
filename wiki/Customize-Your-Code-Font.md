# Brackets 0.42 and newer

To change the code editor font, choose _**View > Themes...**_, then change the "Font Family" setting.

* Use the same syntax as you would `font-family` in CSS
* The font must be installed on your OS (there's no way to load a web font)
* One exception: `'SourceCodePro-Medium'`, the default code editor font, is a web font that's loaded automatically by Brackets (installing Brackets doesn't install any fonts globally on your system)

Changing your font can be useful as a workaround for certain international text-display issues, such as [lack of Russian/Cyrillic support](https://github.com/adobe/brackets/issues/3465).

# Older Brackets & Adobe Edge Code

Here's a quick hack you can use to change your font in earlier releases:

### Approach A: Write an extension

[Write a Brackets extension](https://github.com/adobe/brackets/wiki/How-to-write-extensions) using this code as the basis for your _main.js_:

```
define(function (require, exports, module) {
    "use strict";

    var ExtensionUtils = brackets.getModule("utils/ExtensionUtils");

    ExtensionUtils.addEmbeddedStyleSheet(".CodeMirror { font-family: 'Comic Sans MS'; }");
});
```

...except pick a less horrible font! :-)

> Note: There's no need to write a package.json file since this extension is just for yourself; package.json is only needed if you want to publish the extension for others to use. 

If you store this extension in the location given by Help > Show Extensions Folder, then you can freely upgrade to newer Brackets versions and your extension will remain in place.

_Caveat: This approach only works for fonts installed on your OS._ If you want to use a web font (loaded via `@font-face`), you need to edit the Brackets core source code.


### Approach B: Edit Brackets source

1. [Set up a Brackets dev environment](https://github.com/adobe/brackets/wiki/How-to-Hack-on-Brackets#setting-up-your-dev-environment) (you can omit the fork step and clone the official repo directly, however)
2. In the cloned source, edit the file <i>src/styles/brackets_theme_default.less</i>
3. Add your `@font-face` import directive if needed 
4. Find the `.code-font()` rule and change the `font-family` as desired
5. Restart Brackets

_Caveats:_ In order to keep up to date with Brackets updates, you'll need to to manually pull down source code updates (this can be tricky if you're not familiar with Git). Brackets loads slightly slower in a dev environment because the source isn't minified.