---
layout: post
title:  "Agrona Idle Strategy - Exponential Back Off"
date:   2024-05-23 06:00:00 +0800
categories: performance-engineering
author: Isuru
---

# Agrona

Agrona is a compilation of threading and data structures which essentially a utility library, which is the underpinning of [Aeron](https://github.com/real-logic/aeron), 
[SBE](https://github.com/real-logic/simple-binary-encoding) and [Artio](https://github.com/real-logic/artio). 
These are all parts of the [real-logic stack](https://github.com/real-logic) for building high performance financial applications. 

# AgentRunner, Agent, Duty-Cycle and the Idle Strategy

Let's visualize a new approach of writing business logic. Your logic resides in an Agent class. Which has a method **doWork()** that get executed in a loop. 
Each iteration is called a **Duty Cycle**. And each Agent is managed by an **AgentRunner**. AgentRunner is essentially a [Runnable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Runnable.html).
Using AgentRunners static method, AgentRunner can be executed in a thread using default non-daemon ThreadFactory.

The thread will run following piece of code in an infinite loop. Until AgentRunner is closed. 

```java
final int workCount = agent.doWork();
idleStrategy.idle(workCount);
```

doWork() method returns an integer value, the **idiomatic** usage of this return value is to indicate the number of work carried out by the Agent. 
For an example number of network messages handled by the Agent during the Duty Cycle. Idle Strategy is used by the AgentRunner to decide take breaks between Duty Cycles.
If during the previous Duty Cycle doWork indicate no work has been done, then the Idle Strategy will put the thread in to sleep. 

```java
public void idle(int workCount) {
    if (workCount <= 0) {
        LockSupport.parkNanos(this.sleepPeriodNs);
    }
}
```

The rationale behind this approach is to free up the CPU cores if application has no new events to execute. Then release the CPU core by putting the thread in to sleep. 
This is what we called the voluntary context switching. It may be not the best choice of action in terms of the particular application.
But it's good for overall system performance, where multiple applications run's on same OS host. 

# References

[Agrona Showdown](https://github.com/busy-spin/aeron-showdown/tree/main/agrona-agent-samples)