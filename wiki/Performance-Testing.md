Typing responsiveness
---------------------
**Summary of test results**

* Brackets responds to keystrokes about half as fast as a typical text editor (169 ms vs. avg 83 ms)
* Brackets responds about 50% slower than a bare instance of CodeMirror
* Our native app shell (CEF) does not slow down typing compared to running in the browser
* File size does not seem to have a strong effect on typing speed _(preliminary data)_
* Having the JSLint panel open slows down typing by 30% (presumably also true of the Find in Files dialog). Unclear whether this is due to layout, dropshadow rendering, or something else.
* Laying out a bare instance of CodeMirror via flex-box slows down typing by ~18%. The slowdown effect _may_ be stronger in the full Brackets UI.
* Brackets processes key events very quickly when a key is held down (key repeat). Unclear why this doesn't translate into low latency in normal typing too.

**Test methodology**

Typing performance in Brackets is bottlenecked mainly by native browser layout and rendering, not by JavaScript code execution speed. This makes performance hard to measure: it's easy to write JS code that automatically times its own execution speed, but JS is too decoupled from the browser's layout/rendering to accurately record how long that phase takes. Our best attempt at monitoring keystroke responsiveness via JS code reported a response time more than 2.5x faster than reality.

So, believe it or not... the way we've gotten more accurate data is by recording the computer screen (and keyboard) with a high-speed camera. This is painstaking, but the results are dramatically more reliable.

<a name="proposals"></a>**Optimization proposals**

[Alex](https://github.com/chicu123) has proposed three changes to improve typing & scrolling performance:
* [#1007](https://github.com/adobe/brackets/pull/1007): Hoist the CodeMirror instances out of the layout. Place them near the root of the document and programmatically move/size them as appropriate.
    * _Test results:_ Scrolling framerate is roughly doubled -- basically pegged at 60 Hz now. Typing response time is cut by 1/3, putting Brackets roughly on par with the average text editor.
    * _Risks/downsides:_ The editor reacts to horizontal resizing less smoothly than before.  Noticeable mainly by looking at the vertical scrollbar, or moreso with the inline editor rule list if one is open.  Unusual DOM placement makes CSS selectors and event bubbling a bit counterintuitive.  The patch also has a couple bugs that are presumably fixable (breaks project panel resizing; breaks quick open popup; hides the "[ ]" background).
    * _Next steps:_
        * How much of the gain is from getting CodeMirror closer to the document root, vs. getting CodeMirror out of a flex-box container? We should try leaving the CodeMirror editor in its current place in the DOM, but making its parents not use flex-box. Probably not too hard, since we already have to programmatically kick CodeMirror on vertical resizes -- and it avoids the horizontal resize problem noted above.
        * Try the newer implementation of flex-box that's accessible in Chrome(ium) via a special flag. It may be better optimized than the current impl.
* [adobe/CodeMirror2#60](https://github.com/adobe/CodeMirror2/pull/60): (a) remove the 20ms timeout that delays CodeMirror's key event processing; and (b) optimize the line-rendering code in CodeMirror by batching string to HTML parsing, reducing the frequency of DOM element creation, etc.
    * _Test results:_ Scrolling framerate is unchanged (slow scrolling) or slightly worse (fast scrolling: 10-15% lower framerate). Typing response time is 10-30 ms faster.
    * _Risks/downsides:_ The timeout's purpose is unclear; it's been in the CodeMirror source for a very long time, although in early incarnations it seemed to get short-circuited for most key events.  And of course, reducing scrolling performance isn't good.
    * _Next steps:_
        * How much of the gain/loss is from the rendering change vs. elimination of the timeout?  My guess is the timeout change is the more valuable part.  We should ping Marijn to understand more about its purpose.  Is it even needed on modern browsers if they fire "textinput" and/or "keypress" reliably?


Opening files
-------------
**Summary of test results**
* Average-sized files open in about 1/3 sec
* Large files open in about 1 sec
* Files with very long lines (e.g. minified JS) take dramatically longer to load -- about 7x longer than the unminified version of the same code, in one test
* Brackets opens files a bit slower than most text editors, but there's a lot of variation in how fast other editors are. In particular, some editors are relatively slow on small files while some become very slow on large files (in some cases taking up to 4x longer than Brackets).

Unlike keystroke handling, the speed of opening files is affected by a mixture of JS code execution and native layout/rendering. In limited tests with a high-speed camera, the results recorded via JS code were much closer to the camera's numbers: off by 25-30% for small files, and off by 10% for larger files. The summary above is based entirely on measurements recorded via JS.