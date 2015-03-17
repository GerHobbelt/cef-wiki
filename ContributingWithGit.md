This Wiki page provides information about using Git to contribute code changes to CEF.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

The CEF project uses the Git source code management system hosted via Bitbucket. The easiest way to contribute changes to CEF is by creating your own personal fork of the CEF Git repository and submitting pull requests with your proposed modifications.

# Initial Setup

Git can maintain your changes both locally and on a remote server. To work with Git efficiently the remote server locations must be configured properly.

1\. Log into Bitbucket and create a forked version of the cef.git repository: https://bitbucket.org/chromiumembedded/cef/fork.

2\. Check out CEF/Chromium source code as described on the [BranchesAndBuilding](BranchesAndBuilding.md) Wiki page.

3\. Change the remote origin of the CEF checkout so that it points to your personal forked repository. This is the remote server location that the `git push` and `git pull` commands will operate on by default.

```
cd /path/to/chromium/src/cef

# Replace <UserName> with your Bitbucket user name.
git remote set-url origin https://<UserName>@bitbucket.org/<UserName>/cef.git
```

4\. Set the remote upstream of the CEF checkout so that you can merge changes from the main CEF repository.

```
git remote add upstream https://chromiumembedded@bitbucket.org/chromiumembedded/cef.git
```

5\. Verify that the remotes are configured correctly.

```
git remote -v
```

You should see output like the following:

```
origin  https://<UserName>@bitbucket.org/<UserName>/cef.git (fetch)
origin  https://<UserName>@bitbucket.org/<UserName>/cef.git (push)
upstream        https://chromiumembedded@bitbucket.org/chromiumembedded/cef.git (fetch)
upstream        https://chromiumembedded@bitbucket.org/chromiumembedded/cef.git (push)
```

# Development

You can now commit changes to your personal repository and merge upstream changes from the main CEF repository.

## Commit Private Changes

Create a new personal branch for your changes.

```
cd /path/to/chromium/src/cef

# Start with the branch that your changes will be based upon.
git checkout master

# Create a new personal branch for your changes.
# Replace <BranchName> with your new branch name.
git checkout -b <BranchName>
```

Make local modifications and commit them to your personal branch.

```
# For example, add a specified file by path.
git add path/to/file.txt

# For example, add all existing files that have been modified or deleted.
git add -u

# Commit the modifications locally.
git commit -m "A good description of the fix (issue #1234)"

# Push the modifications to your personal remote repository.
git push origin <BranchName>
```

You can also modify an existing commit if you need to make additional changes.

```
# For example, add all existing files that have been modified or deleted.
git add -u

# Update the current HEAD commit with the changes.
git commit --amend

# Push the modifications to your personal remote repository.
# Using the `--force` argument is not recommended if multiple people are sharing the same branch.
git push origin <BranchName> --force
```

Once you no longer need a branch you can delete it both locally and remotely.

```
# Delete the branch locally.
git branch -D <BranchName>

# Delete the branch remotely.
git push origin --delete <BranchName>
```

## Merge Upstream Changes

The main CEF repository will receive additional commits over time. You will want to include these changes in your personal repository.

```
cd /path/to/chromium/src/cef

# Fetch changes from the main CEF repository. This does not apply them to any particular branch.
git fetch upstream

# Check out the branch that you want to update.
# Replace <BranchName> with the name of your branch.
git checkout <BranchName>

# Merge commits from the main CEF repository branch into your local branch.
# Replace "master" with a different branch name as appropriate (e.g. "2171", "2272", etc).
git merge upstream/master

# Push the modifications to your personal remote repository
git push origin <BranchName>
```

# Pull Requests

Once your personal changes are complete you can request that they be merged into the main CEF repository. This is done using a pull request. You can submit a pull request via the "Create pull request" option in Bitbucket's left-hand navigation bar. In order to be accepted pull requests should:

* Be submitted against the current [CEF master branch](https://bitbucket.org/chromiumembedded/cef/src/?at=master) unless explicitly fixing a bug in a CEF release branch.
* Be related to an issue in the [CEF Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) and reference that issue in the commit description.
* Follow the style of existing CEF source files. In general CEF uses the [Chromium coding style](http://www.chromium.org/developers/coding-style).
* Include new or modified unit tests as appropriate to the functionality.
* Not include unnecessary or unrelated changes.

Your pull request will be reviewed by one or more CEF developers. Please address any comments and update the branch on which your pull request is based (once pushed to your personal repository the change will be [automatically reflected](https://blog.bitbucket.org/2014/04/22/bitbucket-now-auto-updates-pull-requests/) in the pull request). Once your changes are deemed acceptable they will be merged into the main CEF repository.

Detailed information about working with Bitbucket pull requests is available [here](https://confluence.atlassian.com/display/BITBUCKET/Work+with+pull+requests).

