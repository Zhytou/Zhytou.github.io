---
title: "Stanford CS144-Lab总结"
date: 2023-08-04T22:18:06+08:00
draft: true
---

> [CS144: Introduction to Computer Networking](https://cs144.github.io/)

## 1 Networking Warmup

前面的就不同

## 2 Stitching Substrings Into A Byte Stream

一开始使用`map<uint64_t, char>`来表示滑动窗口，但后面发现这样虽然逻辑上没有问题，但是针对test13特别容易超时。于是后面还是将滑动窗口的数据结构修改成`map<uint64_t, string>`，维护待写入字符串，避免一个一个位置处理。

## 3 The TCP Receiver
