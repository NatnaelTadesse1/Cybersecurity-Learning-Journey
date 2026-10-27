# 🚩 OverTheWire: Bandit — CTF Write-up & Learning Journal

Welcome to my write-up and learning reference for the **OverTheWire: Bandit** wargame! Bandit is a fantastic, hands-on CTF designed to build and sharpen core Linux command-line skills, security concepts, network utilities, and Git operations.

Instead of just dumping raw answers, this document focuses on **how the tools work**, the **reasoning behind every approach**, and **real-world security takeaways** learned along the way.

> 🔒 **Ethical CTF Note:** In accordance with CTF etiquette, actual game passwords are excluded/redacted. All command syntax, logic, and methodology remain fully detailed so you can follow along seamlessly in your own session.

---

## 🛠️ Environment & Prerequisites

To follow along or run through these steps, you'll need:
* **Terminal Environment:** Linux, macOS, or Windows (via WSL or PowerShell SSH).
* **GNU Coreutils:** `ls`, `cat`, `find`, `grep`, `sort`, `uniq`, `tr`, `tar`, `gzip`, `bzip2`.
* **Networking & Security Tools:** `ssh`, `nc` (netcat), `openssl`, `nmap`.
* **Version Control:** `git`.

### Server Connection Details
* **Host:** `bandit.labs.overthewire.org`
* **Port:** `2220`
* **Level 0 User:** `bandit0`
* **Level 0 Pass:** `bandit0`

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

---

## 📚 Level Walkthroughs

### Level 0 → Level 1: SSH Basics & File Access
* **Objective:** Log into the game server and retrieve the flag from `readme`.
* **Commands:**
  ```bash
  ssh bandit0@bandit.labs.overthewire.org -p 2220
  cat readme
  ```
* **Takeaway:** SSH provides secure remote shell access. File permissions determine what a user can read after authenticating.

---

### Level 1 → Level 2: Filenames Starting with `-`
* **Objective:** Read a file named `-` in the home directory.
* **Commands:**
  ```bash
  cat ./-
  # Alternative:
  cat -- -
  ```
* **Reasoning:** Command-line utilities interpret arguments starting with `-` as options. Prepending `./` (current directory) or using `--` tells the parser that standard options end here.

---

### Level 2 → Level 3: Spaces in Filenames
* **Objective:** Read `spaces in this filename`.
* **Commands:**
  ```bash
  cat "spaces in this filename"
  # Alternative:
  cat spaces\ in\ this\ filename
  ```
