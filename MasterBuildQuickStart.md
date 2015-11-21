This Wiki page provides a quick-start guide for creating a Debug build of CEF/Chromium using the current master (development) branch.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

This page provides a quick-start guide for setting up a minimal development environment and building the master branch of Chromium/CEF for development and/or debugging purposes. This page is not intended for CEF end users seeking a pre-built binary distribution for use in third-party applications (those people should go [here](http://magpcss.net/cef_downloads/)).

Systems can be configured using dedicated hardware or a [VMware](http://www.vmware.com/products/player), [Parallels](http://www.parallels.com/eu/products/desktop/download/) or [VirtualBox](https://www.virtualbox.org/wiki/Downloads) virtual machine. For a comprehensive overview of the available tools and configurations visit the [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md) Wiki page.

# File Structure

The same file structure will be used on all platforms. `~` can be any path that does not include spaces or special characters. We'll be building this directory structure for each platform in the following next sections.

```
~/code/
  automate/
    automate-git.py   <-- CEF build script
  chromium_git/
    cef/              <-- CEF source checkout
    chromium/
      src/            <-- Chromium source checkout
    update.[bat|sh]   <-- Bootstrap script for automate-git.py
  depot_tools/        <-- Chromium build tools
```

# Windows

In this example `~` is `c:\`. All of the below commands should be run using the system `cmd.exe` and not a Cygwin shell.

1\. Install Windows 7 64-bit or newer and Visual Studio 2013 Professional Update 4. Visual Studio 2015 Professional is also supported but the [bugs](https://bitbucket.org/chromiumembedded/cef/issues/1691/windows-reduce-debug-build-link-time#comment-21297285) can be rather annoying. At least 8GB of RAM and 40GB of disk space are required to successfully build Chromium/CEF.

2\. Create the following directories.

```
c:\code\automate
c:\code\chromium_git
```

3\. Download [depot_tools.zip](https://src.chromium.org/svn/trunk/tools/depot_tools.zip) and extract to `c:\code\depot_tools`. Do not use drag-n-drop or copy-n-paste extract from Explorer, this will not extract the hidden ".git" folder which is necessary for depot_tools to auto-update itself. You can use "Extract all..." from the context menu though. [7-zip](http://www.7-zip.org/download.html) is also a good tool for this.

4\. Run `update_depot_tools.bat` to install Python, Git and SVN.

```
cd c:\code\depot_tools
update_depot_tools.bat
```

5\. Add the `c:\code\depot_tools` folder to your system PATH. For example, on Windows 10:

A\. Run the `SystemPropertiesAdvanced` command

B\. Click the `Environment Variables...` button.

C\. Double-click on `Path` under `System variables` to edit the value.

6\. Download the [automate-git.py](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py) script to `c:\code\automate\automate-git.py`.

7\. Create the `c:\code\chromium_git\update.bat` script with the following contents.

```
set GYP_MSVS_VERSION=2013
python ..\automate\automate-git.py --download-dir=c:\code\chromium_git --depot-tools-dir=c:\code\depot_tools --no-distrib --no-build
```

8\. Run the `c:\code\chromium_git\update.bat` script and wait for CEF and Chromium source code to download. CEF source code will be downloaded to `c:\code\chromium_git\cef` and Chromium source code will be downloaded to ``c:\code\chromium_git\chromium\src`.

```
cd c:\code\chromium_git
update.bat
```

9\. Create a Debug build of CEF/Chromium using Ninja. Edit the CEF source code at `c:\code\chromium_git\chromium\src\cef` and repeat this step multiple times to create incremental builds while developing/debugging.

```
cd c:\code\chromium_git\chromium\src
ninja -C out\Debug cefclient cefsimple cef_unittests
```

10\. Run the resulting cefclient sample application.

```
cd c:\code\chromium_git\chromium\src
out\Debug\cefclient.exe
```

# Mac OS X

In this example `~` is `/users/marshall`.

1\. Install OS X 10.10 or newer and Xcode 6.4 (available for download from https://developer.apple.com/downloads/ with an Apple ID). If you have multiple versions of Xcode installed use the [xcode-select](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/xcode-select.1.html) tool to select the 6.4 version. At least 8GB of RAM and 40GB of disk space are required to successfully build Chromium/CEF.

2\. Create the following directories.

```
mkdir ~/code/automate
mkdir ~/code/chromium_git
```

3\. Download `~/code/depot_tools` using Git.

```
cd ~/code
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

4\. Add the `~/code/depot_tools` directory to your PATH. Note the use of an absolute path here.

```
export PATH=/users/marshall/code/depot_tools:$PATH
```

5\. Download the `~/automate/automate-git.py` script.

```
cd ~/code/automate
wget https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py
```

6\. Create the `~/code/chromium_git/update.sh` script with the following contents.

```
#!/bin/bash
python ../automate/automate-git.py --download-dir=/users/marshall/code/chromium_git --depot-tools-dir=/users/marshall/code/depot_tools --no-distrib --no-build
```

Give it executable permissions.

```
cd ~/code/chromium_git
chmod 755 update.sh
```

7\. Run the `~/code/chromium_git/update.sh` script and wait for CEF and Chromium source code to download. CEF source code will be downloaded to `~/code/chromium_git/cef` and Chromium source code will be downloaded to ``~/code/chromium_git/chromium/src`.

```
cd ~/code/chromium_git
./update.sh
```

8\. Create a Debug build of CEF/Chromium using Ninja. Edit the CEF source code at `~/code/chromium_git/chromium/src/cef` and repeat this step multiple times to create incremental builds while developing/debugging.

```
cd ~/code/chromium_git/chromium/src
ninja -C out/Debug cefclient cefsimple cef_unittests
```

9\. Run the resulting cefclient sample application.

```
cd ~/code/chromium_git/chromium/src
open out/Debug/cefclient.app
```

# Linux

In this example `~` is `/home/marshall`.

1\. Install [Ubuntu 14.04 LTS 64-bit](http://www.ubuntu.com/download/desktop). At least 6GB of RAM and 40GB of disk space are required to successfully build Chromium/CEF. To build on Debian 7 see the [BuildingOnDebian7](https://bitbucket.org/chromiumembedded/cef/wiki/BuildingOnDebian7.md) Wiki page for additional instructions.

2\. Create the following directories.

```
mkdir ~/code/automate
mkdir ~/code/chromium_git
```

3\. Download and run `~/code/install-build-deps.sh` to install build dependencies. Answer Y (yes) to all of the questions.

```
cd ~/code
curl 'https://chromium.googlesource.com/chromium/src/+/master/build/install-build-deps.sh?format=TEXT' | base64 -d > install-build-deps.sh
chmod 755 install-build-deps.sh
sudo ./install-build-deps.sh
```

4\. Install the `libgtkglext1-dev` package required by the cefclient sample application.

```
sudo apt-get install libgtkglext1-dev
```

5\. Download `~/code/depot_tools` using Git.

```
cd ~/code
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

6\. Add the `~/code/depot_tools` directory to your PATH. Note the use of an absolute path here.

```
export PATH=/home/marshall/code/depot_tools:$PATH
```

7\. Download the `~/automate/automate-git.py` script.

```
cd ~/code/automate
wget https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py
```

8\. Create the `~/code/chromium_git/update.sh` script with the following contents.

```
#!/bin/bash
python ../automate/automate-git.py --download-dir=/home/marshall/code/chromium_git --depot-tools-dir=/home/marshall/code/depot_tools --no-distrib --no-build
```

Give it executable permissions.

```
cd ~/code/chromium_git
chmod 755 update.sh
```

9\. Run the `~/code/chromium_git/update.sh` script and wait for CEF and Chromium source code to download. CEF source code will be downloaded to `~/code/chromium_git/cef` and Chromium source code will be downloaded to ``~/code/chromium_git/chromium/src`.

```
cd ~/code/chromium_git
./update.sh
```

10\. Create a Debug build of CEF/Chromium using Ninja. Edit the CEF source code at `~/code/chromium_git/chromium/src/cef` and repeat this step multiple times to create incremental builds while developing/debugging.

```
cd ~/code/chromium_git/chromium/src
ninja -C out/Debug cefclient cefsimple cef_unittests chrome_sandbox
```

11\. Set up the [Linux SUID sandbox](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_suid_sandbox_development.md). This only needs to be done a single time.

```
export CHROME_DEVEL_SANDBOX=/usr/local/sbin/chrome-devel-sandbox
cd ~/code/chromium_git/chromium/src
sudo ./build/update-linux-sandbox.sh
```

12\. Run the cefclient sample application.

```
cd ~/code/chromium_git/chromium/src
./out/Debug/cefclient
```


