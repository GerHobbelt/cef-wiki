This Wiki page provides a quick-start guide for creating a Debug build of CEF/Chromium using the current master (development) branch.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

This page provides a quick-start guide for setting up a minimal development environment and building the master branch of Chromium/CEF for development purposes. For a comprehensive discussion of the available tools and configurations visit the [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md) Wiki page.

This guide is NOT intended for:

- Those seeking a prebuilt binary distribution for use in third-party apps. Go [here](https://cef-builds.spotifycdn.com/index.html) instead.
- Those seeking to build the binary distribution in a completely automated manner. Go [here](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-automated-method) instead.

Development systems can be configured using dedicated hardware or a [VMware](http://www.vmware.com/products/player), [Parallels](http://www.parallels.com/eu/products/desktop/download/) or [VirtualBox](https://www.virtualbox.org/wiki/Downloads) virtual machine.

The below steps can often be used to develop the most recent release branch of CEF/Chromium in addition to the master branch. Chromium build requirements change over time so review the build requirements listed on the [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-release-branches) Wiki page before attempting to build a release branch. Then just add `--branch=XXXX` to the automate-git.py command-line where "XXXX" is the branch number you wish to build.

# File Structure

The same file structure will be used on all platforms. "~" can be any path that does not include spaces or special characters. We'll be building this directory structure for each platform in the following sections.

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

With this file structure you can develop multiple CEF/Chromium branches side-by-side. For example, repeat the below instructions using "chromium_git1" as the directory name instead of "chromium_git".

# Windows Setup

**What's Required**

- Windows Build Requirements as listed on the [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-development) Wiki page.
- Install required Visual Studio sub-components as described [here](https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup#markdown-header-windows-configuration).
- Install the exact Windows SDK version specified in the default location to avoid build issues.
- At least 8GB of RAM (32GB+ recommended) and 90GB of free disk space (for a Debug build).
- Approximately 4 hours with a fast internet connection (100Mbps) and fast build machine (2.4Ghz, 16 logical cores, SSD).

**Step-by-step Guide**

All of the below commands should be run using the system "cmd.exe" and not a Cygwin shell.

1\. Create the following directories.

```
c:\code\automate
c:\code\chromium_git
```

WARNING: If you change the above directory names/locations make sure to (a) use only ASCII characters and (b) choose a short file path (less than 35 characters total). Otherwise, some tooling may fail later in the build process due to invalid or overly long file paths.

2\. Download [depot_tools.zip](https://storage.googleapis.com/chrome-infra/depot_tools.zip) and extract to "c:\code\depot_tools". Do not use drag-n-drop or copy-n-paste extract from Explorer, this will not extract the hidden ".git" folder which is necessary for depot_tools to auto-update itself. You can use "Extract all..." from the context menu though. [7-zip](http://www.7-zip.org/download.html) is also a good tool for this.

3\. Run "update_depot_tools.bat" to install Python and Git.

```
cd c:\code\depot_tools
update_depot_tools.bat
```

4\. Add the "c:\code\depot_tools" folder to your system PATH. For example, on Windows 10:

- Run the "SystemPropertiesAdvanced" command.
- Click the "Environment Variables..." button.
- Double-click on "Path" under "System variables" to edit the value.

5\. Download the [automate-git.py](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py) script to "c:\code\automate\automate-git.py".

6\. Create the "c:\code\chromium_git\update.bat" script with the following contents.

```
set GN_DEFINES=is_component_build=true
# Use vs2017 or vs2019 as appropriate.
set GN_ARGUMENTS=--ide=vs2019 --sln=cef --filters=//cef/*
python ..\automate\automate-git.py --download-dir=c:\code\chromium_git --depot-tools-dir=c:\code\depot_tools --no-distrib --no-build
```

Run the "update.bat" script and wait for CEF and Chromium source code to download. CEF source code will be downloaded to "c:\code\chromium_git\cef" and Chromium source code will be downloaded to "c:\code\chromium_git\chromium\src". After download completion the CEF source code will be copied to "c:\code\chromium_git\chromium\src\cef".

```
cd c:\code\chromium_git
update.bat
```

7\. Create the "c:\code\chromium_git\chromium\src\cef\create.bat" script with the following contents.

```
set GN_DEFINES=is_component_build=true
# Use vs2017 or vs2019 as appropriate.
set GN_ARGUMENTS=--ide=vs2019 --sln=cef --filters=//cef/*
call cef_create_projects.bat
```

Run the "create.bat" script to generate Ninja and Visual Studio project files.

```
cd c:\code\chromium_git\chromium\src\cef
create.bat
```

This will generate a "c:\code\chromium_git\chromium\src\out\Debug_GN_x86\cef.sln" file that can be loaded in Visual Studio for debugging and compiling individual files. Replace “x86” with “x64” in this path to work with the 64-bit build instead of the 32-bit build. Always use Ninja to build the complete project. Repeat this step if you change the project configuration or add/remove files in the GN configuration (BUILD.gn file).

8\. Create a Debug build of CEF/Chromium using Ninja. Edit the CEF source code at "c:\code\chromium_git\chromium\src\cef" and repeat this step multiple times to perform incremental builds while developing.

```
cd c:\code\chromium_git\chromium\src
ninja -C out\Debug_GN_x86 cef
```

Replace "Debug" with "Release" to generate a Release build instead of a Debug build. Replace “x86” with “x64” to generate a 64-bit build instead of a 32-bit build.

9\. Run the resulting cefclient sample application.

```
cd c:\code\chromium_git\chromium\src
out\Debug_GN_x86\cefclient.exe
```

See the [Windows debugging guide](https://www.chromium.org/developers/how-tos/debugging-on-windows) for detailed debugging instructions.

# Mac OS X Setup

**What's Required**

- macOS Build Requirements as listed on the [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-development) Wiki page.
- Building on an Intel Mac is supported with all versions. Building on an Apple Silicon (ARM64) Mac is supported starting with M93 (4577 branch).
- At least 8GB of RAM (16GB+ recommended) and 100GB of free disk space (for a Debug build).
- Approximately 4 hours with a fast internet connection (100Mbps) and fast build machine (2.4Ghz, 16 logical cores, SSD).

**Step-by-step Guide**

In this example "~" is "/Users/marshall". Note that in some cases the absolute path must be used. Environment variables described in this section can be added to your "~/.bash_profile" file to persist them across sessions.

1\. Create the following directories.

```
mkdir ~/code
mkdir ~/code/automate
mkdir ~/code/chromium_git
```

2\. Download "~/code/depot_tools" using Git.

```
cd ~/code
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

3\. Add the "~/code/depot_tools" directory to your PATH. Note the use of an absolute path here.

```
export PATH=/Users/marshall/code/depot_tools:$PATH
```

4\. Download the [automate-git.py](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py) script to "~/code/automate/automate-git.py".

5\. Create the "~/code/chromium_git/update.sh" script with the following contents.

```
#!/bin/bash
python ../automate/automate-git.py --download-dir=/Users/marshall/code/chromium_git --depot-tools-dir=/Users/marshall/code/depot_tools --no-distrib --no-build --x64-build
```

**⚠** Replace `--x64-build` with `--arm64-build` if using an Apple Silicon Mac instead of an Intel Mac.

Give it executable permissions.

```
cd ~/code/chromium_git
chmod 755 update.sh
```

Run the "update.sh" script and wait for CEF and Chromium source code to download. CEF source code will be downloaded to "~/code/chromium_git/cef" and Chromium source code will be downloaded to "~/code/chromium_git/chromium/src". After download completion the CEF source code will be copied to "~/code/chromium_git/chromium/src/cef".

```
cd ~/code/chromium_git
./update.sh
```

6\. Run the "~/code/chromium_git/chromium/src/cef/cef_create_projects.sh" script to create Ninja project files. Repeat this step if you change the project configuration or add/remove files in the GN configuration (BUILD.gn file).

```
cd ~/code/chromium_git/chromium/src/cef
./cef_create_projects.sh
```

**⚠** Add `export GN_DEFINES=is_component_build=true` before running `cef_create_projects.sh` if using an Apple Silicon Mac instead of an Intel Mac.

7\. Create a Debug build of CEF/Chromium using Ninja. Edit the CEF source code at "~/code/chromium_git/chromium/src/cef" and repeat this step multiple times to perform incremental builds while developing.

```
cd ~/code/chromium_git/chromium/src
ninja -C out/Debug_GN_x64 cef
```

**⚠** Replace "x64" with "arm64" if using an Apple Silicon Mac instead of an Intel Mac. Replace "Debug" with "Release" to generate a Release build instead of a Debug build.

8\. Run the resulting cefclient, cefsimple and/or ceftests sample applications.

```
cd ~/code/chromium_git/chromium/src
open out/Debug_GN_x64/cefclient.app
# Or run directly in the console to see log output:
./out/Debug_GN_x64/cefclient.app/Contents/MacOS/cefclient
```

See the [Mac OS X debugging guide](https://www.chromium.org/developers/how-tos/debugging-on-os-x) for detailed debugging instructions.

# Linux Setup

**What's Required**

- Linux Build Requirements as listed on the [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-development) Wiki page.
- Building with other versions or distros has not been tested and may experience issues.
- At least 6GB of RAM (16GB+ recommended) and 60GB of free disk space (for a Debug build).
- Approximately 3 hours with a fast internet connection (100Mbps) and fast build machine (2.4Ghz, 16 logical cores, SSD).

**Step-by-step Guide**

In this example "~" is "/home/marshall". Note that in some cases the absolute path must be used. Environment variables described in this section can be added to your "~/.profile" or "~/.bashrc" file to persist them across sessions.

1\. Create the following directories.

```
mkdir ~/code
mkdir ~/code/automate
mkdir ~/code/chromium_git
```

2\. Download and run "~/code/install-build-deps.sh" to install build dependencies. Answer Y (yes) to all of the questions.

```
cd ~/code
sudo apt-get install curl
curl 'https://chromium.googlesource.com/chromium/src/+/master/build/install-build-deps.sh?format=TEXT' | base64 -d > install-build-deps.sh
chmod 755 install-build-deps.sh
sudo ./install-build-deps.sh --no-arm --no-chromeos-fonts --no-nacl
```

3\. Download "~/code/depot_tools" using Git.

```
cd ~/code
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

4\. Add the "~/code/depot_tools" directory to your PATH. Note the use of an absolute path here.

```
export PATH=/home/marshall/code/depot_tools:$PATH
```

5\. Download the "~/automate/automate-git.py" script.

```
cd ~/code/automate
wget https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py
```

6\. Create the "~/code/chromium_git/update.sh" script with the following contents.

```
#!/bin/bash
python ../automate/automate-git.py --download-dir=/home/marshall/code/chromium_git --depot-tools-dir=/home/marshall/code/depot_tools --no-distrib --no-build
```

Give it executable permissions.

```
cd ~/code/chromium_git
chmod 755 update.sh
```

Run the "update.sh" script and wait for CEF and Chromium source code to download. CEF source code will be downloaded to "~/code/chromium_git/cef" and Chromium source code will be downloaded to "~/code/chromium_git/chromium/src".  After download completion the CEF source code will be copied to "~/code/chromium_git/chromium/src/cef".

```
cd ~/code/chromium_git
./update.sh
```

7\. Configure `GN_DEFINES` for your desired build environment.

Chromium provides [sysroot images](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/linux/sysroot.md) for consistent builds across Linux distros. The necessary files will have downloaded automatically as part of step 6 above. Usage of Chromium's sysroot is recommended if you don't want to deal with potential build breakages due to incompatibilities with the package or kernel versions that you've installed locally. To use the sysroot image configure the following GN_DEFINES:

```
export GN_DEFINES="use_sysroot=true use_allocator=none symbol_level=1 is_cfi=false use_thin_lto=false"
```

It is also possible to build using locally installed packages instead of the provided sysroot. Choosing this option may require additional debugging effort on your part to work through any build errors that result. On Ubuntu 18.04 the following GN_DEFINES have been tested to work reliably:

```
export GN_DEFINES="use_sysroot=false use_allocator=none symbol_level=1 is_cfi=false use_thin_lto=false use_vaapi=false"
```

Note that the "cefclient" target cannot be built directly when using the sysroot image. You can work around this limitation by creating a [binary distribution](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-manual-packaging) after completing step 9 below, and then building the cefclient target using that binary distribution.

You can also create an [AddressSanitizer build](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/asan.md) for enhanced debugging capabilities. Just add `is_asan=true dcheck_always_on=true` to the GN_DEFINES listed above and build the `out/Release_GN_x64` directory in step 9 below. Run with the `asan_symbolize.py` script as described in the AddressSanitizer link to get symbolized output.

The various other listed GN arguments are based on recommendations from the [AutomateBuildSetup Wiki page](https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md#markdown-header-linux-configuration). You can [search for them by name](https://source.chromium.org/search?q=use_allocator%20gni&ss=chromium) in the Chromium source code to find more details.

8\. Run the "~/code/chromium_git/chromium/src/cef/cef_create_projects.sh" script to create Ninja project files. Repeat this step if you change the project configuration or add/remove files in the GN configuration (BUILD.gn file).

```
cd ~/code/chromium_git/chromium/src/cef
./cef_create_projects.sh
```

9\. Create a Debug build of CEF/Chromium using Ninja. Edit the CEF source code at "~/code/chromium_git/chromium/src/cef" and repeat this step multiple times to perform incremental builds while developing. Note the additional "chrome_sandbox" target may be required by step 10. The "cefclient" target will only build successfully if you set `use_sysroot=false` in step 7, so remove that target if necessary.

```
cd ~/code/chromium_git/chromium/src
ninja -C out/Debug_GN_x64 cefclient cefsimple ceftests chrome_sandbox
```

Replace "Debug" with "Release" to generate a Release build instead of a Debug build.

10\. Set up the [Linux SUID sandbox](https://chromium.googlesource.com/chromium/src.git/+/HEAD/docs/linux/suid_sandbox_development.md) if you are using an older kernel (< 3.8). See [here](https://chromium.googlesource.com/chromium/src.git/+/HEAD/docs/linux/sandboxing.md) for more background on Linux sandboxing technology.

```
# This environment variable should be set at all times.
export CHROME_DEVEL_SANDBOX=/usr/local/sbin/chrome-devel-sandbox

# This command only needs to be run a single time.
cd ~/code/chromium_git/chromium/src
sudo BUILDTYPE=Debug_GN_x64 ./build/update-linux-sandbox.sh
```

11\. Run the cefclient, cefsimple and/or ceftests sample applications. Note that the cefclient application will only have built successfully if you set `use_sysroot=false` in step 7.

```
cd ~/code/chromium_git/chromium/src
./out/Debug_GN_x64/cefclient
```

See the [Linux debugging guide](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_debugging.md) for detailed debugging instructions.

# Next Steps

- If you're seeking a good code editor on Linux check out the [Eclipse](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_eclipse_dev.md) and [Emacs](https://chromium.googlesource.com/chromium/src/+/master/docs/emacs.md) tutorials.
- Review the [Tutorial](https://bitbucket.org/chromiumembedded/cef/wiki/Tutorial.md) and [GeneralUsage](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md) Wiki pages for details on CEF implementation and usage.
- Review the Chromium debugging guide for [Windows](https://www.chromium.org/developers/how-tos/debugging-on-windows), [Mac OS X](https://www.chromium.org/developers/how-tos/debugging-on-os-x) or [Linux](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_debugging.md).
- When you’re ready to contribute your changes back to the CEF project see the [ContributingWithGit](https://bitbucket.org/chromiumembedded/cef/wiki/ContributingWithGit.md) Wiki page for instructions on creating a pull request.