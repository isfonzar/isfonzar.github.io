---
layout: page
title:  "Cryptography"
date: 2020-05-05 20:00:00 +0200
modified_date: 2020-05-05 20:30:00 +0200
categories: security
---

The word `cryptography` is derived from 2 greek words: _`kryptós`_ meaning "secret" or "hidden" and _`graphein`_ which means "writings".

Cryptography is the practice and study of techniques for secure communication. The objectives of Cryptography are:

- Privacy: Ensuring that data that is private remains private.

- Authentication: Cryptography helps us confirm that individuals are who they say they are.

- Integrity: Cryptography helps us assure that the data has not been modified by a third-party.

- Non-repudiation: Both the sender and the receiver can be sure of that status of the other one in a transaction.

## Basics of Cryptography

### Cipher

Algorithm used to encrypt and decrypt data. Ciphers are used when we need to obtain the original data.

#### Block Cipher

#### Stream Cipher

### Hashes

Hashes are used when there's no need to obtain the original data again, meaning the encryption happens one-way only.

A good example of this is proper password management, in which the password is hashed and the hash is saved in a database. Whenever we want to confirm if an inputed password matches the one we have saved, we hash the input and compare the hashes. The original password cannot be obtained from the hash.

### Encryption Algorithms

#### Symmetric

The same key is used to both encrypt and decrypt data. Examples: DES, AES, 3DES, etc.

#### Asymetric

Uses two keys, one to encrypt and another one to decrypt the data. Examples: RSA, DSA, elyptical curve, etc.

### Confusion and Difusion

- Confusion refers to the process of making the relationships between encrypted data and the cipher as complex as possible.

- Difusion refers to the process of shifting bits of data throughout the encryption process. As single bits of the plaintext shift, half of the bits of ciphertext should change, thus making the decryption process rely on having the correct key, as capturing bits of the ciphertext and plaintext would not allow reverse engineering.

## References

- [Cryptography on wikipedia](https://en.wikipedia.org/wiki/Cryptography)
- [CompTIA Security+ Certification Prep by Justin Mitchell](https://linuxacademy.com/cp/modules/view/id/264)