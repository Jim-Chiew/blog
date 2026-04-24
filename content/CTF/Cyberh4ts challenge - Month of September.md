---
title: Cyberh4ts challenge - Month of September
description:
draft: false
tags:
  - cryptography
  - CTF
created: 15/37/2025 19:37
updated: 23/02/2026 21:02
---
```
Background: 
You’ve encountered a server claiming to use a one-time pad for encryption. It gives you a cypher flag and lets you submit a plaintext to get a ciphertext. But something feels off. In reality, the key is reused within the same connection, which means the encryption scheme isn’t as secure as it seems. Can you exploit this flaw to recover the flag?
🖥 nc 34.31.1.240 1337
```

Interaction:
```bash
┌──(kali㉿kali)-[~]
└─$ nc 34.31.1.240 1337

=== XORacle: The XOR Encryption Oracle ===
[+] Generating Random Key...
[!] Same key is reused for this connection.

Flag ciphertext: 1a8a385c85dea4fb5a625543195bcf9521eca1c70a04902d4e0ad5494169d1

Give it a try! Enter plaintext:
> this is a test 
cipher = 2d9b334ad7ffe3af48397916181e

Query limit reached. Goodbye.
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ nc 34.31.1.240 1337

=== XORacle: The XOR Encryption Oracle ===
[+] Generating Random Key...
[!] Same key is reused for this connection.

Flag ciphertext: c50940fb774ca5ae3fc02550c492279fbedd00d251bd2c9afab7c9cc5b4cf1

Give it a try! Enter plaintext:
> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
cipher = c73163df4445d09b0dfa3c21f7e257b098c338a3658e32ac888fd7bd6f79cdc3042669e9df99c1c4

Query limit reached. Goodbye.
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ nc 34.31.1.240 1337

=== XORacle: The XOR Encryption Oracle ===
[+] Generating Random Key...
[!] Same key is reused for this connection.

Flag ciphertext: d0be4a369b8da193d3ca48335b70fa0e44eef6a2e56778f5f3a5b9cfa3a08d

Give it a try! Enter plaintext:
> d0be4a369b8da193d3ca48335b70fa0e44eef6a2e56778f5f3a5b9cfa3a08d
cipher = f7f74a36dda4a6d199d328674870f2534782ecf3a42d14b1f5bed1cfb0b5c08f9682fb365c451f2aa27c09e5f7c61da184d9e85643594271cdc86910cedd

Query limit reached. Goodbye.
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ nc 34.31.1.240 1337

=== XORacle: The XOR Encryption Oracle ===
[+] Generating Random Key...
[!] Same key is reused for this connection.

Flag ciphertext: 1854f6b9ae45f662cbdc368e700c592bd835c315cb9399b1f2dedd344451c7

Give it a try! Enter plaintext:
> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
cipher = 1a6cd59d9d4c8357f9e62fff437c2904fe2bfb64ffa0878780e6c3457064fb21927e1d09daf9376a04c98ca2e2d64cd237702b7b2e68f6efa7ba275041f5972f454d71ff8bd0027a4ac0ad4812ce2d2add18939cfddcf982e895763852399a7e0cb3119bad5168e1449671a7f0520b8b4ab732e38fb2a92f085c01

Query limit reached. Goodbye.
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ nc 34.31.1.240 1337

=== XORacle: The XOR Encryption Oracle ===
[+] Generating Random Key...
[!] Same key is reused for this connection.

Flag ciphertext: b9b1cdbae5e4d5ea60041958a56d29484801e362a74caca81d37268f534265

Give it a try! Enter plaintext:
> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
cipher = bb89ee9ed6eda0df523e0029961d59676e1fdb13937fb29e6f0f38fe6777594732316c283b3287987bef3e6c54d81cbbe34fefab0ddb97744214336388eb9b7f8d5cf847d88e664a7f13f37c571c9816a9d204bb4b21104376e2a8bcd05edf60a974a1432ce2c796d3b009b0b7acc1b5e731e52185d4b1ea85eaff

Query limit reached. Goodbye.
```

# Observations: 
1. It is an XOR Encryption Oracle. 
2. It encrypts the text we provide with the same encryption key.
3. XOR Encryption allows us get the same data back when key is provided. But provide the data, we will get the key. 
4. Output seems to be either HEX or Base64.

# Getting the key: 
![[cybperh4ts September 1.png]]

# Getting the flag: 
![[cyberh4ts - September 2.png]]

