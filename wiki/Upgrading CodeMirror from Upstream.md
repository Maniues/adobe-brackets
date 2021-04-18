## Merge Upstream CodeMirror to Master

### Preparation in Brackets repo 

1. From GitBash/Terminal window switch to master branch in your Brackets repo and make sure that you have the latest including all the submodules with the following git commands.
    * `git checkout master`
    * `git pull`
    * `git submodule update`

2. (Optional) Create a new branch for updating CodeMirror SHA. Usually, we directly update CodeMirror SHA in master branch without any code review. However, if you are not comfortable checking into master directly, then you can create a new branch and merge it via GitHub later by yourself.

### Update local CodeMirror and run unit tests
1. In GitBash/Terminal window switch to CodeMirror folder with `cd src/thirdparty/CodeMirror2`.
2. If this is the first time you are updating CodeMirror in your local repo, then you also need to define the upstream repo with `git remote add upstream https://github.com/codemirror/CodeMirror.git`.
3. Grab the latest from upstream CodeMirror with the following git commands.
    * `git checkout master`
    * `git fetch upstream`
    * `git merge upstream/master`
4. Launch Brackets and run all suites of tests. CodeMirror changes tend to break some features and our test suites can capture most of them in the past. However, even if you do not get any test failures, you can also run the smoke test or some scenario testing based on the upstream CodeMirror commit history.

### Merge to remote CodeMirror
If no issue or test failure in the above testing, then merge your local to remote with `git push origin master`. After this, our CodeMirror repo has the latest changes, but you still need to update the submodue SHA in Brackets repo.

### Update CodeMirror SHA in Brackets repo
1. In GitBash/Terminal window switch to Brackets repo with `cd ../../..`.
2. Merge SHA update with the following git commands.
    * `git status` (to verify that you have changes in CodeMirror submodue)
    * `git add src/thirdparty/CodeMirror2`
    * `git commit -m "your update message"`
    * `git push origin master` (or your new branch if you had created one.)
