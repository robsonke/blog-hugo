---
author: admin
comments: true
date: 2015-03-01 16:40:23+00:00
layout: post
slug: alfred-workflow-manage-tomcat
title: An Alfred workflow to manage tomcat instances in Iterm2
categories:
- Alfred
- Tomcat
---

I've always disliked repeating tasks, earlier when working on Windows. It was one of the main reasons to switch to Mac osx years ago. Writing small but effective scripts makes my live easy every day. So once I discovered [Alfred](http://www.alfredapp.com/) I loved it immediately. Earlier I used Quicksilver, not bad either but Alfred is just awesome. Especially with the paid extensions in the powerpack. I have workflows for daily tasks as opening ssh sessions, setting up vpn's, open finder windows and killing processes. 

But I still was struggling with Apache Tomcat. Somehow this is not that easy to handle as with a dos box under Windows. I already wrote some wrapping Bash scripts that start Tomcat and keep logs open and shutdown it nicely, but I always had to open/search the right Iterm2 tabs. There must be a better way! And yet there is. Although I hoped to find a ready to use workflow on the internet, it couldn't find one. So I gave it a shot and wrote my own workflow. I never used Applescript before but it was fairly simple, together with a good amount of examples on the internet.

Let's start with the end result in Alfred:
![End result](alfred-workflow-screenshot.jpg)

The steps of this workflow are simple and short, it's just an input keyword which required a parameter of the right tomcat environment, some Applescript and a notification. And that repeated for the stop mechanism too.

## Step 1: a scriptfilter
Take a scriptfilter component and add a keyword. For me this is "start" and attach a Bash script to this. For me, this lists my different environments:
``` xml
cat<<EOB
<items>
  <item uid="book" arg="book" autocomplete="book" type="default">
    <title>book</title>
  </item>
  <item uid="services" arg="services" autocomplete="services" type="default">
    <title>services</title>
  </item>
  <item uid="portal" arg="portal" autocomplete="portal" type="default">
    <title>portal</title>
  </item>
</items>
EOB
```

## Step 2: an Applescript
This is the main part of this workflow. It's going to do several things but in general: find iterm2, open a new session with a specific profile, give the iterm2 tab a name, start tomcat and make the Iterm2 window the active one. The startup is done with a custom run.sh, that's doing a startup plus log open: "./startup.sh && tail -f ../logs/catalina.out".

```
on alfred_script(q)
  tell application "iTerm"
    activate
    
    tell the first terminal
      launch session "Default"
  
      tell the last session
        set name to q

        if ((q as string) is equal to "services")
          write text "cd ~/tomcats/tomcat7-services/bin"
        end if
        if ((q as string) is equal to "book")
          write text "cd ~/tomcats/tomcat7-book/bin"
        end if
        if ((q as string) is equal to "portal")
          write text "cd ~/tomcats/portal/tomcat-7.0.42/bin"
        end if
        write text "./run.sh"
      end tell
    end tell
  end tell
  tell application "System Events"
    set visible of process "iTerm" to true
  end tell
end alfred_script
```

Now we're able to start a tomcat in a named tab. If you want, you can also implement a second flow to stop tomcat. This script is shorter, it searches the right tab, presses ctrl+c to kill my "tail -f catalina.out" which I always open and call a shutdown script. I'm using a stop.sh, which is nothing more then "./shutdown.sh -force" as I want to be sure that it stops.

```
on alfred_script(q)
  tell application "iTerm"
    activate
    set myterm to (first terminal)
    tell myterm
      repeat with mysession in sessions
        tell mysession
          set session_name to get name
          if session_name contains q then
            tell application "System Events" to keystroke "c" using {control down}
            write text "./stop.sh"
          end if
        end tell
      end repeat
    end tell
  end tell
end alfred_script
```

## Step 3: a notification
This is simple and doesn't require explanation or code. It's just a ping in mac osx. After all, you don't need it.

## Step 4: repeat
Now do the same once more for a stop script. Or whatever tomcat script you like. 

## That's it!

In case you're interested in the source and compiled workflow, you can easily edit that one in Alfred to your needs, take a look here:
https://github.com/robsonke/alfred-tomcat-workflow

After this first custom workflow, I'm afraid that I'm going to write much more of these useful little helps. If useful for others too, I'll share them here or at least through my Github account. 

Some useful links I used:

- https://code.google.com/p/iterm2/wiki/AppleScript
- http://www.alfredforum.com/topic/721-executing-iterm2-terminal-commands-in-current-shell/


