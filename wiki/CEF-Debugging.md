## Version Numbers

CEF keeps several stable branches going in parallel. In a CEF version number like `3.1547.1459`, `1547` is the branch number and `1459` is the SVN revision number (which increments when _any_ branch is updated). These numbers are not connected to the Chromium version number.

#### What CEF version is Brackets using?
Look in [brackets-shell/Gruntfile.js](https://github.com/adobe/brackets-shell/blob/master/Gruntfile.js#L221-L224) and find the `"cef": { "version" : "...." }` declaration (near bottom).

#### What Chromium version is Brackets using?
* For a running copy of Brackets -- Type `navigator.userAgent` in Brackets dev tools. The section like `Chrome/39.0.2171.36` is the Chromium version.
* Otherwise -- Look up the CEF version number at http://cefbuilds.com/ (or https://code.google.com/p/chromiumembedded/wiki/BranchesAndBuilding)


## Comparison Testing

When issues appear with a new version of CEF, it can be hard to tell is the issue is due to CEF, Chromium, or some other factor. One way to find out is to run the same code in different environments:

#### Loading other webpages in brackets-shell

This lets you compare simple testcases (such as a JSFiddle, or a [CodeMirror demo page](codemirror.net/mode/javascript/)) in Chrome vs. brackets-shell.

1. In dev tools, run `window.location="..."`
2. Compare to the same URL running in Chrome

#### Running Brackets outside brackets-shell

This lets you compare _most_ of Brackets core in Chrome vs. brackets-shell.

1. Checkout the `pflynn/in-browser-file-system` branch
2. Point a simple local webserver (like [serf](https://www.npmjs.com/package/serf)) at the Brackets `src` folder.
3. First, verify that this still repros the issue in brackets-shell: run `window.location="localhost:8080/index.html"` in dev tools to load this in brackets-shell
4. Next, load `localhost:8080/src/index.html` in Chrome also

#### Testing with cefclient

"cefclient" is a bare-bones reference app that uses CEF. This lets you see if a bug is due to brackets-shell customizations vs. core CEF.

1. Go to http://cefbuilds.com/
2. Find the _exact_ version of CEF that matches the version Brackets is integrating (you may have to click the gray "More revisions" link)
3. Click the "Test App" link to download the matching cefclient build
4. Unzip and run (no installer needed). Now you can...
    * Compare a simple testcase in cefclient vs. Chrome, or cefclient vs. brackets-shell (similar to "Loading other webpages in brackets-shell" above)
    * Compare Brackets in-browser in cefclient vs. Chrome, or cefclient vs. brackets-shell (similar to "Running Brackets outside brackets-shell" above)

Note: your version of cefclient will rarely be an exact match for the current stable version of Chrome, so this _isn't a perfect test_ (there are two variables that differ: CEF vs. Chrome, and Chromium version slightly differs).

#### Testing in-browser with special flags

Brackets-shell doesn't configure CEF/Chromium engine exactly the same as a normal browser does. For example, GPU acceleration is disabled and on some OSes process security sandboxing is also disabled.

For testing, you can launch Chrome with flags to make it behave more like brackets-shell:

1. Create an empty folder somewhere on disk to act as a throwaway Chrome user profile
2. On the command prompt, run: <br>
    ```
    "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --user-data-dir=<<absolute path to empty folder>> <<add extra flags here>>
    ```
    <br> -or- <br>
    ```
    /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --user-data-dir=<<absolute path to empty folder>> <<add extra flags here>>
    ```
3. This will not interfere with your normal running copy of Chrome

Some useful options:

* Disable GPU acceleration: `--disable-gpu`
* Disable process sandboxing: `--no-sandbox`

Full list of options: http://peter.sh/experiments/chromium-command-line-switches/ (updated in sync with Chromium source)