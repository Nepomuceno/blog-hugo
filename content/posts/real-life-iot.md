---
title: Dealing with real life IOT
date: 2018-01-29 01:01:01
---

When you start working with IOT scenarion on the real work you realise that the combination of changing firsmware updates dealing with battery life and remote devices can be way more complex than you first imagine. In this post I will be exploring a huge deployment with remote devices that need to last for years and communicate over long distances.

<!--more-->

## Challenge size

Just stating the problem this company is planing for hundreds of thousands of devices to be deployed over 2019 and we were trying to prove that the proof of concept would work.

For the device we were aiming for devices that could be deployed than not touched for years in a row. so the technology that we end up choosing was [LoraWan](https://www.thethingsnetwork.org/docs/lorawan/) due to the fact that the devices consume low energy and can communicate over long distances but more than that, devices that have 2 way communications built in.

## Setting up the architecture

Since we are using LoraWan devices those devices broadcast their signal and can be 