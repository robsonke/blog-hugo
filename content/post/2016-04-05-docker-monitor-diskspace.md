---
author: Rob
date: "2016-04-08 13:00:00+00:00"
layout: post
slug: docker-monitor-diskspace
title: Monitor disk space in your docker container
categories:
- docker
- bash
---

A while ago I wrote a basic Bash script to check the disk space inside running Docker containers. As I couldn't find much examples for this simple purpose, I'd thought I'd share mine. It's available in my Github Gist:

<script src="https://gist.github.com/robsonke/c5c478bae476adb32d48.js"></script>

In essential it takes the following steps, made possible with default Docker commands and a set of string manipulations:

1. List and loop through all running Docker containers
2. Execute some "df" commands within the container with Docker exec
3. Collect the values and email whenever something is more full then you want

Of course, monitoring by email is not the best way but it's fairly easy to move a script like this to be used in Zabbix, Nagios or other monitoring tools.


