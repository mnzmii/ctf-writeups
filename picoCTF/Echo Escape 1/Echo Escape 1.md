# Echo Escape 1

## CTF Writeup

### Challenge Information

**Challenge Name:** Echo Escape 1

**Platform:** PicoCTF

**Category:** Binary Exploitation 
Author: Yahaya Meddy

**Difficulty:** 100 points

**Date Solved:** March 13, 2026

---

## Description

The challenge presents a C program that echoes back a user's name. Despite being called "secure," the service uses a fixed-size buffer while allowing a much larger input. A "hidden" `win()` function exists in the code that reads `flag.txt`, but it is never naturally called by the program's logic.

---

## Initial Thoughts

- The use of `read(0, buf, 128)` into a `char buf[32]` is a textbook **Buffer Overflow** vulnerability.
- Since there is a `win()` function that prints the flag, the objective is to perform a **Ret2Win** (Return-to-Win) attack.
- The goal is to overwrite the **Saved Return Address** on the stack to redirect execution to `win()`.

---

## Files Provided

- `vuln`: The compiled ELF binary.
- `vuln.c`: The source code.

---

## Investigation

### Step 1: Locating the Target Address

Using the `nm` command, I identified the memory address where the `win()` function begins.

Bash

![image.png](Echo%20Escape%201/image.png)

**Observation:** The address is `0x401256`. The length of the address confirms this is a **64-bit (x86_64)** binary.

### Step 2: Calculating the Offset

The stack needs to be filled exactly until the Return Address is reached.

- **Buffer Size:** 32 bytes.
- **Saved RBP (Base Pointer):** 8 bytes (Standard for 64-bit).
- **Total Padding Required:** 32 + 8 = 40 bytes.

---

## Key Discovery

The core of this challenge was the **Stack Buffer Overflow**. By providing 40 bytes of "junk" data, the input spilled out of the allocated `buf` tray and into the **Return Address** tray. By replacing the original return address with `0x401256`, the CPU was forced to "jump" to the `win()` function as soon as `main()` tried to finish.

---

## Flag

After running the exploit against the remote server:

![image.png](Echo%20Escape%201/image%201.png)

**Flag:** 

![image.png](Echo%20Escape%201/image%202.png)

---

## Lessons Learned

- **Memory Safety:** Never trust user input length. Functions like `read()` should always be restricted to the size of the destination buffer.
- **The Stack:** Understanding how the CPU uses the stack to track function returns is fundamental to binary exploitation.
- **Endianness:** Data is stored "backwards" in memory on x86 systems; failing to reverse the address would have caused the jump to fail.

## Tools Used

- **nm:** To find function addresses in the binary.
- **Python 3:** To construct the raw byte payload.
- **Netcat (nc):** To interact with the remote challenge server.

