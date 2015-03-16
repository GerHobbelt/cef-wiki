This Wiki page provides an overview of the CEF architecture.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Background

The Chromium Embedded Framework (CEF) is an open source project founded by Marshall Greenblatt in 2008 to develop a Web browser control based on the [Google Chromium](http://www.chromium.org/Home) project. CEF currently supports a range of programming languages and operating systems and can be easily integrated into both new and existing applications. It was designed from the ground up with both performance and ease of use in mind. The base framework includes C and C++ programming interfaces exposed via native libraries that insulate the host application from Chromium and WebKit implementation details. It provides close integration between the browser control and the host application including support for custom plugins, protocols, JavaScript objects and JavaScript extensions. The host application can optionally control resource loading, navigation, context menus, printing and more, while taking advantage of the same performance and HTML5 technologies available in the Google Chrome Web browser.

# Dependencies

The CEF project depends on a number of other projects maintained by third parties. The major projects that CEF depends on are:

  * [Chromium](http://www.chromium.org/Home) - Provides general functionality like network stack, threading, message loop, logging and process control needed to create a fully functional Web browser. Implements "platform" code that allows WebKit to communicate with V8 and Skia. Many Chromium design documents can be found at http://dev.chromium.org/developers.
  * [WebKit](http://www.webkit.org/) - Provides DOM parsing, layout, event handling, rendering and HTML5 JavaScript APIs. Some HTML5 implementations are split between the WebKit and Chromium code bases.
  * [V8](http://code.google.com/p/v8/) - JavaScript engine.
  * [Skia](http://code.google.com/p/skia/) - 2D graphics library used for rendering non-accelerated content. More information on how Chromium integrates Skia is available [here](http://www.chromium.org/developers/design-documents/graphics-and-skia).
  * [Angle](http://code.google.com/p/angleproject/) - 3D graphics translation layer for Windows that converts GLES calls to DirectX. More information about accelerated compositing is available [here](http://dev.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome).

# Versions

There have been three versions of CEF to date.

  * CEF1 - Single process implementation using the [Chromium WebKit API](http://trac.webkit.org/wiki/Chromium).
  * CEF2 - Multi process implementation built on the Chromium browser.
  * CEF3 - Multi process implementation using the [Chromium Content API](http://www.chromium.org/developers/content-module/content-api).

# Common API Usage

All versions of CEF expose a simple, easy-to-use API designed to insulate users from the underlying Chromium and WebKit code complexity. Common usage is as follows:

1. Initialize CEF by calling CefInitialize().

2. Perform work on the UI message loop by calling CefRunMessageLoop() or CefDoMessageLoopWork().

3. Create a browser window by calling CreateBrowser() or CreateBrowserSync() and passing in a CefClient instance.

4. Shut down CEF by calling CefShutdown() before the application exits.

# C Wrapper API

The libcef shared library exports a C API that isolates the user from the CEF runtime and code base. The libcef\_dll\_wrapper project, which is distributed in source code form as part of the binary release, wraps this exported C API in a C++ API that is then linked into the client application. The code for this C/C++ API translation layer is automatically generated by the [translator](https://bitbucket.org/chromiumembedded/cef/src/master/tools/translator.README.txt?at=master) tool. Direct usage of the C API is described on the [UsingTheCAPI](UsingTheCAPI.md) Wiki page.

# CEF1

The single process architecture used by CEF1 integrates Chromium and WebKit directly into the client application. Advantages to the single process arcitecture include reduced memory usage and closer integration with the client application. Disadvantages include [reduced performance](https://bitbucket.org/chromiumembedded/cef/issue/304) with certain types of accelerated content and [crashes](https://bitbucket.org/chromiumembedded/cef/issue/242) due to plugins like Flash running in the same process.

## API Usage

Below is an overview of the main CEF1 interfaces and their uses. For more information on specific supported features and interfaces visit the GeneralUsage Wiki page or browse the CEF1 include files.

  * [CefApp](http://magpcss.org/ceforum/apidocs/projects/(default)/CefApp.html) - This interface is passed to CefInitialize() and allows the application to customize global functionality like resource loading and proxy handling.
  * [CefClient](http://magpcss.org/ceforum/apidocs/projects/(default)/CefClient.html) - This interface is passed to CefCreateBrowser() or CefCreateBrowserSync() and acts as the connection between an individual CEF browser instance and the client application. Additional interfaces for request handling, display handling, etc. are exposed via this interface.
  * [CefBrowser](http://magpcss.org/ceforum/apidocs/projects/(default)/CefBrowser.html) - Exposes capabilities provided by the browser. This includes back/forward navigation, source retrieval, request loading, etc. A single CefBrowser may have one or more child [CefFrame](http://magpcss.org/ceforum/apidocs/projects/(default)/CefRrame.html) objects.

## Threading Considerations

CEF1 includes UI, IO and FILE threads. The UI thread creates browser windows and is used for all interaction with WebKit and V8. The IO thread is used for handling schema and network requests. The FILE thread is used for the application cache and other miscellaneous activities.

When using CEF1 you should keep the following threading considerations in mind:

  * Do not perform blocking operations on the UI thread. This can lead to serious performance issues.
  * The UI thread will be the same as the main application thread if CefInitialize() is called with a CefSettings.multi\_threaded\_message\_loop value of false.
  * All WebKit and V8 interation must take place on the UI thread. Consequently, some CEF API functions can only be called on the UI thread. Functions that have this limitation will be documented accordingly in the associated CEF header file.
  * The [CefPostTask](http://magpcss.org/ceforum/apidocs/projects/(default)/(_globals).html#CefPostTask(CefThreadId,CefRefPtr<CefTask>)) method can be used to post tasks asnychronously between the various threads.

## Implementation Details

CEF1 has the following major implementation classes:

  * CefContext - Represents the global CEF context. A single CefContext object is created by CefInitialize() and destroyed by CefShutdown().
  * CefProcess - Used by CefContext to create and manage the CEF threads.
  * BrowserWebKitInit - Manages the global WebKit environment as exposed by the [Chromium WebKit API](http://trac.webkit.org/wiki/Chromium).
  * WebViewHost - Provides the native window "wrapper" implementation for a WebView. This class extends WebWidgetHost which provides functionality shared in common with popup widgets (like select menus) on some platforms.
  * CefBrowserImpl - Implements the CefBrowser interface, creates the WebViewHost and provides the glue code for a single browser instance.
  * BrowserWebViewDelegate - Implements the WebKit interfaces that provide communication between CefBrowserImpl and the underlying WebView.


# CEF2

CEF2 was discontinued when Chromium announced support for the Content API. Information about CEF2 can be found [here](http://magpcss.org/ceforum/viewtopic.php?f=10&t=122).


# CEF3

CEF3 uses the same multi process architecture as the Chromium Web browser via the [Chromium Content API](http://www.chromium.org/developers/content-module/content-api). This architecture provides a number of advantages over the single process architecture used by CEF1:

  * Support for both single-process and multi-process run modes.
  * More code sharing with the Chromium browser.
  * Improved performance and less breakage due to use of the "supported" code path.
  * Faster access to new features.

In most cases CEF3 will have the same performance and stability characteristics as the Chromium Web browser. Information about CEF3 development status can be found [here](http://magpcss.org/ceforum/viewtopic.php?f=10&t=645).

## API Usage

CEF3 initialization needs to account for multiple processes. On Windows and Linux all processes may share the same executable or optionally use a separate executable for sub-processes (via the CefSettings.browser\_subprocess\_path value). On Mac OS-X all sub-processes must be launched using a separate app bundle. This is necessary because on Mac OS-X the sub-processes must have a different Info.plist file.

The CefExecuteProcess() function is responsible for executing sub-process logic. It examines the "type" command-line flag to determine the type of process currently being executed. If called for the browser process (identified by no "type" command-line value)
it will return immediately with a value of -1. If called for a recognized secondary process it will block until the process should exit and then return the process exit code. If executing the browser process the application should continue with standard process initialization as described in the "Common API Usage" section. See [cef\_app.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/cef_app.h?at=master) for a complete description of initialization functions and their usage. See the [cefclient\_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefclient/cefclient_win.cc?at=masterp) wWinMain function for an example implementation on Windows.

The "--single-process" command-line flag or CefSettings.single\_process value can be used to run CEF3 in a single process. This is useful for debugging purposes but is not recommended for production use.

Below is an overview of the main CEF3 interfaces and their uses. For more information on specific supported features and interfaces visit the GeneralUsage Wiki page or browse the CEF3 include files.

  * [CefApp](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefApp.html) - This interface is passed to CefExecuteProcess() and allows the application to customize global functionality like resource loading and proxy handling. Some functionality is shared by all processes, some must be implemented in the browser process and some must be implemented in the render process. See comments in the header file for details.
  * [CefClient](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefClient.html) - This interface is passed to CefCreateBrowser() or CefCreateBrowserSync() in the browser process and acts as the connection between an individual CEF browser instance and the client application in the browser process. Additional interfaces for request handling, display handling, etc. are exposed via this interface.
  * [CefBrowser](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefBrowser.html) - Exposes capabilities provided by the browser in both the browser and render processes. This includes back/forward navigation, source retrieval and resource loading. A single CefBrowser may have one or more child [CefFrame](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefFrame.html) objects. Some methods must be called in a specific process or on a particular thread so read the documentation carefully.
  * [CefBrowserHost](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefBrowserHost.html) - Exposes functionality related to hosting the browser window that is only available in the browser process. For example, retrieving the native parent window handle or destroying the browser window.
  * [CefRenderProcessHandler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefRenderProcessHandler.html) - Exposes WebKit and V8 integration capabilities to the application in the render process. An instance of this object is returned via CefApp.

## Process Considerations

CEF3 uses a number of processes for different purposes.

  * Browser process - This process is the same as the client application process that calls CefInitialize().
  * Render process - Web content (WebKit and V8) is hosted in a this process.
  * Plugin process - Plugins like Flash are hosted in this process. More information about the plugin architecture is available [here](http://www.chromium.org/developers/design-documents/plugin-architecture).
  * GPU process - Accelerated content is rendered in this process. More information about accelerated compositing is available [here](http://dev.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome).
  * Utility process - Some miscellaneous tasks are executed in this process.

The Chromium process model as a whole is described in detail [here](http://www.chromium.org/developers/design-documents/multi-process-architecture) and [here](http://www.chromium.org/developers/design-documents/process-models).

The various CEF processes communicate using [inter-process communication](http://www.chromium.org/developers/design-documents/inter-process-communication). CefBrowser and CefFrame exist in both the browser and render processes and are passed as arguments to various callbacks. A [CefProcessMessage](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefProcessMessage.html) can be sent from and delivered to a particular browser in a particular process using the CefBrowser::SendProcessMessage function and the OnProcessMessageRecieved callback in CefClient (browser process) and CefRenderProcessHandler (render process). See the "JavaScript Binding" example in the cefclient application for sample usage.

XMLHttpRequest (XHR), WebSockets or similar can also be used for communication between processes. For example, an XHR request can be sent from JavaScript running in the render process to a resource handler implemented in the browser process via [CefRequestHandler::GetResourceHandler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefRequestHandler.html) or a [custom scheme handler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefSchemeHandler.html).

## Threading Considerations

CEF1 includes a number of threads as described in the [cef\_types.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/internal/cef_types.h?at=master) cef\_thread\_id enumeration. When using these threads pay close attention to how they are intended to be used and in what process they're supported as documented in that header file.

When using CEF3 you should keep the following threading considerations in mind:

  * Do not perform blocking operations on any thread other than the browser process FILE thread. This can lead to serious performance issues.
  * The UI thread in the browser process will be the same as the main application thread if CefInitialize() is called with a CefSettings.multi\_threaded\_message\_loop value of false.
  * All WebKit and V8 interation must take place on the render process RENDERER thread.
  * The [CefPostTask](https://bitbucket.org/chromiumembedded/cef/src/master/include/cef_task.h?at=master) method can be used to post tasks asnychronously between different threads in the same process.

## Implementation Details

CEF3 has the following major implementation classes:

  * [CefMainDelegate](https://bitbucket.org/chromiumembedded/cef/src/libcef/common/main_delegate.h?at=master) - Implements the common process bootstrap logic.
  * [CefContentClient](https://bitbucket.org/chromiumembedded/cef/src/libcef/common/content_client.h?at=master) - Implements the Content API callbacks that are common to all processes.
  * [CefContext](https://bitbucket.org/chromiumembedded/cef/src/libcef/libcef/browser/context.h?at=master) - Represents the global CEF context in the browser process. A single CefContext object is created by CefInitialize() and destroyed by CefShutdown().
  * [CefBrowserMainParts](https://bitbucket.org/chromiumembedded/cef/src/libcef/browser/browser_main.h?at=master) - Implements the browser process bootstrap logic.
  * [CefContentBrowserClient](https://bitbucket.org/chromiumembedded/cef/src/libcef/browser/content_browser_client.h?at=master) - Implements the Content API callbacks for the browser process.
  * [CefBrowserHostImpl](https://bitbucket.org/chromiumembedded/cef/src/libcef/browser/browser_host_impl.h?at=master) - Implements the CefBrowser and CefBrowserHost interfaces in the browser process. Provides the glue code and implements interfaces for communicating with the RenderViewHost.
  * [CefContentRendererClient](https://bitbucket.org/chromiumembedded/cef/src/libcef/renderer/content_renderer_client.h?at=master) - Implements the Content API callbacks for the render process.
  * [CefBrowserImpl](https://bitbucket.org/chromiumembedded/cef/src/libcef/renderer/browser_impl.h?at=master) - Implements the CefBrowser interface in the render process. Provides the glue code and implements interfaces for communicating with the RenderView.