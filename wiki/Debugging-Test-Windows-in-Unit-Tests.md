> See [[Running Brackets Unit Tests]] for how to _run_ unit tests.

When creating Unit Tests, it's often helpful to create a new Brackets window. The `createTestWindowAndRun()` and `closeTestWindow()` functions in SpecRunnerUtils make this easy to do.

One problem with testing in this way is that you can't step into the code in the debugger because it's in a separate window which is created and closed too quickly to be able to manually open the Dev Tools. Luckily, there are several ways to do this:

## Option A: Quick One-Off

1. Set a breakpoint at the start of the unit test code (after the window would be opened)
2. Run the test
3. While the test is paused on the breakpoint, open a new browser tab and go to http://localhost:9234. Click the link corresponding to the popup test window -- this opens a _second_ instance of dev tools just for the test window.
4. Set breakpoints in the popup window's code as desired
5. Resume execution in the test-runner window -- the test will continue & hit your breakpoint(s) (in the popup window's dev tools).

## Option B: Repeatable Setup

1. At some point after the test window has been created, add this temporary line of code: <br>`testWindow.brackets.app.showDeveloperTools();` <br>(I put it in the callback passed to `SpecRunnerUtils.createTestWindowAndRun()`, but it can go anywhere).

2. Restart Brackets (because sometimes Reload doesn't reload everything...)

3. Open SpecRunner: Debug > Run Tests

4. On Mac, if you're debugging a Live Development unit test, do the following to open Chrome in debug mode: shutdown Chrome, start Live Dev, open an empty tab in Chrome, and stop Live dev.

5. From SpecRunner: Show Developer Tools

6. Set breakpoint(s) in unit test code as desired.

7. Leave SpecRunner Dev Tools Window open. I wasn't hitting my breakpoint when I closed it.

8. Run your unit test

You should hit your breakpoint(s) and be able to step into code. Note that there are 2 Dev Tools windows open: 1 for the SpecRunner (Jasmine) window, and 1 for the Brackets test window. When you step or run from code in the SpecRunner Dev Tools window to a code or a breakpoint in the Brackets test Dev Tools window, there is no indication in the SpecRunner Dev Tools Window and you are not automatically switched to the other window, so you'll need to manually change to the other Dev Tools window.

Opening a Dev Tools window for every Brackets test window slows down tests, so you may want to disable all tests except the one you are debugging.

If this is helpful, we could build it in to the `createTestWindowAndRun()` function and control it by a parameter or global flag. Let us know!