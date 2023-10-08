---
layout: post
title:  "Benchmarking real-logic SBE"
date:   2022-05-18 23:56:20 +0800
categories: top performance-engineering
author: Isuru
---

# What is real-logic SBE - TL;DR

[real-logic SBE](https://github.com/real-logic/simple-binary-encoding) project is a reference implementation of
[SBE](https://github.com/FIXTradingCommunity/fix-simple-binary-encoding). Objective is to enable financial application with
low latent binary messages encoding and decoding. SBE takes field-less binary encoding approach. 
In which the binary message contains no information about the fields or their type. 
This makes the messages light without scaffolding information to support decoding. This is achieved via generated encoder and decoders 
