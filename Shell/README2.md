# 🧠 Linux Privilege Escalation Handbook

## Table of Contents
1. Introduction
2. Post‑Compromise Enumeration
3. SUID / GUID Binaries
4. Sudo Misconfigurations
5. Capabilities & File Capabilities
6. Cron Jobs & Scheduled Tasks
7. Misconfigured NFS Shares
8. Kernel Exploits & Privilege Escalation
9. Containers, Docker, LXC Escalation
10. Network & Pivoting Vectors
11. Summary & Checklist

---

## Introduction
Once you gain access to a Linux machine—whether as a limited user or root—privilege escalation becomes your next major objective.  
This handbook compiles all major vectors and techniques to escalate privileges on Linux.

---

## Post‑Compromise Enumeration
Before exploiting anything, enumeration is essential.

### Key Commands
```
hostname
uname -a
cat /proc/version
cat /etc/issue
ps aux
env
sudo -l
id
ifconfig || ip a
netstat -lntp
find / -perm -u=s -type f 2>/dev/null
```

---

## SUID / GUID Binaries
SUID = Set User ID → program runs with owner’s privileges  
GUID = Set Group ID → program runs with group’s privileges  

### Enumeration
```
find / -type f -perm -4000 -ls 2>/dev/null
```

---

## Sudo Misconfigurations
Check allowed sudo commands:
```
sudo -l
```

LD_PRELOAD exploit, GTFOBins, and NOPASSWD are common vectors.

---

## Capabilities & File Capabilities
Capabilities allow giving root‑level powers without full root.

### Find capabilities
```
getcap -r / 2>/dev/null
```

---

## Cron Jobs & Scheduled Tasks
Writable or misconfigured cron scripts are a strong PrivEsc path.

### Check cron
```
cat /etc/crontab
crontab -l
```

---

## Misconfigured NFS Shares
NFS entries with **no_root_squash** allow root privilege escalation remotely.

### Check exports
```
showmount -e <IP>
```

---

## Kernel Exploits
Old kernels may be vulnerable (DirtyCow, OverlayFS, DirtyPipe).

### Check kernel
```
uname -a
```

---

## Containers & Docker
Misconfigured Docker can give root on the host.

```
docker ps
docker run -v /:/host -it alpine chroot /host sh
```

---

## Network Enumeration & Pivoting
```
netstat -lntp
ss -lntu
route -n
```

---

## Summary & Checklist
- SUID binaries  
- Sudo misconfigs  
- Cron jobs  
- NFS  
- Capabilities  
- Docker / containers  
- Kernel exploits  
- Weak permissions  
