---
layout: post
title: "ActiveMQ High Availablity"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2011-06-10T15:42:46-07:00
---

One of the features that looks great about ActiveMQ is the network of brokers. While it looks great in documentation it absolutly does not work. There are two many issues with the alogrithm it uses to hop, which ends up stranding messages in the broker as well as duplicates in a topic. So without a active-active setup the next best case becomes active-passive setup. This is done by configuring ActiveMQ to write its journal’s to a shared disk where the first broker to aquire a lock on the journal becomes the active. This does not work well too when only the shared network journal is unavailable as ActiveMQ holds on the the existing connections. Thus the only way is to write your own locking mechanism to detect unavailability of shared network drive and shutdown ActiveMQ.