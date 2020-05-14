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
LOAD_BALANCER_IP=172.31.108.99
CLOUD_USER=cloud_user

ssh -L 6443:localhost:6443 ${CLOUD_USER}@${LOAD_BALANCER_IP}
```

```bash
cd ~/kthw

# set up a new kubernetes cluster in the local configuration for kubectl
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://localhost:6443

# this is going to work because of the ssh tunnel, the traffic will be forwarded through the ssh tunnel to the load balancer
kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem

# a context is just a set of data we use to connect to a server
kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin

kubectl config use-context kubernetes-the-hard-way
```

Then, we can verify that everything is working:

```bash
# for now, there are no pods, but it should return no resources found
kubectl get pods

# should return the two worker nodes
kubectl get nodes

# will return client and server version
kubectl version
```

## Networking

Kubernetes provides a powerful networking model which allows pods to communicate with one another over a virtual network.

### What problems does the networking model solve?

- How will containers communicate with each other?

- What if the containers are on different hosts (worker nodes)?

- How will containers communicate with services?

- How will containers be assigned unique IP addresses?

### The Docker Model

Kubernetes Model is a response to the Docker Model

Docker allows containers to communicate with one another using a virtual network bridge configured on the host.

Each host has its own virtual network serving all the containers on that host. But we run on problems when we have containers on different hosts, then we have to proxy the traffic from the host to the containers, also making sure that no two containers use the same port on a host.

The Kubernetes networking model was created to overcome those limitations.

### The Kubernetes Networking Model

- One virtual network for the whole cluster

- Each pod has a unique IP within the cluster

- Each service has a unique IP that is in a different range than pod IPs.

#### Resources

- [Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

## Cluster Network Architecture

CIDR is a way of allocating IP addresses, and it has a way to specifying a CIDR range.

- Cluster CIDR: value that we will pass to kubernetes and it tells kubernetes what ip range to use when setting up the network (we've used it already when setting up some services: example: systemd unit file for the kube-controller-manager, you can see the flag, cluster cidr)

- Service Cluster IP Range: ip range for services in the cluster, should **not** overlap with the cluster CIDR range.

- Pod CIDR: we don't specify a pod CIDR manually in this guide. the pod CIDR is an ip range for pods on a **specific worker node**, need to fall within the cluster CIDR, but can't overlap with the pod CIDR for any other nodes. So we are allocating to each node a range of IPs.
    - Our networking solution automatically handles this.

#### Resources

- [Weave](https://github.com/weaveworks/weave)