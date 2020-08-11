---
title: 浅析equals()和hashcode()方法
date: 2020-08-06 10:11:18
tags:
---
需要注意的是当equals()方法被override时，hashCode()也要被override。按照一般hashCode()方法的实现来说，相等的对象，它们的hash code一定相等。