---
layout: post
title:  "How to set up a Kubernetes Cluster- Part 1"
date:   2020-05-05 08:25:31 +0200
---

This is going to be a series of posts on how to set up a Kubernetes cluster from scratch.
Although there are many<sup>[1](https://aws.amazon.com/eks/),[2](https://cloud.google.com/kubernetes/),[3](https://www.oracle.com/cloud/compute/container-engine-kubernetes.html),[4](https://www.digitalocean.com/products/kubernetes/)</sup> managed Kubernetes providers, setting up a Kubernetes cluster yourself helps you to better understand the underlying workings of it and that's the goal of this series of posts.

## Introduction

### What is Kubernetes?

Kubernetes is an [open-source](https://github.com/kubernetes/kubernetes) container orchestration and management system. It was originally designed by Google in 2014 and donated to the [Cloud Native Computing Foundation](https://www.cncf.io/) in 2015.

### Why use Kubernetes?

Kubernetes provides the tools to help us with the task of running containers in a distributed environment.
Kubernetes helps us with:

- Ensuring high availability

- Automaticly recovering when a container fails.

- Auto-scaling based on metrics (for instance, CPU or memory consumption)

- Service discovery

- Automated rollouts and rollbacks

- Automatic load balancing between containers

Among many other tasks. 

Without an orchestration system for our containers, we would have to implement all of those solutions manually. Not only the resources could be better spent somewhere else (for instance, writing our application), but we would also be more prone to errors.

## Kubernetes Components




## Resources

- [Kubernetes Oficial documentation](https://kubernetes.io/docs/home/)