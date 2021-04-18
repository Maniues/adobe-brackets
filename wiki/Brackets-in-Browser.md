Although Brackets is packaged as a desktop application, it's built almost entirely in HTML/CSS/JS - and most of that code will run in a browser with no modifications.  The functionality provided by Brackets's native desktop shell (brackets-shell) needs to be replaced or disabled, however.

Some basic parts of this work have been done by the core team in the **[pflynn/in-browser-file-system branch](https://github.com/adobe/brackets/tree/pflynn/in-browser-file-system)** ([diff](https://github.com/adobe/brackets/compare/master...pflynn/in-browser-file-system)). However, **the core Brackets project does not provide a complete turnkey solution for running in browser.**


## Pieces You Must Provide

Some aspects of running Brackets in a browser are very use-case dependent, so each implementor must provide their own code:

- **Server** - We do not provide code for serving up the Brackets app files. Any web server should do, though.
- **Backend storage** - You must implement your own backend storage layer to store the text the user is editing. Provide a [File System Implementation](https://github.com/adobe/brackets/wiki/File-System-Implementations) module that talks to this backend on Bracket's behalf, allowing Brackets to save and load files (among other operations). The impl acts as a bridge between the Brackets core code and your backend.
- **Authentication** - Presumably, your users must log in to determine which set of files in the backend storage they have access to. Brackets does not provide any code for authentication & authorization.
- **Deploying builds** - [Use Grunt to generate a minified build of Brackets](https://github.com/adobe/brackets/wiki/Building-Brackets-Releases). How you deploy to your webserver is up to you, though.


## Missing Pieces

Major Brackets features that do not work in-browser yet:

* **Live Preview** - Live Preview currently requires connecting to the Chrome Remote Debugging API, which is only accessible when Chrome is launched with a special command-line argument. [We hope to avoid this requirement in the future](https://github.com/adobe/brackets/wiki/Live-Development:-lifecycle-research-and-future-directions#live-development-managed-with-unprivileged-scripts). Live Preview also needs to run a small web server to serve up instrumented copies of the user's files; doing this from a remote server is likely trickier than the simple local Node server that Brackets uses on the desktop.
* **Extensions** - Implementing user-installable extensions in-browser requires several considerations:
    * Security - Extensions are loaded in the same domain as Brackets, so they can take any action on your website with your user's auth credentials.
    - Storage - `ExtensonLoader` must be modified to load extensions from your server instead of fixed file:// URLs. To allow each user to install different extensions, your server would need to store a separate extensions folder for each user.
    - Install mechanics - Brackets relies on Node-side code (part of brackets-shell) to unzip & validate extension packages. Your server would need to run this code or implement the same functionality in your preferred server-side language.
    - Desktop-only extensions - Some extensions depend on Node (or other APIs only available in brackets-shell), and thus wouldn't work in browser.
* **Loading feedback** - Loading Brackets, or opening a large file, can take much longer than usual over a slow network connection. Brackets doesn't give any feedback and may contain race condition bugs during such unanticipated delays.
* **Preferences** - In 0.36, preferences storage moved from browser `localStorage` to local-disk JSON files. This is not easily pluggable yet, but your File System implementation can redirect reads/writes on this path to your preferred backend preferences storage mechanism.
* **Quick View, Image preview** - These features expect image files to exist at a URL that can be read directly by the browser via an `<img src>` link. Currently, the URL is assumed to be identical to the FileSystem path; these features probably need to support adding a base URL prefix so the images can be properly addressed on the server (and your server will need to expose the images at such a URL for download).
* **JavaScript code hints** - Enabled, but it may cause performance issues since it aggressively reads many files.


Features that are unlikely to _ever_ work in-browser:

- **Menu bar** - Brackets automatically displays an HTML menu bar instead.
- **Certain keyboard shortcuts** - Web pages cannot use Ctrl+W, Ctrl+T, Ctrl+N, Ctrl+Tab (or any of those +Shift). The shortcuts are still displayed in the menu bar, but pressing them does invokes the standard browser command instead.
- **Edit > Cut/Copy/Paste** - On most browsers, it's not possible to invoke clipboard operations programmatically, so these menu items are hidden.
- **Check for Updates** - Since there's no way to install updates.

Everything else not listed here _does_ work: Find/Replace in Files, Quick Edit, Quick Open, Quick View, multiple cursors, etc.


## Cross-Browser Compatibility

* Chrome: should work in Chrome 29+
* Internet Explorer: 10+
* Firefox: ?


## Existing projects

* [Mozilla Nimble](https://github.com/humphd/brackets) - actively under development within the [Mozilla WebMaker](https://webmaker.org/) project. Read more on their [blog](https://blog.webmaker.org/webmaker-experiments-with-brackets) or [wiki](https://wiki.mozilla.org/Webmaker/Concept-Nimble).
* [Treehouse Workspaces](http://teamtreehouse.com/workspaces) - online web development environment for Treehouse students ([screenshot](http://d.pr/i/hnDL))


## Chrome App / Chrome OS

Getting Brackets running in-brower has a lot of overlap with porting Brackets to a Chrome Packaged App or Chrome OS app. Some existing efforts:

* [Quickfire](https://chrome.google.com/webstore/detail/quickfire/mobpfffdclcandcgkkjgjkcalglekegd) - can read/write local files; supports Live Preview with additional Chrome extension; last updated March 2014; ([announcement](https://groups.google.com/forum/#!topic/brackets-dev/5AzyP7eKHTI))
* [Tailor](https://chrome.google.com/webstore/detail/tailor/mfakmogheanjhlgjhpijkhdjegllgenf) - includes experimental Git syncing; last updated July 2013; ([announcement](https://groups.google.com/forum/#!topic/brackets-dev/3LUTWip5ZPk))
* [Work in progress by Maksim Lin](https://github.com/maks/brackets/tree/chrome-app) - last updated July 2013