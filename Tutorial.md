This Wiki page explains how to create a simple application using CEF3.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Introduction

This tutorial explains how to create a simple application using CEF3. It references the [cefsimple example project](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/?at=master). For complete CEF3 usage information visit the [GeneralUsage](GeneralUsage.md) Wiki page.

# Getting Started

CEF provides a sample project that makes it really easy to get started with CEF development. Simply browse over to the [cef-project](https://bitbucket.org/chromiumembedded/cef-project) website and follow the step-by-step instructions. The source files linked from this tutorial are for the current CEF3 master branch and may differ slightly from the versions that are downloaded by cef-project.

## Loading a Custom URL

The cefsimple application loads google.com by default but you can change it to load a custom URL instead. The easiest way to load a different URL is via the command-line.

```
# Load the local file “c:\example\example.html”
cefsimple.exe --url=file://c:/example/example.html
```

You can also edit the source code in [cefsimple/simple\_app.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.cc?at=master) and recompile the application to load your custom URL by default.

```
// Load the local file “c:\example\example.html”
…
if (url.empty())
  url = "file://c:/example/example.html";
…
```

# Application Components

All CEF applications have the following primary components:

  1. The CEF dynamic library (libcef.dll on Windows, libcef.so on Linux, “Chromium Embedded Framework.framework” on OS X).
  1. Support files (\*.pak and \*.bin binary blobs, etc).
  1. Resources (html/js/css for built-in features, strings, etc).
  1. Client executable (cefsimple in this example).

The CEF dynamic library, support files and resources will be the same for every CEF-based application. They are included in the Debug/Release or Resources directory of the binary distribution. See the README.txt file included in the binary distribution for details on which of these files are required and which can be safely left out. See below for a detailed description of the required application layout on each platform.

# Architecture in 60 Seconds

The below list summarizes the items of primary importance for this tutorial:

  * CEF uses multiple processes. The main application process is called the “browser” process. Sub-processes will be created for renderers, plugins, GPU, etc.
  * On Windows and Linux the same executable can be used for the main process and sub-processes. On OS X you are required to create a separate executable and app bundle for sub-processes.
  * Most processes in CEF have multiple threads. CEF provides functions and interfaces for posting tasks between these various threads.
  * Some callbacks and functions may only be used in particular processes or on particular threads. Make sure you read the source code comments in the API headers before you begin using a new callback or function for the first time.

Read the [GeneralUsage](GeneralUsage.md) Wiki page for complete discussion of the above points.

# Source Code

The cefsimple application initializes CEF and creates a single popup browser window. The application terminates when all browser windows have been closed. Program flow is as follows:

  1. The OS executes the browser process entry point function (main or wWinMain).
  1. The entry point function:
    1. Creates an instance of SimpleApp which handles process-level callbacks.
    1. Initializes CEF and runs the CEF message loop.
  1. After initialization CEF calls SimpleApp::OnContextInitialized(). This method:
    1. Creates the singleton instance of SimpleHandler.
    1. Creates a browser window using CefBrowserHost::CreateBrowser().
  1. All browsers share the SimpleHandler instance which is responsible for customizing browser behavior and handling browser-related callbacks (life span, loading state, title display, etc).
  1. When a browser window is closed SimpleHandler::OnBeforeClose() is called. When all browser windows have closed the OnBeforeClose implementation quits the CEF message loop to exit the application.

Your binary distribution may include newer versions of the below files. However, the general concepts remain unchanged.

## Entry Point Function

Execution begins in the browser process entry point function. This function is responsible for initializing CEF and any OS-related objects. For example, it installs X11 error handlers on Linux and allocates the necessary Cocoa objects on OS X. OS X has a separate entry point function for helper processes.

  * Windows platform implementation: [cefsimple/cefsimple\_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/cefsimple_win.cc?at=master)
  * Linux platform implementation: [cefsimple/cefsimple\_linux.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/cefsimple_linux.cc?at=master)
  * Mac OS X platform implementation
    * For the browser process: [cefsimple/cefsimple\_mac.mm](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/cefsimple_mac.mm?at=master)
    * For sub-processes: [cefsimple/process\_helper\_mac.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/process_helper_mac.cc?at=master)

## SimpleApp

SimpleApp is responsible for handling process-level callbacks. It exposes some interfaces/methods that are shared by multiple processes and some that are only called in a particular process. The CefBrowserProcessHandler interface, for example, is only called in the browser process. There’s a separate CefRenderProcessHandler interface (not shown in this example) that is only called in the render process. Note that GetBrowserProcessHandler() must return |this| because SimpleApp implements both CefApp and CefBrowserProcessHandler. See the [GeneralUsage](GeneralUsage.md) Wiki page or API header files for additional information on CefApp and related interfaces.

  * Shared implementation: [cefsimple/simple\_app.h](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.h?at=master), [cefsimple/simple\_app.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.cc?at=master)

## SimpleHandler

SimpleHandler is responsible for handling browser-level callbacks. These callbacks are executed in the browser process. In this example we use the same CefClient instance for all browsers, but your application can use different CefClient instances as appropriate. See the [GeneralUsage](GeneralUsage.md) Wiki page or API header files for additional information on CefClient and related interfaces.

  * Shared implementation: [cefsimple/simple\_handler.h](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler.h?at=master), [cefsimple/simple\_handler.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler.cc?at=master)
  * Windows platform implementation: [cefsimple/simple\_handler\_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler_win.cc?at=master)
  * Linux platform implementation: [cefsimple/simple\_handler\_linux.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler_linux.cc?at=master)
  * Mac OS X platform implementation: [cefsimple/simple\_handler\_mac.mm](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler_mac.mm?at=master)

# Build Steps

Build steps vary depending on the platform. Explore the CMake files included with the binary distribution for a complete understanding of all required steps. The build steps common to all platforms can generally be summarized as follows:

  1. Compile the libcef\_dll\_wrapper static library.
  1. Compile the application source code files. Link against the libcef dynamic library and the libcef\_dll\_wrapper static library.
  1. Copy libraries and resources to the output directory.

## Windows Build Steps

  1. Compile the libcef\_dll\_wrapper static library.
  1. Compile/link cefsimple.exe.
    * Required source code files include: cefsimple\_win.cc, simple\_app.cc, simple\_handler.cc, simple\_handler\_win.cc.
    * Required link libraries include: comctl32.lib, shlwapi.lib, rcprt4.lib, libcef\_dll\_wrapper.lib, libcef.lib, cef\_sandbox.lib. Note that cef\_sandbox.lib (required for sandbox support) is a static library currently built with Visual Studio 2013 and it may not compile with other Visual Studio versions. See comments in cefsimple\_win.cc for how to disable sandbox support.
    * Resource file is cefsimple.rc.
    * Manifest files are cefsimple.exe.manifest and compatibility.manifest.
  1. Copy all files from the Resources directory to the output directory.
  1. Copy all files from the Debug/Release directory to the output directory.

The resulting directory structure looks like this for 2526 branch:

```
Application/
    cefsimple.exe  <= cefsimple application executable
    libcef.dll <= main CEF library
    icudtl.dat <= unicode support data
    libEGL.dll, libGLESv2.dll, ... <= accelerated compositing support libraries
    cef.pak, devtools_resources.pak, ... <= non-localized resources and strings
    natives_blob.bin, snapshot_blob.bin <= V8 initial snapshot
    locales/
        en-US.pak, ... <= locale-specific resources and strings
```

## Linux Build Steps

  1. Compile the libcef\_dll\_wrapper static library.
  1. Compile/link cefsimple.
    * Required source code files include: cefsimple\_linux.cc, simple\_app.cc, simple\_handler.cc, simple\_handler\_linux.cc.
    * Required link libraries include: libcef\_dll\_wrapper.a, libcef.so and dependencies (identified at build time using the “pkg-config” tool).
    * Configure the rpath to find libcef.so in the current directory (“-Wl,-rpath,.”) or use the LD\_LIBRARY\_PATH environment variable.
  1. Copy all files from the Resources directory to the output directory.
  1. Copy all files from the Debug/Release directory to the output directory.
  1. Set SUID permissions on the chrome-sandbox executable to support the sandbox. See binary distribution build output for the necessary command.

The resulting directory structure looks like this for 2526 branch:

```
Application/
    cefsimple <= cefsimple application executable
    chrome-sandbox <= sandbox support binary
    libcef.so <= main CEF library
    icudtl.dat <= unicode support data
    cef.pak, devtools_resources.pak, ... <= non-localized resources and strings
    natives_blob.bin, snapshot_blob.bin <= V8 initial snapshot
    locales/
        en-US.pak, ... <= locale-specific resources and strings
    files/
        binding.html, ... <= cefclient application resources
```

## Mac OS X Build Steps

  1. Compile the libcef\_dll\_wrapper static library.
  1. Compile/link/package the “cefsimple Helper” app.
    * Required source code files include: process\_helper\_mac.cc.
    * Required link frameworks include: AppKit.framework, Chromium Embedded Framework.framework (unversioned, included in the binary distribution).
    * App bundle configuration is provided via “cefsimple/mac/helper-Info.plist”.
    * Use “install\_name\_tool -change” to rewrite the framework link so that it points to  the “Chromium Embedded Framework.framework/Chromium Embedded Framework” library in the correct location.
  1. Compile/link/package the “cefsimple” app.
    * Required source code files include: cefsimple\_mac.mm, simple\_app.cc, simple\_handler.cc, simple\_handler\_mac.mm.
    * Required link frameworks include: AppKit.framework, Chromium Embedded Framework.framework (unversioned, included in the binary distribution).
    * App bundle configuration is provided via “cefsimple/mac/Info.plist”.
    * Use “install\_name\_tool -change” to rewrite the framework link so that it points to  the “Chromium Embedded Framework.framework/Chromium Embedded Framework” library in the correct location.
  1. Create a Contents/Frameworks directory in the cefsimple.app bundle. Copy the following files to that directory: “cefsimple Helper.app”, “Chromium Embedded Framework.framework”.

The resulting directory structure looks like this for 2526 branch:

```
cefsimple.app/
    Contents/
        Frameworks/
            Chromium Embedded Framework.framework/
                Chromium Embedded Framework <= main application library
                Resources/
                    cef.pak, devtools_resources.pak, ... <= non-localized resources and strings
                    icudtl.dat <= unicode support data
                    natives_blob.bin, snapshot_blob.bin <= V8 initial snapshot
                    en.lproj/, ... <= locale-specific resources and strings
            cefsimple Helper.app/
                Contents/
                    Info.plist
                    MacOS/
                        cefsimple Helper <= helper executable
                    Pkginfo
        Info.plist
        MacOS/
            cefsimple <= cefsimple application executable
        Pkginfo
        Resources/
            cefsimple.icns, ... <= cefsimple application resources
```
