# Appointment Writeup
**[Type: HTB | Lab]**  
**Date:** 2025-08-13  
**Difficulty:** Easy  
**Target IP/URL:** 10.129.135.32  
**Objective:** Learn basics of SQL injection and server-side injection  

---

## 1. Reconnaissance
**Tools & Commands Used:**
- Initial Nmap scan:
  ```bash
  nmap -sV -p 80 -oN scan.txt -iL ip-address.txt
  ```

**Findings:**
- Port 80: Apache httpd 2.4.38 (Debian)  
  *(Image: scan result)*

---

## 2. Enumeration
**Tools & Commands Used:**
- Service and version detection:
  ```bash
  nmap -p 80 --script http-server-header,http-title -sV -oN service-scan.txt -iL ip-address.txt
  ```
- Directory enumeration with Gobuster:
  ```bash
  gobuster dir -u http://10.129.135.32 -w ~/wordlist-tools/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
  ```

**Findings:**
- HTTP title revealed a login page.  
  *(Image: login page)*
- No significant directories found, only default ones.  
  *(Image: Gobuster output)*

---

## 3. Exploitation
**Vulnerability Identified:**
- **Name:** SQL Injection (A03:2021-Injection, OWASP Top Ten 2021)  
- **Why Vulnerable:** The login page does not properly validate user input, allowing the username field to bypass password checks by manipulating the SQL query (e.g., ignoring the password column).

**Exploit Steps:**
- Tested common credentials (e.g., `admin:admin`, `guest:guest`, `user:user`, `root:root`, `administrator:password`)—all failed.
- Used SQL injection in the username field:
  ```
  admin' #
  ```
- This commented out the password check in the SQL query, granting access to the admin panel without a password.  
  *(Image: successful login)*

---

## 4. Privilege Escalation
**Method:**
- SQL injection via login bypass.

**Commands & Output:**
- Input in username field:
  ```
  admin' #
  ```
- Result: Logged in as admin, bypassing password verification.

---

## 5. Results
- **User Flag:** `e3d0796d002a446c0e622226f42e9672`  
  *(Image: flag captured)*

---

## 6. Lessons Learned
**Key Takeaways:**
- SQL (Structured Query Language) is commonly vulnerable to injection attacks (A03:2021-Injection, OWASP Top Ten 2021).
- HTTPS typically uses port 443; HTTP uses port 80.
- SQL injection can be mitigated with input validation, parameterized queries, stored procedures, and a Web Application Firewall (WAF).
- Proper input validation on forms is critical to prevent malicious input like `admin' #`.

**Mistakes to Avoid:**
- Assuming default credentials will work without testing for injection vulnerabilities.
- Not verifying open ports before running tools like Gobuster.

**Tools/Techniques to Remember:**
- Used `gobuster` with `SecLists` wordlist:
  ```bash
  git clone https://github.com/danielmiessler/SecLists.git
  ```
- Kali Linux provides built-in wordlists: `/usr/share/wordlists`.

---

## 7. Images
- Nmap scan result
- Login page
- Successful login via SQL injection
- Gobuster directory enumeration output
- User flag

---

## 8. References
- [SecLists GitHub](https://github.com/danielmiessler/SecLists) (Wordlist)
- [Gobuster Documentation](https://www.kali.org/tools/gobuster) (Tool)
- [OWASP Top Ten 2021](https://owasp.org/Top10) (Injection Reference)
