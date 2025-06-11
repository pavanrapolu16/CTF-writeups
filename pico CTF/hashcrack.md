# 📁 picoCTF - hashcrack

**Category:** Cryptography  
**Challenge Name:** hashcrack  
**Points:** 🟢 (Beginner level)  
**Description:**
> A company stored a secret message on a server which got breached due to the admin using weakly hashed passwords. Can you gain access to the secret stored within the server?  
> Access the server using:  
> `nc verbal-sleep.picoctf.net 52014`

---

## 🧠 Concept

This challenge involves:
- Identifying different types of hashes (MD5, SHA1, SHA256)
- Using online decrypters or hash databases to reverse weak hashes
- Submitting the correct passwords to reveal the final flag

---

## 🧪 Walkthrough

### ➤ Step 1: Connect to the server

```bash
nc verbal-sleep.picoctf.net 52014
```

You’ll get output like:

```
Welcome!! Looking For the Secret?
We have identified a hash: 482c811da5d5b4bc6d497ffa98491e38
Enter the password for identified hash:
```

---

### ➤ Step 2: Crack MD5 Hash

**Hash:**
```
482c811da5d5b4bc6d497ffa98491e38
```

**Detected as:** MD5  
**Used Tool:** [https://crackstation.net](https://crackstation.net)

**Result:**  
`password123`

✔️ Enter `password123` → proceeds to next hash.

---

### ➤ Step 3: Crack SHA1 Hash

**Hash:**
```
b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3
```

**Detected as:** SHA1  
**Used Tool:** [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)

**Result:**  
`letmein`

✔️ Enter `letmein` → proceeds to next hash.

---

### ➤ Step 4: Crack SHA-256 Hash

**Hash:**
```
916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745
```

**Detected as:** SHA-256  
**Used Tools:**
- [https://crackstation.net](https://crackstation.net)
- Manual brute-force validation using `sha256sum`

**Verified with:**
```bash
echo -n "qwerty098" | sha256sum
```

✅ Result:
```
916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745
```

✔️ Enter `qwerty098` → final flag revealed.

---

## 🏁 Flag

After entering the correct final password, the server reveals:

```
picoCTF{cr4ck1ng_h45h3s_l1k3_4_b055}
```

---

## 🛠️ Tools Used

- 🔓 [https://crackstation.net](https://crackstation.net) – For MD5 & SHA1 decryption
- 🧩 [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash) – Public hash search engine
- 💻 `sha256sum` (Linux CLI) – For manually verifying SHA-256 hashes

---

## ✅ Takeaway

Weak passwords + unsalted hashes = vulnerable systems.  
Always use:
- Strong, unique passwords
- Salted hashing algorithms like bcrypt, scrypt, or Argon2


