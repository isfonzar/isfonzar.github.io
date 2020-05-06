---
layout: page
title:  "Public Key Infrastructure"
date: 2020-05-05 20:00:00 +0200
modified_date: 2020-05-05 20:30:00 +0200
---

Public Key Infrastructure (PKI) is not a specific technology, but a framework.

It uses asymmetric algorityhms for communications, but certificates are issued by a trusted Certificate Authority (CA).

PKI provides confidentiality, authentication and confidentiality.

- Authentication is provided because the digital signature of sender is required to send messages.
- It provides confidentiality through data being encrypted before being transmitted.
- Non-repudiation is provided through an assurance that both the sender sent the message and the receiver did in fact, receive the message.

A sender encrypts the message with the recipient's public key. The message is then sent to the receipter, which uses its private key to decrypt the mesage.

The content of the message is protected because only someone with the private key can decrypt it.

In a PKI, the sender will also digitally sign the message using **his own private key**. The message is sent to the receiver, and the receiver will use the sender's **public key** to verify the signature. This confirms the identity of the sender.

### Certificate Types

- Root Certificate: this is the most important certificate in a PKI environment, it identifies the root Certificate Authority (CA). Everything in the environment starts with this certificate. The root certificate will issue all other certificates. It allows for the creation of any trusted certificate.

- User Certificates are used to associate a user with a certificate. This acts as a digital identification for an user.

#### Web Server SSL Certificates

- Domain Validation (DV) when using a DV certificate, the CA verifies that the applicant has been validated by proving some control over the DNS domain associated with it.

- Extended Validation (EV): when using an EV ceritifcate, the CA provides verification of the requestor's identity to verify that they are the legal entity controling the website.

- Subject Alternate Name (SAN) this certificate allows you to put a subject alternate name extension, where you can list all of the domain names that would be associated with the certificate.

#### Internal Certificates

- Self-signed certificates: If the server is only acessed inside the network, so we can establish our own CA.

#### Certificate File Types

- DER: Distinguished Encoding Rule

- PEM: Privacy-Enhanced Mail

- PFX: Binary format for storing a server certificate

- CER: contains only the public key.

- P7B

Any of these forms can be converted to any other form using, for instance, OpenSSL.

### Components

- Digital certificates are data files that are used to link an entity (user or system) with a public key. Used to verify the identity of the linked entity. Also contain information about the issuer of that certificate, so that the certificate itself can be validated.

- Certificate Signing Request (CSR)

- Certificate Authority (CA): trusted entity that issues digital certificates

- Certificate Revocation List (CRL): list of certificates that have been revoked by the CA.

- Online Certificate Status Protocol (OCSP) to check the status of certificates, OCSP verifies the status of the requested certificate, this allows systems to verify that certificates have not been added to the CRL.

## References

- [CompTIA Security+ Certification Prep by Justin Mitchell](https://linuxacademy.com/cp/modules/view/id/264)