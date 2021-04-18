The first thing you're going to need to do is clone the brackets and brackets-shell repositories and build brackets-shell following the instructions from the brackets-shell repo.

Next you need to get the Depot_Tools and add it to your path

    cd ~/
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

Next open your .bash_profile or .bashrc and add ~/depot_tools to your path:

    export PATH="$PATH":~/depot_tools

You also need to execute that line in bash or open a new terminal window.

Next you need to add googlecode to the list of trusted certificates for svn

    svn list https://sctp-refimpl.googlecode.com/svn/trunk/KERN/usrsctp/usrsctplib

Next you need to get gyp into a search path.  I just copied the one from ~/brackets-shell/gyp to ~/depot_tools. 

Next you need to start the process of getting the chromium and cef sources:

    cd ~/
    svn checkout http://chromiumembedded.googlecode.com/svn/branches/1547/cef3/tools/automate automate
    cd ~/automate
    python automate.py --url http://chromiumembedded.googlecode.com/svn/branches/1547/cef3 --download-dir download --ninja-build --no-release-build --force-build --force-distrib --force-config
 

Other flags (documented as we find and need them):
    
    -–force-clean     (deletes intermediates)
    --force-update    (deletes webkit so it can be downloaded again)


### See also

**[[CEF Debugging tips|CEF Debugging]]**