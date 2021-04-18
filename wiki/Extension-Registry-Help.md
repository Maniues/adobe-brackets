Help for using the [Brackets Extension Registry](http://brackets-registry.aboutweb.com/)

## Writing an extension ##

See [How to Write Extensions](https://github.com/adobe/brackets/wiki/How-to-Write-Extensions).

## Publishing your extension in the Registry ##

1. Add a [package.json file](https://github.com/adobe/brackets/wiki/Extension-package-format#packagejson-format) next to your main.js
2. ZIP up your entire extension folder (the GitHub "Download ZIP" button is handy for this -- or use the command `git archive --format zip -o yourextension.zip master` to generate a zip file).
    * Note: we've had difficulty with ZIP files created from Finder on the Mac. If you get an error when uploading your ZIP file, try creating it from the command line instead.
3. Go to the [Brackets Extension Registry website](https://brackets-registry.aboutweb.com/)
4. Click "Sign in via GitHub." We never see your GitHub password and your extension itself doesn't need to be on GitHub (see below).
5. Drag and drop your zip file onto the big upload square, or click the square to browse to your zip file

## Updating your extension ##

Increase the version number in your package.json and then follow the exact same steps above to upload it again. The Registry will detect that you're updating an existing extension since the id (`"name"` in package.json) is the same. If you forget to increase the version number, the Registry will refuse the upload.

You must be signed in as the same GitHub user who originally uploaded the extension.

_Note:_ If the latest version of your extension isn't compatible with a user's current Brackets version, the Extension Manager in Brackets will still allow the user to download an older version of your extension. The user will see a message warning that they need to upgrade Brackets in order to install the latest version of the extension.

## Do I have to have my code on GitHub? ##

No. Right now, the Registry requires a GitHub account in order to identify you to the registry. You can use any source repository and zip utility to create the extension package file that you upload.

## Is the Registry open source? ##

Yes! It has [its own repository](https://github.com/adobe/brackets-registry) on GitHub.

## Removing an extension from the Registry ##

Visit the [Brackets Extension Registry](https://registry.brackets.io/) and sign in as the same GitHub user who originally uploaded the extension. Find the extension in the listing and click the Delete button.

## I need some other help! ##

If you have questions that are not answered here, feel free to ask on the [email list](https://groups.google.com/forum/#!forum/brackets-dev).