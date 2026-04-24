---
title: Creating Self-Signed SSL Certificates
description:
draft: false
tags:
  - ssl/tls
  - web
  - cryptography
  - ssl
created: 17/25/2025 22:25
updated: 23/00/2026 21:00
---
# Generate the Private Key:
```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key -aes256
```
Common algorithms: `RSA`, `DSA`, `DH`, `EC`, `ED25519`, `ED448` `X25519`, `X448`
Encryption algorithms: `-aes256`, `-aes128`, `-des3`, `-des`, `-idea`
RSA key size options:  `1024`, `2048`, `3072`, `4096`

# Generating a certificate:
```bash
openssl req -x509 -newkey rsa:2048 -key server.key -out server.crt -sha256 -days 1095 
```

# Convert to PKCS12
```bash
openssl pkcs12 -inkey server.key -in server.crt -export -out server.pfx
```

# Resource:
https://tecadmin.net/step-by-step-guide-to-creating-self-signed-ssl-certificates/
https://www.ssldragon.com/how-to/openssl/create-self-signed-certificate-openssl/
https://www.baeldung.com/openssl-self-signed-cert
https://stackoverflow.com/questions/10175812/how-can-i-generate-a-self-signed-ssl-certificate-using-openssl

https://docs.openssl.org/3.0/man1/openssl-req/#examples
https://docs.openssl.org/3.0/man1/openssl-genpkey/
https://docs.openssl.org/3.0/man1/openssl-genpkey/#parameter-generation-options

https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/
https://dev.to/techschoolguru/how-to-create-sign-ssl-tls-certificates-2aai
https://medium.com/@talyitzhak/understanding-digital-certificates-and-self-signed-certificates-b1cdca759bbc