---
layout: post
title:  "QuickFixJ - FIX Protocol Session Layer Implementation"
date:   2024-05-23 06:00:00 +0800
categories: top
author: Isuru
---

# Session Layer Definitions 

## Retransmission Gap-Fill and Resend

## in-band vs out-of-band communication

## TestRequestThreshold, SendingTimeThreshold , LogOutAckThreshold

# Establishing Connection 

1. Transport Layer Connection
2. Acceptance with optional authentication
3. Message synchronization 


# Testing the spec

## Heartbeat interval determination

QuickFIXJ implementation of heart beat interval determination, is to specify it in the HeartBtInt(108) tag of the logon request (35=A) send by the initiator.
This is as per FIX session layer implementation guide. Although its worth noting the session layer specification allow other methods as well. 

[QFJ Configuration - Initiator](https://www.quickfixj.org/usermanual/2.3.0/usage/configuration.html#Initiator)

Configuration   | Description                                              | Values
---             |----------------------------------------------------------| ---
HeartBtInt      | Heartbeat interval in seconds. Only used for initiators.	|  positive integer	



# References

[FIX Session Layer Test Cases](https://www.fixtrading.org/standards/fix-session-layer/)

[FIX Session Layer Complete References](https://www.fixtrading.org/standards/fix-session-layer-online/)