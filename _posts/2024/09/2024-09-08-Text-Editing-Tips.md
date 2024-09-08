---
layout: post
title: 'Early Days - Text Editing Tips'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

How do you master editing text? Practice is only part of the journey. Knowing what to practice is just as important.

<!--more-->

## The Basics

In my last post I advocated that mastering how to edit text was a fundamental skill for all developers. Great, but what do I need to learn? I am going to walk through some of the basics that I feel you should memorize above all others. These are universal keyboard shortcuts that will work in almost every Microsoft tool (Visual Studio Code, Visual Studio, SQL Server Management Studio, Microsoft Word). Many tools built for Windows also have the same keyboard shortcuts (JetBrains Rider, Notepad++ for example).

**NOTE** These keyboard shortcuts were popularized by Apple, Microsoft adopted them in Windows later.

One of the most basic operations on a text file you can learn is to cut, copy, or paste text. You should know these shortcuts from day one:

Chord | Result
------------------- | -------------------
`CTRL+X` | Cut the selected text and place it in the clipboard.
`CTRL+C` | Copy the selected text and place it in the clipboard.
`CTRL+V` | Copy the contents of the clipboard into the file starting from the cursor location.
`CTRL+Z` | Undo the last operation.

## File Navigation

Navigating through a text file using keyboard shortcuts is liberating. It makes you feel like you can do anything. Using the `UP`, `DOWN`, `LEFT` and `RIGHT` arrow keys can move the cursor anywhere you like. You can hold down the arrow key and it will keep moving the cursor without you having to press it over and over. Using the arrow keys alone won't get you anywhere quickly. To speed up navigating a file you will need to use shortcuts:

Shortcut | Result
------------------- | -------------------
`PAGE-DOWN` | Move the cursor one 'page' down.
`PAGE-UP` | Move the cursor one 'page' up.
`HOME` | Move the cursor to the beginning of the current line.
`END` | Move the cursor to the end of the current line.
`CTRL+HOME` | Move the cursor to the beginning of the first line in the file.
`CTRL+END` | Move the cursor to the end of the last line in the file.
`CTRL+G` | Goto a line in the current file.

`CTRL+F` find a piece of text in the file. Press `F3` to move to the next instance of the text. Press `SHIFT+F3` to move to the previous instance.

## Selecting Text

Being able to select text is crucial in being free from using the mouse when editing text. You can use the `SHIFT` and  arrow keys to select text:

Shortcut | Result
------------------- | -------------------
`SHIFT+RIGHT` | Select text from the cursor moving to the right, one character at a time
`SHIFT+LEFT` | Select text from the cursor moving to the left, one character at a time
`SHIFT+CTRL+RIGHT` | Select text from the cursor moving to the left, one word at a time
`SHIFT+CTRL+LEFT` | Select text from the cursor moving to the left, one word at a time
`SHIFT+UP` | Select text from the cursor moving up
`SHIFT+DOWN` | Select text from the cursor moving down

Using your cursor as a starting point, you can use the `SHIFT` key and the Arrow keys (`UP`, `DOWN`, `LEFT`, and `RIGHT`) to select text. Holding `SHIFT` and `CTRL` down at the same time allows you to select entire words of text at a time, rather than one character at a time.

In addition, `CTRL+A` is a great shortcut. It will select all the text in the current file.

If you want to select one or more lines of text, move your cursor to the start of the line you want to select. Hold the `SHIFT` key down and use the `UP` and `DOWN` arrow keys to select the lines you want to select.

## Tab Navigation

It doesn't take long for you to have more than one tab open at a time. Navigating between tabs using the keyboard can be a huge productivity gain. Press `CTRL+TAB` to navigate to the next tab. The editor will switch to the next tab once you let go of the `CTRL` key. Pressing `CTRL+TAB` again will return you to the previous tab. It can be very useful when attempting to switch back and forth quickly between two tabs.

Try pressing `CTRL+TAB` and then keep pressing the `CTRL` key and let go of the `TAB` key. A small window usually appears letting you see all the tabs you can navigate to. While still pressing the `CTRL` key, use the `UP` and `DOWN` arrow keys to move the cursor onto the tab you would like to open. This is very useful when you have a lot of tabs open.

Use `CTRL+W` to close a tab in Visual Studio Code. Most other Microsoft apps require you to use `CTRL+F4`.

## Replacing Text

Use `CTRL+H` to search for and replace text in a file. When the replace window is open, you can use the following shortcuts to enable/disable the search options:

Shortcut | Result
------------------- | -------------------
`ALT+C` | Toggle case sensitive search
`ALT+Q` | Toggle whole word search
`ALT+R` | Toggle regex search

Press `CTRL+A` to replace all the selected occurrences in the file (`CTRL+ALT+ENTER` in Visual Studio Code)

## The Secret

The secret to using keyboard shortcuts is to have a clear mental picture of all the files you are currently working with.

You need to see in your mind that the previous file you were working with is available with `CTRL+TAB`. This means you shouldn't think of your files in terms of where they are located on your file system. You need to think of them in terms of where they are in your editor. Was the file I need the last one I was looking at or was it 2 or 3 files back?

When you copy text into the clipboard, you need to feel what's in there as you navigate to the place where you want to use it. You need to avoid any detours along the way and focus on pasting the text as the most important action.

These mental images only come with repetition and practice. Focus on learning these shortcuts and before long it will become second nature to you. You will forget the mouse exists.
