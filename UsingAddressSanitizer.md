This Wiki page explains how to use AddressSanitizer with CEF-based applications.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

[AddressSanitizer](https://code.google.com/p/address-sanitizer/) (ASan) is a fast memory error detector based on compiler instrumentation (LLVM). The below instructions relate to using ASan with CEF-based applications on Linux. ASan is also supported on OS X and Windows (experimental) but usage with CEF has not been tested on those platforms. For examples of ASan output see [this bug](https://bitbucket.org/chromiumembedded/cef/issue/1457) and [this bug](https://bitbucket.org/chromiumembedded/cef/issue/1458).

# System Setup

The below instructions are based on the CEF 2357 branch built on an Ubuntu 14 LTS 64-bit Linux system. To set up this system:

1\. Install [Ubuntu 14 LTS 64-bit](http://www.ubuntu.com/download/desktop). This can be done using dedicated hardware or a [VMware](http://www.vmware.com/products/player), [Parallels](http://www.parallels.com/eu/products/desktop/download/) or [VirtualBox](https://www.virtualbox.org/wiki/Downloads) virtual machine. At least 6GB of RAM and 30GB of disk space are required to successfully build Chromium/CEF.

2\. Install the required build dependencies (full instructions are [here](https://code.google.com/p/chromium/wiki/LinuxBuildInstructionsPrerequisites)).

```
# Download the install-build-deps.sh script at the correct revision (initially base64 encoded).
wget -O install-build-deps.txt https://chromium.googlesource.com/chromium/src/+/fedbbb0aa762826ba2fc85a2b934dc9660639aa7/build/install-build-deps.sh?format=TEXT
# Decode the script.
python -m base64 -d install-build-deps.txt > install-build-deps.sh
# Mark the script as executable.
chmod 755 install-build-deps.sh

# Run the script excluding unnecessary components.
sudo ./install-build-deps.sh --no-lib32 --no-arm --no-chromeos-fonts --no-nacl

# Install additional packages required by cefclient.
sudo apt-get install libgtkglext1-dev
```

3\. Download Chromium and CEF source code at the correct branch and without building (see [BranchesAndBuilding](BranchesAndBuilding.md) for complete CEF build instructions).

```
python automate-git.py --download-dir=/path/to/chromium_git --depot-tools-dir=/path/to/depot_tools --no-distrib --no-build --force-clean --branch=2357
```

4\. Add depot\_tools to the PATH.

```
export PATH=/path/to/depot_tools:$PATH
```

## Building CEF with ASan

To build Chromium/CEF with ASan enabled follow these steps:

1\. Configure Chromium to build using ASan (see [Testing/AddressSanitizer](http://www.chromium.org/developers/testing/addresssanitizer) for complete Chromium-related ASan instructions).

```
export GYP_DEFINES='asan=1 dcheck_always_on=1'
```

2\. Generate CEF project files.

```
cd /path/to/chromium_git/chromium/src/cef
./cef_create_projects.sh
```

3\. Create a Release build of CEF. ASan must always be used with Release builds.

```
cd /path/to/chromium_git/chromium/src
ninja -C out/Release cefclient cefsimple cef_unittests chrome_sandbox
```

4\. Run the resulting executable and pipe output to the ASan post-processing script so that stack traces will be symbolized.

```
cd /path/to/chromium_git/chromium/src
./out/Release/cef_unittests 2>&1 | tools/valgrind/asan/asan_symbolize.py
```

# Using CEF and ASan with third-party executables

ASan on Linux is built as a static library by default. This works fine with applications like cefclient and cef\_unittests that are built at the same time as libcef and consequently specify the “-fsanitize=address” flag. However, in order to use libcef and ASan with pre-built third-party executables (e.g. Java and [JCEF](https://code.google.com/p/javachromiumembedded/)) it’s necessary to build ASan as a shared library (see [here](https://code.google.com/p/address-sanitizer/issues/detail?id=271) for more information). This works as follows (see step 8 below for an example):

A. The ASan library (“libclang\_rt.asan-x86\_64.so” on 64-bit Linux) is specified via LD\_PRELOAD so that it will be loaded into the main executable at runtime.

B. The main executable delay loads “libcef.so” which depends on “libc++.so” and “libclang\_rt.asan-x86\_64.so” provided by the Chromium Clang build.

C. Output is piped to the ASan post-processing script in order to symbolize stack traces.

To build ASan as a shared library the following changes are required to Chromium’s default Clang build and GYP configuration:

1\. Create a custom build of Clang with ASan configured as a shared library.

A. Edit the “tools/clang/scripts/update.sh” script and add "-DCOMPILER\_RT\_BUILD\_SHARED\_ASAN=ON" to all CMake command-lines (see [AsanAsDso](https://code.google.com/p/address-sanitizer/wiki/AsanAsDso) and [HowToBuild](https://code.google.com/p/address-sanitizer/wiki/HowToBuild) for more information).

```
diff --git tools/clang/scripts/update.sh tools/clang/scripts/update.sh
index 10a4645..bd7c5ab 100755
--- tools/clang/scripts/update.sh
+++ tools/clang/scripts/update.sh
@@ -472,6 +472,7 @@ if [[ -n "${bootstrap}" ]]; then
   pushd "${LLVM_BOOTSTRAP_DIR}"
 
   cmake -GNinja \
+      -DCOMPILER_RT_BUILD_SHARED_ASAN=ON \
       -DCMAKE_BUILD_TYPE=Release \
       -DLLVM_ENABLE_ASSERTIONS=ON \
       -DLLVM_TARGETS_TO_BUILD=host \
@@ -567,6 +568,7 @@ add_subdirectory(\${CHROMIUM_TOOLS_SRC} \${CMAKE_CURRENT_BINARY_DIR}/a)
 EOF
 rm -fv CMakeCache.txt
 MACOSX_DEPLOYMENT_TARGET=${deployment_target} cmake -GNinja \
+    -DCOMPILER_RT_BUILD_SHARED_ASAN=ON \
     -DCMAKE_BUILD_TYPE=Release \
     -DLLVM_ENABLE_ASSERTIONS=ON \
     -DLLVM_ENABLE_THREADS=OFF \
@@ -612,6 +614,7 @@ pushd "${COMPILER_RT_BUILD_DIR}"
 
 rm -fv CMakeCache.txt
 MACOSX_DEPLOYMENT_TARGET=${deployment_target} cmake -GNinja \
+    -DCOMPILER_RT_BUILD_SHARED_ASAN=ON \
     -DCMAKE_BUILD_TYPE=Release \
     -DLLVM_ENABLE_ASSERTIONS=ON \
     -DLLVM_ENABLE_THREADS=OFF \
@@ -655,6 +658,7 @@ if [[ -n "${with_android}" ]]; then
   pushd ${LLVM_BUILD_DIR}/android
   rm -fv CMakeCache.txt
   MACOSX_DEPLOYMENT_TARGET=${deployment_target} cmake -GNinja \
+      -DCOMPILER_RT_BUILD_SHARED_ASAN=ON \
       -DCMAKE_BUILD_TYPE=Release \
       -DLLVM_ENABLE_ASSERTIONS=ON \
       -DLLVM_ENABLE_THREADS=OFF \
```

B. Run the “tools/clang/scripts/update.sh” script to create a local build of Clang.

```
cd /path/to/chromium_git/chromium/src
./tools/clang/scripts/update.sh --force-local-build --without-android
```

This will create “libclang\_rt.asan-x86\_64.so” (assuming 64-bit Linux) in the “third\_party/llvm-build/Release+Asserts/lib/clang/3.7.0/lib/linux” directory.

2\. Copy “libclang\_rt.asan-x86\_64.so” to the “out/Release/lib” directory so that binaries built as part of the Chromium build can find it.

3\. Add '-shared-libasan' or modify related exclusions everywhere that ‘-fsanitize=address’ is mentioned for Linux in “build/common.gypi”, “build/sanitizers/sanitizers.gyp”, “sandbox/linux/sandbox\_linux.gypi” and “third\_party/libvpx/libvpx.gyp” (see [AsanAsDso](https://code.google.com/p/address-sanitizer/wiki/AsanAsDso) for details). Also, specify '-mllvm -asan-globals=0' in “base/common.gypi” (see [here](https://code.google.com/p/address-sanitizer/issues/detail?id=82#c18) for details).

```
diff --git build/common.gypi build/common.gypi
index 066b0b4..6756a25 100644
--- build/common.gypi
+++ build/common.gypi
@@ -4292,16 +4292,18 @@
               ['_toolset=="target"', {
                 'cflags': [
                   '-fsanitize=address',
+                  '-shared-libasan',
                   # TODO(earthdok): Re-enable. http://crbug.com/427202
                   #'-fsanitize-blacklist=<(asan_blacklist)',
                 ],
                 'ldflags': [
                   '-fsanitize=address',
+                  '-shared-libasan',
                 ],
               }],
             ],
             'conditions': [
-              ['OS=="mac"', {
+              ['OS=="mac" or OS=="linux"', {
                 'cflags': [
                   '-mllvm -asan-globals=0',  # http://crbug.com/352073
                 ],
@@ -4865,9 +4867,11 @@
               # binaries on x86_64 host is problematic.
               # TODO(eugenis): re-enable.
               '-fsanitize=address',
+              '-shared-libasan',
             ],
             'ldflags!': [
               '-fsanitize=address',
+              '-shared-libasan',
               '-Wl,-z,noexecstack',
               '-Wl,--gc-sections',
               '-Wl,-O1',
diff --git build/sanitizers/sanitizers.gyp build/sanitizers/sanitizers.gyp
index 4126d22..1e3ef49 100644
--- build/sanitizers/sanitizers.gyp
+++ build/sanitizers/sanitizers.gyp
@@ -45,6 +45,7 @@
       'cflags/': [
         ['exclude', '-fsanitize='],
         ['exclude', '-fsanitize-'],
+        ['exclude', '-shared-libasan'],
       ],
       'direct_dependent_settings': {
         'ldflags': [
diff --git sandbox/linux/sandbox_linux.gypi sandbox/linux/sandbox_linux.gypi
index 4305b41..9ca48de 100644
--- sandbox/linux/sandbox_linux.gypi
+++ sandbox/linux/sandbox_linux.gypi
@@ -213,9 +213,11 @@
       # Do not use any sanitizer tools with this binary. http://crbug.com/382766
       'cflags/': [
         ['exclude', '-fsanitize'],
+        ['exclude', '-shared-libasan'],
       ],
       'ldflags/': [
         ['exclude', '-fsanitize'],
+        ['exclude', '-shared-libasan'],
       ],
     },
     { 'target_name': 'sandbox_services',
```

4\. Configure Chromium to build using ASan (see [Testing/AddressSanitizer](http://www.chromium.org/developers/testing/addresssanitizer) for complete Chromium-related ASan instructions).

```
export GYP_DEFINES='asan=1'
```

5\. Generate CEF project files.

```
cd /path/to/chromium_git/chromium/src/cef
./cef_create_projects.sh
```

6\. Create a Release build of CEF. ASan must always be used with Release builds.

```
cd /path/to/chromium_git/chromium/src
ninja -C out/Release cefclient cefsimple cef_unittests chrome_sandbox
```

7\. Copy “libcef.so”, “libc++.so” and “libclang\_rt.asan-x86\_64.so” from the “out/Release/lib” directory to the third-party project’s binary directory (e.g. “out/Debug” for JCEF).

8\. Run the third-party executable pre-loading “libclang\_rt.asan-x86\_64.so” and piping output to the ASan post-processing script so that stack traces will be symbolized. For example, using JCEF’s [run.sh script](https://code.google.com/p/javachromiumembedded/source/browse/trunk/src/tools/run.sh):

```
LD_PRELOAD=$LIB_PATH/libclang_rt.asan-x86_64.so java -cp "$CLS_PATH" -Djava.library.path=$LIB_PATH tests.$RUN_TYPE.MainFrame "$@" 2>&1 | /path/to/chromium_git/chromium/src/tools/valgrind/asan/asan_symbolize.py
```
