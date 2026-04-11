# Hidden Cipher 1

## Challenge Information

**Challenge Name:** Hidden Cipher 1

**Platform:** picoCTF

**Category:** Reverse Engineering / Cryptography

**Difficulty:** Easy

**Date Solved:** March 2026

---

## Description

The flag is right in front of you; just slightly encrypted. All you have to do is figure out the cipher and the key.

Connect to the program with netcat:

nc [candy-mountain.picoctf.net](http://candy-mountain.picoctf.net/) 61786

Hints:

- The binary can be unpacked using a tool that's often pre-installed on Linux
- The program hides a secret. Look at how it's defined and used.
- Think XOR. What happens when you XOR something twice with the same key?

---

## Initial Thoughts

- The challenge name “Hidden Cipher” suggests encryption is involved.
- The hints specifically mention **XOR**.
- Likely the binary contains a secret key that we need to extract.
- Since picoCTF flags always start with `picoCTF{`, we can use that as known plaintext.

---

## Files Provided

- `hiddencipher.zip`

After extraction:

- `hiddencipher` (binary executable)

---

## Investigation

### Step 1

Check readable strings inside the binary.

Command used:

```
strings hiddencipher
```

Result:

```
flag.txt
Here your encrypted flag:
%02x
get_secret
```

![image.png](Hidden%20Cipher%201/image.png)

Observation:

- The binary reads the flag from `flag.txt`
- It prints the encrypted flag as hexadecimal (`%02x`)
- The function `get_secret` likely contains the key

---

### Step 2

Check file type.

Command used:

```
file hiddencipher
```

![image.png](Hidden%20Cipher%201/image%201.png)

Observation:

- Output shows `ELF 64-bit executable`
- Confirms this is a Linux binary

---

### Step 3

Connect to the remote server to get the encrypted flag.

Command used:

```
nc candy-mountain.picoctf.net61786
```

Result (example ciphertext):

```
235a201d702015483b1d412b265d3313501f0c072d135f0d2002302d5011305120100a452e
```

![image.png](Hidden%20Cipher%201/image%202.png)

Observation:

- This is the encrypted flag in hexadecimal
- Likely XOR encrypted, possibly with a repeating key

---

### Step 4 — Use known plaintext to recover the key

Since picoCTF flags always start with `picoCTF{`, we can XOR the first bytes of the ciphertext with the ASCII bytes of this known plaintext.

Ciphertext (first bytes):

```
23 5a 20 1d 70 20 15 48
```

Plaintext (`picoCTF{` in ASCII):

```
70 69 63 6f 43 54 46 7b
```

XOR operation:

| Cipher | Plaintext | Result |
| --- | --- | --- |
| 0x23 | 'p' | S |
| 0x5a | 'i' | 3 |
| 0x20 | 'c' | C |
| 0x1d | 'o' | r |
| 0x70 | 'C' | 3 |
| 0x20 | 'T' | t |

Observation:

The repeating pattern forms the key:

```
S3Cr3t
```

This key is repeated across the ciphertext during encryption.

---

### Step 5 — Decrypt the full flag

Python script:

```
cipher_hex = "235a201d702015483b1d412b265d3313501f0c072d135f0d2002302d5011305120100a452e"
cipher = bytes.fromhex(cipher_hex)
key = b"S3Cr3t"

flag =""
for i inrange(len(cipher)):
flag += chr(cipher[i] ^ key[i % len(key)])

print(flag)
```

![image.png](Hidden%20Cipher%201/image%203.png)

Observation:

- Decrypting with `"S3Cr3t"` produces a readable flag.

---

## Key Discovery

- The key was determined **from the known plaintext of the flag** (`picoCTF{`) instead of being stored explicitly in the binary.
- Using the XOR property:

```
ciphertext XOR key = plaintext
```

- Once the key `"S3Cr3t"` was known, decrypting the full ciphertext revealed the flag.

---

## Flag

```
picoCTF{xor_unpack_4nalys1s_cecbcb91}
```

![image.png](Hidden%20Cipher%201/image%204.png)

---

## Lessons Learned

- XOR encryption is reversible using the same key.
- Known plaintext attacks are powerful — if you know part of the plaintext, you can often recover the key.
- Simple tools like `strings` and `file` help inspect binaries, but sometimes the key doesn’t need to be explicitly extracted.
- Always consider patterns in CTF flags (`picoCTF{…}`) when reasoning about encryption.

---

## Tools Used

- strings
- file
- Python 3
- netcat (`nc`)
