This page is mainly about modifying core Brackets code. If you're adding a new feature, consider **[writing an extension](How to write extensions)** instead.

If you're interested in **submitting a pull request**, review the [guidelines for contributing code](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#contributing-code). Most importantly:

1. Discuss any major changes or questions beforehand in the [brackets-dev newsgroup](http://groups.google.com/group/brackets-dev).
2. Follow the [Pull Request Checklist](https://github.com/adobe/brackets/wiki/Pull-Request-Checklist) to ensure a good-quality pull request.
3. Sign the [Brackets Contributor License Agreement (CLA)](http://dev.brackets.io/brackets-contributor-license-agreement.html) — we can't merge your code otherwise. You only need to do this once.


## Quick Start ##

### Requirements ###

* Latest [Brackets installer build](http://download.brackets.io) (or if you'd rather build the native bits yourself instead, [see below](#nativeshell)).
* Git command line tools — follow the setup instructions [on GitHub](https://help.github.com/articles/set-up-git) or download [here](http://git-scm.com/downloads)
* NodeJS installed (https://nodejs.org/en/download/current/)
* PHP 7
* [Composer 1.8.4](https://getcomposer.org/download/) (Make sure that it is added to the path, and the `composer` command works)

### Setting up your dev environment ###

1. Install the latest Brackets build (this gives you the native shell binaries which you'll use in step 6)
2. Fork the [brackets repo](https://github.com/adobe/brackets)
3. Clone your fork of the repo: `git clone https://github.com/<username>/brackets.git`
4. Fetch submodules: `cd brackets` and `git submodule update --init`
5. Install `npm` dependencies: `npm install`
6. Add an "upstream" remote: `git remote add upstream https://github.com/adobe/brackets.git`
7. Run `setup_for_hacking` script:

    |  |  |
    |---|---|
    | Mac | `tools/setup_for_hacking.sh "/Applications/Brackets.app"` |
    | Windows | `tools\setup_for_hacking.bat "C:\Program Files (x86)\Brackets"` <br>_(MUST be run in a Command Prompt started with "Run as Administrator". If you opened a new Command Prompt you might need to navigate back to the repository folder you cloned in step 3 with eg. `cd C:/path/to/brackets`)._ |
    | Linux | `sudo tools/setup_for_hacking.sh "/opt/brackets"` |

Now, when you launch the Brackets app it will load your git copy of the source code, rather than the original source code that was distributed with the installer. It will share the same preferences & set of installed extensions that a regular installed build would use.

**Making changes** is simple:

1. Edit the source code and save your changes (you can even do this _with_ Brackets)
2. Choose _Debug > Reload With Extensions_ to reload Brackets and pick up the changes. _No build step is needed._

> For more in-depth instructions see ["Getting a Copy of the Code" below](#wiki-getcode).

> For hacking on the native code or building the app binaries yourself without running an installer, [see "Hacking on brackets-shell" below](#nativeshell).

### Learning the Brackets code ###

* **API docs:** http://brackets.io/docs/current (also inline in the code as JSDoc comments)
* [Use **Dev Tools** to debug Brackets](https://github.com/adobe/brackets/wiki/Debugging-Brackets)
* [Core APIs & architecture overview](https://github.com/adobe/brackets/wiki/Brackets-Development-How-Tos)
* [How to use common APIs](https://github.com/adobe/brackets/wiki/How-to-write-extensions#common-how-tos)

### Submitting a pull request ###

1. Make sure your local copy of source is up to date: `git fetch upstream && git merge upstream/master`
2. Make sure submodules are up to date: `git submodule sync` and `git submodule update --init`
3. Create a feature/bugfix branch: `git checkout -b <branchname>`
4. Hack on Brackets!
5. Commit your changes: `git commit -am "Your commit message here"`
6. Run unit tests
7. Push changes to your fork of the repo: `git push origin <branchname>`
8. Make sure you've reviewed the [Pull Request Checklist](https://github.com/adobe/brackets/wiki/Pull-Request-Checklist) and signed the [Contributor Licence Agreement (CLA)](http://dev.brackets.io/brackets-contributor-license-agreement.html). (See also overall [guidelines for contributing code](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#contributing-code)).
9. Submit pull request on GitHub, including a descriptive title and links to any relevant bugs you're fixing.


## Getting Started ##

### How Brackets is Organized ###
Brackets is primarily built in HTML/JS/CSS, but it currently runs as a desktop
app inside a thin native app shell called [brackets-shell](https://github.com/adobe/brackets-shell),
based on [CEF](http://code.google.com/p/chromiumembedded/),
that lets it access local files. (It doesn't run in the browser yet, but we're hoping to work on an in-browser version soon.)

The HTML/JS/CSS files are installed along with brackets-shell, but you can set up a separate development copy of these files and then load them in brackets-shell instead of the default installed copy. See [Running Your Copy of the Code](#setup_for_hacking) for more details.

For details on working with Brackets's architecture and APIs, see [[Brackets Development How-Tos]].

### What should I hack on? ###
Whatever you want, of course! Check out the [CONTRIBUTING guide](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#where-do-i-start) for some ideas.

If you're planning to do something other than a small bugfix, please start a discussion on the [brackets-dev Google group](http://groups.google.com/group/brackets-dev) or the [#brackets IRC channel on freenode](http://webchat.freenode.net/?channels=brackets) to get feedback. There might already be some prior thinking on what you're working on, or some reason that it hasn't already been done. We don't want you to do tons of work and then have to rewrite half of it.

### What's the process? ###
First, [sign the Brackets Contributor License Agreement (CLA)](http://dev.brackets.io/brackets-contributor-license-agreement.html). This is for your protection as well as that of the Brackets project.

Then, just submit changes as pull requests from your own fork of brackets or
brackets-shell. The core dev team works in 2.5-week sprints (weird length,
but it works for us). We'll try to review small pull requests quickly
in the current sprint. Larger submissions may take longer — but discussing them
in advance will smooth the process!

Before you submit your pull request, please make sure it's merged with master and fully tested as described in the [Pull Request Checklist](https://github.com/adobe/brackets/wiki/Pull-Request-Checklist).

[Read more on how pull requests are reviewed...](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#the-code-review-process)


### <a name="getcode"></a> Getting a Copy of the Code ###
**Note:** Don't use the GitHub "ZIP" to get a copy of the source. The auto-generated ZIP file will be missing important dependencies.

**New to git?** If all this git stuff seems scary, check out [GitHub's git tutorial](http://try.github.io) or the [Pro Git book](http://git-scm.com/book).

The first step is to fork the repo on GitHub so you can start making changes in your local repository. For the HTML/JS/CSS code that comprises the bulk of Brackets, you only need to fork the [brackets repo](https://github.com/adobe/brackets). (See ["Hacking on brackets-shell" below](#nativeshell) if you want to work on the native code as well). To fork a repo, simply click the "Fork" button at the top of the page.

Next pull the repositories down to your local machine. 

```bash
git clone https://github.com/<your username>/brackets.git
cd brackets
git submodule update --init
```

_Don't skip the last line!_ Brackets uses submodules for third-party dependencies (like [CodeMirror](http://codemirror.net/)), so it won't work until you run this command to set them up.

> You may also need to run `git submodule update` when you switch branches or pull from upstream, since submodule changes aren't updated automatically. If you see third-party code showing up as modified in `git status` (something like `M	src/thirdparty/CodeMirror2`), then you need to run this command.


### <a name="setup_for_hacking"></a> Running Your Copy of the Code ###

If you're only hacking on HTML/JS/CSS files, you can have the installed Brackets shell run your local copy of the HTML/JS/CSS code (instead of the default installed copy) by running the `tools/setup_for_hacking` script. Here's how:

1. Download and install the [latest Brackets sprint build](http://download.brackets.io).
2. Follow ["Getting a Copy of the Code" above](#getcode) to fork & clone the brackets repo.
3. On a Mac: 
  1. Open a Terminal window
  2. `cd` to the root of your brackets repo
  3. run `tools/setup_for_hacking.sh`, passing the full pathname to your installed Brackets.app. For example:
  ```bash
  tools/setup_for_hacking.sh "/Applications/Brackets.app"
  ```
4. On Windows:
  1. Open a Command Prompt _using "Run as Administrator"_
  2. `cd` to the root of your brackets repo
  3. run `tools\setup_for_hacking.bat`, passing the full path of the directory where Brackets.exe is installed. For example:
  ```bat
  tools\setup_for_hacking.bat "C:\Program Files (x86)\Brackets"
  ```
5. Launch the installed copy of Brackets, select _Help > About_, and verify that the version number says "sprint xx _development_ build" instead of "sprint xx _experimental_ build". This indicates that you're running Brackets from your git repo instead of the installed build.

You can revert back to running the installed version of the Brackets source at any time by running `tools/restore_installed_build.sh` (Mac) or `tools\restore_installed_build.bat` (Windows) from your Brackets repo.

Once you are set up, Brackets will pick up the latest changes to the code every time it starts — _no build step is needed._ (The Grunt scripts are used only to [generate a final release build](https://github.com/adobe/brackets/wiki/Building-Brackets-Releases)).


### Getting Updates from the Main Repository ###
It's important to keep up to date with the main Brackets repository to make sure your pull requests match the latest Brackets builds. First you'll first need to link your local clone of Brackets to track the main Brackets repository on GitHub. Run this command from the _brackets_ directory:

```bash
cd brackets
git remote add upstream https://github.com/adobe/brackets.git
```

Your repo will now have two "remotes": `origin` refers to your fork on GitHub, while `upstream` refers to the original, official Brackets repo.

> If you want to avoid getting branches other than master, you can add the `--track master` argument after `add`. However, that will mean that if you need to pull a different branch, you'll need to explicitly fetch it. 

#### Getting the latest changes

Getting the latest changes on the Brackets `master` branch is a two-step process. First:

```
git fetch upstream
```

This brings down any new commits into your repo, but doesn't actually update any of your branches. Next, update your current local branch:

```
git merge upstream/master
```

This merges any changes from the main Brackets repository into whichever branch you currently have checked out locally. (To update multiple local branches, you'll need to `git checkout` each one in turn and repeat the merge command).

You may also need to run `git submodule update` at this point — if the output of `git fetch` said "Fetching submodule" or if `git status` shows an unexpected diff in a third-party library.

Rarely, an entire _new_ submodule is added to Brackets. You'll need to run `git submodule update --init` when that happens.

#### What changes did I get?

You can see a diff before merging with `git difftool ...upstream/master`.

For a higher-level overview (with API important changes called out), check the [[Release notes]] after each sprint.


### Brackets-shell Updates ###

_If_ you also forked the brackets-shell repository, repeat the steps above for the brackets-shell repo, then rebuild the shell once you've pulled down updates.

If you did _not_ fork the brackets-shell repo (i.e. you're using the 'setup_for_hacking' step above), you should update your installed build of Brackets each sprint and rerun the 'setup_for_hacking' script. This ensures you're using the latest app binaries.



## Contributing Code ##

### Useful Tools for Development ###
If you use Brackets to edit Brackets, you can quickly reload the app itself by choosing *Debug > Reload Brackets* from the in-app menu. You can also [debug Brackets using dev tools](Debugging Brackets) — but to keep Reload working properly while dev tools are open, be sure to **disable caching** as instructed in that link.

You can use *Debug > Run Tests* to [run our unit test suite](Running Brackets Unit Tests).

### Saving Your Code Changes ###
If you've found an issue you want to fix or a feature you want to implement, eventually you'll want to submit a _pull request_ back to Brackets upstream. Here's how to organize your changes so they're ready to turn into a pull request.

First, create a new branch off of `master` for the change you want to work on. This allows your `master` branch to stay in sync with the main Brackets repository — and if your change doesn't work or breaks something, you can always start fresh from your local `master` again.

```
git checkout master
git branch mynewfeature
git checkout mynewfeature
```

That creates a new branch called `mynewfeature` and sets it as your working branch. Any changes you make now will be linked to that branch. While you work on it, you should still keep your branch in sync with Brackets master following the "Getting Updates from the Main Repository" steps above. 

Now go ahead and modify some code, make your fix, and be sure that it works in your copy of Brackets. As always, please follow the [guidelines for contributing code](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#contributing-code) to ensure your code matches what's expected for Brackets contributions.

Next it's time to _commit_ your changes to your local git repo. Use `git add <filename>` to stage any modified files for committing, then use `git commit -m "COMMIT MESSAGE"` to commit those files. (Or just use use `git commit -am "COMMIT MESSAGE"` to commit all modified files). Be sure to write a thorough commit message that describes the changes you're making and why. 

The last step before submitting a pull request is to _push_ those changes to your GitHub account — so far the changes are only stored in your local copy of the Brackets repository. Remember that your GitHub fork of the Brackets repo is called "origin"... so to push your changes use:

```
git push origin mynewfeature
```

That command creates a matching branch on your GitHub-hosted repo and copies your branch's commits into it. Now your commits are stored on the GitHub server and not just locally.

This is a good time to review the **[[Pull Request Checklist]]** — make sure your code passes JSLint, doesn't break any unit tests, etc.
 

### Submitting a Pull Request ###
Now you're ready to submit a pull request. Go to the GitHub page for your fork of Brackets and choose your branch from the dropdown near the upper left (it says `master` by default). Now you're looking at the code for that branch. 

Click the Pull Request button in the top right and you'll be brought to a page that describes the pull request. Make sure you're submitting _to_ the `adobe/brackets` repository, _from_ the branch you've been working on. 

Review the "Commits" and "Files Changed" tabs to make sure you're submitting only the changes you intend. Write a detailed description of what your pull request does, and include links to any relevant bugs ("#1234" is automatically turned into a link — though avoid writing "fixes #1234", since GitHub will auto-close the bug too soon if it sees a string like that). Then click "Send pull request."

Congratulations on submitting your pull request — you're helping make Brackets even better! Read about [the pull request review process](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#the-code-review-process) for what happens next.

#### Updating an Existing Pull Request
During code review, you will probably be asked to make some changes to your code. First, make sure you're working on the correct branch:

```
git checkout mynewfeature
```

Then make the necessary code changes. When done, repeat the `git commit` and `git push` steps from above.

_Ta daa!_ As soon as you've pushed those changes to "origin" (your fork on GitHub), your pull request is automatically updated with the new code. Finally, add a comment to the pull request describing the new changes — this ensures the code reviewer is notified of your update.


## <a name="nativeshell"></a> Hacking on brackets-shell ##
For most of your changes to Brackets, you should only need to edit the HTML/JS/CSS code in the brackets repo. You won't need the source for brackets-shell because you can use a pre-built binary from a Brackets installer to run your modified HTML/JS/CSS code ([see instructions above](#setup_for_hacking)).

However, you may want to set up a dev environment for brackets-shell if:
* You want to hack on the native code.
* You want to pull the latest from the brackets repo `master` and it requires a new brackets-shell build that hasn't been released as a binary yet. (This is uncommon, but it can happen).

1. Similar to the steps above, fork the [brackets-shell repo](https://github.com/adobe/brackets-shell) and `git clone` it.
2. Follow the [build instructions on the brackets-shell wiki](https://github.com/adobe/brackets-shell/wiki/Building-Brackets-Shell).