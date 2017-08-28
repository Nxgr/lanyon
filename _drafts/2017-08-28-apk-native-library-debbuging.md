---
published: false
layout: post
title: Apk native library debbuging
---
## Context

During [HITB CTF](https://hitb.xctf.org.cn) last weekend I faced an APK reverse engineering task.
Getting the code from an APK is pretty straightforward nowadays, there are even online [services](http://www.javadecompilers.com/apk).

The code was basic, however it called a method from an embeded compiled library that was essential in order to get the final flag.

In this blog post, I am going to describe the steps I took to be able to interact and debug this library in an APK context.

DISCLAIMER : I am an absolute neophyte regarding APK or android studio. I may not be doing this the right way.
> If it's stupid and it works, it's not stupid.

## Walkthrough