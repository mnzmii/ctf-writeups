# Binary Digits

## CTF Writeup

### Challenge Information

**Challenge Name:** Binary Digits
**Platform:** PicoCTF

Author: Yahaya Meddy
**Category:** Forensics
**Difficulty:** 100 points
**Date Solved:** March 13, 2026

---

### Description

The challenge provided a file named `digits.bin`. The description hinted that the file might look like random 1s and 0s, but contained something meaningful. The goal was to extract the hidden data to find the flag.

---

### Initial Thoughts

- The file extension `.bin` and the content (pure binary) suggest that this is a raw data dump.
- In CTFs, long binary strings often represent a file that has been converted to text.
- The first few bits might reveal a "Magic Byte" signature (File Header) once converted.

---

### Files Provided

- `digits.bin`: A text file containing a continuous string of binary digits (0 and 1).

![image.png](Binary%20Digits_Images/image.png)

---

### Investigation

### Step 1: Analyzing the Binary Data

I opened the file and saw a massive wall of `1`s and `0`s. To see if this was a known file type, I took the first few segments of the binary:
`11111111 11011000`

Converting `11111111` to Hex gives `FF`, and `11011000` gives `D8`.

**Observation:** `FF D8` is the standard header for a **JPEG** image file. This confirmed the binary data was actually an image encoded in base-2.

### Step 2: Decoding with CyberChef

I used CyberChef to automate the conversion of the entire string.

**Recipe Used:**

1. **From Binary:** To convert the text-based 1s and 0s into raw bytes.
2. **Render Image:** To display the resulting bytes as a visual file.

![image.png](Binary%20Digits_Images/image%201.png)

### Step 3: Extracting the Flag

Once the recipe was applied, an image appeared in the output pane. The image contained a stylized graphic.

**Observation:** The image clearly displayed the flag.

![image.png](Binary%20Digits_Images/image%202.png)

---

### Key Discovery

The core of this challenge was recognizing that binary data in a text file is often a representation of a hidden file format. By identifying the **JPEG Magic Bytes** (`FF D8`) at the very beginning of the binary string, I was able to determine the correct decoding method.

---

### Flag

Based on the image content, the flag/solution is:
picoCTF{h1dd3n_1n_th3_b1n4ry_cc2099d3}

---

### Lessons Learned

- **File Headers (Magic Bytes):** Always check the first few bytes of unknown data. Knowing that `FF D8` is JPEG and `89 50 4E 47` is PNG is vital for forensics.
- **Data Representation:** Data can be represented as Binary, Hex, or Base64. Being able to pivot between these quickly using tools like CyberChef is essential.

---

### Tools Used

- **CyberChef:** For "From Binary" and "Render Image" operations.
- **Hex Editor / Text Editor:** For initial file inspection.

