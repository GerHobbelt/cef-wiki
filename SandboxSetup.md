This Wiki page describes sandbox usage and requirements for CEF.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

The Chromium/CEF [multi-process model](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md#markdown-header-processes) uses separate processes for the renderer, network stack, GPU, etc. This model allows Chromium to restrict access to system resources based on the sub-process type. For example, a renderer sub-process that executes JavaScript code does not need direct access to the file system or network stack. Allowing unnecessary access can be a security concern if a sub-process is compromised while executing content (e.g. JavaScript code) from a hostile website.

The [Chromium sandbox](https://chromium.googlesource.com/chromium/src/+/master/docs/design/sandbox.md) is used to apply per-process access restrictions based on the process type. With the sandbox enabled a compromised sub-process will often be terminated before it can do any harm. CEF supports the Chromium sandbox on each platform as described below.

# Usage

Sandbox capabilities are OS-specific and different requirements apply to each platform.

## Windows

To enable the sandbox on Windows the following requirements must be met:

1. Use the same executable for the browser process and all sub-processes. The `CefSettings.browser_subprocess_path` setting cannot be used in combination with the sandbox.
2. Link the executable with the `cef_sandbox` static library.
3. Call the `cef_sandbox_info_create()` function from within the executable (not from a separate DLL) and pass the resulting pointer into both the` CefExecutProcess()` and `CefInitialize()` functions via the `windows_sandbox_info` parameter.


See [cef_sandbox_win.h](https://bitbucket.org/chromiumembedded/cef/raw/master/include/cef_sandbox_win.h) for additional details. See [cefsimple_win.cc](https://bitbucket.org/chromiumembedded/cef/raw/master/tests/cefsimple/cefsimple_win.cc) for an example implementation.

## macOS

To enable the sandbox on macOS the following requirements must be met:

1. Link the helper process executable with the `cef_sandbox` static library.
2. Call the `cef_sandbox_initialize()` function at the beginning of the helper executable `main()` function and before loading the CEF framework library.

See [cef_sandbox_mac.h](https://bitbucket.org/chromiumembedded/cef/raw/master/include/cef_sandbox_mac.h) and [cef_library_loader.h](https://bitbucket.org/chromiumembedded/cef/raw/master/include/wrapper/cef_library_loader.h) for additional details. See [cefsimple_mac.mm](https://bitbucket.org/chromiumembedded/cef/raw/master/tests/cefsimple/cefsimple_mac.mm) and [process_helper_mac.cc](https://bitbucket.org/chromiumembedded/cef/raw/master/tests/cefsimple/process_helper_mac.cc) for an example implementation.

The [V2 sandbox](https://chromium.googlesource.com/chromium/src/+/master/sandbox/mac/seatbelt_sandbox_design.md) was enabled starting with 3538 branch. See [this announcement](https://groups.google.com/d/msg/cef-announce/Fith0A3kWtw/6ds_mJVMCQAJ) for a description of the related changes.

## Linux

Different approaches are used based on system capabilities. See [this document](https://chromium.googlesource.com/chromium/src/+/refs/tags/57.0.2956.2/docs/linux_sandboxing.md) for details.

# Building cef_sandbox (Windows/macOS)

Linking the `cef_sandbox` static library is required, as described above, for using the sandbox on Windows and macOS. This library must be built with special configuration settings in order to link successfully with third-party applications. For example, Chromium's custom bundled libc++ version must be disabled to avoid linker conflicts when using the default platform runtime library.

The `cef_sandbox` library build is usually performed by the `automate_git.py` tool as part of the official binary distribution build (see the [AutomatedBuildSetup](https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md) Wiki page). The special sandbox-specific configuration comes from the `GetConfigArgsSandbox` function in the [gn_args.py](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/gn_args.py) tool and the sandbox build is performed by `automate_git.py` in a separate `out\[Config]_sandbox` output directory. A final post-processing action by the [make_distrib.py](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/make_distrib.py) tool combines multiple lib files into a single `cef_sandbox.lib` (Windows) or `cef_sandbox.a` (macOS) file which is included with the binary distribution.

The `cef_sandbox` library can also be built locally for development and testing purposes as described below. This example creates a build that is compatible with the official 32-bit Windows distribution available for download from [Spotify](http://opensource.spotify.com/cefbuilds/index.html). To perform this build you must first set up a local Chromium/CEF checkout as described on the [MasterBuildQuickStart](https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md) Wiki page and check out the correct branch/version by passing the appropriate `--branch=XXXX` or `--checkout=HHHHHHHH` command-line flags to `automate_git.py`. You can then build the `cef_sandbox` library as follows:

```
# Discover what `args.gn` contents are required:
cd c:\path\to\chromium\src\cef
set GN_DEFINES=is_official_build=true
tools\gn_args.py

# Create the `chromium\src\out\Release_GN_x86_sandbox\args.gn` file with the specified contents.

# Build the cef_sandbox target in the new output directory:
cd c:\path\to\chromium\src
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
gn gen out\Release_GN_x86_sandbox
ninja -C out\Release_GN_x86_sandbox cef_sandbox

# Create a binary distribution for the cef_sandbox build:
cd c:\path\to\chromium\src\cef\tools
make_distrib.bat --allow-partial --sandbox --ninja-build --no-archive --no-symbols --no-docs
```

The resulting `chromium\src\cef\binary_distrib\cef_binary_*_windows32_sandbox\Release\cef_sandbox.lib` file can then be tested in combination with an existing official binary distribution at the same version.

If a different platform or configuration is desired you can find the appropriate `GN_DEFINES` on the [AutomatedBuildSetup](https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md) Wiki page.