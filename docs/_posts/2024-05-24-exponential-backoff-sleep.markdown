---
layout: post
title:  "Agrona Idle Strategy"
date:   2024-05-23 06:00:00 +0800
categories: performance-engineering top
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

# Simple example 

We have simple Agent implementation called **RandomWorkCountAgent** which simulate Agent with duty cycles which work has been carried out 
and also duty cycles where no work has been carried out.

```java
import org.agrona.concurrent.Agent;
import java.util.Random;

public class RandomWorkCountAgent implements Agent {

    private final Random random = new Random();

    @Override
    public void onStart() {
        System.out.printf("[%s], Agent started\n", Thread.currentThread().getName());
    }

    @Override
    public int doWork() throws Exception {
        return random.nextInt(0, 3);
    }

    @Override
    public void onClose() {
        Agent.super.onClose();
    }

    @Override
    public String roleName() {
        return "random-agent";
    }
}
```

Idle strategy implementation is **LogSleepIdleStrategy** where is work count is zero it will make the thread sleep for 2 seconds.
And if work count is greater than zero, log the work count and return. 


```java
import org.agrona.concurrent.IdleStrategy;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

public class LogSleepIdleStrategy implements IdleStrategy {
    @Override
    public void idle(int workCount) {
        if (workCount <= 0) {
            System.out.printf("[%s], Going for a sleep\n", Thread.currentThread().getName());
            LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2L));
        } else {
            System.out.printf("[%s] - work count = %d\n", Thread.currentThread().getName(), workCount);
        }
    }

    @Override
    public void idle() {
    }

    @Override
    public void reset() {
    }

    @Override
    public String alias() {
        return IdleStrategy.super.alias();
    }
}
```
Main program. 

```java
import org.agrona.concurrent.AgentRunner;

public class SimpleAgentSample {
    public static void main(String[] args) {
        AgentRunner agentRunner = new AgentRunner(new LogSleepIdleStrategy(),
                throwable -> {}, null, new RandomWorkCountAgent());
        AgentRunner.startOnThread(agentRunner);
    }
}
```

Once you run the program you may observe something similar to bellow output. Where Idle strategy logs work count or indicate thread sleep. 
In duty cycles where work count is zero. Thread sleep before execute the next duty cycle. 

```shell
[random-agent], Agent started
[random-agent] - work count = 1
[random-agent] - work count = 2
[random-agent], Going for a sleep
[random-agent] - work count = 1
[random-agent] - work count = 2
[random-agent] - work count = 2
[random-agent] - work count = 2
[random-agent] - work count = 1
[random-agent] - work count = 1
[random-agent] - work count = 2
[random-agent] - work count = 1
[random-agent] - work count = 2
[random-agent], Going for a sleep
[random-agent] - work count = 1
[random-agent], Going for a sleep
```

# Is this a sensible approach

Answer to that question simply is **NO**. Our objective is to process events as fast as possible. Just because a duty cycle reported no work has been done, 
does not imply that next duty cycle will also have no events or work to be done. By making thread move to voluntary context switch we waste lot of CPU cycles and reload the CPU instructions. 
But it may be also impossible to predict how work count in one duty cycle could be used to predict the work count for next duty cycle. 

There are many Idle wait strategies Agrona provide to cater different use cases. You can use them to be more CPU conservative or less CPU conservative. 
Choice is really up to you. Each come with their own perks. 

However, decoupling the CPU conservativeness from business logic in my opinion is what makes Agrona Agent's standout against other threading libraries. 

# Writing Custom Idle Wait Strategy

Imagine extra CPU conservative idle strategy, in which the idle time exponentially increase with more and more duty cycles reporting no work has been done.
This strategy is a suitable candidate for network reconnection, or low throughput high latency message processing. 

As illustrated bellow, after first duty cycle with zero work count, thread sleep for 100ms, and as duty cycles keep producing zero work count. The sleep time increase to 200ms, 400ms.
We usually cap the maximum wait time as the wait time will increase to very large value by few iterations. And that big of a wait is meaningless. 

![backoff](/assets/img/agrona-agents/backoff.png)



```java
import org.agrona.concurrent.IdleStrategy;
import java.util.concurrent.locks.LockSupport;

public class ExponentialBackOffIdleStrategy implements IdleStrategy {

    private final long maxBackOff;

    private final long initialBackoff;

    private long backoffCounter;

    private long currentBackOff = 0;

    public ExponentialBackOffIdleStrategy(long initialBackoff, long maxBackOff) {
        this.maxBackOff = maxBackOff;
        this.initialBackoff = initialBackoff;
    }


    @Override
    public void idle(int workCount) {
        if (workCount > 0) {
            backoffCounter = 0;
            currentBackOff = 0;
        } else {
            if (currentBackOff < maxBackOff) {
                long sleepTime = initialBackoff * Math.round(Math.pow(2, backoffCounter));
                backoffCounter++;
                currentBackOff = Math.min(sleepTime, maxBackOff);
                System.out.printf("A new backoff calculated %dms\n", Math.round(currentBackOff * 1e-6));
            }

            LockSupport.parkNanos(currentBackOff);
        }
    }

    @Override
    public void idle() {
        LockSupport.parkNanos(this.initialBackoff);
    }

    @Override
    public void reset() {

    }

    @Override
    public String alias() {
        return IdleStrategy.super.alias();
    }
}
```

# References

[Agrona Showdown](https://github.com/busy-spin/aeron-showdown/tree/main/agrona-agent-samples)