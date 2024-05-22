---
layout: post
title:  "Agrona Idle Strategy - Exponential Back Off"
date:   2024-05-23 06:00:00 +0800
categories: performance-engineering
author: Isuru
---

# Agrona

Agrona is a compilation of threading and data structures which essentially a utility library, which is at the heart of [Aeron](https://github.com/real-logic/aeron), 
[SBE](https://github.com/real-logic/simple-binary-encoding) and [Artio](https://github.com/real-logic/artio). 
These are all parts of the [real-logic stack](https://github.com/real-logic) for building high performance financial applications. 

# A new perspective the Threads , Duty Cycle and Idle wait

Java Thread API provide a way to start a thread by providing the function to execute. It also allows the developer to name it for better performance analysis and mark 
it as daemon to control the program exit behaviour.  
And sometimes thread lives throughput the lifespan of the process, by executing events as and when they occur. 
This single iteration of processing, in Agrona terminology is often called a Duty Cycle. This is synonym to single iteration in an event loop. 
Managing a thread's execution frequency is important to overall system performance. 

# References

[Agrona Showdown](https://github.com/busy-spin/aeron-showdown/tree/main/agrona-agent-samples)