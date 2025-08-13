# Linux Shortcuts and Commands

This document outlines various Linux shortcuts and commands to navigate and manage files, directories, permissions, and system configurations effectively. The explanations are kept simple and in my own words, with typos corrected and incorrect explanations clarified where necessary. Sections marked as "I don't understand" or "will come back later" are left as is.

## Linux Shortcuts
These shortcuts help navigate and interact with Linux more efficiently.

- **Ctrl + Alt + T**: Opens the Linux terminal.
- **Ctrl + Shift + +**: Increases font size in the terminal.
- **Ctrl + -**: Decreases font size in the terminal.
- **clear or Ctrl + L**: Clears the terminal screen.
- **Ctrl + C**: Terminates the currently running process.
- **Ctrl + Z**: Pauses the currently running process (sends it to the background).
- **Tab**: Autocompletes commands or file names; double-tap Tab to cycle through available files or commands.

## Linux File Management
Commands for managing files and directories.

- **pwd**: Prints the current working directory.
- **ls**: Lists files and directories in the current directory.
  - **-l**: Lists in a detailed table format.
  - **-a**: Includes hidden files and directories.
  - **-al**: Combines detailed table format with hidden files.
  - **-lh**: Lists files with sizes in a human-readable format (e.g., KB, MB).
  - **-lR <directory-name>**: Recursively lists all files and directories in the specified directory.
  - Example: `ls -alsh /etc/ | grep proxychains.conf` lists files in `/etc/` in a human-readable, detailed format, filtering for `proxychains.conf`.
- **cd**: Changes the current directory (e.g., `cd /path/to/directory`).
- **touch**: Creates an empty file (e.g., `touch filename`).
- **whatis**: Displays a brief description of a command (e.g., `whatis ls`).
- **echo**: Writes text to a file or displays it in the terminal (e.g., `echo "text" > file.txt`).
- **cat**: Displays the contents of a file or redirects contents to another file.
  - Example: `cat file1.txt > file2.txt` copies contents of `file1.txt` to `file2.txt`.
- **rm**: Removes a file (e.g., `rm file.txt`).
- **mkdir**: Creates a directory (e.g., `mkdir new_folder`).
- **cp**: Copies a file to a directory (e.g., `cp file.txt /path/to/destination`).
- **rm -r <directory-name>**: Removes a directory and its contents recursively.
- **rmdir**: Removes an empty directory (e.g., `rmdir empty_folder`).
- **mv**: Moves a file to a directory or renames a file.
  - Example: `mv oldname.txt newname.txt` renames the file.
- **nano, vim**: Text editors for editing files.

## Linux File and Directory Permissions
Permissions define who can read, write, or execute files and directories.

- **Ownership**: Displayed as `user:group` (e.g., `kali:kali`).
- **Permission Types**:
  - **r (read)**: Permission to read the file.
  - **w (write)**: Permission to modify the file.
  - **x (execute)**: Permission to execute the file.
- **Permission Structure** (e.g., `-rw-r--r--`):
  - First character: `-` for a file, `d` for a directory.
  - Next three characters: Permissions for the owner (`rw-`).
  - Next three: Permissions for the group (`r--`).
  - Last three: Permissions for others (`r--`).
- **chmod**: Changes file or directory permissions.
  - **Symbolic Method**:
    - `chmod u=rwx filename`: Sets read, write, and execute permissions for the user (owner). Use `u` (user), `g` (group), `o` (others), or `a` (all).
    - `chmod go-wx filename`: Removes write and execute permissions for group and others.
  - **Octal Method**:
    - Permissions: `r = 4`, `w = 2`, `x = 1`.
    - `chmod 444 filename`: Sets read-only permissions for user, group, and others.
    - `chmod 744 filename`: Sets `rwx` for user, `r` for group and others (7 = 4+2+1).
    - `chmod -R 744 directory-name`: Applies permissions recursively to a directory.
- **chown**: Changes file or directory owner (e.g., `chown root filename`).
- **chgrp**: Changes file or directory group (e.g., `chgrp root filename`).

## Linux Grep and Piping
- **grep**: Searches for strings or patterns in a file.
  - Example: `grep "pattern" filename` (case-sensitive). Use `-i` for case-insensitive search: `grep -i "pattern" filename`.
  - Piping: `cat filename | grep "pattern"` filters the output of `cat` for the specified pattern.
- **Piping (|)**: Sends the output of one command as input to another.
- **Redirection (>)**: Redirects output to a file (e.g., `cat file1.txt > file2.txt`).

## Linux Finding Files With Locate
- **locate**: Finds files by name.
  - Example: `locate --all passwd` finds files containing "passwd".
  - Narrow search: `locate /etc --all passwd` or `locate passwd | grep "/etc/passwd"`.
  - Wildcard: `locate --all "*.config" | grep resolve` finds `.config` files containing "resolve".
  - Count matches: `locate --all -c proxychains` counts matching files.
  - Case-insensitive: `locate -i proxychains`.

## Enumerating Distribution and Kernel Information
Commands to gather system and user information.

