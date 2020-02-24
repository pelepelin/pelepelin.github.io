---
layout: post
title: Developer's work is a waterfall
---

In order to do something you always need to do something else.

That's how my software development always goes. No matter if it's a job task or my own wish, before I can achieve what was an initial goal, before I can finish the task, I need to learn and finish a thousand of other things.

Like now. I want to make this blog alive but apart of intial easy setup (thanks to the man whose setup I cloned) it involves perpetual unintended nested learning. Look, I want to make the blog bilingual but the initial template is not suited. So I need to experiment with other templates, but it requires local Jekyll environment. Which I don't want to set up on my Windows laptop, so I need a good virtualized environment. Also, I'm a bit tired by small annoyances while developing Node.JS applications on Windows, so I'd move all development into this environment and so a seemingly easy short path of using Docker for Windows is less attractive. I'd try [WSL2](https://docs.microsoft.com/en-us/windows/wsl/wsl2-index) but it's not in a Windows public release build yet and I failed to join the Windows Insider Program (maybe it's for better). So I stick to VirtualBox. Unfortunately, heavy applications in VirtualBox lose up to 30% of performance. Also, on my job workplace I still use some Windows applications during the day. So this time I've learned that there are interesting alternatives to using a Linux desktop inside of a VirtualBox window. Namely, [VcXsrv](https://sourceforge.net/projects/vcxsrv/) and [Xpra](https://xpra.org/) look promising enough for seamless integration of Linux and Windows desktops. VcXsrv is good enough but Xpra is especially interesting as it promises that Linux sessions can survive Windows host reboots the same way as if I use VirtualBox UI. Unfortunately, setup is not that seamless and there are some bugs I'm trying to resolve. So that all becomes really long.
