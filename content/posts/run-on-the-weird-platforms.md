---
title: "Run on the weird platforms"
date: 2024-02-07T21:49:33+02:00
tags: ['linux', 'freebsd', 'openbsd']
---

Run your C code on the bizarre platforms, on the `POSIX` ones and the `BSD` ones. 

Run your code in [OpenBSD](https://www.openbsd.org/) it will crash if you do something incorrectly. 

Run your code in [FreeBSD](https://www.freebsd.org/) it will make sure stuff is location agnostic (location of binaries and the usage of `env` instead of `/bin/xxx`), and it's pretty similar to `Linux`.

Run your code in [Alpine Linux](https://www.alpinelinux.org/) it will compile your code against [musl](https://musl.libc.org/) C library which is a `glibc` replacement with `POSIX` enforcement.

Cross compile your code to different cpus, like `arm`, it will find you bugs so fast you'd be suprised. (alignment, page sizes, you name it). 

Use `qemu` and enjoy.

