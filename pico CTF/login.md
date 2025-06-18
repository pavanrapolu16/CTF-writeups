## 📘 Challenge: login  
**Category:** Web Exploitation  
**Author:** BrownieInMotion  
**Description:**  
> My dog-sitter's brother made this website but I can't get in; can you help?  
>  
> URL: [http://login.mars.picoctf.net](http://login.mars.picoctf.net)

---

### 🔍 Initial Recon

We are given a basic login page with two input fields: **username** and **password**.

📜 View the page source → nothing suspicious.

We check the browser's **Developer Tools (F12)** → Inside the `<script>` tag, we find JavaScript code like this:

```javascript
(async () => {
    await new Promise((e => window.addEventListener("load", e))),
    document.querySelector("form").addEventListener("submit", (e => {
        e.preventDefault();
        const r = {
            u: "input[name=username]",
            p: "input[name=password]"
        }
        const t = {};
        for (const e in r)
            t[e] = btoa(document.querySelector(r[e]).value).replace(/=/g, "");
        return "YWRtaW4" !== t.u ? alert("Incorrect Username") :
               "cGljb0NURns1M3J2M3JfNTNydjNyXzUzcnYzcl81M3J2M3JfNTNydjNyfQ" !== t.p
               ? alert("Incorrect Password")
               : void alert(`Correct Password! Your flag is ${atob(t.p)}.`);
    }));
})();
```

---

### 🧠 Understanding the Logic

The JavaScript:
- Waits for the page to load.
- Listens for the login form submission.
- Grabs the username and password fields.
- Encodes both in **Base64** (using `btoa()`, without `=` padding).
- Compares the encoded versions with hardcoded Base64 strings.

We decode them manually:

---

### 🔓 Decode Username

```bash
echo "YWRtaW4" | base64 -d
# Output:
admin
```

✅ Username = `admin`

---

### 🔓 Decode Password

```bash
echo "cGljb0NURns1M3J2M3JfNTNydjNyXzUzcnYzcl81M3J2M3JfNTNydjNyfQ" | base64 -d
# Output:
picoCTF{53rv3r_53rv3r_53rv3r_53rv3r_53rv3r}
```

✅ Password = `picoCTF{53rv3r_53rv3r_53rv3r_53rv3r_53rv3r}`

---

### ✅ Final Answer

Go to the login page and enter:

- **Username:** `admin`  
- **Password:** `picoCTF{53rv3r_53rv3r_53rv3r_53rv3r_53rv3r}`

You will get a popup saying:
```
Correct Password! Your flag is picoCTF{53rv3r_53rv3r_53rv3r_53rv3r_53rv3r}
```

---

### 🏁 Flag

```
picoCTF{53rv3r_53rv3r_53rv3r_53rv3r_53rv3r}
```

---

### 🔧 Takeaways

- Check for hardcoded client-side validation in JavaScript.
- `btoa()` and `atob()` are Base64 encode/decode functions in JS.
- Developer Tools + basic decoding skills = easy CTF win!
