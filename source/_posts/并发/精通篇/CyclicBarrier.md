---
title: CyclicBarrier.md
date: 2020-08-10 22:46:35
categories: 并发
tags: JUC
---

### 简单介绍

CyclicBarrier就像起跑线，规定了有多少个跑道(等待的线程数)，必须等所有的选手到位(最后一个线程执行完)之后，才可以开始跑。

* CyclicBarrier初始值的设置需要与使用的数量相同，否则将会一直等待下去
* CyclicBarrier可以设置最大等待时间，超出最大等待时间抛出TimeoutException并且先运行，运行之后会破坏Barrier，抛出BrokenBarrierException。当其他线程运行时，都会抛出BrokenBarrierException