---
layout: post
title: OSX Firewire kernel debugging
category: OSX
tags: ['firewire', 'kernel']
date: 2015-02-18T16:13:26+02:00
draft: true
---

Introduction
------------

If you ever need to debug a newly (Yosemite) OSX kernel using firewire here is a small tutorial to help you through. 

Downloads 
-----------
Grab the needed Kernel Debug Kit dmg file from [apple](https://developer.apple.com/downloads/index.action), make sure you download the exact kernel version otherwise it will not work. 

Possible cables 
-----------

Please verify that your connectors and the cable match before buying.

1. [firewire adapter](http://store.apple.com/us/product/MD464ZM/A/apple-thunderbolt-to-firewire-adapter)
2. [firewire cable](http://store.apple.com/us/product/HA834ZM/A/belkin-firewire-800-99-pin-cable-94-pin-adapter?fnode=51)

Debuggee machine configuration
----------------------

You will need to configure the nvram to support firewire debugging. The man page of [fwdkp](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/fwkdp.1.html) suggests using `$ fwkdp -r`, 
but it does not configure the nvram correctly (there is a bug in the `fwkdp`). 

So issue, 

    $ sudo nvram boot-args="debug=0x146 kdp_match_name=firewire kext-dev-mode=1 fwkdp=0x8000"
    
The `kext-dev-mode` is only needed if you are loading unsigned kexts.

After you reboot your machine you can issue NMI [interrupt](https://developer.apple.com/library/mac/qa/qa1264/_index.html), then the debuggee will wait for a kernel debugger to connect. 

Note that the `debug` flags can be changed (see [documentation](https://developer.apple.com/library/mac/documentation/Darwin/Conceptual/KernelProgramming/build/build.html#//apple_ref/doc/uid/TP30000905-CH221-BABDGEGF))

Debugger (guest) machine configuration
------------------------------------

Mount the kernel debug kit from the DMG you downloaded before. From kernel 10.10.1 you will need to install the SDK into your machine instead of simply loading it from `/Volumes/Kernel Debug Kit`.

Open a terminal and issue: 

	$ fwkdp

This will pass all commands from lldb to the firewire, so just leave it running. 

*(You might encounter a connection problems from time to time so simply close and reopen the `fwkdp`. if you want more information you can run it with `fwdkp -v` to see some raw packets, if you really miss embedded debugging)*

For the actual debugging open another terminal and issue:

    $ lldb /Library/Developer/KDKs/KDK_10.10_14B25.kdk/System/Library/Kernels/kernel

*(if you run a kernel older than 10.10.1, you will simply replace the path to `/Volumes/Kernel Debug Kit/kernel`)*

run the xnu debugging utilities script: 

    (lldb) command script import "/Library/Developer/KDKs/KDK_10.10_14B25.kdk/System/Library/Kernels/kernel.dSYM/Contents/Resources/Python/kernel.py"

connect to the debuggee with:

    (lldb) kdp-remote localhost

If you are able to connect, horay! Everything is working correctly, but just make sure you are running the correct version of the kernel. 

Test your debugger with a simple command like: 

	(lldb) showproctree

If it does not complain about anything and you get a tree of processes you are good to go. 

