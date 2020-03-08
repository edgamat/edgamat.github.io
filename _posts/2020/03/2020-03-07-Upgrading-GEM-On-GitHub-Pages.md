---
layout: post
title: 'Upgrading GEM on GitHub Pages'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Whenever I receive an email from GitHub indicating a new security advisory on my blog, I can never remember how to update things.

This is meant to list the steps so I don't have to relearn things each time.

<!--more-->

Here is the alert: (https://github.com/edgamat/edgamat.github.io/network/alert/Gemfile.lock/nokogiri/open)[https://github.com/edgamat/edgamat.github.io/network/alert/Gemfile.lock/nokogiri/open]

It instructs me to add the following to the Gemfile:

```
gem "nokogiri", ">= 1.10.8"
```

I am previewing things on my local PC, I use WSL to run the build:

```
wsl bundle exec jekyll serve --drafts
```

After modifying the Gemfile, I get the following error:

```
You have requested:
  nokogiri >= 1.10.8

The bundle currently has nokogiri locked at 1.10.4.
Try running `bundle update nokogiri`
```

So I ran this:

```
wsl bundle update nokogiri
```

After being prompted for the WSL admin password, the install completes. The `Gemfile.lock` file is also updated.

Yay. And the villagers rejoice.
