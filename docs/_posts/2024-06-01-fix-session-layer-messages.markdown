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
Such as keeping track of **NextNumIn**, **NextNumOut**, **Retransmission**, **Gap-Fill** and **Resend** of messages. 

![Session exists across sequential connections](/assets/img/fix_session_layer/session_lives_across_connections.png)

Image Reference - [Session Across Sequential Connections](https://www.fixtrading.org/standards/fix-session-layer-online/)

## Establishing FIX Connection

Establishing FIX connection has three parts.

1. Transport Layer Connection
2. Acceptance with optional authentication
3. Message synchronization


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


# Heartbeat interval determination

In QuickFIXJ initiator will specify the HeartBtInt(108) tag in the logon request (35=A).
Acceptor which may or may not be QuickFIXJ based implementation, will also send HeartBeatInt(108) in its logon request. 
Acceptor may echo back the value send by the initiator, or send a different value. Or it can even reject the logon request. 
Initiator can also reject the logon message send by acceptor if it does not agree with the amend for the HeartBeatInt.
 

[QFJ Configuration - Initiator](https://www.quickfixj.org/usermanual/2.3.0/usage/configuration.html#Initiator)

Configuration   | Description                                              | Values
---             |----------------------------------------------------------| ---
HeartBtInt      | Heartbeat interval in seconds. Only used for initiators.	|  positive integer	

Following is a QFJ log from the perspective of the accepting peer. HeartBtInt configuration is mandatory for QFJ initiator peer.
Acceptor will switch to the heart beat interval set by the initiator in the logon request.

![QFJ HeartBtInt determination](/assets/img/fix_session_layer/heartbtint_determination.png)

Above screenshot is from the output of the [qfj-fix-shell](https://github.com/busy-spin/qfj-fix-shell)

# Message Recovery

FIX message synchronization after logon request has been send considered to be recoverable if the followings are true.

```
peer1.NextNumOut >= peer2.NextNumIn
peer2.NextNumOut >= peer1.NextNumIn
```

What this means is peer's NextNumOut should be always equal or smaller than other peer's expected NextNumIn. 
If there is need for message synchronization, then it can be done using ResendRequest(35=2) and SequenceReset(35=4) messages which 
follows the Logon(35=A) messages.  

## Scenario 1 - Acceptor requires retransmission of messages from initiator

Let's take a scenario where Initiator's NextNumOut is larger number than the Acceptor's NextNumIn.

```
    initiator.NextNumOut > acceaptor.NextNumIn 
    initiator.NextNumIn = acceaptor.NextNumOut
```


![Sequence Numbers](/assets/img/fix_session_layer/recovery_1_sequence_numbers.png)

Above figure shows a scenario I have created using [qfj-fix-shell](https://github.com/busy-spin/qfj-fix-shell).
Where initiator's NextNumIn and acceptor's NextNumOut matches(83), but initiator's NextNumOut (199) is larger than acceptor's NextNumIn (20).

Let's assume on the initiator side message sequence for un-synced outgoing messages would look like this.

Sequence Number(s) | Type of Message
----               | ---
20-152             | Session Layer Messages
153                | Application Layer Message
154                | Session Layer Message
155                | Application Layer Message
156-199            | Session Layer Messages

### Recovery process

![Message Recovery](/assets/img/fix_session_layer/recovery_1_fix_log.png)

Step    | Remark 
---     | ---
1       | Initiator send logon(35=A) request
2       | Acceptor respond with logon(35=A) request
3       | Acceptor resend reqeust (35=2) for missing messages between 20 to **infinity** BeginSeqNo<7>=20 and EndSeqNo<16>=0.
4       | Initiator send sequence reset (35=4) with NewSeqNo<36>=153 and GapFill<123>=Y. Reason is there is no application messages till sequence number 152
5       | Initiator send application message NewOrderSingle(35=D) with PossDupFlag<43>=Y to denote that this is possible duplicate message.
6       | Initiator send sequence reset (35=4) with NewSeqNo<36>=155 and GapFill<123>=Y. Reason is there is no application messages till sequence number 154
7       | Initiator send applicatoin message NewOrderSingle(35=D) with PossDupFlag<43>=Y to denote that this is possible duplicate message.
8       | Initiator send sequence reset (35=4) with NewSeqNo<36>=200 and GapFill<123>=Y. Reason is there is no application messages till sequence number 199
9       | Acceptor send HeartBeat message (35=0), and conclude the message recovery


# Best Practices

# Always send a TestRequest(35=1) upon Login(35=A)

Initiator and Acceptor require to perform message synchronization after each peer received the logon request, and logon request 
SequenceNumber does not match with the expected NextNumIn for the respective peer. A good practice is to send TestRequest(35=1) to force message synchronization 
the peer to send a HeartBeat(35=0), before sending any queued application level messages. 

# References

[FIX Session Layer Test Cases](https://www.fixtrading.org/standards/fix-session-layer/)

[FIX Session Layer Complete References](https://www.fixtrading.org/standards/fix-session-layer-online/)