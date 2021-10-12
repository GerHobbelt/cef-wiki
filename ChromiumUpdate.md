This Wiki page describes how to update CEF to use the newest Chromium revision.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

The Chromium developers work very hard to introduce new features and capabilities as quickly as possible. Consequently, projects like CEF that depend on Chromium must also be updated regularly. The update process can be complicated and must be done carefully to avoid introducing new bugs and breakage. Below are the steps to follow when updating CEF to work with a new Chromium revision.

# Recommended workflow

## 1\. Setup a local developer checkout

Setup a local developer checkout of the CEF/Chromium master branch as described on the [MasterBuildQuickStart](https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md) Wiki page. The standard `automate-git.py` parameters discussed on that Wiki page are represented by the `[...]` placeholder in the below examples.

## 2\. Start the update

### A\. Identify the target Chromium version

Start the update by running `automate-git.py` with the following parameters:

```
python automate-git.py [...] --log-chromium-changes --no-build --fast-update --chromium-channel=canary
```

You will see output like the following:

```
--> Computed Chromium update for win canary at distance 0
--> Compat:  2018-05-09 03:16:29 UTC refs/tags/68.0.3425.0 695380fc6b1f9aee91d4b3933000c8a7da1d25f8 (#557062)
--> Target:  2018-05-20 03:14:11 UTC refs/tags/68.0.3436.0 11cc97646015a81933d79d43bc335351452460e6 (#560157)
--> Channel: 2018-05-20 03:14:11 UTC refs/tags/68.0.3436.0 11cc97646015a81933d79d43bc335351452460e6 (#560157)
```

