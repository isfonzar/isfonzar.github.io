## Wireless Cryptographic Protocols

WEP uses an RC4 stream cipher for both authentication and encryption.

The key is shared with every device on the wireless network.

The key is only 40 bits (standard) or 104 bits (commercial), but is combine with a 24-bit initialization vector (IV) making the total key size 64 or 128 bits.

Since the IV is so small, this leads to it being reused, which in turn, makes it easier to crack.

For these reasons, WPA was introduced as the successor to WEP.

### WPA (Wifi-Protected Access)

WPA is also based on the RC4 cipher, however with several enhancements over it's predecessor WEP.
Especially the use of Temporal Key Integrity Protocol (TKIP). TKIP has a set of functions to improve wireless security.

- Use of 256-bit keys

- Per-packet key mixing (each packet has a unique key generated for it)

- Automatic broadcast of updated keys

- Message integrity checker

- Larger IV size (48 bits)

### WPA2

WPA2 replaces the RC4 cipher and TKIP with two stronger encryption and authentication mechanisms: AES and Counter Mode with Cipher Block Chaining Message Authentication Code Protocol (CCMP).

WPA2 also supports TKIP as a fallback if a device cannot support CCMP.

It protects data confidentiality by allowing only authorized network users to receive data, and uses CCMP's block chaining message authentication code to ensure message integrity.

WPA2 also introduced easier roaming, allowing clients to move from one access point to another without reauthentication (preauthentication)

## Wireless Authentication Protocols

### Extensible Authentication Protocol (EAP)

More of a framework, used to create different types of authentication.

### EAP-FAST

The first widely deployed EAP method was LEAP, however it contained some serious security vulnerabilities, so to address these, Cisco developed EAP-FAST (Flexible Authentication via Secure Tunneling). EAP-FAST addresses these vulnerabilities by performing this authentication over a Transport Layer Security (TLS) tunnel.

### EAP-TLS

EAP over a TLS connection.

### PEAP

Protected EAP, refers to sending EAP within an encrypted and authenticated TLS tunnel.

### IEEE 802.1X

Is a port-based network access control. You cannot have access to network resources until you have completed the authentication process.
This is usually used in conjunction with some previously mentioned authentication process from our IAM lessons.

3 devices involved in authentication:
- Device requesting access, the supplicant
- The switch (or wireless AP), sometimes referred as the authenticator
- Authentication server, pushes authentication requests to this server to verify the requestor.