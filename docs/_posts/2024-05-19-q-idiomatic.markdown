---
layout: post
title:  "Idiomatic Q-uirky"
date:   2024-05-26 13:00:00 +0800
categories: top kdb-q
author: Isuru
---

# Idiomatic Q

Computer scientist Arthur Whitney invented the programing language [Q](https://code.kx.com/q4m3/0_Overview/) and the time series database KDB+.
Which has now become the most widely used timeseries database among Quants all over the world. Major banks, hedge funds widely use this technology,
mainly due to its efficiency. 

Q is a product of radical novelty. With a language grammar that challenges norms set out by preceding languages such as Java, C.

Only way to embrace the radical novelty is approaching it in the idiomatic way. 
Best way to learn Q is to forget about what you know about your core competence language, which for me is Java. 


Like this wise monk said, **empty your cup**.

![empty-your-cpu](/assets/img/kdb-q/empty_your_cup.gif)


# Q-uirky 101

I always tell this joke that Q stands for quirky. This post is not about comprehensive guide for all data structures utility methods Q has.
But to illustrate the Q-uirkiness, the simplicity behind this sophisticated language.

![quirky](/assets/img/kdb-q/quirky.gif)

# 5 minutes crash course for Q

## Variable assignment 

**=** is the test operator

```shell
q)2=1
0b
q)
```

**:** is the assignment operator

```shell
q)q:1
q)q
1
```

## Evaluating expressions/statements

Expressions and Statements are evaluated right-to-left. 
Or more cheeky way of saying this is left-of-right.

```shell
q)show b:1+a:42
43
q)
```

This statement is interpreted as 

1. a assigned to 42
2. add 1 to a
3. Assign the result to b
4. show value of b

# Temporal types Date & Timespan

Date are represented using integers. 
2000.01.01 is the magic day which has the integer value 0.
```shell
q)2000.01.01=0
1b
q)
```

```shell
q)2000.01.01+4
2000.01.05
q)2000.01.01-365
1999.01.01
q)
```

Similar to date for timespan 00:00:00 is the integer value 0.

```shell
q)00:00:00=0
1b
q)
```

## Data structures

Only base data structure Q has is list. All Other data structures are derived using list.

List
```shell
q)1 2 3
1 2 3
```

For an example dictionary data structure can be viewed as two list with same length. 

```shell
q)`a`b`c!1 2 3
a| 1
b| 2
c| 3
q)
```