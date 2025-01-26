---
layout: post
title: 'Sharing PowerShell Profiles'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I use PowerShell on several computers. I need a means to share the profile I prefer on all these computers.
I found a simple way using GitHub.

<!--more-->

In Linux, it is common to store your dot files (e.g. `.bashrc`) in `git` and configure their use in your operating system.
This is a great way to backup your settings and keep track of changes. I want something similar for my
PowerShell profile. I have aliases configured for several commands, as well as configurations for using
`posh-git` that I want to use on all my computers (2 home computers and 2 work computers).

I have created a repo on GitHub to store my settings, and instructions for how to install the modules
I include in my profile:

[https://github.com/edgamat/PowerShellProfiles](https://github.com/edgamat/PowerShellProfiles)

## Clone the Repo

The first step is to clone the repo to my `source` folder:

```powershell
cd ~/source
git clone git@github.com:edgamat/PowerShellProfiles.git
```

## Run the script from my profile

Open the profile in VS Code:

```powershell
code $PROFILE
```

Add the setup code:

```powershell
# Use the correct path to the cloned repo
$ProfileRepoPath = "C:\Users\$env:USERNAME\source\PowerShellProfiles"

# Select a profile to load
$SelectedProfile = "GeneralProfile.ps1"  # Change as needed

# Load the selected profile
if (Test-Path "$ProfileRepoPath\$SelectedProfile") {
    . "$ProfileRepoPath\$SelectedProfile"
    Write-Host "Loaded profile: $SelectedProfile"
} else {
    Write-Host "Profile not found: $SelectedProfile"
}
```

## Is that it?

Well, yes. It isn't complicated. Storing your settings in GitHub allows you to access them from any
machine. And with a simple script, you can include these settings and have a consistent experience
from each machine. I'm sure I'll be adding more to this over time, more aliases, more modules, etc.
Having them centrally accessible is key.
