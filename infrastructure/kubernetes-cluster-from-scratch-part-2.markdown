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

### Creating the etcd cluster

Log in both of the controller servers, for this you are going to execute commands in both of the controllers.

First, let's install `etcd` for both of the controllers

```bash
# Download binaries
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"

# Extract the contents
tar -xvf etcd-v3.3.5-linux-amd64.tar.gz

# Move the executables
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/

# Create some necessaries directories to run etcd
sudo mkdir -p /etc/etcd /var/lib/etcd

# Move the certificates file into the correct location
# certificate authority, kubernetes certificate key and pem
# don't move them, but copy, as they will be used later
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

Now, we need to create the systemd file to etcd. Let's make some environment variables to make it easier:

```bash
CONTROLLER0_HOST=icaro1c.mylabserver.com
CONTROLLER0_IP=172.31.98.105
CONTROLLER1_HOST=icaro2c.mylabserver.com
CONTROLLER1_IP=172.31.105.100

# has to be different for each controller
ETCD_NAME=icaro1c.mylabserver.com
# using the hostname might make it easier to identify

# internal/private ip of each of the servers
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
# this only works because the server's are amazon ec2 instances

# all the servers in the cluster
INITIAL_CLUSTER=${CONTROLLER0_HOST}=https://${CONTROLLER0_IP}:2380,${CONTROLLER1_HOST}=https://${CONTROLLER1_IP}:2380
```

Then, create the systemd unit file

```bash
cat << EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${INITIAL_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

After the creation of the systemd unit file, it's necessary to start and enable the etcd service:

```bash
# necessary to do every time we modify/add a new unit file, so systemd see the changes
sudo systemctl daemon-reload 

# start automatically every time the server comes up
sudo systemctl enable etcd

# start the etcd service
sudo systemctl start etcd
```

Verify if it's working by running:

```bash
# check if etcd is running
sudo systemctl status etcd

# get a list of all the etcd hosts
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

The last command should display both servers, showing that the etcd cluster has been created correctly.

## References

- [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)