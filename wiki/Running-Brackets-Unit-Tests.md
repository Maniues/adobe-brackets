## Tips and Tricks for Running Brackets Unit Tests

Unit Tests can only be run from a cloned version of Brackets -- not from an installer build.

Use `Debug > Run Tests` to invoke the Jasmine SpecRunner dialog which looks something like this:

![Jasmine SpecRunner dialog](http://i.imgur.com/ZzozdSA.png)

You can try to run "All" tests from the "All" tab, but due to memory constraints,
usually all of the tests don't pass. Here's a more reliable recipe:

1. Switch to "Unit" tab and click "All" to run all tests in this category.
2. Switch to "Integration" tab and run "All".
3. Close SpecRunner dialog, shutdown/restart Brackets, `Debug > Run Tests`.
4. Switch to "Live Preview" tab and run "All".
5. Switch to "Main View" tab and run "All". (added since screenshot was taken)
6. Switch to "Performance" tab and run "All".
7. Switch to "Extensions" tab and run "All".

### If tests fail
If any tests fail, try running only that suite of tests alone (e.g. "CSS Parsing").
If all tests for that suite pass when run alone, the problem is likely to be a bug in the unit test setup instead of a real bug in Brackets.

Sometimes tests fail and cleanup code does not get executed so temporary files are left behind which causes subsequent test to fail. Run `git status` to verify that there are no unexpected files in the test folder. If so, manually delete the files and re-run the tests that failed.

If any tests consistently fail, open an Issue with High Priority.

## Debugging test failures

* For non-integration tests, click "Show Developer Tools"
* For integration tests (tests that pop up a new window), see [[Debugging Test Windows in Unit Tests]]