<div align="center">

# 🐧 Linux Command-Line Operations

**System enumeration, processes, permissions, networking, storage, logs, and shell operations.**

![Platform](https://img.shields.io/badge/platform-Parrot%20OS%20%7C%20Kali%20%7C%20Debian-1793d1?style=for-the-badge&logo=linux&logoColor=white)
![Level](https://img.shields.io/badge/level-Intermediate%20to%20Advanced-orange?style=for-the-badge)
![Domain](https://img.shields.io/badge/domain-Administration%20%7C%20Cybersecurity-critical?style=for-the-badge)

</div>

| | |
|---|---|
| **Domain** | Linux Administration, Cybersecurity, System Troubleshooting |
| **Environment** | Parrot OS, Kali Linux, Debian-based systems |
| **Focus** | System enumeration, process management, permissions, networking, storage, logs, shell operations |

---

## 📚 Table of Contents

| # | Section |
|---|---------|
| 1 | [Introduction](#1--introduction) |
| 2 | [System Enumeration](#2--system-enumeration) |
| 3 | [File and Directory Management](#3--file-and-directory-management) |
| 4 | [Permissions and Ownership](#4--linux-permissions-and-ownership) |
| 5 | [Process Management](#5--process-management) |
| 6 | [Services and Systemd](#6--services-and-systemd) |
| 7 | [Networking and Sockets](#7--linux-networking-and-socket-analysis) |
| 8 | [Storage and Filesystems](#8--storage-and-filesystem-analysis) |
| 9 | [Logs and Security Investigation](#9--logs-and-security-investigation) |
| 10 | [Pipelines and Text Processing](#10--shell-pipelines-and-text-processing) |
| 11 | [Archives and Backups](#11--archives-compression-and-backups) |
| 12 | [Package Management](#12--package-management) |
| 13 | [Environment Variables](#13--environment-variables-and-shell-configuration) |
| 14 | [Privileges and Sudo](#14--permissions-privileges-and-sudo) |
| 15 | [Troubleshooting Methodology](#15--advanced-troubleshooting-methodology) |
| 16 | [Common Risks](#16--common-command-line-risks) |
| 17 | [Technical Summary](#17--technical-summary) |

---

## 1. 📖 Introduction

Linux command-line proficiency is essential in system administration, penetration testing, digital forensics, and cybersecurity operations.

Advanced usage is more than memorizing commands. It means understanding how processes, filesystems, permissions, network interfaces, services, and system logs interact.

This reference covers practical commands for inspecting system state, diagnosing problems, managing resources, analyzing files, and automating administrative tasks.

---

## 2. 🖥️ System Enumeration

Establish the operating-system environment, kernel version, hardware resources, and available storage.

### 2.1 Operating system information

```bash
cat /etc/os-release
```

```bash
uname -a
```

```bash
hostnamectl
```

| Command | Purpose |
|---------|---------|
| `cat /etc/os-release` | Distribution identification |
| `uname -a` | Kernel and system information |
| `hostnamectl` | Show or manage hostname and related settings |

### 2.2 Hardware and resources

```bash
lscpu
```

```bash
free -h
```

```bash
lsblk
```

```bash
df -hT
```

| Command | Purpose |
|---------|---------|
| `lscpu` | CPU architecture and processor information |
| `free -h` | Memory usage in human-readable units |
| `lsblk` | Block devices and their relationships |
| `df -hT` | Filesystem capacity, usage, and type |

### 2.3 Uptime and load

```bash
uptime
```

```bash
top
```

`uptime` shows uptime, logged-in users, and load averages. `top` gives a live view of processes, CPU, and memory.

> 💡 Load averages count runnable and certain uninterruptible tasks over time. Read them alongside CPU count and actual utilization.

---

## 3. 📁 File and Directory Management

### 3.1 Recursive file discovery

Find regular files beneath Documents:

```bash
find ~/Documents -type f
```

By extension:

```bash
find ~/Documents -type f -name "*.pdf"
```

Modified in the last 24 hours:

```bash
find ~/Documents -type f -mtime -1
```

Directories only:

```bash
find ~/Documents -type d
```

`find` can filter by name, type, modification time, permissions, ownership, and more.

### 3.2 File metadata

```bash
stat filename
```

```bash
file filename
```

```bash
du -sh directory/
```

- `stat` shows size, ownership, permissions, and timestamps
- `file` identifies a file's type from its contents
- `du -sh` estimates a directory's total disk usage

### 3.3 Locate executables

```bash
command -v python3
```

```bash
which bash
```

```bash
whereis ssh
```

`command -v` is generally the best way to check how the current shell resolves a command.

### 3.4 Search file contents

```bash
grep -Rni "error" ~/Documents
```

| Option | Meaning |
|--------|---------|
| `-R` | Search recursively |
| `-n` | Show line numbers |
| `-i` | Ignore case |

Search system logs:

```bash
grep -i "failed" /var/log/auth.log
```

> The log path depends on the distribution and logging setup. Access may need elevated privileges.

---

## 4. 🔑 Linux Permissions and Ownership

### 4.1 Inspect permissions

```bash
ls -l
```

```bash
stat filename
```

Example output:

```text
-rwxr-x--- 1 user security 2048 Oct 20 12:00 script.sh
```

| Symbol | Meaning |
|--------|---------|
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `-` | Not granted |

The first character is the file type. The next nine are permissions for **owner**, **group**, and **others**.

### 4.2 Change permissions

```bash
chmod 640 filename
```

Owner: read and write. Group: read. Others: none.

```bash
chmod u+x script.sh
```

Adds execute permission for the owner.

### 4.3 Change ownership

```bash
sudo chown user:group filename
```

For a directory tree:

```bash
sudo chown -R user:group directory/
```

> [!CAUTION]
> Recursive ownership changes affect many files. Verify the target path first.

### 4.4 Permission security checks

Regular files writable by others:

```bash
find ~/Documents -type f -perm -002
```

Regular files with at least one execute bit:

```bash
find ~/Documents -type f -perm /111
```

> 🧠 An executable file is not automatically unsafe, and permissions alone don't prove a system is vulnerable.

---

## 5. ⚙️ Process Management

A process is a running instance of a program. Each gets a process ID (PID).

### 5.1 Enumeration

```bash
ps aux
```

```bash
ps -ef
```

```bash
pgrep -a ssh
```

- `ps aux` is a detailed process listing
- `ps -ef` is the same idea in a different standard format
- `pgrep -a ssh` finds processes matching "ssh" and shows their command lines

### 5.2 Process relationships

```bash
pstree -p
```

```bash
ps -o pid,ppid,user,%cpu,%mem,cmd -p PID
```

Replace `PID` with a real process ID. The parent PID (PPID) shows which process created another.

### 5.3 Terminating processes

```bash
kill PID
```

```bash
kill -TERM PID
```

`SIGTERM` asks a process to exit gracefully. If it doesn't respond, `SIGKILL` forces termination:

```bash
kill -KILL PID
```

> ⚠️ Use forced termination only when necessary. It skips the process's normal cleanup.

### 5.4 Resource monitoring

```bash
top
```

```bash
ps aux --sort=-%cpu
```

```bash
ps aux --sort=-%mem
```

> 🧠 Investigate high resource usage using workload expectations and logs before concluding a process is malicious.

---

## 6. 🛠️ Services and Systemd

### 6.1 Service status

```bash
systemctl status ssh
```

```bash
systemctl is-active ssh
```

```bash
systemctl is-enabled ssh
```

- `status` shows service state and recent information
- `is-active` checks whether it's running now
- `is-enabled` checks whether it's configured to start automatically

Service names vary by distribution.

### 6.2 Service management

```bash
sudo systemctl restart ssh
```

```bash
sudo systemctl stop ssh
```

```bash
sudo systemctl start ssh
```

> [!WARNING]
> Restarting a network or remote-access service can cut active sessions. Confirm the service and its role first.

### 6.3 Service logs

```bash
journalctl -u ssh
```

```bash
journalctl -u ssh --since "1 hour ago"
```

```bash
journalctl -p warning
```

| Option | Meaning |
|--------|---------|
| `-u` | Filter by unit |
| `--since` | Restrict the time range |
| `-p warning` | Warning priority and more severe |

---

## 7. 🌐 Linux Networking and Socket Analysis

### 7.1 Interfaces and addresses

```bash
ip addr show
```

```bash
ip route
```

```bash
ip link show
```

- `ip addr show` shows interface addresses
- `ip route` shows routing-table entries
- `ip link show` shows interfaces and link state

### 7.2 Active connections and listening ports

```bash
ss -tuln
```

```bash
ss -lntp
```

`ss -tuln` lists listening TCP and UDP sockets with numeric addresses. `ss -lntp` lists listening TCP sockets with process details where permitted.

| Option | Meaning |
|--------|---------|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-p` | Associated processes (where available) |

> 🧠 A listening port means a service is waiting for connections. It doesn't automatically mean the service is reachable externally.

### 7.3 Connectivity testing

```bash
ping -c 4 192.0.2.10
```

```bash
ip route get 192.0.2.10
```

`192.0.2.10` is a documentation-reserved example address. A failed ping doesn't prove a host is offline, since ICMP may be filtered. `ip route get` shows how the kernel would route traffic to a destination.

### 7.4 DNS resolution

```bash
getent hosts example.com
```

```bash
dig example.com
```

`getent hosts` uses the system's configured name-resolution mechanisms. `dig` shows detailed DNS responses (if installed).

---

## 8. 💾 Storage and Filesystem Analysis

### 8.1 Disk usage

```bash
df -h
```

```bash
du -h --max-depth=1 ~
```

```bash
lsblk -f
```

- `df -h` shows filesystem capacity and free space
- `du -h --max-depth=1 ~` summarizes immediate directories under home
- `lsblk -f` shows devices, filesystems, labels, and UUIDs

### 8.2 Identify large files

Files over 500 MiB:

```bash
find ~/Downloads -type f -size +500M
```

Top 20 entries by size:

```bash
du -ah ~/Downloads | sort -h | tail -n 20
```

> Directories and their contents may both appear, so results can overlap.

### 8.3 Mounted filesystems

```bash
findmnt
```

```bash
mount
```

`findmnt` shows mounted filesystems and relationships. `mount` lists mounts, or performs mounts when given arguments and privileges.

> [!CAUTION]
> Before modifying partitions or filesystems, identify the correct device and confirm backups exist.

---

## 9. 🔎 Logs and Security Investigation

### 9.1 Authentication logs

On systems using the traditional Debian auth log, show the last 100 lines:

```bash
sudo tail -n 100 /var/log/auth.log
```

Follow new entries live:

```bash
sudo tail -f /var/log/auth.log
```

The file may not exist on systems that use the systemd journal or a different logging setup.

### 9.2 Search for failed authentication

```bash
sudo grep -i "failed" /var/log/auth.log
```

> 🧠 The word "failed" appears in many contexts and doesn't independently prove an attack.

### 9.3 Journal analysis

```bash
journalctl --since today
```

```bash
journalctl -p err
```

```bash
journalctl -b
```

- `--since today` shows entries from today
- `-p err` shows errors and more severe
- `-b` shows the current boot

### 9.4 Evidence preservation

- Record timestamps and relevant host details
- Preserve original logs where required
- Avoid unnecessary changes to the affected system
- Restrict access to sensitive evidence
- Document commands used and your interpretation of results

> 🧠 Missing logs don't prove an event didn't happen. Retention, configuration, permissions, and rotation affect what remains.

---

## 10. 🔗 Shell Pipelines and Text Processing

Pipelines send one command's output into another command's input.

### 10.1 Pipeline example

```bash
ps aux | grep '[s]sh'
```

Filters the process list for "ssh". The bracket trick usually keeps `grep` from matching itself.

### 10.2 Sort and filter

```bash
ps aux --sort=-%mem | head
```

Shows the top of the list sorted by memory, highest first.

```bash
grep -Rni "TODO" ~/Projects
```

Searches project files recursively for "TODO" with line numbers, ignoring case.

### 10.3 Count results

```bash
find ~/Documents -type f | wc -l
```

### 10.4 Extract fields

```bash
cut -d: -f1 /etc/passwd
```

Pulls the first colon-separated field from each line, typically the account name.

### 10.5 Redirect output

```bash
command > output.txt
```

```bash
command >> output.txt
```

```bash
command 2> error.log
```

| Operator | Effect |
|----------|--------|
| `>` | Redirect standard output, **overwriting** the file |
| `>>` | Append standard output |
| `2>` | Redirect standard error |

---

## 11. 📦 Archives, Compression, and Backups

### 11.1 Create a tar archive

```bash
tar -cf backup.tar Documents/
```

### 11.2 Create a compressed archive

```bash
tar -czf backup.tar.gz Documents/
```

`-z` uses gzip compression.

### 11.3 List and extract

```bash
tar -tf backup.tar.gz
```

```bash
tar -xzf backup.tar.gz
```

| Option | Meaning |
|--------|---------|
| `-t` | List contents |
| `-x` | Extract |
| `-f` | Archive file |
| `-z` | gzip compression |

> [!IMPORTANT]
> Inspect archive paths before extracting, especially from untrusted sources.

### 11.4 Verify integrity

```bash
sha256sum backup.tar.gz
```

A hash helps detect changes when compared to a trusted reference. It doesn't prove the original file was authentic or safe.

---

## 12. 📥 Package Management

On Debian-based systems, APT manages packages and dependencies.

### 12.1 Package information

```bash
apt search wireshark
```

```bash
apt show wireshark
```

```bash
dpkg -l
```

- `apt search` searches package metadata
- `apt show` displays package details
- `dpkg -l` lists installed packages and their states

### 12.2 Install and update

```bash
sudo apt update
```

```bash
sudo apt install package-name
```

```bash
sudo apt upgrade
```

`apt update` refreshes metadata only. It doesn't upgrade anything. `apt install` installs a package and its dependencies. `apt upgrade` upgrades installed packages.

> Review proposed changes before confirming, especially on a penetration-testing workstation where tool versions matter.

### 12.3 Identify package ownership

```bash
dpkg -S /usr/bin/ssh
```

Shows which installed package owns a path. Replace the path with the file you're investigating.

---

## 13. 🌱 Environment Variables and Shell Configuration

### 13.1 Inspect variables

```bash
printenv
```

```bash
echo "$HOME"
```

```bash
echo "$PATH"
```

- `printenv` lists exported environment variables
- `$HOME` is your home directory
- `$PATH` lists directories searched for commands

> [!CAUTION]
> Don't publish full environment dumps. They can contain secrets.

### 13.2 Temporary variables

```bash
export LAB_MODE=testing
```

```bash
echo "$LAB_MODE"
```

Exports a variable to the current shell and its child processes. It doesn't persist after the session ends unless set in a startup file or other persistent config.

### 13.3 Command history

```bash
history
```

History can contain sensitive arguments or operational details. Review it before sharing terminal screenshots or diagnostics.

---

## 14. 🛡️ Permissions, Privileges, and Sudo

`sudo` lets an authorized user run commands with elevated privileges according to system policy.

### 14.1 Check identity and groups

```bash
id
```

```bash
groups
```

### 14.2 Inspect sudo authorization

```bash
sudo -l
```

Lists what the current user may run via `sudo`. A user can log in successfully and still have no sudo rights. Grant admin permissions by least privilege.

### 14.3 Security considerations

- Avoid unnecessary root sessions
- Verify the target path before recursive commands
- Understand a command's effects before running it privileged
- Never paste unknown commands into a root shell
- Protect files containing credentials or keys
- Avoid unsafe changes like making sensitive directories universally writable

---

## 15. 🧭 Advanced Troubleshooting Methodology

Good troubleshooting starts with observation, not random configuration changes.

```mermaid
flowchart LR
    A[Check service state] --> B[Review logs]
    B --> C[Check listening sockets]
    C --> D[Inspect network config]
    D --> E[Test the connection]
    E --> F[Record findings]
```

**Example: a service is unavailable**

**Step 1: Check service state**

```bash
systemctl status SERVICE_NAME
```

**Step 2: Review recent logs**

```bash
journalctl -u SERVICE_NAME -n 100
```

**Step 3: Check listening sockets**

```bash
ss -lntp
```

**Step 4: Inspect network configuration**

```bash
ip addr show
```

```bash
ip route
```

**Step 5: Test the relevant connection.** Use a test appropriate to the service and your authorized environment.

**Step 6: Record the findings.** Document the failure, supporting output, corrective action, and verification result.

> Replace `SERVICE_NAME` with the actual systemd unit name. These commands give diagnostic evidence but don't identify every possible root cause.

---

## 16. ⚠️ Common Command-Line Risks

| Risk | Why it matters | Safer approach |
|------|----------------|----------------|
| Incorrect recursive path | Can modify or delete large directory trees | Verify paths before running |
| Overwriting files with `>` | Existing output can be lost | Check the destination or use a new filename |
| Unnecessary root privileges | Raises the cost of mistakes | Use least privilege |
| Killing the wrong PID | May interrupt essential services | Verify the process first |
| Extracting untrusted archives | May overwrite files or create unexpected paths | Inspect contents first |
| Sharing raw logs | May expose usernames, addresses, or secrets | Sanitize evidence |
| Changing network services remotely | Can disconnect your session | Plan access and recovery first |

---

## 17. 🏁 Technical Summary

Advanced Linux command-line proficiency means knowing how to inspect, manage, troubleshoot, and secure a Linux system with native utilities.

**This reference covers**

- System and hardware enumeration
- File discovery and metadata analysis
- Permission and ownership management
- Process inspection and termination
- Service management with systemd
- Network-interface and socket analysis
- Filesystem and storage investigation
- Log analysis and security monitoring
- Shell pipelines and text processing
- Archive management and file integrity
- Package management on Debian-based systems
- Environment variables and privilege controls
- Structured troubleshooting

**Skill focus:** Linux administration, command-line troubleshooting, system enumeration, network diagnostics, and security-focused log analysis.

### 📎 References

- [Linux manual pages](https://man7.org/linux/man-pages/)
- [GNU Coreutils](https://www.gnu.org/software/coreutils/manual/)
- [systemd documentation](https://www.freedesktop.org/software/systemd/man/)
- [Debian documentation](https://www.debian.org/doc/)
- [Arch Linux Wiki](https://wiki.archlinux.org/)

*Documentation type: Linux command-line technical reference. Add sanitized terminal output and verified examples from your own Parrot OS or Kali Linux environment to demonstrate practical application.*

---

<div align="center">

**Observe first. Change second. Document everything.** 🐧

</div>
