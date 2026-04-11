# Hidden Cipher 2

# CTF Writeup

## Challenge Information

**Challenge Name:** Hidden Cipher 2

**Platform:** picoCTF 

**Category:** Reverse Engineering 

Author: Yahaya Meddy

**Difficulty:** 100 points

**Date Solved:** 2026-03-12

---

## Description

The flag is right in front of you... kind of. You just need to solve a basic math problem to see it. But to get the real flag, you’ll have to understand how that math answer is used. You can download the program files here.

Connect to the program with netcat: 

`nc crystal-peak.picoctf.net 52202`

---

## Initial Thoughts

- The challenge mentions a "basic math problem" that reveals the flag.
- The math answer is likely used as a key or a multiplier for an encoded string.
- The `flag.txt` provided in the zip is likely a decoy, and the real flag is generated dynamically by the binary.

---

## Files Provided

- `hiddencipher2.zip` (Contains the binary `hiddencipher2` and a decoy `flag.txt`)

---

## Investigation

### Step 1

Connect to the server to see the behavior of the program.

**Command used:**

`nc crystal-peak.picoctf.net 52202`

**Result:**

The server asks "What is 3 + 3?". Upon answering `6`, it returns a series of large integers labeled as "Encoded flag values":

`672, 630, 594, 666, 402, 504, 420, 738, 654, 312, 696, 624, 570, 588, 306, 624, 294, 660, 600, 570, 594, 294, 672, 624, 306, 684, 570, 612, 336, 594, 606, 330, 582, 600, 324, 750`

![image.png](Hidden%20Cipher%202/image.png)

### Step 2

Analyze the relationship between the input (6) and the output. Since standard flag formats start with `picoCTF{`, I checked if the first number (672) related to 'p' (ASCII 112).

**Calculation:**

672 / 6 = 112

The result matches the ASCII value for 'p'. It appears the program multiplies the ASCII value of each flag character by the user's input.

### Step 3

Automate the decryption using a Python script to divide each encoded value by the key (6) and convert it back to characters.

**Command used:**

nano solve.py

![image.png](Hidden%20Cipher%202/image%201.png)

python3 solve.py

![image.png](Hidden%20Cipher%202/image%202.png)

---

## Key Discovery

The critical discovery was realizing that the math problem wasn't just a gatekeeper, but provided the **divisor** for the cipher. The program takes the flag string, iterates through it, and performs `char * input` before printing it to the terminal.

---

## Flag

**picoCTF{m4th_b3h1nd_c1ph3r_f8e7ad6}**

---

## Lessons Learned

- Input provided to a program in a CTF is often used as a variable in the encryption/obfuscation logic.
- Comparing known-plaintext (like the "pico" prefix) against encoded output can quickly reveal simple arithmetic ciphers.
- Don't trust the local `flag.txt` if the challenge involves a remote connection; it's often a placeholder.

---

## Tools Used

- `netcat` (nc)
- `unzip`
- `Python3`
- `ASCII Reference Table`