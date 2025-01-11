---
layout: post
title: 'Helpful Git Commands'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I recently was trying to figure out which files I had modified with my feature branch in `git`. It
wasn't as easy as I thought. I decided to document some of the `git` commands I frequently use (so
I don't have to keep searching fo them again and again)
<!--more-->

## What is in my Branch?

I asked ChatGPT for help with this one. I asked it to tell me how to list all the files I had
changed in my `git` branch. Here's the command it recommended:

```bash
# This command lists all files that have changed in your current branch compared to main
git diff --name-only main...HEAD
```

Well, that didn't give me what I wanted. With some trial and error, I found this to work:

```bash
git diff --name-only main
```

You can also see only the files that are changed, but not staged:

```bash
git diff --name-only
```

And if you want, you can see just the staged files:

```bash
git diff --name-only --cached
```

## Undo my last commit

This happens a lot. I commit something to the wrong branch (usually the main branch in my local repository).
Here is the command to undo that last commit:

```bash
git reset HEAD~
```

All the changes are back in an unstaged state and the commit is gone. You can now continue to do what you
should have done before you created the commit.

Reference: [https://stackoverflow.com/questions/927358/how-do-i-undo-the-most-recent-local-commits-in-git](https://stackoverflow.com/questions/927358/how-do-i-undo-the-most-recent-local-commits-in-git)

## Set my user name/email

Upon a fresh install of `git`, you should always set your user name and email:

```bash
git config --global user.name "Matt Edgar"
git config --global user.email "edgamat@outlook.com"
```

I often work for multiple clients and my user name and email are different per `git` repository. For
these cases, I just leave out the `--global` argument and the changes will only apply to the current
repository.

```bash
git config user.name "Matt Edgar"
git config user.email "edgamat@outlook.com"
```

If I want to view these config values, I can use this command:

```bash
git config --list
```

If you want to remove these local settings you can use this command:

```bash
git config --unset user.name
git config --unset user.email
```

## Change the Default Editor

I usually use `vim` as my default editor with `git`. But sometimes it can be helpful to change it.
Here is the command to change the default editor to Visual Studio Code:

```bash
git config --global core.editor "code -n -w --disable-extensions"
```

Or Notepad++:

```bash
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

And then you can remove it if you change your mind:

```bash
git config --global --unset core.editor
```

## How Much Have I Changed?

When I look at a feature branch I often want to get a sense of how large it is. Using `git` you can
get a count of the number of files and the number of lines that have changed:

To get a detailed list of file changes:

```bash
git diff --stat main
```

To get a one line summary of the counts:

```bash
git diff --shortstat main
```

To show added and removed lines for each file:

```bash
git diff --numstat main
```

## Revert a File

Again, this occurs frequently. In a feature branch, I have several commits. In one of the commits I
have changed a file. Then I realize that I need to revert it back to its original version (the version
currently in the main branch).

Here is the command to revert the file:

```bash
# Fetch the latest version of the main branch
git fetch origin

# Replace the file with its version from main
git checkout origin/main -- path/to/file
```

If you haven't linked your local repo to a remote repo you can just use this command:

```bash
git checkout main -- path/to/file
```
