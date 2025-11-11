# CTF Writeup — Service Enumeration & Flags

**Target:** `10.201.14.166` (CTF / lab)

> **Scope:** Port scanning (1–10000), service enumeration, credential discovery for `eddie` and `quinn`, and flag retrieval from HTTP, SSH and FTP services.

---

## TL;DR (quick results / flags)

* **Highest open port < 10,000:** `80xx`
* **Open port > 10,000:** `10xxx`
* **Number of open TCP ports:** `X`
* **Flag in HTTP server header:** `THM{web_server_xxxxx}`
* **Flag hidden in HTTP challenge at `:8080`:** `THM{f744xxx9}`
* **Flag in SSH server header:** `THM{946219583xxx}`
* **FTP server (nonstandard port) version:** `vsftpd x.x.x`
* **Flag accessed via FTP from user files:** `THM{321452667xxxx}`

---

## Artifacts (uploaded / collected)

* `rustscan_nmap.nmap` — nmap / rustscan raw output (scan evidence)
* `hydraoutput.txt` — hydra command history and results
* `ftp_flag.txt` — retrieved FTP flag file containing `THM{32145266xxxx}`

Keep these files in the repository under an `artifacts/` directory for submission and verification.

---

## Tools & commands used (examples)

> Only run these against authorized targets (CTF / lab). Do not brute-force or scan systems you do not own or have explicit permission to test.

* **Fast discovery:** `rustscan` / `masscan`
* **Verification & scripting:** `nmap` (-sV, scripts)
* **Web checks:** `curl`, browser
* **SSH / SFTP:** `ssh`, `sftp`, `scp`
* **Credential guessing (CTF only):** `hydra`, `sshpass` (use small curated lists)

Example commands used during the engagement:

```bash
# fast port discovery with rustscan -> nmap
rustscan -a 10.201.14.166 -p 1-10000 -- -sV -oN rustscan_nmap.nmap

# inspect HTTP headers
curl -I http://10.201.14.166:8080

# check SSH banner
nc -v 10.201.14.166 22

# enumerate FTP on non-standard port
nmap -p 10021 -sV --script=banner 10.201.14.166

# targeted hydra example (CTF only; small list)
hydra -l quinn -P quinn_hits.txt ssh://10.201.14.166 -s 22 -t 4 -f -o hydra_result.txt
```

---

## Step-by-step methodology

1. **Port discovery:** ran a fast scan for TCP ports (1–10000). Verified results with `nmap`. Counted 6 open TCP ports; `8080` was the highest under 10k and `10021` was found above 10k.

2. **HTTP analysis (port 8080):** browsed to `http://10.201.14.166:8080` — a small challenge page returned `THM{f744xxxx}` after solving. Checked HTTP response headers with `curl -I` and found `THM{web_server_2xxxx}` in the `Server` header.

3. **SSH banner (port 22):** connected to port 22 to view the SSH banner and found `THM{94621958xxxx}`.

4. **FTP enumeration (port 10021):** identified `vsftpd 3.0.5` in the FTP banner on port `10021`. Located user files and retrieved a flag file via FTP/SFTP. Saved `ftp_flag.txt` locally containing `THM{32145266xxxx}`.

5. **Credential discovery (eddie / quinn):** social-engineering provided the usernames. A rockyou-like file present on the host was inspected for candidate passwords. Used curated candidate lists and targeted guessing. Where necessary and authorized, `hydra`/`sshpass` were used with small lists only.

6. **Flag retrieval:** downloaded the relevant file(s) and inspected contents. Verified each flag and recorded evidence.

---

## Evidence & saved outputs

* `rustscan_nmap.nmap` — raw scan output (place under `artifacts/`).
* `hydraoutput.txt` — command history showing authentication attempts (place under `artifacts/`).
* `ftp_flag.txt` — the file retrieved via FTP containing the FTP flag (place under `artifacts/`).

Include timestamps and command outputs where possible for traceability.

---

## Scoring (suggested rubric)

*Total: 100 pts*

* Recon & Port Discovery — **20 pts** (identified open ports, counts, and highest <10k)
* HTTP analysis — **20 pts** (header flag + web challenge flag)
* SSH enumeration — **15 pts** (banner flag)
* FTP enumeration & retrieval — **25 pts** (ftp version, file retrieval, flag)
* Documentation & evidence — **20 pts** (artifacts, commands, reproducibility)

Adjust weights to match your CTF event scoring rules.

---

## Remediation & defensive notes

1. **Remove sensitive data from banners** (SSH/HTTP). Banners leak information and should not contain secret tokens or flags.
2. **Harden FTP/SFTP:** prefer SFTP (SSH-based) with key-based auth, or FTPS with modern TLS. Disable anonymous access and remove unnecessary files (like password wordlists) from reachable locations.
3. **Patch & configuration:** keep services (vsftpd, OpenSSH, web server) updated and restrict ciphers and protocols.
4. **Protect sensitive artifacts:** do not store password lists such as `rockyou.txt` on test systems; restrict permissions.
5. **Rate-limiting and monitoring:** implement fail2ban or rate-limiting to slow down brute-force attempts and log suspicious activity.

---

## Repro steps (concise)

1. Run `rustscan` (or `masscan`) for ports 1–10000, verify with `nmap`.
2. `curl -I http://<IP>:8080` to get HTTP header flag; visit the page to solve the challenge and get the web flag.
3. `nc <IP> 22` to read SSH banner for SSH flag.
4. `nmap -p10021 -sV --script=banner <IP>` to enumerate FTP banner and version.
5. If credentials are available/derived, use `sftp` or `ftp` to download files and retrieve the flag.

---

## Notes & ethics

* All scanning and password-guessing must be performed only on systems you are authorized to test. Unauthorized scanning or brute forcing is illegal and unethical.
* For CTF writeups, always sanitize any sensitive or identifying data before public posting.

---

If you want this repository to include: a PDF export of this writeup, screenshots of the web challenge solution, or a short slide deck for a client briefing, mention which artifacts to include and they will be generated into the `docs/` folder.