- **whoami**: Displays the current user.
- **hostname**: Shows the system's hostname. Edit with `sudo vim /etc/hostname`.
- **id**: Displays the current user's ID and group memberships.
- **groups <username>**: Lists groups a user belongs to.
- **lsb_release -a**: Displays Linux distribution details (may vary by distribution).
  - Alternatives: `cat /etc/issue` or `cat /etc/*release`.
- **lscpu**: Shows CPU information.
- **uname**: Displays kernel information.
  - `-a`: All kernel details.
  - `-s`: Kernel name.
  - `-r`: Kernel version.
  - `-p`: Processor type.
  - `-i`: Hardware platform.
  - `-o`: Operating system.

## Linux Finding Files With Find
- **find**: Searches for files or directories.
  - Example: `sudo find / -type f -name "proxychains.conf"` searches for a file named `proxychains.conf` starting from `/`.
  - `-type f`: Files, `-type d`: Directories.
  - `-iname`: Case-insensitive name search.
  - `-size +1M`: Finds files larger than 1MB.
  - `-perm 400`: Finds files with specific permissions.

## Linux Shells and Bash Configuration
- Default shell for Debian/Ubuntu: Bash (Bourne Again Shell).
- **echo $SHELL**: Displays the current shell.
- **cat /etc/shells**: Lists available shells.
- Switch shells: Run `sh`, `bash`, or `dash`.
- **cat /etc/passwd | grep <username>**: Shows the user's assigned shell.
- **chsh**: Changes the default login shell (e.g., `chsh -s /usr/bin/dash`).
- **ls -al | grep .bash**: Lists Bash configuration files.
- Clear Bash history:
  - `cat /dev/null > ~/.bash_history`
  - `history -c`: Clears command history.

## Linux Disk Usage
- **du**: Shows file sizes in the current directory.
  - `du -sh *`: Displays sizes in human-readable format (`-s` for summary, `-h` for human-readable, `-c` for total).
- **df**: Shows disk usage for file systems.
  - `df -h`: Human-readable format.
  - `df -ht ext4`: Filters for ext4 file systems.

## File Compression and Archiving With tar
- **tar**: Archives files.
  - `tar -cf file.tar File/`: Creates an archive (`-c` for create, `-f` for file).
  - `tar -xvf file.tar`: Extracts an archive (`-x` for extract, `-v` for verbose).
  - `tar -czf file.tar.gz File/`: Creates a gzip-compressed archive.
  - `tar -xzf file.tar.gz`: Extracts a gzip archive.

## Users and Groups and Permissions With Visudo
- Add a user: `sudo useradd -m -c "chima added" -s /bin/bash chima`
  - `-m`: Creates a home directory.
  - `-c`: Adds a comment.
  - `-s`: Sets the default shell.
- Set password: `sudo passwd chima`.
- **visudo**: Edits sudoers file for permissions.
  - Example: `chima ALL=(ALL) ALL` grants `chima` full sudo privileges.
- **groupadd**: Creates a group (e.g., `sudo groupadd developers`).
- **usermod**: Adds a user to a group (e.g., `sudo usermod -aG developers chima`).
- **groups <username>**: Lists a user's groups.

## Linux Networking (ifconfig, netstat, and netdiscover)
- **ip route show**: Displays the routing table.
- **ip addr**: Shows IP addresses.
- **ifconfig**: Lists all network interfaces and their IP addresses.
- **netstat**: Displays network connections and services.
  - `-r`: Routing table.
  - `-t`: TCP connections.
  - `-l`: Listening ports.
  - `-lt`: Listening TCP ports.
  - `-lu`: Listening UDP ports.
- **netdiscover**: Scans for devices on the network.
  - Example: `sudo netdiscover -i eth0`.
- **DNS**:
  - Edit DNS: `sudo vim /etc/resolv.conf` (e.g., add `nameserver 8.8.8.8` for Google's DNS).
  - Check DNS: `systemd-resolve --status`.
  - Block ads via hosts: `sudo vim /etc/hosts` (e.g., `0.0.0.0 google.com` blocks `google.com`).
- Restart network: `sudo systemctl restart network-manager.service`.

## TOR and Proxychains
- **TOR**:
  - Start: `sudo systemctl start tor`.
  - Check status: `sudo systemctl status tor`.
  - Stop: `sudo systemctl stop tor`.
  - Verify TOR port: `netstat -lt | grep 9050`.
- **Proxychains**:
  - Edit config: `sudo vim /etc/proxychains.conf`.
  - Run: `proxychains firefox dnsleaktest.com` (start TOR first).

## Service and Process Management (htop and systemctl)
- **top**: Displays system resource usage and running processes.
- **htop**: A more user-friendly alternative to `top`.
- **free -h**: Shows RAM usage in human-readable format.
- **ps**: Lists running processes.
  - `ps aux`: Detailed snapshot of all processes.
- **kill <pid>**: Terminates a process by ID (e.g., `kill 4887`). Note: This does not stop the service itself (I don't understand how to explain better, will come back to this later).
- **systemctl**: Manages system services.
  - List services: `systemctl`.
  - Enable service: `sudo systemctl enable tor`.
  - Disable service: `sudo systemctl disable tor`.
  - Check status: `sudo systemctl is-enabled tor`.

## SSH and SSH Security
- SSH connections involve a client-server setup (I don't understand this fully and will come back to it later when I learn how to build and configure servers).