---
layout: post
title:  "Kubernetes: How to create a Single Node Cluster"
date:   2020-05-26 12:40:31 +0200
modified_date: 2020-05-26 15:30:00 +0200
categories: infrastructure kubernetes
comment: true
---

Recently, I decided to set up a kubernetes cluster for one of my projects and found out that most of the guides about kubernetes are about setting up multi-node environments.

This makes sense since you can't take use of all of the kubernetes features unless you have a multi-node setup. However, I did not want to go through all the work to set up a whole multi-node environment for a small project that I was not sure if I would keep developing in the future. I wanted something easier, that I could expand upon if needed.

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

Without an orchestration system for our containers, we would have to implement all of those solutions manually (or be left without them). Not only the resources could be better spent somewhere else, for instance, writing our application, but we would also be more prone to errors.

## Kubernetes Components

![](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)
_(Kubernetes cluster and its components, from the oficial kubernetes documentation)_

A Kubernetes cluster consists of an amount of worker machines (worker nodes), those will run the containers containing the applications and at least one worker node is necessary in a cluster.

Those worker nodes will host the [**Pods**](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) (basic unit of kubernetes, holds the containers that run the applications).

### Control Plane

The Control Plane is the container orchestration layer that exposes the API and interfaces to define, deploy and manage the lifecycle of containers.<sup>[1](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane)</sup>

Although Control Plane's components can be run on any machine in the cluster, in order to use the most out of Kubernetes, it's recommended to use a dedicated machine for it.

#### [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

`kube-apiserver` exposes the Kubernetes API and is the main way of interaction within the cluster.

#### [etcd](https://etcd.io/)

etcd is an [open-source](https://github.com/etcd-io/etcd) key-value store for distributed systems. It's used by Kubernetes to coordinate all the nodes in the cluster and share data with all its members.

#### kube-scheduler

The kube-scheduler is a component that assigns Pods to nodes, selecting the most suitable node for a pod by weighting in multiple factors like resources requirements, replication, etc.

#### [kube-controller-manager](https://kubernetes.io/docs/concepts/overview/components/#kube-controller-manager)

This component of the Control Plane is responsible for running controller processes. It contains multiple processes that, in order to reduce complexity, run as a single process.

#### cloud-controller-manager

This is the component that holds cloud-specific control logic, since we will not be using a cloud provider in this guide, this will not be present in our control plane.

### Worker Nodes

#### kubelet

kubelet is an agent that runs on each node in the cluster, it makes sure that containers are running in a pod.

#### kube-proxy

This is the component that maintains network rules on nodes and allows for the pods to communicate inside and outside of the cluster.

#### Container runtime

The software responsible for running the containers. Example: Docker, containerd, etc.

____ 

This first post is an introduction to the cluster architecture and its components. In the next part, we will begin setting up our cluster starting from the Control Plane.

## Resources

- [Kubernetes Oficial documentation](https://kubernetes.io/docs/home/)