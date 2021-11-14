---
title: Semaphore.md
date: 2020-08-10 22:46:35
categories: 并发
tags: JUC
---



### 使用信号量的注意点

* 获取和释放的数量保持一致
* 注意在初始化Semaphore的时候设置公平性，一般设置为true会更合理
* 并不是必须由获取许可证的线程释放那个许可证，

