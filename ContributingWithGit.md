This Wiki page provides information about using Git to contribute code changes to CEF.

**Note to Editors: Changes made to this Wiki page without prior approval via the [CEF Forum](http://magpcss.org/ceforum/) or[ Issue Tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) may be lost or reverted.**

***
[TOC]
***

# Overview

The CEF project uses the Git source code management system hosted via Bitbucket. The easiest way to contribute changes to CEF is by creating your own personal fork of the CEF Git repository and submitting pull requests with your proposed modifications. This document is intended as a quick-start guide for developers who already have some familiarity with Git. If you are completely new to Git you might want to review the series of [Git tutorials](https://www.atlassian.com/git/tutorials/) provided by Bitbucket.

# Initial Setup

Git can maintain your changes both locally and on a remote server. To work with Git efficiently the remote server locations must be configured properly.

1\. Log into Bitbucket and create a forked version of the cef.git repository: https://bitbucket.org/chromiumembedded/cef/fork.

2\. Check out CEF/Chromium source code as described on the [MasterBuildQuickStart](MasterBuildQuickStart.md) or [BranchesAndBuilding](BranchesAndBuilding.md) Wiki page. Note that if you use the automate-git.py tool you will want to back up your customized CEF checkout before changing branches or performing a clean build.

3\. Change the remote origin of the CEF checkout so that it points to your personal forked repository. This is the remote server location that the `git push` and `git pull` commands will operate on by default.

```
cd /path/to/chromium/src/cef

# Replace <UserName> with your Bitbucket user name.
git remote set-url origin https://<UserName>@bitbucket.org/<UserName>/cef.git
```

4\. Set the remote upstream of the CEF checkout so that you can merge changes from the main CEF repository.

```
git remote add upstream https://bitbucket.org/chromiumembedded/cef.git
```

5\. Verify that the remotes are configured correctly.

```
git remote -v
```

You should see output like the following:

```
origin  https://<UserName>@bitbucket.org/<UserName>/cef.git (fetch)
origin  https://<UserName>@bitbucket.org/<UserName>/cef.git (push)
upstream        https://bitbucket.org/chromiumembedded/cef.git (fetch)
upstream        https://bitbucket.org/chromiumembedded/cef.git (push)
```

6\. Configure your name and email address.

```
git config user.name "User Name"
git config user.email user@example.com
```

7\. Configure the correct handling of line endings in the repository.

```
# Use this value on Windows. Files will be converted to CRLF line endings
# in the working directory and LF line endings in the object database.
git config core.autocrlf true

# Use this value on other platforms. Files will be unchanged in the working
# directory and converted to LF line endings in the object database.
git config core.autocrlf input

# Cause Git to abort actions on files with mixed line endings if the change is
# not reversible (e.g. changes to binary files that are accidentally classified
# as text).
git config core.safecrlf true
```

# Working With Private Changes

You can now commit changes to your personal repository and merge upstream changes from the main CEF repository. To facilitate creation of a pull request or the sharing of your code changes with other developers you should make your changes in a branch.

## Creating a Branch

Create a new personal branch for your changes.

```
# Start with the branch that your changes will be based upon.
git checkout master

# Create a new personal branch for your changes.
# Replace <BranchName> with your new branch name.
git checkout -b <BranchName>
```

## Creating a Commit

After making local modifications you can commit them to your personal branch.

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

## Modifying a Commit

You can also modify an existing commit if you need to make additional changes.

```
# For example, add all existing files that have been modified or deleted.
git add -u

# Update the current HEAD commit with the changes.
git commit --amend

# Push the modifications to your personal remote repository.
# Using the `--force` argument is not recommended if multiple people are sharing the
# same branch.
git push origin <BranchName> --force
```

## Rebasing on Upstream Changes

The main CEF repository will receive additional commits over time. You will want to include these changes in your personal repository. To keep Git history correct (showing upstream CEF commits on the CEF branch instead of on your personal branch) you will need to rebase the local CEF branch before rebasing your local personal branch.

```
# Fetch changes from the main CEF repository. This does not apply them to any
# particular branch.
git fetch upstream

# Check out the local CEF branch that tracks the upstream CEF branch.
# Replace "master" with a different branch name as appropriate (e.g. "2171", "2272", etc).
git checkout master

# Rebase your local CEF branch on top of the upstream CEF branch.
# After this command your local CEF branch should be identical to the upstream CEF branch.
git rebase upstream/master

# Check out the personal branch that you want to update with changes from the CEF branch.
# Replace <BranchName> with the name of your branch.
git checkout <BranchName>

# Rebase your personal branch on top of the local CEF branch.
# After this command your local commits will come after all CEF commits on the same branch.
git rebase master

# Push the modifications to your personal remote repository.
git push origin <BranchName>
```

You may get merge conflicts if your personal changes conflict with changes made to the main CEF respository. For instructions on resolving merge conflicts see [this articicle](https://help.github.com/articles/resolving-merge-conflicts-after-a-git-rebase/).

For more information on using the rebase command go [here](https://www.atlassian.com/git/tutorials/merging-vs-rebasing).

## Deleting a Branch

Once you no longer need a branch you can delete it both locally and remotely. Do not delete branches that are associated with open pull requests.

```
# Delete the branch locally.
git branch -D <BranchName>

# Delete the branch remotely.
git push origin --delete <BranchName>
```

## Cleaning a Checkout

You can remove all local changes from your checkout using the below commands.

```
# Check the current state of the repository before deleting anything.
git status

# Remove all non-committed files and directories from the local checkout.
# If you run this command with JCEF it will also remove all third_party directories and you will
# need to re-run the `gclient runhooks` command.
git clean -dffx

# Remove all local commits from the current branch and reset branch state to match 
# origin/master. Replace "origin/master" with a different remote branch name as appropriate.
git reset --hard origin/master
```

# Working With Pull Requests

Once your personal changes are complete you can request that they be merged into the main CEF (or JCEF) repository. This is done using a pull request. Before submitting a pull request you should:

* Rebase your changes on the upstream CEF (or JCEF) branch (see "Rebasing on Upstream Changes").
* Squash your changes into a single commit (see "Squashing Commits").
* Find or create an appropriate issue in the [CEF issue tracker](https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open) (or [JCEF issue tracker](https://bitbucket.org/chromiumembedded/java-cef/issues?status=new&status=open) if the change targets that project). Make sure the issue number is referenced in your commit description.
* Follow the style of existing CEF (or JCEF) source files. In general CEF uses the [Chromium coding style](http://www.chromium.org/developers/coding-style).
* Include new or modified unit tests as appropriate to the functionality.
* Remove unnecessary or unrelated changes.

## Squashing Commits

Before creating a pull request you should squash, or combine, all of the commits on your personal branch into a single commit that includes all of your changes. This will make it easier for the CEF developers to review your changes. Note that this is a destructive process so you might want to use a new branch just in case you make a mistake.

1\. Create a new personal branch that will be squashed.

```
# Start with the existing personal branch that contains all of your commits.
git checkout <ExistingBranchName>

# Create a new personal branch that will be squashed.
git checkout -b <BranchName>
```

2\. Identify the first commit where your personal branch diverged from an existing CEF branch.

```
# Replace <BranchName> with your new branch name.
# Replace "master" with a different CEF branch as appropriate (e.g. "2272", "2171", etc).
git merge-base <BranchName> master
```

3\. Start an interactive rebase using the commit hash returned from step 2.

```
git rebase --interactive <hash>
```

This will launch a text editor with a list of all commits in your personal branch. It should look something like this:

```
pick 59d2a23 Initial implementation
pick 752ae4c Add more features
pick cd302a3 Fix something
pick 5410de3 Fix something else
```

4\. Change all but the first line to say "squash" instead of "pick". The contents should now look like this:

```
pick 59d2a23 Initial implementation
squash 752ae4c Add more features
squash cd302a3 Fix something
squash 5410de3 Fix something else
```

5\. Save the changes and close the file. A new file will now open containing the commit messages from all of the commits. Reword the commit message then save the changes and close the file.

6\. Push the modifications to your personal remote repository.

```
# If the branch has already been pushed to the remote repository you will need to add
# the  `--force` argument.
git push origin <BranchName>
```

For more information on using interactive rebase see [this article](https://help.github.com/articles/about-git-rebase/).

## Creating a Pull Request

You can create a pull request via the "Create pull request" option in Bitbucket's left-hand navigation bar. Pull requests will only be accepted if they meet the requirements described above. Detailed information about working with Bitbucket pull requests is available [here](https://confluence.atlassian.com/display/BITBUCKET/Work+with+pull+requests).

## Reviewing a Pull Request

Your pull request will be reviewed by one or more CEF developers.  Please address any comments and update your pull request. The easiest way to update a pull request is by pushing new commits to the same branch -- those new commits will be [automatically reflected](https://blog.bitbucket.org/2014/04/22/bitbucket-now-auto-updates-pull-requests/) in the pull request. Once your changes are deemed acceptable they will be squashed and merged into the main CEF repository.

Detailed instructions for locally testing a pull request created by someone else are available [here](http://www.electricmonk.nl/log/2014/03/31/test-a-pull-merge-request-before-accepting-on-bitbucket/).

The contents of a pull request can also be downloaded as a patch file and applied to your local Git checkout:

```
# Download the patch file.
curl -u user:password https://bitbucket.org/api/2.0/repositories/{user}/{repo}/pullrequests/{pull_no}/patch -L -o name.patch

# Apply the patch file to your local Git checkout.
git apply name.patch
```
