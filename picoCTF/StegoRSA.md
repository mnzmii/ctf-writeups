# StegoRSA

# 🧠 CTF Writeup: StegoRSA

## **Challenge Information**

- **Challenge Name:** StegoRSA
- **Platform:** picoCTF
- **Category:** Steganography / Cryptography
- **Difficulty:** Medium
- **Date Solved:** March 14, 2026

---

## **Description**

A message has been encrypted using RSA. The public key is gone… but someone might have been careless with the private key. Can you recover it and decrypt the message?

---

## **Initial Thoughts**

- The challenge title combines **Steganography** and **RSA**, suggesting that the RSA private key is hidden within the provided image file.
- RSA decryption requires the private key ($d$, $n$) to unlock a ciphertext encrypted with the corresponding public key ($e$, $n$).
- The hints point toward **metadata** as the storage medium for the key and mentioned that it is stored in a **hexadecimal** format.

---

## **Tools Used**

| **Tool** | **Purpose** |
| --- | --- |
| **exiftool** | Used to extract hidden metadata strings from the image. |
| **xxd** | Used to convert the extracted hex string back into a binary `.pem` key file. |
| **openssl** | Used to perform the actual RSA decryption using the recovered private key. |

---

## **Step-by-Step Solution**

### **1. Metadata Extraction**

I began by inspecting the metadata of the provided image to find any "careless" traces of the private key.

Bash

`exiftool image.png`

**Observation:** I found a long, continuous string of hexadecimal characters located in the `Comment`  field. This matched the hint regarding "Hex" values.

![image.png](StegoRSA/image.png)

### **2. Converting Hex to RSA Key**

Since RSA tools cannot read a raw hex string directly as a key, I saved the hex string into a file called `key.hex` and used `xxd` to "reverse" the hex into a binary format.

`# Convert the plain hex string to a binary file
xxd -r -p key.hex > private_key.pem`

To verify the conversion was successful, I checked the file header:

`cat private_key.pem`

The file correctly began with `-----BEGIN PRIVATE KEY-----`.

![image.png](StegoRSA/image%201.png)

### **3. Decrypting the Flag**

With the private key recovered, I used the OpenSSL utility to decrypt the `flag` file (which contained the RSA ciphertext).

Bash

`openssl pkeyutl -decrypt -inkey private_key.pem -in flag -out flag.txt`

- **`decrypt`**: Specifies the decryption operation.
- **`inkey`**: Points to our recovered private key.
- **`in`**: The encrypted flag file.

### **4. Final Flag Retrieval**

Reading the resulting `flag.txt` revealed the plaintext flag.

Bash

`cat flag.txt`

![image.png](StegoRSA/image%202.png)

---

## **Vulnerability Analysis**

The primary security failure here is **Insecure Sensitive Data Storage**.

- **The Root Cause:** Storing a private key (even if hex-encoded) inside a public asset like image metadata is a form of "Security through Obscurity."
- **The Impact:** Because the RSA private key was leaked, the mathematical trapdoor of the RSA algorithm was bypassed, allowing anyone with the image to decrypt the secret message.

---

## **Final Flag**

`picoCTF{rs4_k3y_1n_1mg_66388eb3}`

---

## **Lessons Learned**

- **Metadata is Persistent:** Always strip metadata from files before distribution in a production environment.
- **Key Management:** Private keys should be stored in secure vaults or Hardware Security Modules (HSMs), never inside the data they are meant to protect.
- **Hex is not Encryption:** Converting a key to hexadecimal format provides zero security; it is simply a different way to represent the same underlying data.
