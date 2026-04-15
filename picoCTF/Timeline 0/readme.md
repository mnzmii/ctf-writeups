# Timeline 0

## Challenge Information

Challenge Name: Timeline 0

Platform: picoCTF

Category: Forensics

Difficulty: Easy

Date Solved: 11 March 2026

---

## Description

Can you find the flag in this disk image? Wrap what you find in the picoCTF flag format.

Hints:

- Create a Sleuthkit MAC timeline!
- Sloppy timestomping can yield strange (very old) timestamps

---

## Initial Thoughts

The hint suggests creating a **MAC timeline using Sleuthkit tools**.

This indicates the challenge likely involves **file timestamps** (Modified, Accessed, Changed).

The second hint mentions **timestomping**, a technique where attackers modify timestamps to hide malicious activity. If timestomping was done poorly, files might have **abnormally old timestamps**, which would stand out in a timeline analysis.

My hypothesis:

- The disk image contains a suspicious file with **unnatural timestamps**.
- By generating a timeline, we can identify files with strange dates and investigate them.

---

## Tools Used

- Sleuthkit (`fls`, `mactime`, `icat`)
- base64
- strings
- Linux command line (`grep`, `head`)

---

## Files Provided

- partition4.img

---

## Investigation

### Step 1

First, identify the type of file provided.

Command used:

```
file partition4.img
```

Result:

```
partition4.img: Linux rev 1.0 ext4 filesystem data
```

!(image.png)

Observation:

The file is directly an **ext4 filesystem image**, meaning it does not contain a partition table. Therefore, we can analyze the filesystem directly using Sleuthkit tools.

---

### Step 2

Generate a **bodyfile** containing filesystem metadata.

Command used:

```
fls -r -m / partition4.img > bodyfile.txt
```

Explanation:

- `r` recursively lists files
- `m /` sets the mount point for timeline generation

This produces metadata required for building a timeline.

---

### Step 3

Create a **MAC timeline** from the bodyfile.

Command used:

```
mactime -b bodyfile.txt > timeline.txt
```

Observation:

This generates a timeline showing file activities including:

- Modified
- Accessed
- Changed
- Created

These timestamps help identify suspicious activity.

---

### Step 4

Based on the hint mentioning **very old timestamps**, the first attempt was to search for files from the year **1970**, which is commonly seen in timestomping cases.

Command used:

```
grep 1970 timeline.txt
```

!(image%201.png)

Observation:

The results only showed normal system files from `/lib/modules/...` and nothing obviously suspicious or related to a flag. Therefore, this approach did not reveal the flag.

---

### Step 5

Since the grep search did not reveal anything useful, the next step was to inspect the **earliest entries in the timeline**.

Command used:

```
head timeline.txt
```

Output:

```
Wed Jan 02 1985 01:00:00       41 macb r/rrw-r--r-- 0 0 4945 /bin/bcab
```

!(image%202.png)

Observation:

This entry immediately stood out because:

- The timestamp is **1985**, which is unusually old.
- The file path `/bin/bcab` looks suspicious.
- It is not a typical system binary.

The inode number associated with this file is **4945**, so this file was investigated further.

---

### Step 6

Extract the suspicious file using its inode.

Command used:

```
icat partition4.img 4945
```

Output:

```
NzFtMzExbjNfMHU3MTEzcl9oM3JfNDNhMmU3YWYK
```

!(image%203.png)

Observation:

The output appears to be **Base64 encoded text**.

---

### Step 7

Decode the Base64 string.

Command used:

```
echo NzFtMzExbjNfMHU3MTEzcl9oM3JfNDNhMmU3YWYK | base64 -d
```

Output:

```
71m311n3_0u7113r_h3r_43a2e7af
```

!(image%204.png)

Since picoCTF requires the flag format `picoCTF{}`, the decoded text was wrapped accordingly.

---

## Key Discovery

The most important clue was the **abnormally old timestamp (1985)** found in the timeline. This indicated **timestomping**, where timestamps were manipulated to hide malicious files.

By identifying the suspicious file `/bin/bcab` and extracting it using `icat`, a Base64 encoded string was discovered. After decoding it, the hidden flag content was revealed.

---

## Flag

picoCTF{71m311n3_0u7113r_h3r_43a2e7af}

---

## Key Takeaways

- Timeline analysis is very useful in **digital forensics investigations**.
- **Timestomping** can be detected by identifying abnormal timestamps.
- Sleuthkit tools such as `fls`, `mactime`, and `icat` are powerful for analyzing disk images.
- If searching specific timestamps (like `1970`) fails, examining the **earliest timeline entries** can reveal anomalies.
- Encoded data such as **Base64** is commonly used in CTF challenges to hide flags.
