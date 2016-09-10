---
author: Rob
date: "2016-09-10 14:17:00+00:00"
layout: post
slug: cli-image-resize
title: Resize images using the command line in bash/zsh
categories:
- sips
- zsh
- bash
---

Another command line script for sharing purpose. It's a simple script that loops through the current directory and asks you for the new filenames, maximum width and height. I've used the OSX built in sips tool but you can rewrite that easily to Imagemagick if you prefer more options or a script that works on Linux as well. The original script is available through my Gist:

<script src="https://gist.github.com/robsonke/1b0e06e060e9836e6fb88e3e0e6eabde.js"></script>

Happy resizing! And to give some credits to Sips, it's blazingly fast.

Next stop will probably be a multi-tenant post on Spring Boot.