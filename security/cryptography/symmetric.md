---
layout: page
title:  "Symmetric Encryption"
date: 2020-05-05 20:00:00 +0200
modified_date: 2020-05-05 20:30:00 +0200
---

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

Symmetric algorithms have some shortcomings which might be an issue depending on the situation. For instance, how do you exchange the key in a secure way? This problem is solved with the use of [asymmetric encryption](/security/cryptography#Asymmetric).