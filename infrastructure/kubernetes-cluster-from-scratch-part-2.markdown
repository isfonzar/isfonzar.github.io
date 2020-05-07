---
layout: post
title:  "Kubernetes Cluster from Scratch - Part 2"
date:   2019-12-27 08:25:31 +0200
categories: infrastructure devops kubernetes
---

## Kubeconfigs

Kubeconfigs are Kubernetes configuration file, it stores information about clusters, users, namespaces and authentication mechanisms. It contains the configuration data that we need in order to connect and to interact with a Kubernetes cluster.

## Requirements

To generate the kubeconfig files, we need to have the `kubectl` tool installed

```bash
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
$ kubectl version --client # check the installation
```

## Generating the required kubeconfigs

```bash
# Ip address of the Kubernetes API
KUBERNETES_ADDRESS=172.34.2.0

WORKER0_HOST=worker0.mylabserver.com
WORKER0_IP=172.34.1.0
WORKER1_HOST=worker1.mylabserver.com
WORKER1_IP=172.34.1.1

CONTROLLER0_IP=172.34.0.0
CONTROLLER1_IP=172.34.0.1

# Specify the user you will be connecting to those servers with
CLOUD_USER=cloud_user
```

```bash
for instance in ${WORKER0_HOST} ${WORKER1_HOST}; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

Now, we generate the kubeconfig for kubeproxy. Only 1 file is needed for all the workers as they will share this file.

```bash
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

Now, for the kube controller manager, here we use the default IP for localhost, the kube-controller-manager will not go through the load-balancer, but will access API from whatever host it is in.

```bash
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```

The kube-scheduler kubeconfig:

```bash
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
```

And lastly for the admin user:

```bash
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
```

If you open one of those files we just generate, you can see the certificate authority, the server (load balancer IP), username and certificates for the client. All the information we generated on Part 1 is now present within those files.

## Distributing the kubeconfig files

Moving the kubeconfig to the worker nodes

```bash
scp ${WORKER0_HOST}.kubeconfig kube-proxy.kubeconfig ${CLOUD_USER}@${WORKER0_IP}:~/
scp ${WORKER1_HOST}.kubeconfig kube-proxy.kubeconfig ${CLOUD_USER}@${WORKER1_IP}:~/
```

Moving the kubeconfig files to the controller nodes:

```bash
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${CLOUD_USER}@${CONTROLLER0_IP}:~/
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${CLOUD_USER}@${CONTROLLER1_IP}:~/
```

## Generating the Kubernetes Data Encryption Config

Kubernetes supports the ability to encrypt secret data at rest, so any sensitive data is always encrypted. In order to make use of this feature, we need to provide kubernetes with an encryption key.

```bash
CONTROLLER0_IP=10.0.1.39
CONTROLLER1_IP=10.0.1.90

CLOUD_USER=cloud_user

# Generate a random encryption key
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

Now, let's create the encryption config file:

```bash
cat > encryption-config.yaml << EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

And lastly, let's copy this encryption config file to the controller servers.

```bash
scp encryption-config.yaml ${CLOUD_USER}@${CONTROLLER0_IP}:~/
scp encryption-config.yaml ${CLOUD_USER}@${CONTROLLER1_IP}:~/
```

## Bootstrapping the etcd cluster

[etcd](https://github.com/coreos/etcd) is a distributed key value store that provides a reliable way to store data across a cluster of machines.

It provides a way to store data across a distributed cluster of machines and make sure the data is synchronized across all machines.

Kubernetes uses etcd to store all of its internal data about cluster state.

This data needs to be stored, but it also needs to be reliably synchronized across all controller nodes in the cluster.

We just need to install it on our controller nodes. 

More information on the official kubernetes documentation on [configuring etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)



## References

- [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)