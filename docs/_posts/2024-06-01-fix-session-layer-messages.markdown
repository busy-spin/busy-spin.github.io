---
layout: post
title:  "QuickFixJ - FIX Protocol Session Layer Implementation"
date:   2024-06-02 06:00:00 +0800
categories: top
author: Isuru
---

# FIX Protocol Session Layer 

FIX protocol messages are divided into two types. 

1. Session Layer Messages
2. Application Messages 

Session Layer Messages are for establishing, recovering and terminating FIX session. 
While application messages are to carry out business purposes such as buy, sell, sending/subscribing market data or sending execution reports. 

FIX protocol session layer is a realization of layer 5 session layer of OSI model.

![OSI Session Layer](/assets/img/fix_session_layer/osi_session_layer.jpg)
Image Reference - [Data Networks - MIT](https://web.mit.edu/modiano/www/6.263/Lecture1.pdf)

FIX session can exist multiple sequential FIX connections. FIX connection can be terminated due to various reasons such as application or network outages.
But the FIX session can continue after re-establishing the FIX connection. 
There are various means provided in the FIX session layer specification to recover the session as well.
Such as **NextNumIn**, **NextNumOut***, **Retransmission**, **Gap-Fill** and **Resend** of messages. 

![Session exists across sequential connections](/assets/img/fix_session_layer/session_lives_across_connections.png)

Image Reference - [Session Across Sequential Connections](https://www.fixtrading.org/standards/fix-session-layer-online/)

## Establishing FIX Connection

Establishing FIX connection has three parts.

1. Transport Layer Connection
2. Acceptance with optional authentication
3. Message synchronization

Due to number 2 and 3 steps. Some experts argue that FIX is an all-in-one protocol. 
Mixing the session layer with connection layer. To establish a FIX connection peers should accept the session and sync the messages, sounds counterintuitive right. 

## FIX session time-span

FIX session usually exists for agreed time-span by two peers (acceptor and initiator). It can be weekly or daily sessions. 
This weekly and daily session timing are agreed by **out-of-band** communications. Which means no session layer message will specify the duration of the session.
Which we called the **in-band** communication. Two counterparties agree upon session timing as part of their rules of engagement. 

Since I briefly mention **in-band** communication. Example for **in-band** communication is the heart beat interval. 
Which used by peers to check on each other's live-ness.
HeartBtInt(tag 108) is set in initiators logon request (35=A) to specify this value. Hence, its **in-band** communication. 


## Important out-of-band parameters 

Following out-of-band parameters are important to the session layer. 

Configuration               | Description                                              
---                         |---
TestRequestThreshold        | Amount of time expressed as a multiplier of heartbeat interval before TestRequest(35=1) send to the peer.		
SendingTimeThreshold        | Amount of time expressed in seconds in which SendingTime(tag 52) in an inbound message can differ from system time of the receiving peer. 
LogOutAckThreshold          | Amount of time express in seconds where FIX peer who has sent Logout(35=5) will wait for its peer to reply with Acknowledgement, if execeed it will terminate the transport layer connection.


# Testing the spec

## Heartbeat interval determination

QuickFIXJ implementation of heart beat interval determination, is to specify it in the HeartBtInt(108) tag of the logon request (35=A) send by the initiator.
This is as per FIX session layer implementation guide. Although its worth noting that the session layer specification allow other methods as well. 

[QFJ Configuration - Initiator](https://www.quickfixj.org/usermanual/2.3.0/usage/configuration.html#Initiator)

Configuration   | Description                                              | Values
---             |----------------------------------------------------------| ---
HeartBtInt      | Heartbeat interval in seconds. Only used for initiators.	|  positive integer	

Following is a QFJ log from the perspective of the accepting peer. HeartBtInt configuration is mandatory for QFJ initiator peer.
Acceptor will switch to the heart beat interval set by the initiator in the logon request.

![QFJ HeartBtInt determination](/assets/img/fix_session_layer/heartbtint_determination.png)

Above screenshot is from the output of the [fix-shell](https://github.com/busy-spin/fix-protocol-showcase)

```mermaid
  graph TD;
      A-->B;
      A-->C;
      B-->D;
      C-->D;
```

# References

[FIX Session Layer Test Cases](https://www.fixtrading.org/standards/fix-session-layer/)

[FIX Session Layer Complete References](https://www.fixtrading.org/standards/fix-session-layer-online/)