# EVEN RSA CAN BE BROKEN???
**CTF:** picoCTF  
**Challenge:** EVEN RSA CAN BE BROKEN???  
**Category:** Crypto  
**Author:** Michael Crotty   
**Difficulty:** Easy to Medium  
**Tags:** RSA, GCD Attack, Shared Prime

---

## 🧠 Description

> This service provides you an encrypted flag. Can you decrypt it with just N & e?  
> Connect to the program with netcat:  
> `$ nc verbal-sleep.picoctf.net 51434`  
> The program's source code can be downloaded here.

---

## 📂 Files Provided

- `encrypt.py` (source code)
- Remote server interaction: `nc verbal-sleep.picoctf.net 51434`

---

## 🔍 Analysis

We are told we only get **N**, **e**, and **ciphertext**. The provided source (`encrypt.py`) reveals that:

- Each time you connect, it:
  - Generates an RSA key: `N = p * q`
  - Encrypts the same flag with `e = 65537`
  - Returns `N`, `e`, and the encrypted flag

🔐 But here's the **critical flaw**:
```python
from setup import get_primes
p, q = get_primes(k // 2)
```
This means the challenge is using a **custom prime generator**, likely reusing or weakly randomizing primes.

---

## ⚠️ Vulnerability: Shared Prime Attack

If two RSA public keys (`N₁` and `N₂`) share a common prime (`p`), then:
- `gcd(N₁, N₂) = p`  
- You can factor both `N`s easily:
  - `q₁ = N₁ // p`
  - `q₂ = N₂ // p`
- Then compute the private key:
  - `φ(N) = (p-1)(q-1)`
  - `d = inverse(e, φ(N))`
- Finally, decrypt the ciphertext.

---

## 🧪 Exploit Script

We collected multiple `(N, ciphertext)` pairs from the server using:

```bash
$ nc verbal-sleep.picoctf.net 51434
```

Then we wrote this Python script:

```python
from Crypto.Util.number import inverse, long_to_bytes
from math import gcd

# Paste your (N, e, ciphertext) values here
data = [
    {
        "N": 17005850958993068544611951375497018247709871571188996787072328644472548672517405275024848323413559150476672536105686487996466870041168786691151952104738298,
        "cipher": 2832137017681705990578066529046951418593056515732107828347791580201444568626900412693334538709786459246962908003578904607182716500634117529490867115834887
    },
    {
        "N": 19336074753524478568555502187537727081579048246881251731604194737897382861248258695211789616373461221614093258643351699510217418855462084387025714856337146,
        "cipher": 7598786673464066308169266903524977213144889659226260492306018197428515605287664104861363402838957180294298825237940531352623629456861222519027035308624587
    },
    # Add more N/ciphertext pairs from the challenge here...
]

e = 65537

for i in range(len(data)):
    for j in range(i + 1, len(data)):
        Ni, Nj = data[i]["N"], data[j]["N"]
        p = gcd(Ni, Nj)

        if 1 < p < min(Ni, Nj):  # Found shared prime
            print(f"[+] Shared prime found between pair {i} and {j}")
            q = Ni // p
            phi = (p - 1) * (q - 1)
            d = inverse(e, phi)
            c = data[i]["cipher"]
            m = pow(c, d, Ni)
            flag = long_to_bytes(m)
            print("[+] Decrypted flag:", flag.decode())
            exit()
```

---

## 🏁 Output

```
[+] Shared prime found between pair 0 and 1
[+] Decrypted flag: picoCTF{tw0_1$_pr!m381772be5}
```

---

## 🧠 Takeaway

- **RSA is secure only if p and q are truly random and unique**
- If even a single prime is reused across keys, **entire encryption collapses**
- This is known as the **Common Modulus GCD Attack**

---

## ✅ Flag

```
picoCTF{tw0_1$_pr!m381772be5}
```

---

## 🔒 Lessons Learned

- Always use **secure prime generation** in cryptography
- RSA is only as strong as its **implementation**
- GCD attacks are powerful against naive key generation

