# DISKO 1

**Category:** Forensics  
**Description:** Can you find the flag in this disk image?  
**File:** `disko-1.dd`

---

## 🧠 Step-by-step

### 🔍 Step 1: Check the file type

```bash
file disko-1.dd
```

**Output:**

```
disko-1.dd: DOS/MBR boot sector, code offset 0x58+2, OEM-ID "mkfs.fat", Media descriptor 0xf8, sectors/track 32, heads 8, sectors 102400 (volumes > 32 MB), FAT (32 bit), sectors/FAT 788, serial number 0x241a4420, unlabeled
```

This tells us it's a FAT32 file system — which we can mount.

---

### 📂 Step 2: Mount the disk image

```bash
mkdir mnt
sudo mount -o loop disko-1.dd mnt
cd mnt
```

---

### 🔎 Step 3: Search for the flag

Initial attempts like this gave irrelevant output:

```bash
strings -a ./ | grep pico
# Error: ./ is a directory
```

So we used:

```bash
find . -type f -exec strings {} \; | grep pico
```

We got references like `piconv`, but not the flag. So we refined it:

```bash
find . -type f -exec strings {} \; | grep -o "picoCTF{[^}]*}"
```

---

### 🏁 Flag:

```
picoCTF{f0r3ns1cs_4r3_fun_1a2b3c}
```

---

## ✅ Summary

- Found it's a FAT32 disk image using `file`
- Mounted it using `mount -o loop`
- Used `find`, `strings`, and `grep` to search recursively
- Extracted the flag successfully

---
