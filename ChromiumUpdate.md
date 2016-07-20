This Wiki page describes how to update CEF to use the newest Chromium revision.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***

The Chromium developers work very hard to introduce new features and capabilities as quickly as possible. Consequently, projects like CEF that depend on Chromium must also be updated regularly. The update process can be complicated and must be done carefully to avoid introducing new bugs and breakage. Below are the steps to follow when updating CEF to work with a new Chromium revision.

1\. Identify the commit hash for the last known compiling revision of Chromium (origin/lkcr):

```
cd /path/to/chromium/src

# Fetch the most recent Chromium sources without applying them to a branch.
git fetch

# Query the current value of origin/lkcr
git log -1 origin/lkcr
```

You can also view current build status on the [Chromium build waterfall](https://chromium-build.appspot.com/p/chromium/console).

2\. Create a diff of relevant directories between the last Chromium commit hash and new Chromium commit hash.

With CEF3 the following files should be evaluated for changes:

```
content/shell/*
content/content_shell.gypi
chrome/browser/component_updater/chrome_component_updater_configurator.cc
chrome/browser/extensions/api/tabs/tabs_api.[cc|h]
chrome/browser/extensions/chrome_component_extension_resource_manager.[cc|h]
chrome/browser/extensions/chrome_extension_web_contents_observer.[cc|h]
chrome/browser/extensions/chrome_url_request_util.[cc|h]
chrome/browser/extensions/component_loader.[cc|h]   (for libcef/browser/extensions/extension_system.cc)
chrome/browser/extensions/extension_service.[cc|h]   (for libcef/browser/extensions/extension_system.cc)
chrome/browser/guest_view/mime_handler_view/*
chrome/browser/pdf/*
chrome/browser/plugins/plugin_info_message_filter.[cc|h]
chrome/browser/printing/*
chrome/browser/renderer_host/chrome_resource_dispatcher_host_delegate.[cc|h]
chrome/browser/renderer_host/pepper/*
chrome/common/extensions/api/_api_features.json
chrome/common/extensions/api/_permission_features.json
chrome/renderer/chrome_content_renderer_client.[cc|h]
chrome/renderer/extensions/chrome_extensions_renderer_client.[cc|h]
chrome/renderer/pepper/*
chrome/renderer/printing/*
content/browser/webui/shared_resources_data_source.cc   (for libcef/browser/chrome_scheme_handler.cc)
content/browser/web_contents/web_contents_view_guest.cc   (for libcef/browser/web_contents_view_osr.cc)
extensions/shell/*
```

On Windows you can create a batch script called `diff.bat` in the directory that contains the `src` folder where `[list of paths]` is the above list of file paths excluding wildcards.

```
@echo off
cd src
call git diff --relative --no-prefix %1..%2 -- [list of paths] > ../diff_%1-%2.diff
cd ..
```

And run it as follows:
```
diff.bat OLDHASH NEWHASH
```

This will be your guide to identifying what has changed. CEF began life as a customized version of content\_shell and there's still a one-to-one relationship between many of the files.

3\. Use CEF's `patch_updater.py` tool to update the Chromium patch files in the `patch/patches` directory. Use of this tool requires the `patch` binary which is distributed with Posix systems (Linux, OS X, Cygwin on Windows).

```
cd /path/to/chromium/src/cef/tools

# Attempt to update patch files. Any merge conflicts will be highlighted in the output.
python patch_updater.py

# After manually resolving merge conflicts re-save the patch files.
python patch_updater.py --resave
```

4\. Make the necessary changes to CEF, build (clean if necessary) and fix whatever is broken.

5\. Run cef\_unittests and the various tests available via the Tests menu in cefclient to verify that everything still works.

In most cases (say, 90% of the time) any code breakage will be due to naming changes, minor code reorganization and/or project name/location changes. The remaining 10% can require pretty significant changes to CEF, usually due to the ongoing refactoring in Chromium code. If you identify a change to Chromium that has broken a required feature for CEF, and you can't work around the breakage by making reasonable changes to CEF, then you should work with the Chromium team to resolve the problem.

1\. Identify the specific Chromium revision that broke the feature and make sure you understand why the change was made.

2\. Post a message to the chromium-dev mailing list explaining why the change broke CEF and either seeking additional information or suggesting a fix that works for both CEF and Chromium.

3\. After feedback from the Chromium developers create an issue in the [Chromium issue tracker](http://crbug.com) and a [code review](http://www.chromium.org/developers/contributing-code) with the fix and publish it with the appropriate (responsible) developer(s) as reviewers.

4\. Follow through with the Chromium developer(s) to get the code review committed.

The CEF build currently contains a patch capability that should be used only as a last resort or as a stop-gap measure if you expect the code review to take a while. The best course of action is always to get your Chromium changes accepted into the Chromium trunk if possible.
