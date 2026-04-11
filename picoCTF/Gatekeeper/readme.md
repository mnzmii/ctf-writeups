# Gatekeeper

## Challenge Information

- **Challenge Name:** gatekeeper
- **Platform:** picoCTF 2026
- **Category:** Reverse Engineering
- **Difficulty:** 100 points
- **Date Solved:** March 10, 2026
- **Author:** Yahaya Meddy

---

## Description

What’s behind the numeric gate? You only get access if you enter the **right** kind of number.

**Connection:** `nc green-hill.picoctf.net 53162`

---

## Initial Thoughts

The binary requires a numeric code. Based on the file strings, there is a "Too small" and "Too high" check. The presence of `is_valid_hex` and `strtol` suggests we can bypass standard decimal limits using hexadecimal input.

---

## Investigation

## Step 1: Analyze with Radare2

Opening the file and analyzing functions revealed the logic inside `main`.

Bash

`r2 gatekeeper
aaa
s main
pdf`

![image.png](Gatekeeper/image.png)

## Step 2: Identify the Logic Constraints

The disassembly showed three specific requirements to trigger the `reveal_flag` function:

1. **Lower Bound:** The input must be greater than `0x3e7` (999).
2. **Upper Bound:** The input must be less than or equal to `0x270f` (9999).
3. **Length Check:** The string length of the input must be exactly **3** characters.

![image.png](image%201.png)

## Step 3: Finding the Bypass

A decimal number greater than 999 is at least 4 characters long (e.g., "1000"). To satisfy the **3-character length** while remaining **numerically > 999**, a hexadecimal value is required.

- **Input:** `3E8`
- **Hex Value:** `0x3E8` = `1000` (Passes numeric check)
- **String Length:** `len("3E8")` = `3` (Passes length check)

tolong delete…

![image.png](image%202.png)

---

## Flag Recovery

## 1. Execute the Bypass

Bash

`nc green-hill.picoctf.net 53162
# Enter: 3E8`

![image.png](image%203.png)

## 2. Decode the Result

The server returns a scrambled string with `ftc_oc_ip` injected as junk.  Use the following Python script to clean and reverse the output:

Python

`# Raw output from the server
raw = "}adbftc_oc_ipe082ftc_oc_ipf_99ftc_oc_ip9_TGftc_oc_ip_xehftc_oc_ip_tigftc_oc_ipid_3ftc_oc_ip{FTCftc_oc_ipocipftc_oc_ip"

# Remove the junk and reverse the string
flag = raw.replace("ftc_oc_ip", "")[::-1]
print(f"Flag: {flag}")`

![image.png](image%204.png)

---

## Final Flag

![image.png](image%205.png)

**`picoCTF{3_digit_hex_GT_999_f280ebda}`**

---

## Lessons Learned

- **Hex Bypasses:** String length checks often fail to account for different numeric bases (Hex/Octal).
- **Obfuscation:** Simple string reversal and junk injection are common ways to hide flags even after the logic is broken.
