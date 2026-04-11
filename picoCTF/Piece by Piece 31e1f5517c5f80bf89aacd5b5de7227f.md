# Piece by Piece

# CTF Writeup: Piece by Piece

## **Challenge Information**

- **Challenge Name:** Piece by Piece
- **Platform:** picoCTF
- **Category:** General Skills / File Manipulation
- **Difficulty:** Easy
- **Date Solved:** March 10, 2026

---

## **Description**

After logging in, you will find multiple file parts in your home directory. These parts need to be combined and extracted to reveal the flag.
**SSH Host:** `dolphin-cove.picoctf.net:60338`**User:** `ctf-player`**Password:** `f3b61b38`**Hint:** Use Linux commands to combine the parts. The ZIP is password protected with `supersecret`.

---

## **Initial Thoughts**

- The challenge presents a classic "split file" scenario. In Linux, large files are often broken into smaller chunks (e.g., `part_aa`, `part_ab`) for easier transport.
- I need to concatenate (join) these pieces back into their original order to reconstruct a valid ZIP archive.
- Since the parts are named alphabetically (`aa`, `ab`, `ac`, etc.), the standard `cat` command with a wildcard should automatically join them in the correct sequence.
- Once joined, the `unzip` utility will be needed to handle the password protection.

---

## **Tools Used**

| **Tool** | **Purpose** |
| --- | --- |
| **SSH** | Used to access the remote challenge environment. |
| **cat (Concatenate)** | The primary tool for merging multiple file parts into one single archive. |
| **unzip** | Used to extract the contents of the reconstructed ZIP archive using a password. |
| **ls -l** | Used to verify file sizes and ensure the "parts" were present before merging. |

---

## **Challenge Overview**

- **Description:** The flag is hidden within multiple file parts in the home directory of a remote server. These parts need to be combined and extracted to reveal the flag.
- **Host:** `dolphin-cove.picoctf.net`
- **SSH Credentials:** `ctf-player` / `f3b61b38`
- **Key Concepts:** SSH, Linux File Concatenation (`cat`), Password-protected ZIP extraction.

---

## **Step-by-Step Solution**

### **1. Remote Connection**

First, I connected to the challenge server using SSH.

![image.png](Piece%20by%20Piece/image.png)

`ssh ctf-player@dolphin-cove.picoctf.net -p 60338`

### **2. Initial Reconnaissance**

Once logged in, I listed the files in the home directory. I found five "parts" (`part_aa` through `part_ae`) and an `instructions.txt` file.

Reading the instructions revealed the following hints:

- The flag is split into multiple parts as a zipped file.
- The parts need to be combined.
- The ZIP file is protected with the password: **`supersecret`**.

![image.png](Piece%20by%20Piece/image%201.png)

### **3. Combining the Parts**

To reconstruct the original archive, I used the `cat` command with a wildcard to merge all files starting with "part_a" into a single file named `combined.zip`.

Bash

`cat part_a* > combined.zip`

### **4. Extracting the Flag**

With the archive reconstructed, I used the `unzip` command. When prompted, I entered the password `supersecret` provided in the instructions.

`unzip combined.zip
# [combined.zip] flag.txt password: supersecret`

### **5. Reading the Flag**

Finally, I listed the directory again to find `flag.txt` and read its contents.

![image.png](Piece%20by%20Piece/image%202.png)

`cat flag.txt`

---

## **Final Flag**

`picoCTF{z1p_and_spl1t_f1l3s_4r3_fun_81362b37}`

---

## **Key Takeaways**

- **`cat [parts]* > [output]`**: A powerful way to join files that were split using the Linux `split` utility.
- **`unzip`**: Standard utility for extracting compressed archives; can handle password prompts in the terminal.