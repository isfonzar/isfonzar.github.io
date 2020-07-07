---
layout: post
title:  "System Design Foundamentals: Reliability, Scalability and Maintainability"
date:   2020-06-03 22:00:00 +0200
categories: system-design
---

There are many factors that influence the design of an application, however there are three qualitites that are important in most software systems: reliability, scalability and maintainability.

In this post, I'm going to review those attributes and give a brief overview of what each one of them means.

## Reliability

Reliability is the concern that an application will perform its intended functions satisfactoryly, even when faults or errors occur.

To put it in another words it means that the system will continue to work correctly even in the face of adversities.

A system that is prepared to properly handle faults is called a _fault-tolerant_ or _resilient_ system.
A fault differs from a failure in the sense that a fault is usually defined as one component of the system deviating from its spec, while failure is when the system as a whole stops providing the required service.

It is impossible to prepare for every possible kind of fault, it is however advisible to design fault-tolerance mechanisms to prevent faults from causing failures (resulting in the service becoming unavailable)

Many failures happen due to pool error handling, so many companies choose nowadays to take a proactive approach to faults and try to increase the rate in which they happen. One example of this is
[Netflix's Chaos Monkey](https://github.com/Netflix/chaosmonkey), which randomly terminates instances with the purpose of discovering faults and guarantee that services are resilients to them.

### Hardware Reliability

Hardware issues happen all the time, and the mechanisms to cope with that vary from disks set up in RAID configuration, servers with dual power supplies, hot-swappable CPUs, and datacenters with diesel generators for backup power.

This is all done so that when a component inevitably dies, a redundant component can take its place while the broken component is replaced. The lower the downtime the better.

Again, it's impossible to prevent those hardware issues from happening, but reliability means that we aim to prevent those problems from causing failures.

There is a move towards systems that can tolerate loss of components and even entire machines. 

### Software Errors

## Scalability

Scalability means that as the traffic, complexity and data volume grow, there are reasonable ways of dealing with that growth.

## Maintainability

Maintainability means that different people are able to work on the system productively, maintaining the current behavior as well as improving it.