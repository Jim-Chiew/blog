---
title: YBN CTF 2024 - Crypto - Hero 1
description:
draft: true
tags:
  - CTF
  - cryptography
created: 07/49/2025 21:49
updated: 23/24/2026 21:24
---
```
Crypto: Hero 1 

RSA challenge; Do you have what it takes?

Flag Format: YBN24{...}

Challenge Author: Gr0undUp
```

```python
from Crypto.Util.number import getPrime, bytes_to_long

flag = b"YBN24{????????????????????????}"

p = getPrime(256)
q = getPrime(256)
y = getPrime(256)
e = getPrime(64)
c = getPrime(32)


try:
    a = int(eval(input("a: ")))
    b = int(eval(input("b: ")))

    assert a > 0
except:
    quit()


g = q * e
n = ((a) ** (b + c)) * p * q * y

enc = pow(bytes_to_long(flag), e, n)

ct = enc * y

print("g = {}".format(g))
print("n = {}".format(n))
print("ct = {}".format(ct))
```

# Code Review:
Seems to be a modified implementation of RSA.

## Initial setup:
when prompted. we can set `a = 1`. thus simplifying  `n` to `1 * p * q * y` or `p * q * y`
```python
n = ((a) ** (b + c)) * p * q * y
```

## Retrieving `y`:
From the code above we see that `n` and `ct` both uses `y`. 
```python
n = ((a) ** (b + c)) * p * q * y
ct = enc * y
```

we know that `p`, `q`, `y` are primes. and `ct` are not factored by `p` and `q`. thus the greatest common devisor or GCD is therefore y. 
```python
from math import gcd

y = gcd(ct, n)
```

# Retrieving the rest of the perimeter:
using the same logic we can get the rest of the fields
```python
q = gcd(n, g)
p = n // (y * q)
e = g // q
```

# Decrypting the cypher text:
Using the formula of RSA to decrypt the ciphertext
```python
# text = cryp ^ d mod n
# d = d * e mod ETF(n) = 1
etf = (y-1) * (q-1) * (p-1)
d = pow(e, -1, etf)

enc = ct // y
print("Decrypted: ", long_to_bytes(pow(enc, d, n)))
```
## Resources: 
https://www.geeksforgeeks.org/computer-networks/rsa-algorithm-cryptography/


# Solutions code
```python
from math import gcd
from Crypto.Util.number import long_to_bytes

g = 1368513670940007321313035369142922130020137145341212299224064232013612470286863465551037416143739
n = 574529479575461968992868506876733714292268982183267422698802742123027496147724134375835429416499617689426089343212318094896257354507538787255157843525359348242880612441463871238640201528368363588853872969488502087382584300964332807
ct = 9213639383703656966690861813765184644503725773051514396364577088559583449914096077655568479199676939610679482625979412979087647125463922771029890239680195856732952698648857178835136043392251000153402337175737737286645670884561682338282748524590146100125875173239984800102063878409339821411314826861983245931

y = gcd(ct, n)
q = gcd(n, g)
p = n // (y * q)
e = g // q

# text = cryp ^ d mod n
# d = d * e mod ETF(n) = 1
etf = (y-1) * (q-1) * (p-1)
d = pow(e, -1, etf)

enc = ct // y
print("Decrypted: ", long_to_bytes(pow(enc, d, n)))
```

![[Hero 1.png]]