* *Compat* is the Chromium version that is currently compatible with CEF as listed in the [CHROMIUM_BUILD_COMPATIBILITY.txt](https://bitbucket.org/chromiumembedded/cef/src/master/CHROMIUM_BUILD_COMPATIBILITY.txt) file
* *Target* is the Chromium version that has been selected for this update.
* *Channel* is newest available version from the Chromium canary channel as listed at https://omahaproxy.appspot.com/.

**If you would prefer to manually pick the Chromium version see the "Choosing the Chromium version manually" section below. This is useful when updating to a specific Chromium milestone version, for example.**

### B\. Run CEF's `patch_updater.py` script

This will update the Chromium patch files in the `patch/patches` directory. If patch files cannot be updated automatically the `automate-git.py` script will fail.

**If one or more patch files have unresolved conflicts you will get a failure like the following:**

```
--------------------------------------------------------------------------------
!!!! FAILED PATCHES, fix manually and run with --resave
content_2015:
  Hunk #4 FAILED at 4220.
  1 out of 5 hunks FAILED -- saving rejects to file content/browser/frame_host/render_frame_host_impl.cc.rej
storage_partition_1973:
  Hunk #1 FAILED at 167.
  1 out of 1 hunk FAILED -- saving rejects to file content/browser/shared_worker/shared_worker_service_impl.cc.rej
--------------------------------------------------------------------------------
```

Use a text editor to manually fix the specified files, and then re-run the `patch_updater.py` script:

```
$ cd /path/to/chromium/src/cef/tools
$ python patch_updater.py
```

**If a patched file is missing (moved or deleted) you will get a failure like the following:**

```
--> Reading patch file /path/to/chromium/src/cef/patch/patches/content_mojo_3123.patch
--> Skipping non-existing file /path/to/chromium/src/content/public/browser/document_service_base.h
--> Applying patch to /path/to/chromium/src
(Stripping trailing CRs from patch; use --binary to disable.)
can't find file to patch at input line 5
...
Skip this patch? [y] 
Skipping patch.
2 out of 2 hunks ignored
--> Saving changes to /path/to/chromium/src/cef/patch/patches/content_mojo_3123.patch
Traceback (most recent call last):
  File "/path/to/chromium/src/cef/tools/patch_updater.py", line 294, in <module>
    raise Exception('Failed to add paths: %s' % result['err'])
Exception: Failed to add paths: fatal: pathspec '/path/to/chromium/src/content/public/browser/document_service_base.h' did not match any files

```

You can use this Git command to discover what happened to the missing Chromium file:

```
> cd /path/to/chromium/src
> git log --full-history -1 -- content/public/browser/document_service_base.h
```

Once you know the offending Git commit hash you can use the `git show <hash>` command or load `https://crrev.com/<hash>` in a web browser to see the contents of the change. Edit the patch file manually in a text editor to fix the paths and then re-run the `patch_updater.py` script.

**To create a new patch file use this command:**

```
$ cd /path/to/chromium/src
$ git diff --no-prefix --relative [path1] [path2] > [name].patch
```

Copy the resulting `[name].patch` file to the `src/cef/patch/patches` directory and add an appropriate entry to the `src/cef/patch/patch.cfg` file.

**To add additional files to an existing patch file use this command:**

```
$ cd /path/to/chromium/src/cef/tools
$ python patch_updater.py --resave --patch [name] --add [path1] --add [path2]
```

All paths are relative to the `src` directory by default.

**After all patch files have been fixed you can re-run `automate-git.py` with the additional `--resave` command-line flag to resave the patch files and continue the update process:**

```
python automate-git.py [...] --log-chromium-changes --no-build --fast-update --chromium-channel=canary --resave
```

### C\. Identify potentially relevant Chromium changes

This step creates a `chromium_update_changes.diff` file in your download directory that will act as your guide when updating the CEF source code. CEF began life as a customized version of content\_shell and there's still a one-to-one relationship between many of the files. The list of relevant paths is taken from CEF's [CHROMIUM_UPDATE.txt](https://bitbucket.org/chromiumembedded/cef/src/master/CHROMIUM_UPDATE.txt) file.

### D\. Identify problematic patterns in the Chromium source code

This step looks for patterns in Chromium src files that may cause issues for CEF. If these patterns are found then the `automate-git.py` script will create a `chromium_update_patterns.txt` file in your download directory and fail with output like the following:

```
Evaluating pattern: static_cast<StoragePartitionImpl\*>(
ERROR Matches found. See chromium_update_changes.diff for output.
```

In that case the contents of `chromium_update_patterns.txt` might look like this:

```
!!!! WARNING: FOUND PATTERN: static_cast<StoragePartitionImpl\*>(
New instances in non-test files should be converted to call StoragePartition methods.
See storage_partition_1973.patch.

content/browser/loader/navigation_url_loader_impl.cc:1295:  auto* partition = static_cast<StoragePartitionImpl*>(storage_partition);
content/browser/renderer_interface_binders.cc:189:        static_cast<StoragePartitionImpl*>(host->GetStoragePartition())
```

Use the process described in part B to fix these failures.

## 3\. Build CEF and fix whatever else is broken

The diff file created in step C above will assist you in this process. Refer to the [MasterBuildQuickStart](https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md) Wiki page for build instructions. If Chromium changes are required refer to the "Resolving Chromium breakage" section below.

## 4\. Run CEF tests

Run `ceftests` and the the various tests available via the Tests menu in `cefclient` to verify that everything still works. If any particular subsystem had significant changes (like printing, for example) then make sure to give that subsystem additional testing.

## 5\. Update the compatible Chromium version

Update the compatible Chromium version listed in CEF's [CHROMIUM_BUILD_COMPATIBILITY.txt](https://bitbucket.org/chromiumembedded/cef/src/master/CHROMIUM_BUILD_COMPATIBILITY.txt) file.

**WARNING: Remove the `--chromium-channel` switches from your `automate-git.py` command-line at this time to avoid accidentally performing another Chromium update. **

## 6\. Create and submit a PR

Create a commit with your changes and submit a PR as described on the [ContributingWithGit](https://bitbucket.org/chromiumembedded/cef/wiki/ContributingWithGit.md) Wiki page. See the [CEF commit history](https://bitbucket.org/chromiumembedded/cef/commits/branch/master) for examples of the expected commit message format.

## 7\. Test your changes on other supported platforms

You can update, build and run tests automatically on those platforms with the following command:

```
python automate-git.py [...] --run-tests --build-failure-limit=1000 --fast-update --force-cef-update
```

If running in headless mode on Linux install the `xvfb` package and add the `--test-prefix=xvfb-run --test-args="--no-sandbox"` command-line flags.

Access to automated builders as described on the [AutomatedBuildSetup](https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md) Wiki page is recommended for this step.

# Resolving Chromium breakage

In most cases (say, 90% of the time) any code breakage will be due to naming changes, minor code reorganization and/or project name/location changes. The remaining 10% can require pretty significant changes to CEF, usually due to the ongoing refactoring in Chromium code. If you identify a change to Chromium that has broken a required feature for CEF, and you can't work around the breakage by making reasonable changes to CEF, then you should work with the Chromium team to resolve the problem.

1\. Identify the specific Chromium revision that broke the feature and make sure you understand why the change was made.

2\. Post a message to the chromium-dev mailing list explaining why the change broke CEF and either seeking additional information or suggesting a fix that works for both CEF and Chromium.

3\. After feedback from the Chromium developers create an issue in the [Chromium issue tracker](https://crbug.com) and a [code review](http://www.chromium.org/developers/contributing-code) with the fix and publish it with the appropriate (responsible) developer(s) as reviewers.

4\. Follow through with the Chromium developer(s) to get the code review committed.

The CEF build currently contains a patch capability that should be used only as a last resort or as a stop-gap measure if you expect the code review to take a while. The best course of action is always to get your Chromium changes accepted into the Chromium trunk if possible.

# Choosing the Chromium version manually

You can also choose a Chromium version or commit hash manually. It's best to choose a version that is likely to build successfully on most platforms.

* Current Chromium channel version information can be found at https://omahaproxy.appspot.com/
* Good choices for master updates include the canary channel "current_version" value or the value from the most recent branch announcement email sent to the chromium-dev mailing list. For example, [this email](https://groups.google.com/a/chromium.org/d/msg/chromium-dev/ieGUR8J2JXM/t0xL98wkFAAJ) was sent for the M73 branch. These versions should have 0 as the last component (e.g. "73.0.3683.0").
* Good choices for release branch updates include the most recent "current_version" value listed for that branch. These versions should not have 0 as the last component (e.g. "73.0.3683.75").

To manually specify the Chromium version and update the checkout of master:

```
python automate-git.py [...] --no-build --fast-update --log-chromium-changes --chromium-checkout=refs/tags/73.0.3683.0
```

To manually specify the Chromium version and update the checkout of an existing release branch:

```
python automate-git.py [...] --no-build --fast-update --force-cef-update --branch=3683 --chromium-checkout=refs/tags/73.0.3683.75
```