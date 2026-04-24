---
title: HTB Challenge - Digital Safety Annex
description:
draft: false
tags:
  - HackTheBox
  - CTF
created: 20/54/2025 08:54
updated: 23/08/2026 21:08
---
# Overview: 
The improper implementation of generating the nonce(k) in DSA allowed the private key(x) to be derived from the signature. 

# DIGITAL SIGNATURE STANDARD (DSS): Digital Signature Algorithm (DSA):
As the name suggest, it is a digital signature generation and verification

## How it works:
The following 2 websites goes in detail on how it works:
[FIPS PUB 186: DIGITAL SIGNATURE STANDARD (DSS)](https://nvlpubs.nist.gov/nistpubs/Legacy/FIPS/fipspub186.pdf)
[Digital Signature Algorithm (DSA)](https://www.geeksforgeeks.org/digital-signature-algorithm-dsa/)

Few perimeters to note:
Public perimeters: 
	- p (prime modulus)
	- q (prime devisor)
	- g (generator)
	- y (public key)
Private perimeter:
	- x (private key)
	- k (number to only use once (nonce))
Output of DSA:
	- r
	- s

Main note: 
- Nonce(k) should only be used once for every new signing.
- Nonce(k) should be random/pseudorandom and unpredictable.

# Code Review:
## General information:
When accessing the service you will be prompted with the following: 
```
Welcome to the Digital Safety Annex!
We will keep your data safe so you don't have to worry!

[0] Create Account
[1] Store Secret
[2] Verify Secret
[3] Download Secret
[4] Developer Note
[5] Exit

[+] Option >
```

Getting global information for DSA:
First of is getting p, q, g. looking at the code we can get it with option 4: used the hardcoded password from code provided: Admin:5up3r_53cur3_P45sw0r6
``` python
# server.py: elif user_inp == '4':
p, q, g = annex.dsa.get_public_params()
print(f'{p = }')
print(f'{q = }')
print(f'{g = }')
```

you'll then be prompted with `Test user log`. Selecting yes and entering the password from earlier would give you signature logs:
```python
# server.py: elif user_inp == '4':
if inp == 'y':
    if annex.users['Admin'].login():
        print(f'\n{annex.user_log}')
```

The logs is a list in the following format: `(r, s), hash`
```python
# class Annex: log_info()
self.user_log.append((sig, h))
```
## Downloading flag:
in order to download the flag we need the following information:
- username
- message's request id
- nonce value
- private key
```python
elif user_inp == '3':
	uname = input("\nPlease enter the username that stored the message: ")
    if not uname in annex.vault:
        print("\n[!] Sorry, need valid existing username to download secret!")
        continue

    req_id = input("\nPlease enter the message's request id: ")
    if not req_id.isdigit() or not (0 <= int(req_id) < len(annex.vault[uname])):
        print("\n[!] Sorry, need valid request id to download secret!")
        continue
            
    req_id = int(req_id)
    if uname == account_username:
        account = annex.users[account_username]

        if account.login():
            h, msg, sig = annex.vault[uname][req_id]
            print(f"\n[+] Here is your message: {msg}")
        else:
            print("[-] Invalid username and/or password!")
    else:
        k = input_number("\nPlease enter the message's nonce value: ")
        if not k:
            print("\n[!] Sorry, need a valid nonce to download secret!")
            continue
            
        x = input_number("\n[+] Please enter the private key: ")
        if not x:
            print("\n[!] Sorry, need a valid private key to download secret!")
            continue

        annex.download(x, k, req_id, uname)
```


## Nonce implementation
DSA sign function: 
The k is generated using `random.randint()`. It is based on the range of the variable `self.k_min` and `k_max`.
```python
# class DSA
def sign(self, h, k_max):
        k = random.randint(self.k_min, k_max)
        r = pow(self.g, k, self.p) % self.q
        s = (pow(k, -1, self.q) * (int(h, 16) + self.x * r)) % self.q
        return (r, s)
```
k_min is hardcoded to the value of: `self.k_min = 65500`

k_max is based of the username:
```python
# class Account: init()
    self.k_max = int(len(self.username) ** 6)
    if self.k_max < 65536:
        self.k_max += 1000000
    self.stored_msgs = 0
```

## Extracting Nonce:
The nonce is generated at random and is only use once for each signing but it is predictable and small enough that we can brute force it.

first, define k_max: copied based of the original code provided:
```python
# class Account: __init__()
def k_max(username):
    k_max = int(len(username) ** 6)
    if k_max < 65536:
        k_max += 1000000
    return k_max
```

DSA output of `r` is created with the following formula:
r = (g^k mod p) mod q

so brute forcing  is done as follows: return if brute_force_r(_tmp_r) is equal to r of the flag.
```python
def brute_force_nounce_K(k_max):
    for k in range(65500, k_max):
        _tmp_r = pow(g, k, p) % q
        print(f"K: {k}/{k_max}   R: {_tmp_r}", end="\r")
        if _tmp_r == r:
            print()
            print("Done: k = ", k)
            return k
```


## Extracting private key:
Got the formula from: [Recovering cryptographic keys from partial information, by example](https://eprint.iacr.org/2020/1506.pdf)
under: 5.1.3 Nonce recovery and (EC)DSA security.

Formula for extracting private key is as follows: 
d = r^−1(ks − h) mod n
rewritten as:
x = r^-1(ks - h) mod q

```python
def get_private_key(k):
    private_key_x = (pow(r, -1, q) * (k * s - int(h, 16))) % q
    return private_key_x
```

Something to note is that 
because or math. r^-1 needing to be in the modular context/space of q. therefore in python it is coded as `pow(r, -1, q)`.  thank you chatGPT.

full code:
```python
p = 20058102826947027761726273389333641000473209403091483561688258175474618601652139736893398659346630928652751628210535866966174806611503803394381292517633627535972278871370830094783450733238968689752413918182169457979355619552040476632604774686835047711846859563092914448154777535938794839477090700185938874506051142633507235238994726295432433181598932903325659269382377560542846407973521373601240432263570508092416933080510006586477348948986872778825706425549462174507217907939325420005727838080004771112021634591212781469428787966948093385063943772671190401677513622755781596873916464994847698192583488156665714740201
q = 18478753825720897807965232352344040078711991004973966756850639946907
g = 5454890307785540506234894466991210041471350454808132083244073827364153041253779152459340870659797719958405266884658205176533803149272101030804459704106157690006122743166258315304398044000391485687290003375146201948777738800640780480538645133274246332591883157787291114482946238650411421191718850610096282925743209679949488110554538225309874958358880642812761961762782130293533228880090336445181268630697403778738602688868971050139946780644415768724727279969852227991466839819586154883761102489035147950247606406684747883884401123725025197452321498267830645773435561873075508490036199827804239939969322205847774423378

signature_log = [((7902293523193251739094651692616902831726626417672586972487687359358, 17535331798777849331476732285254313700579651588554790870604440361946), 'a0aad39c9280260016dabaed8ca0c16d812ed8f2ccaa79eed07908f6bc74fb48'), ((9826812942588625649320101343670826570594096074825702340370631356394, 10137314708928771713900697239492273200369538161608712088637235874345), 'b61c744b656adfca5049503c898073fefce49413e072505541e78460c02345ac'), ((17624840125802501522042911709668698938251314550751325207016364372704, 16580061935305462382410766413396780093498172615385560350386996020390), '625e8f4530e14927bd3095b35917e127a056517ba11fa7570deae5485a1a8503'), ((4691944194958109856614993030873053717815914173248561690123851605580, 10491582655966412832045413234889042903660077394068103961611358674735), '0e9722da720ecebce84ce77efa3f047e7a9b1c1fb11264be7d5cc1adbb8a73a5'), ((15507395143375705895613897318790060845259010377907490204638849899941, 5066821542716643994660905866303542078097576002780646545638369629443), '36f6e72c03df0167409761fa929d4574b13a7d513a903a8c49193836c0cb34cd'), ((6862307253552894306169323633745704866599455960419995352794964790924, 144489041945987062361084833349764136658068914289242420261544317315), 'e20824d197269d9df0949caf37c7df2676ffc9e6b26bf3fd6e34bcb872651445')]

# Flag logs.
r = signature_log[5][0][0]
s = signature_log[5][0][1]
h = signature_log[5][1]


def k_max(username):
    k_max = int(len(username) ** 6)
    if k_max < 65536:
        k_max += 1000000
    return k_max


def brute_force_nounce_K(k_max):
    for k in range(65500, k_max):
        _tmp_r = pow(g, k, p) % q
        print(f"K: {k}/{k_max}   R: {_tmp_r}", end="\r")
        if _tmp_r == r:
            print()
            print("Done: k = ", k)
            return k


def get_private_key(k):
    private_key_x = (pow(r, -1, q) * (k * s - int(h, 16))) % q
    return private_key_x


if __name__ == "__main__":
    k_max = k_max("ElGamalSux")
    nounce_k = brute_force_nounce_K(k_max)
    private_key_x = get_private_key(nounce_k)
    print(f"K(Nounce): {nounce_k} \nPrivate_key: {private_key_x}")
```

![[Pasted image 20250302200857.png]]

![[Pasted image 20250302200916.png]]