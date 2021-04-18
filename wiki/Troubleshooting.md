## ⚠️ On September 1, 2021, Adobe will end support for Brackets. If you would like to continue using, maintaining, and improving Brackets, you may fork the project on GitHub. Through Adobe’s partnership with Microsoft, we encourage users to migrate to Visual Studio Code, Microsoft’s free code editor built on open source.


Brackets not working for you? Maybe the following will help:

* [Having trouble installing Brackets?](#installation)
* [Brackets not launching?](#launching)
* [Having trouble with Live Preview?](#livedev)
    * [Live Editing](#live-edit)
    * [Page Not Loading](#livedev-not-loading)
    * [Using a Local Server](#local-server)
    * [Chrome Issues](#livedev-chrome)
    * [Experimental MultiBrowser implementation](#livedev-multibrowser)
    * [Other Live Preview Issues](#livedev-other)
* [Is Brackets running slowly?](#slow)
* [Having trouble installing extensions?](#extension-install)
* [Other: Brackets Is Acting Weird](#other)

Other information that may help you:

* [What are Brackets' system requirements?](#requirements)
* [How do I associate an extension with a type of file?](#fileassoc)

[Question not yet answered?](#notsolved)

## <a name="requirements"> </a>System Requirements

* Mac OSX 10.11, 10.12, 10.13, 10.14 
* Windows 7 with Service Pack 1, Windows 8.1, or Windows 10 (installer requires administrator access)
* Linux Ubuntu 16.04, 18.04, 18.10 (x32 and x64)
* Linux Debian 8, 9
* At least 2 GB of RAM for Live Development

## <a name="installation"> </a>Can't Install Brackets

### Windows Vista: Nothing happens when launching installer

Some Windows Vista computers will block installers downloaded from the Internet, so nothing at all happens when you try to run the installer. To work around this: right-click the installer file, choose Properties, and click the Unblock button.

### Windows error: "Installation directory must be on a local drive"

This can happen on some Windows machines. To work around this, try executing the installer from an elevated command prompt:

1. Open an elevated command prompt using one of the techniques on this page: http://www.sevenforums.com/tutorials/783-elevated-command-prompt.html
2. `cd` to the folder containing the installer.
3. Run the installer using msiexec, e.g.: `msiexec /i "brackets-sprint-xx-WIN.msi"` (where "xx" is the sprint number)

### Windows: Installer stuck on 0% progress

There may be a long delay at the start of the installation process as Windows checks, prepares, and displays the UAC prompt.  During this delay, you will see the "Installing Brackets" - "Please wait while Brackets is installed" page of the installer but with 0% progress.  The length of this delay may vary, depending on your individual system.


## <a name="launching"> </a>Can't Launch Brackets

### <a name="clear-cache"> </a>Clear The Cache
If you had previously used Brackets, your cache may have information that is conflicting with the most recent version. [Find your cache folder](https://github.com/adobe/brackets/wiki/Cache-Folder) and delete the cache. _Warning: this will reset all of your Brackets preferences._

### Check the File Permissions

If Brackets won't launch, check the permissions of the main executable files (e.g. using `ls -l`). On Mac:

* `bin/mac/Brackets.app` should be `drwxr-xr-x`
* `bin/mac/Brackets.app/Contents/MacOS/Brackets` should be `-rwxr-xr-x`

To fix permissions, use a command like `chmod +x bin/mac/Brackets.app/Contents/MacOS/Brackets`.

### Run Brackets From The Command Line
Next, try running Brackets from the command line. Open up a Terminal (or Command Prompt in Windows), navigate to the executable, and run Brackets. (On Mac, type `open bin/mac/Brackets.app`.). Did an error appear? If so, file an issue or find us on IRC or the mailing list and we'll try to figure it out.

### Linux: libudev.so.0 Error
This may happen when trying to run 32-bit Brackets on 64-bit Linux. Please verify that you've downloaded the correct build.

### Linux: GLIBC_2.14 Error
This occurs when trying to run Brackets on Debian 7 (Wheezy). Brackets currently requires Debian 8 (Jessie). (But for some _potential_ workarounds, [see issue #4816](https://github.com/adobe/brackets/issues/4816)).


## <a name="livedev"> </a>Live Preview Isn't Working

### <a name="live-edit"> </a>Live Editing

#### CSS, HTML, and JavaScript
Currently, Live Development works differently for different types of files:

* For CSS, all changes are applied in the browser immediately as you type, without reloading the page.
    * CSS preprocessors are not currently supported - they are treated as "other file types" below
* For HTML, _most_ changes are applied in the browser immediately as you type. **Updating pauses** when the page is syntactically invalid (e.g. after you type '<' for a new tag but before you type the closing '>'). The line number and Live Preview icon turn red, and the tooltip says "_not updating due to syntax error_". Brackets will resume pushing changes to the browser when syntax becomes valid again.
   * HTML live updating is disabled if you've specified a custom server URL in Project Settings.
* For JavaScript and other _external_ file types, when you save your changes, the page is reloaded to reflect your changes. For _embedded_ JS, you will need to reload browser.

See **[How to Use Brackets](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#wiki-live-preview)** for more details.

#### Files should be in Current Project
You can use `File > Open` to open any file on your computer, but Brackets' definition of a _project_ are the files in the folder opened using `File > Open Folder...`. Some (but not all) Live Development features require a node server, which means being in the current project, so make sure the files that you want to use with Live Development are in the current project.

#### HTML File should be in Working Set

There is a [known issue](https://github.com/adobe/brackets/issues/7886) (which is fixed in release 0.43) that if HTML file is in project tree (i.e. not in Working Set), then element highlighting stops working after switching to a CSS (or other?) file and then back to the HTML file. The workaround is to double-click HTML file so it's added to the Working Set.

#### Live CSS is not working

Updating CSS in Live Preview does not seem to work if &lt;link&gt; has __type="text/css"__, so try removing it.

Known issues:

* [Bug #7935](https://github.com/adobe/brackets/issues/7935): Live CSS does not update if page contains an iframe (including injected iframes such as ads or social media buttons).

#### HTML Page is not Updating in Browser as you Type
If you are using your own local server, HTML will not update live ([see documentation](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#wiki-live-preview)).

Brackets pauses sending updates to browser when it detects an HTML Syntax Error. In this case, it should color the line number in red (but this can be scrolled out of view) so scroll through entire page to verify that there are no highlighted line numbers.

The Live Preview lightning bolt icon should be also colored red and have a tooltip of "Live Preview (not updating due to syntax error)" in this case, but there's a known bug being tracked as [issue #7126](https://github.com/adobe/brackets/issues/7126) where this sometimes doesn't happen. See [issue #7126](https://github.com/adobe/brackets/issues/7126) for an illustrated description including the line number and icon coloring.

Other known issues:

* [Bug #5338](https://github.com/adobe/brackets/issues/5338): Live Preview can never update if initially launched with syntax error - after fixing a syntax error, try stopping and restarting Live Preview.

### <a name="livedev-not-loading"> </a>Live Preview Page not loading

* If you specified a "Base URL," make sure your local server is already running - Brackets will not start it for you.
* Make sure you are not running firewall, network security, or antivirus software that is blocking the connection (try disabling it temporarily to check)
* Make sure you haven't modified your [hosts file](https://en.wikipedia.org/wiki/Hosts_(file)) to remap localhost or 127.0.0.1
* Try shutting down background apps in Chrome. In Chrome, go to Settings > Advanced Settings and then uncheck the "Continue running background apps when Google Chrome is closed" setting.

### <a name="local-server"> </a>Using a Local Server with Live Preview
To use a local server, you need to specify a Base URL in the `File > Project Settings...` dialog (see [How to Use Brackets](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#wiki-live-preview) for details).

If you're using a local server and are seeing these messages such as "Oops! Google Chrome could not connect to localhost:[port]" (in Chrome) or "Unable to load Live Development page" (in Brackets), verify that your local server has been started.

As noted above, live HTML updating is disabled when using your own custom local server.

### <a name="livedev-chrome"> </a>Live Preview Chrome Issues

#### Stable Chrome

Brackets is verified with the current stable Chrome. If chrome is not configured to automatically update to the latest version, then be sure to check for updates. It usually works with current beta, dev, or canary versions of Chrome, but if you are having a problem switch back to stable Chrome before opening an issue.

#### Install Chrome For Multiple User Accounts
##### Windows Only
If you get the error ``An error occurred when launching the browser. (error 2)`` when doing Live Development, installing [Chrome for multiple user accounts](http://support.google.com/chrome/bin/answer.py?hl=en&answer=118663) may solve the issue.
 
#### Check Windows Registry
If Brackets cannot launch the Chrome browser on your Windows system, check the Registry setting here:

* HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\chrome.exe

This is the file path that Brackets uses to launch Chrome. If this is not correct, then try reinstalling the Chrome browser at this location.

#### Uninstall/Reinstall Chrome

When uninstalling Google Chrome on Windows, some users have reported problems with Registry settings that are not removed. If uninstalling and reinstalling Chrome still does not fix problem, try deleting Registry settings in the following locations before re-installling:

* HKEY_CURRENT_USER\SOFTWARE\Google\Chrome\
* HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\

**Warning:** Editing the Windows Registry can easily cause problems with your system, so edit it manually at your own risk. Another option is to use a tool such as Revo Uninstaller to uninstall Chrome.

#### Keep Chrome Open
Try leaving an extra blank tab open in the instance of Chrome that is launched by Live Preview. This prevents Chrome from shutting down and restarting between each file, so Live Preview will launch faster; this may reduce some intermittent errors.

#### Restart Your Computer
If you keep getting errors when trying to launch Chrome, or if you keep getting prompted to restart Chrome, try rebooting your machine. Rebooting has resolved many odd issues with Live Development.

#### Re-install Brackets

On Windows, you may run into issues starting Live Preview if you installed Chrome after installing Brackets. In that case, re-installing Brackets should fix the problem.

### <a name="livedev-multibrowser"> </a>Issues with multi-browser experimental implementation
This section describes known issues and work-arounds specific to [Multi-Browser Live Preview](https://github.com/adobe/brackets/wiki/Live-Preview-Multibrowser).

#### Multi-browser Live Preview does not work with Internet Explorer 11
In order to make it work with Internet Explorer 11, disable all the options that IE uses to include sites in the local intranet (unchecked all the items at `Internet Options > Security > Local Intranet > Sites`). This will allow Live Preview to establish connection to the editor.

### <a name="livedev-other"> </a>Other Live Preview issues

#### Disable Extensions
Use [`Debug > Reload Without Extensions`](#wiki-disable-all-extensions) to quickly see if the problem is being caused by an extension.

The Theseus[\[1\]](https://github.com/adobe-research/theseus/issues/37)[\[2\]](https://github.com/adobe-research/theseus/issues/46) and CSS Shapes Editor[\[3\]](https://github.com/adobe-webplatform/brackets-css-shapes-editor/issues/4) extensions are known to cause problems with Live Preview, and other extensions could potentially interfere also.

#### Clear Live Preview Profile/Cache
Choose _Help > Show Extensions Folder_, go up one level to the parent folder, and remove the 'live-dev-profile' folder. This will not affect any Brackets settings, but may clear up Live Preview problems.

## <a name="slow"> </a>Brackets is Running Slow
This section discusses some of the features that can affect performance and possible solutions.

### Activity Monitor
On Mac OS 10.9 (Mavericks), Activity Monitor will say the Brackets Helper process is "Not Responding" even when it is working normally ([bug #5794](https://github.com/adobe/brackets/issues/5794)). You can safely ignore this unless Brackets is actually failing to respond when you click or type text.

### Extensions
Most Brackets extensions don't impact performance, but some may slow down Brackets (for example Show Whitespace can cause slow typing performance). Try [`Debug > Reload Without Extensions`](#wiki-disable-all-extensions) to quickly check if the problem is being caused by an extension.

### File Searching
Using "Find in Files" and "JS Code Hinting" can be slow because of the number of files that are searched. You can try installing the [experimental Exclude Folders extension](https://github.com/gruehle/exclude-folders) to limit the number of folders that are searched.

### Highlight Active Line
This feature can negatively impact scrolling performance, so try turning it off with: View > Highlight Active Line

### JavaScript Code Hinting
Collecting the information required to build the JS code hint lists can slow down Brackets. Start by reading the [Configuration section of the JavaScript Code Hints guide](https://github.com/adobe/brackets/wiki/JavaScript-Code-Hints#configuration).

You can disable the hints by moving the JavaScriptCodeHints folder out of www/extensions/default (installed version) or src/extensions/default (Git source) folder and into the extensions/disabled folder, and restarting Brackets.

## <a name="extension-install"> </a>Having trouble installing extensions?

### Working behind proxy

If your computer needs to use a proxy to get to the web, you'll need to configure Brackets to use it. Use Debug > Open Preferences File and then add a `"proxy"` property to the JSON file. The value should be the URL of your proxy server. [Read more about configuring Preferences](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#preferences).

You can also install extensions manually:

1. Find the extension in the [online Extension Registry](https://brackets-registry.aboutweb.com/)
2. Click the name of the extension to download it as a zip file
3. In Brackets, open Extension Manager
4. Drag the zip file onto the "Drag .zip here" zone in the lower left of the window

### Check `localhost` IP

Make sure your `localhost` address resolves correctly, normally it should resolve to `127.0.0.1`. You can do it by executing `ping localhost` from the command line. If it fails, add the following entry to your `hosts` file:

```
127.0.0.1	localhost
```

`hosts` file is `/etc/hosts` on Mac and Linux. On Windows it's located in `%windir%\system32\drivers\etc\hosts`

## <a name="other"> </a>Other: Brackets Is Acting Weird

### Preferences are Not Saved

Your preferences may have become corrupt. Follow these instructions to delete your preferences and cache folder: https://github.com/adobe/brackets/wiki/Cache-Folder#resetting-cache--preferences.

### Disable All Extensions
Use `Debug > Reload Without Extensions` to quickly check whether the problem is being caused by an extension. To re-enable your extensions, just quit and relaunch Brackets, or choose `Debug > Reload With Extensions`.

If this fixes the problem, you can identify the problematic extension by re-enabling extensions one-by-one:

#### Disabling extensions individually

1. Start with all extensions disabled: Choose _Help > Show Extensions Folder_ and move all extensions from the "user" folder into the "disabled" folder (this is similar to what Debug > Reload Without Extensions does).
2. Move one extension back into the "user" folder, then quit and re-launch Brackets.
3. Check if the problem has come back. If not, repeat step 2.

**Binary Reduction**: The previous method tests one extension at a time. This works for a small number of extensions but is slow if you have a lot of extensions. Instead, you can test _half_ of your remaining extensions at a time. You're done when you only have 1 extension being tested. As above, you have to quit and re-launch Brackets between each test.

Note: to make it easier to manage which extensions you have tested, create a third folder named "disabled-verified" for extensions which have been verified to not cause the problem.

1. (same as above)
2. Move half (doesn't need to be exact) of your extensions from the "disabled" folder to the "user" folder.
3. If you do not see the problem, move all of the extensions in the "user" folder to the "disabled-verified" folder. Repeat step 2.
4. Otherwise you do see problem, so move all extensions from "disabled" folder to "disabled-verified" folder. Then move half of your extensions from "user" folder back to "disabled" folder. Repeat step 3.

Once you have identified the problematic extension, move all extensions from "disabled-verified" folder back to "user" folder.

**Alternative**: Select _Debug > Show Developer Tools_ and look at the console.  If an error message is present, it may have a link to the code that is failing, which in turn may point out a specific extension.

### Debug with Developer Tools
Choose `Debug > Show Developer Tools` to open an instance of the Developer Tools for Brackets. If you've used the Developer Tools in Chrome this will look familiar. Check the Console tab for errors.

### Can't Paste Text Into Brackets
There's a [known issue](https://github.com/adobe/brackets/issues/2531) with the Webroot Identity Shield software blocking Paste in Brackets. If you're running Webroot, try the [workarounds on their support page](https://community.webroot.com/t5/Webroot-SecureAnywhere-Antivirus/Cut-Copy-or-Paste-Problems-Character-Entry-Issues-Scripting/ta-p/18396#.UhKfpmR4b9F). (Fully up-to-date Webroot should no longer treat Brackets as "unknown"; try [reinstalling](https://github.com/adobe/brackets/issues/2531#issuecomment-23747772) to update your copy).

## <a name="fileassoc"> </a>How do I associate an extension with a type of file?

Let's say you want ".inc" files to be treated like ".php" files. To do this, open an ".inc" file and click the dropdown in the lower-right (for unrecognized files it will say "Text"). Choose the desired file type (e.g. "PHP"), then open the dropdown again and choose "Set as Default for .inc Files."

You can also [edit the `language.fileExtensions` preference directly](https://github.com/adobe/brackets/wiki/Language-Support#preferences).

## <a name="prefs37"> </a>Error reading preferences file (release 37)

In Brackets release 37, there's a known issue in which [an empty preferences file could cause Brackets to display an error message on startup](https://github.com/adobe/brackets/issues/7220). If you see this error, Brackets will still start and will open the (empty) preferences file into the editor. Follow these steps, and you'll be all set:

1. Type this into the editor:
    ```{}```
2. Save
3. Quit and restart Brackets

Starting with Brackets 36, we've added a bunch of preferences that let you tailor how Brackets works for you. Read more on the [How To Use Brackets](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#preferences) page.

The bug described here is fixed in Brackets 38.

## <a name="notsolved"> </a>Still Having a Problem?

* **[File an issue](https://github.com/adobe/brackets/wiki/How-to-Report-an-Issue)** - be sure to include as many details as possible (see link for specific guidelines)
* **Contact us** on [Slack](https://brackets.slack.com) ([request an invite](https://brackets-slack.herokuapp.com) first) or via one of the other channels mentioned in the [README](https://github.com/adobe/brackets/blob/master/README.md#i-want-to-keep-track-of-how-brackets-is-doing)