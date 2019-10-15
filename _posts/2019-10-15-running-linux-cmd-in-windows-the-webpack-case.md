---
layout: post
published: true
title: Running Linux CMD in Windows - the webpack Case
date: '2018-03-21'
Tags: WSL
---
So I am learning now that consistency is key: I was working on a Single Page Application (SPA)  project in windows, but running the tooling (npm, webpack, etc.) from WSL, just because I think the Linux native commands work better from Linux.

Well, it turns out there are larger differences than I thought: In this particular case, I ran npm install from WSL, which in turn created the node_modules dir, and populated all my dependencies. So far so good. no weird errors or anyting, I tried before to run it from PowerShell and I got permission issues.

The bigger problem appeared when I tried to run my project from Visual Studio. I am not certain what commands are run when in VS that trigger the webpack process, and that webpack process failed miserably on my application. I started debugging and I found out I was not even able to run the webpack command from windows CMD!.

It turns out that depending on the environment you are, the files created are registered in different ways in the node_modules/.bin directory. If you are in Windows, it creates .cmd files and junctions to the actual package, while in Linux, it creates symlinks and aliases to run the commands. Obviously, those are named the same, but if you generated the modules in WIn, you cannot run them in Linux and vice-versa.

In conclusion, if you are running in Windows env, use the windows CMD, if you are running in Linux, use the WSL!