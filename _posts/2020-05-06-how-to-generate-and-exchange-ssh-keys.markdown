---
layout: page
title:  "How to generate and exchange SSH keys"
date: 2020-05-06 08:00:00 +0200
categories: security
---

For this we will be using the tool `ssh-keygen`, `ssh-keygen` is a tool to generate, manage and convert authentication keys for SSH.

The type of key to be generated can be specified with the `-t` argument. If left empty, a RSA key will be generated.

First, we need to generate a key pair on the client.

```bash
ssh-keygen -t rsa
```

You'll be prompted for a directory to store the key (press Enter to keep it in the default location) and for a passphrase. The passphrase is used to encrypt the private key file and offer an extra layer of protection in case the key is stolen. It can be changed later with the `-p` option, however, there's no way to recover a lost passphrase and a new key must be generated. (You can leave it empty for now)

Now that the key pair has been generated, all that's needed is to add the key to the server you want to connect to.

The `ssh-copy-id` tool provides us with an easy and secure tool to transfer the keys.

```bash
ssh-copy-id user@10.0.1.10
```

After that, you can test the new key and verify its success by logging into the server.

```bash
ssh user@10.0.1.10
```