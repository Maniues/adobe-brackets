Brackets allows you to create editor themes that users can easily install and switch between. You can create a theme with simple CSS or LESS file and a package.json file with metadata for your theme.

Themes are simple Brackets extensions that can adjust the colors used in the editor area.

## Creating the Extension

* Open your extensions folder by selecting "Help > Show Extensions Folder" in Brackets
* Inside the `user` folder, create a new "yourThemeName" folder
* Inside the yourThemeName folder create package.json and theme.less files.

## package.json

The `package.json` file tells Brackets and, ultimately, the Extension Registry about your extension. Your theme's package.json should look something like this:

```javascript
{
    "name": "yourname.my-shiny-theme",
    "title": "My Shiny Theme",
    "description": "This theme is so shiny that you'll need to wear shades!",
    "homepage": "https://github.com/yourname/my-shiny-theme",
    "version": "1.0.0",
    "author": "Your Name <your@email> (http://your.url)",
    "license": "MIT",
    "theme": {
        "file": "theme.less",
        "dark": true,
        "addModeClass": true
    },
    "keywords": ["theme"]
}
```

This seems like a lot, but giving all of this information will give potential users of your theme the most information in the Extension Manager.

The "theme" part is unique to themes and is the trigger that tells Brackets that this extension is a theme. The `file` property tells Brackets which file in your extension is the stylesheet for your theme. If your theme has a dark appearance, use the `dark` flag to tell Brackets that it should make other elements beyond the editor dark as well. You can use the `addModeClass` flag if you want to have mode-specific classes added to your DOM (more about that below).

Especially important for themes: **include the homepage field** in package.json to point to your GitHub repository. Ideally, the README for your theme will display a screenshot to show what your theme looks like.

Read more about package.json on the [extension package format page](https://github.com/adobe/brackets/wiki/Extension-package-format#packagejson-format).

## Developing Your Theme

Brackets has a workflow that will help you iteratively create your theme. If you set up your theme files as suggested in the "Creating the Extension" section above, your new theme should be available for use right away.

* Start up Brackets
* Select your theme in the theme settings (`View > Themes...`) and choose Done
* Drag your theme.less file onto Brackets to open it

Now, when you save changes to your theme.less file, your changed theme is automatically applied to Brackets.

## Theme Styles

The base theme is the "Brackets Light" theme and all the styles you add will be added to the page after the base theme.

For an up-to-date example of a complete theme, take a look [at the Brackets Dark theme](https://github.com/adobe/brackets/blob/master/src/extensions/default/DarkTheme/main.less).

Your theme's stylesheet is processed as a LESS file that is wrapped with `#editor-holder {`. Setting the main foreground and background colors looks like this:

```css
.CodeMirror, .CodeMirror-scroll {
    background-color: #1d1f21;
    color: #ddd;
}
```

To apply your color scheme to the editor area, LESS will convert this to:

```css
#editor-holder .CodeMirror, #editor-holder .CodeMirror-scroll {
    background-color: @background;
    color: @foreground;
}
```

Beginning with Release 1.1, if you've the `addModeClass` flag set (see [package.json](#packagejson)), you can specify mode-aware styles like:

```css
.cm-m-css.cm-tag {
    color: #6c9ef8;
}
```

Common modes:
* `.cm-m-clike`: PHP
* `.cm-m-css`: CSS, LESS
* `.cm-m-javascript`: JavaScript
* `.cm-m-xml`: HTML, XML

Tips for creating your theme's CSS:

* Starting with an [existing theme](https://github.com/adobe/brackets/blob/master/src/extensions/default/DarkTheme/main.less) is always easier
* Use the Dev Tools element inspector (`Debug > Show Developer Tools`) to view the elements in the editor
* You can customize the styles within an inline code editor by adding `.inline-widget .CodeMirror` to your CSS selector. The inline code editor's background color should typically be slightly darker or lighter than your main editor background color, for contrast.
* Watch out for these colors that aren't displayed all the time:
    * matching brackets - `.CodeMirror-matchingbracket`
    * matching tags in HTML - `.CodeMirror-matchingtag`
    * matches for the "Find" command
        * `.CodeMirror-searching` (all matches), `.CodeMirror-searching.searching-current-match` (current match)
    * matches for the ["highlightMatches"](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#preferences) auto word highlighting - `.cm-matchhighlight`
    * the highlight for the active line (`View > Highlight Active Line`) - `.CodeMirror-focused .CodeMirror-activeline-background` and `.CodeMirror-focused .CodeMirror-activeline .CodeMirror-gutter-elt` (line numbers shown) or `.show-line-padding .CodeMirror-focused .CodeMirror-activeline-background` (line numbers hidden).
    * the Quick View hover highlight - `.quick-view-highlight`
    * cursor in overwrite mode - `.CodeMirror-overwrite .CodeMirror-cursor`
    * code folding gutter triangles - `.CodeMirror-foldgutter-open:after`, `.CodeMirror-foldgutter-folded:after`, `.CodeMirror.over-gutter .CodeMirror-foldgutter-open:after`, `.CodeMirror-activeline .CodeMirror-foldgutter-open:after`
    * code folding collapsed-text placeholders - `.CodeMirror-foldmarker` (border, fill, and fg icon/text color)
* Don't set the editor font to a web font (`@font-face`) - this will cause cursor positioning glitches. Note that users can also override the font set by your extension via the View > Themes dialog.

## What about other UI elements?

Brackets and other editors separate the notion of "editor themes" from that of "UI themes". Editor themes provide colors for the text editor area based on the syntax highlighting engine of the editor. UI themes can style the whole rest of the UI.

Editor themes have certain advantages. They are:

* simpler to create because they don't have very many things to style
* easier to test because you don't need to look through the whole UI to see the impact of changes
* less likely to break between Brackets releases, because the Brackets DOM can change at any time

Brackets currently *only* supports editor themes.

Inline editors other than inline code editors (for example, the inline color editor), are considered "UI" and should not be styled by editor themes.

## Publishing Your Theme

Congratulations! You've created a new theme. Now you can share it with the world:

* zip your theme's folder into a new zip file
* upload the zip file to the [extension registry](https://brackets-registry.aboutweb.com/)