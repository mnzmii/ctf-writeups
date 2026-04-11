# Old Sessions

---

# 🧠 CTF Writeup: Old Sessions

## **Challenge Information**

- **Challenge Name:** Old Sessions
- **Platform:** picoCTF
- **Category:** Web Exploitation
- **Difficulty:** Easy
- **Date Solved:** March 11, 2026

---

## **Description**

Proper session timeout controls are critical for securing user accounts. If a user logs in on a public or shared computer but doesn’t explicitly log out, and session expiration dates are misconfigured, the session may remain active indefinitely.

Your friend built a new social media platform and mentioned that he hates constantly logging into sites, so he made it so that "once you login, you never have to log-out again!"

Browse the site and find the flag!

---

## **Initial Thoughts**

- The challenge is centered on **Insecure Session Management** and **Session Hijacking**.
- The developer's comment about never having to log out suggests that session tokens are long-lived and potentially reused.
- The hints regarding the "Web Inspector" and "Cookies" indicate that the solution involves manual cookie manipulation within the browser.
- I need to find a way to impersonate another user (likely an admin) by obtaining their active session ID.

---

## **Tools Used**

| **Tool** | **Purpose** |
| --- | --- |
| **Web Browser** | Used to navigate the social media platform and interact with the UI. |
| **Developer Tools (F12)** | Used to inspect cookies, modify session values, and view the network requests. |
| **URL Path Discovery** | Manually navigating to hidden endpoints based on user comments found on the site. |

---

## **Step-by-Step Solution**

### **1. Account Registration and Reconnaissance**

I began by creating a dummy account to understand how the application handles sessions.

- **Username:** `test`
- **Password:** `test`

After logging in, I explored the dashboard. In the comments section, I found a post by a user named `mary_jones_8992` stating: *"Hey I found a strange page at /sessions"*.

![image.png](Old%20Sessions/image.png)

### **2. Discovering the Leaked Sessions**

Following the lead from the comment, I navigated to the hidden endpoint:

[http://dolphin-cove.picoctf.net:51591/sessions](http://dolphin-cove.picoctf.net:51591/sessions)

**Observation:** This page displayed a raw list of active sessions stored on the server. I noticed an entry that stood out:

`1) session:jNgSwba3Y29qWvdDj_CCNI0VotYqImbdOs2jpfWWDU, {'_permanent': True, 'key': 'admin'}`

This revealed a valid **Session ID** belonging to the `admin` user.

![image.png](Old%20Sessions/image%201.png)

### **3. Session Hijacking via Cookie Manipulation**

With the admin's session ID in hand, I used the browser's Developer Tools to hijack the session.

1. I pressed **F12** and navigated to the **Application** tab (Storage tab in Firefox).
2. Under **Cookies**, I located the cookie named `session`.
3. I double-clicked the value and replaced my current "test" session ID with the leaked admin ID: `jNgSwba3Y29qWvdDj_CCNI0VotYqImbdOs2jpfWWDU`.

![image.png](Old%20Sessions/image%202.png)

### **4. Refreshing and Flag Retrieval**

After modifying the cookie, I refreshed the page (**Ctrl + R**). Because the server recognized the session ID as valid and associated with the admin account, I was automatically logged in as the administrator.

![image.png](Old%20Sessions/image%203.png)

---

## **Vulnerability Analysis**

The application suffered from two major security flaws:

1. **Information Leakage:** The `/sessions` endpoint was publicly accessible, exposing sensitive session tokens of every logged-in user.
2. **Broken Session Management:** The server relied solely on the session cookie provided by the client without verifying the user's current IP or context, allowing for easy **Session Hijacking**.

---

## **Final Flag**

`picoCTF{s3t_s3ss10n_3xp1rat10n5_ed8964c2}`

---

## **Lessons Learned**

- **Protect Session Data:** Session IDs should never be exposed in the UI or via public API endpoints.
- **Session Expiration:** Implement strict session timeouts to ensure that sessions do not remain active indefinitely.
- **Inspect Every Hint:** In Web CTFs, user comments or "debug" pages often hold the path to the vulnerability.

---
