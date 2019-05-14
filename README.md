# osx-util

TODO: Table of Contents

- - -

## Overview

This repository stores useful scripts for use on OS X.

- - -

## jclean

The current version of this utility is v1.2.0.

The jclean application is the ultimate Java removal tool for Java 8 for OS X. The actions taken by this script are an amalgamation of the incorrect Oracle Java removal documentation (they are missing things), several stack overflow questions and answers, and the testing and experimentation of isaki-x (thank goodness for Time Machine).

In order to work properly, this script needs to run as root (you should execute it via sudo).

    Usage: jclean
    
        h|help             : Display this screen.
        version            : Display version information.
        x|exec             : If this is not specified, this process operates in a
                              what-if mode. Specifying this option requires root
                              access.
        v|verbose          : Enable verbose output from directory removal via
                              File::Path::remove_tree.
        n|no-verify        : This disables the user verification of the cleanup;
                              this options is designed for mass cleanups in an
                              enterprise deployment of Java across many OS X
                              machines. If you are running this for your own
                              personal OS X instance, you should avoid this option
                              unless you have absolute confidence you didn't make
                              any errors in your command line.
    
    Environment Settings:
    
        ANSI_COLORS_DISABLED
            If set to a true value, this will disable the color output. If you
            are seeing strange escaped sequences and no colors in your terminal
            when utilizing this utility, you should set this environment
            variable to clean up your output.


You can run this safely as your normal login user to see what the script would do:

    Checking: /Library/Application Support/Oracle/Java... exists
    Checking: /Library/PreferencePanes/JavaControlPanel.prefPane... symlink
    Checking: /Library/Internet Plug-Ins/JavaAppletPlugin.plugin... exists
    Checking: /Library/LaunchAgents/com.oracle.java.Java-Updater.plist... symlink
    Checking: /Library/Preferences/com.oracle.java.Helper-Tool.plist... exists
    Checking: /Library/LaunchDaemons/com.oracle.java.Helper-Tool.plist... symlink
    Checking: /Library/Java/JavaVirtualMachines/jdk*.jdk... 1 found
     - Found: /Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk
    
    Please re-run this process with --exec as root if this is ok!

Please note that during execution you may see warnings about files not existing; this was fixed in version 1.0.5 of jclean and you need to update to the latest version.

As of jclean 1.2.0, jclean supports removing other JDKs such as Corretto.

- - -

## kextpolicy

The current version of this utility is v1.1.1

This application dumps the contents of our security policy for kernel extensions (i.e. what extensions are installed and have been allowed and which have not). This script is for MacOS High Seirra (10.13) and newer.

    Usage: kextpolicy
    
        h|help             : Display this screen.
        version            : Display version information.
    
    Environment Settings:
    
        ANSI_COLORS_DISABLED
            If set to a true value, this will disable the color output. If you
            are seeing strange escaped sequences and no colors in your terminal
            when utilizing this utility, you should set this environment
            variable to clean up your output.
    
    MINIMUM MACOS VERSION: 10.13


This script will produce output as follows:

    team_id    | bundle_id              | allowed | developer_name | flags
    -----------|------------------------|---------|----------------|------
    XXXXXXXXXX | com.isaki.kext.allowed |       t | Isaki-X        |    12
    YYYYYYYYYY | com.isaki.kext.malware |       F | Not-Isaki-X    |     1

Note that 't' for allowed will be green and 'F' for disallowed will be red when colorization is enabled.

Please also note that as of MacOS Mojave (10.14) this script must be executed as root (you can use sudo). For example:

    sudo ~/git/osx-util/bin/kextpolicy

- - -

## eclipse

The current version of this utility is v1.0.3.

This is a command line launcher for eclipse that has several features.

1. Automatic override of SSH for EGit; this allows you to use your OS X keychain for Git SSH access instead of relying on Eclipse to store it. Please see the notes later in this section regarding this functionality.
2. Allows passing command line options without having to remember the syntax and usage of /usr/bin/open.
3. Allows easy switching between multiple installations of eclipse either by symlinks or an environment variable.

### Utilizing the "Default" Eclipse Installation Path

This script does not default to the location Eclipse is installed if you deployed Eclipse to your machine via the DMG. Future versions of this script may support searching there if enough demand exists. I prefer placing eclipse in my home directory and deploying via the tarball; hence the defaults chosen for this script.

The script assumes (defaults) that you have installed eclipse to ~/opt/eclipse. You can use this install path if you wish; I like to use symlinks to manage my installations but that is not required (and doesn't work well if you use multiple installations constantly).

    $ ls -ld ~/opt/eclipse*
    lrwxr-xr-x  1 isaki-x  staff   16 Apr 26 14:22 /Users/isaki-x/opt/eclipse -> eclipse.mars.SR2
    drwxr-xr-x  3 isaki-x  staff  102 Apr 26 14:22 /Users/isaki-x/opt/eclipse.mars.SR2

Just make sure that Eclipse.app is the root of the install path:

    $ ls -ld ~/opt/eclipse/*
    drwxr-xr-x@ 3 isaki-x  staff  102 Feb 18 03:38 /Users/isaki-x/opt/eclipse/Eclipse.app

### Utilizing Eclipse Installation Path Override

The alternate way to specify where your Eclipse install directory lives is to specify the ECLIPSE_INSTALL_ROOT environment variable. This must point to a directory that houses a valid Eclipse.app.

As an example, if you used the default directory structure supported by this script, your setup would look something like this:

    export ECLIPSE_INSTALL_ROOT="${HOME}/opt/eclipse"

### Important Note Regarding MacOS Sierra (10.12.2 and later)

In 10.12.2, Apple changed the default behavior of SSH to no longer utilize the keychain for managing the passwords to encrypted SSH keys. Thus, when utilizing built in SSH, you will still be promoted for your SSH key password.

In order to restore the ability for MacOS' SSH client to be able to utilize the keychain for password protected SSH keys, you need to either create or modify your local ssh configuration file. This file should be located at: ${HOME}/.ssh/config

The directive that you need to enable is called "UseKeyChain". This directive can either be placed as a global setting for your configuration or it can be setup to only be for specific hosts.

#### Global Example:

    $ cat ~/.ssh/config
    UseKeychain yes

#### Host Specific Example:

    $ cat ~/.ssh/config
    Host github.com
        IdentifyFile ~/.ssh/github.key
        UseKeychain yes
