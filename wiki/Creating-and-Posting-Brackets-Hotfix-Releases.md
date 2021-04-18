This document outlines the steps needed to create a hotfix branch and release for Brackets Sprint (XX).

Please note that the release's version number can consist of a major and minor component.  For example in "34.1", "34" is the major and "1" is the minor version.  The Windows Wix installer will upgrade a previous install in-place, so long as the version (major and minor) to install is greater than the version already installed (and so long as the `UpgradeCode` in 'brackets-shell/installer/win/Brackets.wxs' matches).

### Setup the hotfix branch

1. create a new hotfix branch from major branch in **both** brackets and brackets-shell
```
git checkout sprint-XX
git checkout -b sprint-XX-hotfix
```

2. increment the sprint minor version (Y) in **both** brackets and brackets-shell
```
grunt set-sprint --sprint=XX.Y
```
If you choose to increment the version manually (ie. not using the `grunt` task, please be sure to increment the `product.sprint.number` value defined in `installer/win/brackets-win-install-build.xml`.  Otherwise, in-place upgrades on Windows won't install correctly.

3. commit and push your branches to both GitHub repos.

### Test the hotfix builds

Test installing the resulting hotfix builds on Mac, Win, and Linux platforms.  Since Sprint 34 and later releases install in-place, be sure to check that upgrades also install correctly.  Specifically, be sure to test installing the hotfix builds on systems which:
  * do not have Brackets Sprint 34 or later installed
  * do have the last major Brackets Sprint release installed, in particular, the one with the same major version as the hotfix.

To confirm that the installation worked correctly, confirm that:
  * the installer completes successfully
  * there is just one Brackets application shortcut (ie. that any previously installed, Sprint 34 or later shortcuts were overridden)
  * Brackets launches successfully
  * (Windows-only) there is just one entry to uninstall Brackets in Control Panel > Programs and Features list
  * (Windows-only) the uninstaller completely removes the installed Brackets binaries

### Post the hotfix builds to Brackets.io

As exemplified with the Brackets Sprint 34.1 hotfix release, please post the hotfix builds to download.brackets.io as follows:

1. any hotfix build will replace the previously posted build of the same major version number.
1. the hotfix build installers will be posted with the same installer filenames as the previously posted major release.  To do so, rename the .dmg and .msi installer filenames to remove the .Y suffix.
1. don't delete the previous releases' Release Notes, but just add the hotfix release notes to the list
