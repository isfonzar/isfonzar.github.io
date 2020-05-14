---
layout: post
title:  "Kubernetes Cluster from Scratch - Part 5"
date:   2019-12-27 08:25:31 +0200
categories: infrastructure devops kubernetes
---

## Setting up Kubernetes Remote Access and kubectl

We've already used kubectl to generate the kubeconfigs and some verification steps.

Kubectl is the kubernetes command line tool, it allows us to interact with Kubernetes clusters from the command line.

We will set up kubectl to allow remote access from our machine in order to manage the cluster remotely.

To do this, we will generate a local kubeconfig that will authenticate as the admin user and access the Kubernetes API through the load balancer.

#### Resources

- [Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)

## Configuring kubectl for remote access

Open a ssh tunnel to port 6443 on the kubernetes API load balancer.

```bash
ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
```