This set of commands creates the Brackets API Docs to be published at http://brackets.io/docs. So far, the only set of docs are for the [current version](http://brackets.io/docs/current) of Brackets, but it would probably be useful to provide snapshots of docs for major releases (e.g. Brackets 1.0).

### apify

Brackets API Documents are generated using [apify](https://github.com/jbalsas/apify/) which is an open source project was written by Brackets committer [Chema Balsas](https://github.com/jbalsas). The content is based on [JSDoc Annotations](https://code.google.com/p/jsdoc-toolkit/wiki/TagReference) in Brackets source code which follow [Google Closure Compiler](https://developers.google.com/closure/compiler/docs/js-for-compiler).

### Use Mac

__Note:__ There is a [known bug](https://github.com/jbalsas/apify/issues/3) that the incorrect folder structure and links are generated on Windows, so use Mac to generate documents until that bug is fixed.

### Install node apify package

The following commands assume that the **brackets** and **brackets.io** root folders are side-by-side.

This command installs the node **apify** package globally so you can run it from any folder. Note that **apify** is still being developed, so it's a good idea to re-run this command each time to get the latest package.

```
cd ~
sudo npm install -g jsdocify
```

### Create branch in brackets.io repo

Brackets API documents are posted to the https://github.com/adobe/brackets.io repo, so clone that repo (first time only):

```
git clone https://github.com/adobe/brackets.io.git
cd brackets.io
```

Create a new branch off of latest `gh-pages` branch (default branch in brackets.io).

```
git checkout gh-pages
git pull
git status
git checkout -b yourname/new-branch-name
```

### Checkout Brackets branch

Checkout the Brackets branch that you want to generate API Docs for which is usually `release` branch. Currently, we only generate docs for the current master, but once we reach version 1.0, we'll want to generate docs for each major and minor release.

```
cd ../brackets
git checkout release
git pull
git status
```

### Generate Docs

The source folder (-s) for API Docs is the `brackets/src` folder. The following command assumes that you are in the root of the **brackets** repo.

The output folder (-o) is `brackets.io/docs/current`. 

The title (-t) is `Brackets`.

```
apify -s src/ -o ../brackets.io/docs/current -t Brackets
```

### Submit Pull Request

The API Docs should now be in the **brackets.io** repo in your new branch, so you'll need to submit a pull request and then merge files.