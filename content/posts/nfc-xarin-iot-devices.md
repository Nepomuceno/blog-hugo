---
title: Using Xamarin and NFC for IOT devices.
date: 2018/12/14 01:01:01
draft: true
---

One of the largest challenges when we are talking about IOT it is the cpability of giving maitenance of the devices. Most of the time this can be done over the air or the devices come in pre programed from the manufacturer. But when that is not the case havinv  device that can accpet nfc as a way to program it can be really useful.

This blog post is to explain how can you use Xamarin and IOThub to build an app that can reprogram those devices and provision thousands of them without the need to pay extra for the manufactured to built in your specifications on it.

The full source code demonstrated here can be found at [Elsys Xamarin deployer](http://www.com)

<!-- more -->

## Architecture and deployment

The idea it is really simple we are going to have a bunch of devices comminucating through LoraWan to a gateway that will be passing this reading to IOTHub you can find the code used by Azure Functions to decode this messages in our case at: [Functions provisioner decoder][1].

<!-- Drawning of the architecture -->

In order to setup our devices when we are in the fild we are going to place the device them tap it with a phone. after the device has been tapped it will try to joing our network and start sending data.

## Building the app

First step it is to create a xamaring app we are going to be using the Monodroid solution since Apple does not expose their [NFC for third party developers][2].

After creatign our one page app we need to add NFC to it so you need to include in your manifext file.
