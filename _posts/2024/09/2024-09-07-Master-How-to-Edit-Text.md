---
layout: post
title: 'Early Days - Mater How to Edit Text'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the early part of your software development career, you spend the majority of your time writing code. This is the time to master how to edit text.

<!--more-->

## It's What We Do

Farmers work the soil. Carpenters cut, sand, and paint wood. Name any endeavor and you will find a core skill that defines it. For programmers, it is editing text.

When coding, in my mind there is a vision of what the code should look like. I use text editors to realize that vision. The ideas flow from my mind, through my fingers and into the code using text editors. The keystrokes I press are an extension of my thoughts. 

Frustration will build if I can't express my ideas. I have mastered how to edit text in order to bring ideas to life with as little effort as possible. To achieve this state, I must know how to edit text.

## Early Days

My first job out of college was a long time ago. The Microsoft Windows operating system was just beginning to be popular in the work environment. When I started I was given a brand new PC. It was the first one on the floor with an SVGA graphics card (800x600 resolution). I was asked to configure an MS-DOS-based (not Windows) application to conduct field surveys and questionnaires. The application used the Clipper programming language with a dBASE database storing the data. We also wrote utilities in C to add headers to the questionnaire data files (PK-ZIP'ed directories of files). I still have a copy of that C code today:

![One of my first programs](/assets/img/header.png)

 I spent 8 months on this assignment and learned a lot. All of the programming was done using MS-DOS tools. I had to rely on the keyboard and memorize several shortcuts to get work done efficiently. No GUI, no mouse, no Internet. Those were the days.

My next assignment required me to program on a mainframe computer. While I didn't have to use punch cards (they were phased out a few years before I started), the only way to create programs was using a text-based terminal emulator. The programs were written using PL/I. We had to write JCL scripts to submit the code to be compiled and run (usually over night). We had just been given an upgrade which enabled color coded syntax highlighting. We were over the moon!

The mainframe text editor was very limiting. I hacked together a set of FTP scripts that would allow me to transfer the files to my PC where I could edit the files using the [Programmer's File Editor][pfe]. After making the changes I'd FTP them back to the mainframe.

I learned a lot during these early days of my career. All of the technologies I learned during that time have long been replaced. But one of the things that has endured all these years later is the skills I learned editing text. It's one of the fundamental skills that anyone earlier in their software development career needs to master.

## So Many Choices

Today, deciding which application to use as your primary text editor can be a journey of discovery. There are so many choices for you to explore. If you are primarily using Linux, the usual choices are `vim`, `nano` and `emacs`. But these are no means the only options. With Windows, you will also find an abundance of choices such as Notepad++, TextPad, and Sublime Text. (I've never used a Mac so I am not sure what choices you will find there).

One of the more popular text editors is embodied in Visual Studio Code. This IDE can be used with just about any programming language, so it is worth taking a look at and learning how to use it. It is available on Linux and Windows (and on the Mac) and is free.

Regardless of where your journey takes you, find an editor to master. Find one that you can use for as many situations as possible. You don't want to rely on multiple text editors if you can help it. For years I relied on 2 separate text editors. Each had features that made them the best tool for specific tasks. But I was fooling myself. It turns out that I could do 100% of my work from either of them. I was just lazy and didn't learn how to use each tool to its fullest. So I uninstalled one of them and haven't looked back.

## No Mouse Allowed

Sit down with a developer for 5 minutes and watch them work on a code file. In that short amount of time you will know if they can edit text efficiently or not. A warning sign is their reliance on the mouse. If their primary way of selecting text is using a mouse, then they are not good at editing text. If they rely on the mouse to scroll through the file, they are not good at editing text. Mouse-based operations are slow.  You shouldn't have to use the mouse to manipulate the text.

## Memorize Shortcuts

All text editors have keyboard shortcuts (`CTRL+C` / `CTRL+V`, for example) to help developers manipulate text. Your productivity will increase a great deal if you take the time to learn these shortcuts. Here is a list of all the built-in shortcuts for Windows users of Visual Studio Code:

https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf

Have a copy of this list available when you are editing text. I prefer to have a printed copy. Each time you reach for the mouse, stop and look through the list of shortcuts and see if there is one you can use instead of the mouse. For example, you can use the `SHIFT` and  arrow keys to select text:

Shortcut | Result
------------------- | -------------------
`SHIFT+Right Arrow` | Select text from the cursor moving to the right, one character at a time
`SHIFT+Left Arrow` | Select text from the cursor moving to the left, one character at a time
`SHIFT+CTRL+Right Arrow` | Select text from the cursor moving to the left, one word at a time
`SHIFT+CTRl+Left Arrow` | Select text from the cursor moving to the left, one word at a time
`SHIFT+Up Arrow` | Select text from the cursor moving up
`SHIFT+Down Arrow` | Select text from the cursor moving down

Focus first on the basics:

- Selecting Text
- Cut, Copy, Paste
- Navigating through a file
- Searching / Replacing text
- Switching / Closing tabs

While some of these shortcuts are specific to Visual Studio Code, a good portion of the basic commands are universally available in all Windows text editors.

## Practice, Practice, Practice

It doesn't take long for these shortcuts to feel natural, and you will rely less and less on your mouse. But it does take practice. At first you may feel like it takes longer to get things done. But stick with it. Before you know it, your ability to edit text will improve and you will be on your journey to master how to edit text.

[pfe]: https://en.wikipedia.org/wiki/Programmer%27s_File_Editor
