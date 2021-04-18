# NOTE

As of Brackets 1.11. the Linux version is up-to-date with other platforms and this guide should not be needed anymore.

If some of the issues persists, [do open an issue](https://github.com/adobe/brackets/issues/new)

# Brackets Linux Guide

> Parts of this guide have been contributed by @nathanjplummer and are available at https://nathanjplummer.github.io/Brackets/

How to fix common Brackets issues that may occur with the Linux Operating System.

## Libcrypt11 Missing Dependency in Ubuntu

Brackets depends on libcrypt11 but newer version of Ubuntu ship with libcrypt20. If you’re using Ubuntu or a derivative (Kubuntu, Linux Mint, etc) you may get an error message similar to the following:

```bash
    dpkg: dependency problems prevent configuration of brackets:
     brackets depends on libgcrypt11 (>= 1.4.5); however:
      Package libgcrypt11 is not installed.

    dpkg: error processing package brackets (--install):
     dependency problems - leaving unconfigured
    Processing triggers for hicolor-icon-theme (0.15-0ubuntu1) ...
    Errors were encountered while processing:
     brackets
```

### Solution 1: Manually Install libcrypt11

The links below will download a copy of libcrypt11 from an older version of Ubuntu. Note that libcrypt11 can coexist peacefully with newer versions.

*   [32bit x86](https://launchpad.net/ubuntu/+archive/primary/+files/libgcrypt11_1.5.3-2ubuntu4.2_i386.deb)
*   [64bit x64](https://launchpad.net/ubuntu/+archive/primary/+files/libgcrypt11_1.5.3-2ubuntu4.2_amd64.deb)
*   All other architectures can be found [here](https://launchpad.net/ubuntu/+source/libgcrypt11)

### Solution 2: Install from PPA

Ubuntu has a PPA for Brackets installs which will automatically import libcrypt11 while also keeping Brackets up to date automatically. To install from PPA, open a terminal and then:

1.  `sudo add-apt-repository ppa:webupd8team/brackets`
    *   If asked to install key say “yes”
2.  `sudo apt-get update`
3.  `sudo apt-get install brackets`

## `Save all`-keyboard shortcut issue

> Cause of issue and a workaround contributed by [@haslam22](https://github.com/adobe/brackets/issues/12954#issuecomment-268061096) and [@Pk13055](https://github.com/adobe/brackets/issues/12954#issuecomment-268253676)

If you notice the `Save all`-keyboard shortcut `Ctrl + Alt + S` not working on some Desktop Environments such as Gnome 3, it might be overshadowed by a system shortcut to disable/enable “roll up” / “shade” on an active window.

You can enter this command (in the terminal) to disable this shortcut to get `Save all` working:

``` bash
gsettings set org.gnome.desktop.wm.keybindings toggle-shaded "['disabled']"
```

Alternatively, you can re-map said shortcut to another key group of your choice (for example `Ctrl + Super + S`) with:

``` bash
gsettings set org.gnome.desktop.wm.keybindings toggle-shaded "['<Control><Super>s']"
```

> Note: The Save All option from the drop-down menu works without any modification; you can always resort to that in case you don't want to alter the system shortcut.

## Lack of HiDPI support

If you have a high resolution screen, such as an Apple Retina, Brackets will ignore any HiDPI settings and present you with very small text and icons.

### Step 1: Install Extension “UI Too Small”

The [UI Too Small](https://github.com/1beb/ui-too-small) Extension increases the size of fonts inside the Bracket’s UI. Unfortunately it does not change the size of the icons. You can install the extension from the Brackets Extension manager.

*   From the Brackets File Menu choose “File” -> “Extension Manager”
*   Search for “Ui Too Small”
*   The Extension’s author is: “Brandon Bertelsen”
*   Click `Install` next to the extension name
*   Save your work and restart Brackets

### Step 2: Change your Font Size

If the font in your work view i still too small, you can change you font size by:

*   From the Brackets File Menu choose “View” -> “Themes”
*   You’ll see a section Font Size:
*   Increase the number in front of “px.” Be sure to leave the “px” unit.

# Brackets-Electron

## What is Brackets-Electron?

An alternative to the above issues is to install [Brackets-Electron](https://github.com/zaggino/brackets-electron)- a fork of Brackets that:

1.  Merges newer code that has not yet made it to Brackets standard
2.  Replaces the Brackets CEF3 Shell with a more Linux friendly Electron Shell.

Worth noting: Brackets-Electron is an unofficial fork and not supported by the Brackets team.

While being more cutting edge, and hence, _potentially_ more error prone, Brackets-Electron does fix the above issues. Furthermore, it retains compatibility with most Brackets Extensions and can be installed alongside vanilla Brackets without conflict.

You can download [Brackets-Electron here.](https://github.com/zaggino/brackets-electron/releases)

## Install Brackets-Electron as an AppImage

While the Brackets-Electron team currently publishes .deb packages alongside the more universal AppImage format, plans are to eventually move to only AppImage Packages.

To install an AppImage:

1.  From the terminal, change to the directory where your AppImage is stored. Example:
    *   `cd home/user/Downloads`
2.  Change permissions on the AppImage with command `chmod a+x`
    *   `chmod a+x brackets-electron-1.8.2-x86_64.AppImage`
3.  Run image with ./imagename
    *   `./brackets-electron-1.8.2-x86_64.AppImage`

Once you run the AppImage you will be given the option of installing a shortcut to Brackets-Electron in your program menu.


![Install from AppImage](https://nathanjplummer.github.io/Brackets/images/appimage.jpg)

Click "Yes" to install to your menu.

NOTE: If you change the location of the AppImage your program link will cease to function. Put your AppImage in a more permanent location before you run it.