* **Reasoning:** Shells use whitespace to separate arguments. Wrapping paths in double quotes or escaping spaces using backslashes (`\`) stops word-splitting.

---

### Level 3 → Level 4: Hidden Files
* **Objective:** Locate the password file hidden inside the `inhere` folder.
* **Commands:**
  ```bash
  cd inhere
  ls -la
  cat .hidden_file_name
  ```
* **Takeaway:** Leading dots (`.`) hide files from regular `ls` output. Hiding files is just visual convention—it isn't encryption or access control.

---

### Level 4 → Level 5: Inspecting File Types
* **Objective:** Find the human-readable text file among multiple binary files in `inhere`.
* **Commands:**
  ```bash
  cd inhere
  file ./*
  cat <target-text-file>
  ```
* **Takeaway:** File extensions in Linux are optional and cosmetic. Use `file` to check magic bytes and headers to determine actual file types.

---

### Level 5 → Level 6: Searching by File Attributes
* **Objective:** Find a file in `inhere` that is human-readable, non-executable, and exactly 1033 bytes in size.
* **Commands:**
  ```bash
  find inhere -type f -size 1033c ! -executable
  ```
* **Reasoning:** `find` allows you to stack search predicates (`-type`, `-size`, `! -executable`) to quickly filter down large directory trees.

---

### Level 6 → Level 7: Searching by System Metadata
* **Objective:** Find a file anywhere on the system owned by user `bandit7`, group `bandit6`, and 33 bytes in size.
* **Commands:**
  ```bash
  find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
  ```
* **Reasoning:** Redirecting standard error (`2>/dev/null`) suppresses "Permission denied" warnings while searching root `/`, keeping your output clean.

---

### Level 7 → Level 8: Text Pattern Searching
* **Objective:** Extract the password next to the word `millionth` inside `data.txt`.
* **Commands:**
  ```bash
  grep 'millionth' data.txt
  ```
* **Takeaway:** Searching large files line-by-line manually is inefficient; pattern matching tools like `grep` get instant results.

---

### Level 8 → Level 9: Isolate Unique Lines
* **Objective:** Find the single line of text in `data.txt` that occurs only once.
* **Commands:**
  ```bash
  sort data.txt | uniq -u
  ```
* **Reasoning:** The `uniq` command only compares adjacent lines. Sorting first ensures identical lines are grouped together so `uniq -u` can accurately filter out duplicates.

---

### Level 9 → Level 10: Extracting Strings from Binaries
* **Objective:** Find the printable password string inside binary file `data.txt`, marked by several `=` characters.
* **Commands:**
  ```bash
  strings data.txt | grep '==='
  ```
* **Takeaway:** `strings` pulls out human-readable text sequences from binary data, making it useful for reverse engineering and basic forensic checks.

---

### Level 10 → Level 11: Base64 Decoding
* **Objective:** Decode the Base64-encoded text in `data.txt`.
* **Commands:**
  ```bash
  base64 -d data.txt
  ```
* **Takeaway:** Encoding standardizes data for transmission—it does not offer confidentiality or encryption.

---

### Level 11 → Level 12: Caesar Cipher / ROT13
* **Objective:** Decode text in `data.txt` obfuscated using ROT13.
* **Commands:**
  ```bash
  cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
  ```
* **Takeaway:** ROT13 is simple letter substitution. It can be easily reversed with character-mapping utilities like `tr`.

---

### Level 12 → Level 13: Hex Dumps & Layered Archives
* **Objective:** Reconstruct binary data from a hex dump and unpack multiple compressed archive layers.
* **Commands:**
  ```bash
  mkdir /tmp/workspace && cp data.txt /tmp/workspace/
  cd /tmp/workspace
  xxd -r data.txt > stage1
  file stage1
  ```
* **Extraction Loop:** Inspect each step with `file`, rename files appropriately (`.gz`, `.bz2`, `.tar`), and unpack:
  ```bash
  gzip -d stage1.gz
  bunzip2 stage2.bz2
  tar -xf stage3.tar
  ```
* **Takeaway:** Always work inside isolated temporary directories (`/tmp`). Never trust file extensions—rely on file headers.

---

### Level 13 → Level 14: SSH Private Key Authentication
* **Objective:** Log into user `bandit14` using the provided SSH key `sshkey.private`.
* **Commands:**
  ```bash
  ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
  ```
* **Takeaway:** Asymmetric key pairs replace password authentication. Keep private keys secured and restricted.

---

### Level 14 → Level 15: Local TCP Connections
* **Objective:** Send the current password to a service running on local port `30000`.
* **Commands:**
  ```bash
  nc localhost 30000
  ```
* **Takeaway:** Tools like Netcat (`nc`) interact directly with open TCP ports and network services.

---

### Level 15 → Level 16: Encrypted Connections (SSL/TLS)
* **Objective:** Send the current password to port `30001` using SSL/TLS encryption.
* **Commands:**
  ```bash
  openssl s_client -connect localhost:30001 -quiet
  ```
* **Takeaway:** SSL/TLS wraps TCP traffic in an encrypted tunnel, protecting sensitive data over network hops.

---

### Level 16 → Level 17: Port Scanning & SSH Key Extraction
* **Objective:** Scan ports `31000–32000`, locate the SSL service returning credentials, and log into the next level.
* **Commands:**
  ```bash
  nmap -p 31000-32000 localhost
  openssl s_client -connect localhost:<PORT> -quiet
  ```
* **Login Steps:** Save the returned private key, lock down file permissions, and connect:
  ```bash
  nano /tmp/b17.key
  chmod 600 /tmp/b17.key
  ssh -i /tmp/b17.key bandit17@localhost -p 2220
  ```
* **Takeaway:** Port scanning reveals listening endpoints. SSH keys require strict permission sets (`chmod 600`) to be accepted by SSH clients.

---

### Level 17 → Level 18: File Comparisons (Diffing)
* **Objective:** Find the changed line between `passwords.old` and `passwords.new`.
* **Commands:**
  ```bash
  diff passwords.old passwords.new
  ```
* **Takeaway:** `diff` rapidly highlights differences between file versions or system configs.

---

### Level 18 → Level 19: Bypassing Shell Restrictions via SSH
* **Objective:** Read `readme` when interactive shell access is blocked upon SSH connection.
* **Commands:**
  ```bash
  ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
  ```
* **Takeaway:** You can append commands directly to an SSH login string to execute them remotely without invoking an interactive shell.

---

### Level 19 → Level 20: SUID Privilege Execution
* **Objective:** Use the SUID helper binary (`bandit20-do`) to read protected credentials.
* **Commands:**
  ```bash
  ./bandit20-do cat /etc/bandit_pass/bandit20
  ```
* **Takeaway:** SUID binaries execute with the privileges of the file owner, not the executing user.

---

### Level 20 → Level 21: Port-Based SUID Authentication
* **Objective:** Set up a local listener to receive input from the SUID binary `suconnect`.
* **Terminal 1 (Listener):**
  ```bash
  nc -l -p 12345
  ```
* **Terminal 2 (Trigger):**
  ```bash
  ./suconnect 12345
  ```
* **Takeaway:** Network listeners can receive socket inputs locally to trigger and validate client-server authentication logic.

---

### Level 21 → Level 22: Inspecting Cron Jobs
* **Objective:** Inspect scheduled background tasks inside `/etc/cron.d/` to locate flag files.
* **Commands:**
  ```bash
  cat /etc/cron.d/cronjob_bandit22
  cat /usr/bin/cronjob_bandit22.sh
  cat /tmp/t7O2W9BnBwav2kgb1yVnWaBe4idgFlA3
  ```
* **Takeaway:** System schedulers (cron) run automated background jobs. System auditors check `/etc/cron.*` to monitor scheduled activity.

---

### Level 22 → Level 23: Reversing Hash-Generated Paths
* **Objective:** Analyze a cron script that generates output paths dynamically using MD5 hashes.
* **Commands:**
  ```bash
  cat /usr/bin/cronjob_bandit23.sh
  echo -n "bandit23" | md5sum
  cat /tmp/<MD5_HASH_RESULT>
  ```
* **Takeaway:** Pay attention to string formatting when calculating hashes (e.g., using `echo -n` to exclude trailing newlines).

---

### Level 23 → Level 24: Script Injection in Cron Directories
* **Objective:** Execute arbitrary commands by writing a shell script into a cron job's target directory.
* **Commands:**
  ```bash
  cd /tmp/my_dir
  echo '#!/bin/bash' > script.sh
  echo 'cat /etc/bandit_pass/bandit24 > /tmp/my_dir/flag' >> script.sh
  chmod 777 script.sh
  cp script.sh /var/spool/bandit24/foo/
  ```
* **Takeaway:** Allowing privileged cron scripts to blindly execute files from world-writable paths introduces critical local privilege escalation risks.

---

### Level 24 → Level 25: PIN Brute-Forcing via Shell Loops
* **Objective:** Send the current password along with a 4-digit PIN (`0000–9999`) to port `30002`.
* **Commands:**
  ```bash
  for pin in {0000..9999}; do
    echo "<CURRENT_PASSWORD> $pin"
  done | nc localhost 30002 | grep -v "Wrong"
  ```
* **Takeaway:** Small PIN spaces without rate-limiting or lockout mechanisms are easily brute-forced with simple shell loops.

---

### Level 25 → Level 26: Escaping Restricted Shells via Pagers
* **Objective:** Escape a restricted shell environment running inside `more`.
* **Escape Steps:**
  1. Shrink your terminal window size so `more` pauses output.
  2. Press `v` inside `more` to open `Vim`.
  3. Inside Vim, launch a shell: `:e /etc/bandit_pass/bandit26` or `:set shell=/bin/bash` followed by `:shell`.
* **Takeaway:** Legacy tools and text pagers often contain shell-escape shortcuts that can bypass naive restricted environments.

---

### Level 26 → Level 27: SUID Privileged Command Execution
* **Objective:** Use the binary executable in the home directory to read the target flag.
* **Commands:**
  ```bash
  ./bandit27-do cat /etc/bandit_pass/bandit27
  ```
* **Takeaway:** SUID helpers must strictly constrain user inputs to avoid arbitrary file reading.

---

### Level 27 → Level 28: Cloning Local Git Repositories
* **Objective:** Clone a remote Git repository running on a local port.
* **Commands:**
  ```bash
  git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo /tmp/my_repo
  ```
* **Takeaway:** Version control systems store source code and development artifacts, requiring proper permissions and access controls.

---

### Level 28 → Level 29: Uncovering Hardcoded Credentials in Git Logs
* **Objective:** Retrieve leaked passwords from previous Git commits.
* **Commands:**
  ```bash
  cd /tmp/my_repo
  git log -p
  git show <COMMIT_HASH>
  ```
* **Takeaway:** Deleting sensitive data from your latest code snapshot does not erase it from past Git commit logs.

---

### Level 29 → Level 30: Inspecting Alternative Git Branches
* **Objective:** Find credentials stored inside non-default Git branches.
* **Commands:**
  ```bash
  git branch -a
  git checkout dev
  # Or search log diffs across all branches:
  git log --all -p
  ```
* **Takeaway:** Secrets may be accidentally left behind in feature branches or stale development code.

---

### Level 30 → Level 31: Extracting Secrets from Git Tags
* **Objective:** Find hidden information embedded inside Git tag metadata.
* **Commands:**
  ```bash
  git tag
  git show secret
  ```
* **Takeaway:** Git tags store references to historical states and object metadata that should be audited before pushing repositories publicly.

---

### Level 31 → Level 32: Bypassing `.gitignore` and Triggering Push Hooks
* **Objective:** Create a required file, bypass `.gitignore` restrictions if necessary, and push back to the repository.
* **Commands:**
  ```bash
  git status
  cat README.md
  echo "Hello world" > key.txt
  git add -f key.txt
  git commit -m "Submit solution"
  git push origin master
  ```
* **Takeaway:** Git servers can run custom validation scripts (hooks) during push actions to trigger dynamic checks.

---

### Level 32 → Level 33: Escaping Uppercase Shell Wrappers
* **Objective:** Escape a restricted shell wrapper that forces input to uppercase.
* **Commands:**
  ```text
  $0
  ```
* **Reasoning:** In shell execution, `$0` references the currently running executable (`/bin/sh` or `/bin/bash`). Calling `$0` directly launches a standard interactive shell, bypassing input filters.

---

### Level 33 → Level 34: Graduation! 🎓
* **Objective:** Read the final completion message.
* **Takeaway:** Congratulations! Completing Bandit builds a rock-solid foundation in command-line navigation, file permissions, networking concepts, and security basics.

---

## ⚡ Command Quick-Reference

| Utility | Category | Purpose | Example Usage |
| :--- | :--- | :--- | :--- |
| `ls -la` | Navigation | Show all directory contents (including `.hidden`) | `ls -la` |
| `find` | Search | Filter filesystem by type, size, owner | `find / -type f -size 33c` |
| `grep` | Parsing | Search inside text streams for patterns | `grep 'key' file.txt` |
| `sort` / `uniq` | Parsing | Sort streams and filter duplicate lines | `sort file \| uniq -u` |
| `strings` | Forensics | Pull readable strings from binary files | `strings binary.bin` |
| `base64 -d` | Decoding | Convert Base64 data back to plain text | `base64 -d hash.txt` |
| `xxd -r` | Binaries | Reconstruct binary files from hex dumps | `xxd -r hex.txt > output.bin` |
| `nc` | Network | Connect directly to TCP sockets | `nc localhost 30000` |
| `openssl` | Security | Establish encrypted SSL/TLS client channels | `openssl s_client -connect host:port` |
| `git log -p` | Git | View commit history along with patch diffs | `git log -p` |

---

## 🛡️ Key Cybersecurity Takeaways

1. **Obfuscation ≠ Encryption:** Simple encodings (Base64, ROT13, Hex) convert data formats—they offer no confidentiality.
2. **Audit Git History:** Removing credentials in the latest commit does not remove them from history logs, tags, or unmerged branches.
3. **Lock Down SUID & Scheduled Tasks:** Never allow cron scripts or SUID binaries to execute user-writable files without strict input checks.
4. **Least Privilege Principle:** Always enforce proper file modes (`chmod 600` for keys) and restrict endpoint exposures.

---

*Documented and written as an active learning journal for Linux administration and CTF security fundamentals.*