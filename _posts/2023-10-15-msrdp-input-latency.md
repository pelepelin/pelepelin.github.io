---
layout: post
title: MS RDP Input Latency Setting 
---

Working a long time with MS RDP classic client (so called "Remote Desktop Connection", mstsc.exe),
previously I never knew about this configuration option, found in  
[VirtualBox troubleshooting chapter](https://www.virtualbox.org/manual/ch12.html#ts_win-host-rdp-client):

> RDP client collects input for a certain time before sending it to the RDP server.
> The interval can be decreased by setting a Windows registry key to smaller values
> than the default of 100. The unit for its values is milliseconds.

100ms is a pretty noticeable delay.  

> The key does not exist initially and must be of type DWORD.
> Depending whether the setting should be changed for an individual user or for
> the system, set either of the following.
>
> `HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Min Send Interval`
> `HKEY_LOCAL_MACHINE\Software\Microsoft\Terminal Server Client\Min Send Interval`