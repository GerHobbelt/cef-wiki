This Wiki page describes how to build CEF on Debian 7 systems.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***

**Note: Developing Chromium/CEF on Debian 7 is not recommended. Building CEF on Debian 7 is currently necessary to support Debian 7 client installations until such time as the [GLIBC version issue](https://bitbucket.org/chromiumembedded/cef/issue/1575) is resolved.**

CEF/Chromium is primarily developed on Ubuntu 14 LTS systems as described on the [BranchesAndBuilding](https://code.google.com/p/chromiumembedded/wiki/BranchesAndBuilding) Wiki page. To build CEF/Chromium on a Debian 7 system additional manual steps are required.

The below instructions are based on the CEF 2526 branch built on a Debian 7.8.0 64-bit Linux system. To build using this system:

1\. Install [Debian 7 64-bit](https://www.debian.org/distrib/). This can be done using dedicated hardware or a [VMware](http://www.vmware.com/products/player), [Parallels](http://www.parallels.com/eu/products/desktop/download/) or [VirtualBox](https://www.virtualbox.org/wiki/Downloads) virtual machine. At least 6GB of RAM and 40GB of disk space are required to successfully build Chromium/CEF and create a CEF binary distribution.

2\. Download the install-build-deps.sh script.

```
# Download the install-build-deps.sh script at the correct revision (initially base64 encoded).
wget -O install-build-deps.txt https://chromium.googlesource.com/chromium/src/+/fedbbb0aa762826ba2fc85a2b934dc9660639aa7/build/install-build-deps.sh?format=TEXT
# Decode the script.
python -m base64 -d install-build-deps.txt > install-build-deps.sh
# Mark the script as executable.
chmod 755 install-build-deps.sh
```

3\. Edit the install-build-deps.sh script so that it will run on Debian 7. This includes removing Ubuntu-specific logic and packages that are not available on Debian 7. For example:

```
--- install-build-deps.sh	2015-03-11 16:42:28.546982015 -0400
+++ install-build-deps.sh	2015-03-11 16:43:40.930979602 -0400
@@ -72,12 +72,6 @@
 lsb_release=$(lsb_release --codename --short)
 ubuntu_codenames="(precise|quantal|raring|saucy|trusty)"
 if [ 0 -eq "${do_unsupported-0}" ] && [ 0 -eq "${do_quick_check-0}" ] ; then
-  if [[ ! $lsb_release =~ $ubuntu_codenames ]]; then
-    echo "ERROR: Only Ubuntu 12.04 (precise) through 14.04 (trusty) are"\
-        "currently supported" >&2
-    exit 1
-  fi
-
   if ! uname -m | egrep -q "i686|x86_64"; then
     echo "Only x86 architectures are currently supported" >&2
     exit
@@ -91,12 +85,11 @@
 fi
 
 # Packages needed for chromeos only
-chromeos_dev_list="libbluetooth-dev libxkbcommon-dev"
+chromeos_dev_list=""
 
 # Packages needed for development
 dev_list="apache2.2-bin bison cdbs curl dpkg-dev elfutils devscripts fakeroot
-          flex fonts-thai-tlwg g++ git-core git-svn gperf language-pack-da
-          language-pack-fr language-pack-he language-pack-zh-hant
+          flex fonts-thai-tlwg g++ git-core git-svn gperf
           libapache2-mod-php5 libasound2-dev libbrlapi-dev libav-tools
           libbz2-dev libcairo2-dev libcap-dev libcups2-dev libcurl4-gnutls-dev
           libdrm-dev libelf-dev libexif-dev libgconf2-dev libgl1-mesa-dev
@@ -170,11 +163,6 @@
 nacl_list="${nacl_list} libgl1-mesa-glx${mesa_variant}:i386"
 
 # Some package names have changed over time
-if package_exists ttf-mscorefonts-installer; then
-  dev_list="${dev_list} ttf-mscorefonts-installer"
-else
-  dev_list="${dev_list} msttcorefonts"
-fi
 if package_exists libnspr4-dbg; then
   dbg_list="${dbg_list} libnspr4-dbg libnss3-dbg"
   lib_list="${lib_list} libnspr4 libnss3"
```

4\. Install the necessary packages (run as root).

```
# Run the script excluding unnecessary components.
./install-build-deps.sh --no-lib32 --no-arm --no-chromeos-fonts --no-nacl

# Install additional packages that will be required later on.
# cmake is required when building llvm/clang.
# libgtkglext1-dev is required when building the cefclient sample application.
apt-get install cmake libgtkglext1-dev 
```

5\. Download Chromium and CEF source code at the correct branch and without building (see the [BranchesAndBuilding](BranchesAndBuilding.md) Wiki page for complete CEF build instructions).

```
python automate-git.py --download-dir=/path/to/chromium_git --depot-tools-dir=/path/to/depot_tools --no-distrib --no-build --branch=2526
```

6\. Add depot\_tools to the PATH.

```
export PATH=/path/to/depot_tools:$PATH
```

7\. Build cmake version 2.8.12.2 or newer (required by newer llvm/clang builds).

```
# Download/build in whichever directory you prefer.
cd ~

wget http://www.cmake.org/files/v2.8/cmake-2.8.12.2.tar.gz
tar xzf cmake-2.8.12.2.tar.gz
cd cmake-2.8.12.2
./configure
./make

# Run the following command as root.
./make install
```

Verify that the cmake version is now correct by running `cmake --version`.

8\. Build llvm and clang locally. Debian 7 ships with an [older version of GLIBC](https://code.google.com/p/chromiumembedded/issues/detail?id=1575) than Ubuntu 14 so the binaries distributed with the Chromium checkout cannot be used. See comments in the update.sh script for additional llvm/clang build information.

```
cd /path/to/chromium_git/chromium/src
./tools/clang/scripts/update.sh --force-local-build --without-android --gcc-toolchain '/usr'
```

If you get build errors while compiling llvm/clang for 2526 branch or newer see [this thread](http://www.magpcss.org/ceforum/viewtopic.php?f=6&t=13330) for possible solutions.

9\. Build binutils locally for 2526 branch or newer. The binutils binaries included with these newer branches have the same GLIBC issue as llvm/clang.

A\. Edit the “src/third_party/binutils/build-one.sh” script and change all instances of “/build/output” to “$PWD/build/output”. This script is intended to build inside a chroot environment but we want to build directly on the Debian system instead.

B\. Download and build binutils (based on the contents of “src/third_party/binutils/build-all.sh”):

```
cd /path/to/chromium_git/chromium/src/third_party/binutils

# Download binutils at the version specified in build-all.sh.
wget http://ftp.gnu.org/gnu/binutils/binutils-2.25.tar.bz2
tar xjf binutils-2.25.tar.bz2

cd binutils-2.25

# Apply the Chromium patches specified in build-all.sh.
patch -p1 < ../unlock-thin.patch
patch -p1 < ../plugin-dso-fix.patch

cd ..

# Build binutils (don’t forget to apply the build-one.sh changes from A first!).
./build-one.sh binutils-2.25

# Copy the resulting files to the expected location (backing up the old files first).
mv Linux_x64/Release Linux_x64/Release_old
cp -a binutils-2.25/build/output/x86_64-unknown-linux-gnu Linux_x64/Release
mkdir Linux_x64/Release/include/
cp -a binutils-2.25/build/output/include/plugin-api.h Linux_x64/Release/include/
```

Verify that the resulting executables are correct by running `Linux_x64/Release/bin/objcopy`.

10\. Build Chromium/CEF and create the CEF binary distribution.

```
python automate-git.py --download-dir=/path/to/chromium_git --depot-tools-dir=/path/to/depot_tools --no-update --force-build --force-distrib --branch=2526
```
