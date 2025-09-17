# Crocodile Writeup
**[Type: HTB | Lab]**  
**Date:** 2025-08-13  
**Difficulty:** Easy  
**Target IP/URL:** 10.129.191.5  
**Objective:** Learn basics of SQL injection and server-side injection  

---

## 1. Reconnaissance
**Tools & Commands Used:**
- Nmap scan:
  ```bash
  nmap -sC -sV -oN scan.txt -iL ip-address.txt
  ```

**Findings:**
- Port 21: vsftpd 3.0.3 (FTP)  
- Port 80: Apache httpd 2.4.41 (Ubuntu)  
  ![Nmap Scan](images/nmap.png)

---

## 2. Enumeration
**Tools & Commands Used:**
- Service and version detection:
  ```bash
  nmap -sC -sV -oN scan.txt -iL ip-address.txt
  ```
- Web technology detection:
  ```bash
  whatweb -v 10.129.191.5
  ```
- Directory enumeration:
  ```bash
  gobuster dir -u http://10.129.191.5 -w ~/wordlist-tools/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
  ```

**Findings:**
- Port 21 open, running FTP with anonymous login allowed.  
- Port 80 open, running Apache 2.4.41 with Bootstrap (via WhatWeb).  
  ![WhatWeb Scan](images/whatweb-scan.png)
- Gobuster found a dashboard page.  
  ![Gobuster Scan](images/gobuster-scan.png)
- Website homepage detected.  
  ![Website](images/website.png)

---

## 3. Exploitation
**Vulnerability Identified:**
- **Name:** Weak FTP Authentication (No CVE)  
- **Why Vulnerable:** FTP allows anonymous login, exposing sensitive files (`allowed.userlist`, `allowed.userlist.passwd`).  

**Exploit Steps:**
- Connected to FTP:
  ```bash
  ftp 10.129.191.5
  ```
  - Username: `anonymous`
  - Output:
    ```
    Connected to 10.129.191.5.
    220 (vsFTPd 3.0.3)
    230 Login successful.
    ```
  ![FTP Login Attempt](images/ftp-login-attempt.png)
- Downloaded files:
  ```bash
  ftp> get allowed.userlist
  ftp> get allowed.userlist.passwd
  ```
  ![FTP File Discovery](images/ftp-file-discovery.png)
  ![FTP Files Download](images/ftp-files-download.png)
  ![FTP File Content](images/ftp-file-content.png)
- Used credentials from `allowed.userlist.passwd` on the dashboard login page found via Gobuster.  
  ![Login Page](images/login-page.png)
  ![Login Success](images/login-success.png)

---

## 4. Privilege Escalation
**Method:**
- Credential reuse from FTP files on the web dashboard.

**Commands & Output:**
- Used username/password from `allowed.userlist.passwd` to log into the dashboard, granting access to restricted content.

---

## 5. Results
- **User Flag:** `7b4bec00d1a39e3dd4e021ec3d915da8`  
  ![Flag](images/flag.png)

---

## 6. Lessons Learned
**Key Takeaways:**
- FTP (File Transfer Protocol) transfers files between client and server, often using clear-text authentication. Anonymous login (allowed here) exposes sensitive data.
- Nmap `-sC` revealed anonymous FTP access (code 230).
- Downloaded `allowed.userlist` and `allowed.userlist.passwd` via FTP, which provided credentials for the dashboard.
- WhatWeb and Wappalyzer confirmed Apache 2.4.41 and Bootstrap usage.
- Gobuster identified the dashboard page, critical for login attempts.
- Credential reuse from FTP files enabled web access.

**Mistakes to Avoid:**
- Not checking FTP for anonymous access early.
- Ignoring web dashboard after finding credentials.

**Tools/Techniques to Remember:**
- FTP client for anonymous login and file retrieval.
- `gobuster` for directory enumeration.
- `whatweb` and Wappalyzer for web technology detection.

---

## 7. Images
- ![Nmap Scan](images/nmap.png)
- ![WhatWeb Scan](images/whatweb-scan.png)
- ![Gobuster Scan](images/gobuster-scan.png)
- ![Website](images/website.png)
- ![FTP Login Attempt](images/ftp-login-attempt.png)
- ![FTP File Discovery](images/ftp-file-discovery.png)
- ![FTP Files Download](images/ftp-files-download.png)
- ![FTP File Content](images/ftp-file-content.png)
- ![Login Page](images/login-page.png)
- ![Login Success](images/login-success.png)
- ![Flag](images/flag.png)

---

## 8. References
- [vsftpd Documentation](https://security.appspot.com/vsftpd.html) (FTP)
- [Gobuster Documentation](https://www.kali.org/tools/gobuster/) (Tool)
- [WhatWeb Documentation](https://www.kali.org/tools/whatweb/) (Tool)
- [Wappalyzer](https://www.wappalyzer.com/) (Tool)