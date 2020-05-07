---
layout: page
title:  "Cryptography"
date: 2020-05-05 20:00:00 +0200
modified_date: 2020-05-05 20:30:00 +0200
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

#### Hashing Algorithms

Hashing operates by taking a variable amount of data and compressing it into a fixed length value, refered to as a hash value.

- MD5: processes a variable-size input and produces a 128-bit output.

- Secure Hashing Algorithm (SHA) is very similar to MD5. SHA-0 and SHA-1 have been deprecated. SHA-2 is a family of function are a safe replacement. (SHA-224, SHA-256, SHA-386, SHA-512, where each produces a message digest of the number of bits that it contains in its name. SHA-256 is the most common)

- Hashed Message Authentication Code (HMAC) was designed to avoid collisions attacks that other hashing algorithms are susceptible to.

### Encryption Algorithms

#### Symmetric

Symmetric algorithms are those that use the same key to encrypt and decrypt the data. A good analogy to understand symmetric encryption is thinking of a physical lock and key, the same key is used to both lock and unlock the door.

Examples of symmetric algorithms:

- Data Encryption Standard (DES) was the first commonly used symmetrical algorithm. It uses 64-bit keys, but 8 bits of those are used for parity, which means only 56 bits left for the actual key.

- Triple-DES (3DES): DES started to become insufficient once the avaibility of computional power increased and brute-force attacks became more and more feasible. So DES was deprecated in 1998 and 3DES was created to fill the hole left behind while looking for a new better solution. 3DES encrypts with 112 or 168 bits, depending on the number of keys used.
    - DES EEE2: uses 2 keys, the encryption process happens three times, alternating keys.
    - DES EDE2: uses 2 keys, the first is used twice. Encryption-decryption-encryption.
    - DES EEE3: uses 3 keys, encryption happens 3 times, each time with a different key.
    - DES EDE3: uses 3 keys, operates by encrypting, decrypting, encrypting.

- Advanced Encryption Standard (AES): In 2002, it was chosen as the replacement for DES and it's now the most popular symmetric encryption algorithm. It can be deployed in key sizes of 128. 192 or 256 bits.

- RC4 is a fast stream cipher, well suited for Wi-fi Equivalent Protection (WEP). It's 40 bits in length, but it's paired with a 24 bit IV to create a 64-bit WEP.

Symmetric algorithms have some shortcomings which might be an issue depending on the situation. For instance, how do you exchange the key in a secure way? This problem is solved with the use of [asymmetric encryption](#Asymmetric).

##### Cipher Modes

- Counter Mode (CTR)

- Electronic Codebook (ECB)

- Galois/Counter Mode (GCM)

- Cipher Block Chaining (CBC)

#### Asymmetric

Instead of using the same key to encrypt and decrypt the data, instead, asymmetric algorithms use two keys, one to encrypt and another one to decrypt the data. Those keys are named, respectively: public and private keys.

The private key is kept a secret from anyone else while public keys are openly shared. If one wants to transmit encrypted data, it first encrypts it using the recipient's public key. Since both keys are mathematically related, only those in posession of the private key will be able to decrypt the data and see its content.

For more about how it works internally: [How does RSA encryption work?](https://www.comparitech.com/blog/information-security/rsa-encryption/)

Some asymmetric algorithms:

- Diffie-Hellman was the first public key-exchange algorithm, it allows two users to exchange a secret key over an insecure medium without any prior secrets. It is, however, susceptible to Man in the Middle attacks, as there is no authentication from either participant.

- RSA is the de facto standard for industrial-strenght encryption.

- Digital Signature Algorithm (DSA): Great for determining the integrity of data, but does little to provide confidentiality. Used to provide digital signatures.

- Elliptical Curve Cryptosystem (ECC) is considered more secure as it is harder to crack based on the discrete log problems it employs. 

### Confusion and Difusion

- Confusion refers to the process of making the relationships between encrypted data and the cipher as complex as possible.

- Difusion refers to the process of shifting bits of data throughout the encryption process. As single bits of the plaintext shift, half of the bits of ciphertext should change, thus making the decryption process rely on having the correct key, as capturing bits of the ciphertext and plaintext would not allow reverse engineering.

## References

- [Cryptography on wikipedia](https://en.wikipedia.org/wiki/Cryptography)
- [CompTIA Security+ Certification Prep by Justin Mitchell](https://linuxacademy.com/cp/modules/view/id/264)
- [Wikipedia page on 3DES](https://en.wikipedia.org/wiki/Triple_DES)
- [What is asymmetric encryption?](https://medium.com/@FreedomBen/what-is-asymmetric-encryption-64c74b2a0a82)