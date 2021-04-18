You'll need to install the build you want to test (e.g. Release 0.44 beta)

for the full instructions see [Running tests on Linux](https://github.com/adobe/brackets-shell/wiki/Running-tests-on-Linux)
```text
cd ~/brackets
git checkout fetch
git checkout release
grunt setup
grunt test-integration --suite=integration --shell=/opt/brackets/brackets
mv results.xml integration-results.xml
grunt test-integration --suite=unit --shell=/opt/brackets/brackets
mv results.xml unit-results.xml
grunt test-integration --suite=extension --shell=/opt/brackets/brackets
mv results.xml extension-results.xml
```

You'll want to report the results if there are failures so it may be worth-while to save them.  Just take note of any failures and report them.  You don't need to file an issue but send a note to the dev-forum on google