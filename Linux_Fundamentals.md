# Linux Fundamentals — CyberClub Workshop

## Table of Contents

1. [What is Linux?](#1-what-is-linux)
2. [The Terminal & Shell](#2-the-terminal--shell)
3. [File System Navigation](#3-file-system-navigation)
4. [File Operations](#4-file-operations)
5. [Permissions & Ownership](#5-permissions--ownership)
6. [Users & Groups](#6-users--groups)
7. [Processes & Services](#7-processes--services)
8. [Networking Basics](#8-networking-basics)
9. [Text Processing Power Tools](#9-text-processing-power-tools)
10. [Hands-On Challenges](#10-hands-on-challenges)

---

## 1. What is Linux?

Linux is an **open-source operating system** kernel. Unlike Windows or macOS, Linux is:
- **Free** — no licensing costs
- **Open source** — anyone can view and modify the code
- **Secure** — permission-based architecture
- **Dominant in servers & cybersecurity** — most servers, cloud infra, and hacking tools run on Linux

### Linux Distributions (Distros)

| Distro | Best For |
|--------|----------|
| **Kali Linux** | Penetration testing & CTFs (comes with 600+ tools) |
| **Ubuntu** | Beginners, general desktop/server use |
| **Debian** | Stability-focused servers |
| **Parrot OS** | Security & privacy (lighter than Kali) |
| **Arch Linux** | Advanced users who want full control |

### Linux vs Windows: Key Differences

| Concept | Windows | Linux |
|---------|---------|-------|
| File system | `C:\Users\` | `/home/user/` |
| Path separator | `\` | `/` |
| Case sensitivity | `File.txt` = `file.txt` | `File.txt` ≠ `file.txt` |
| Software install | `.exe` installer | `apt install` / `pacman -S` |
| Admin user | Administrator | `root` (via `sudo`) |
| GUI vs CLI | Mostly GUI | CLI is first-class |

---

## 2. The Terminal & Shell

### What is a Shell?

A **shell** is a program that interprets commands. The most common is **Bash** (Bourne Again SHell).

### Opening the Terminal

- **Kali/Ubuntu:** Right-click desktop → "Open Terminal" or `Ctrl+Alt+T`
- **macOS:** Cmd+Space → type "Terminal"
- **SSH (remote):** `ssh user@ip_address`

### Your First Commands

```bash
# Who am I?
whoami

# Where am I?
pwd        # print working directory

# What's in this directory?
ls
ls -la     # detailed view (including hidden files)

# Clear the screen
clear

# Get help on any command
man ls     # manual page for ls
ls --help  # quick help
```

### Command Structure

```bash
command [OPTIONS] [ARGUMENTS]

# Examples:
ls -la /home        # command=ls, options=-l -a, argument=/home
mkdir -p new/dir    # create nested directories
echo "Hello World"  # print text
```

### Key Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Autocomplete command or filename |
| `↑` / `↓` | Scroll through command history |
| `Ctrl+C` | Kill the current running command |
| `Ctrl+D` | Exit terminal / EOF |
| `Ctrl+L` | Clear screen (same as `clear`) |
| `Ctrl+A` | Go to beginning of line |
| `Ctrl+E` | Go to end of line |
| `Ctrl+W` | Delete word before cursor |
| `Ctrl+R` | Search command history |

---

## 3. File System Navigation

### The Linux File System Tree

```
/                   → Root (top of the tree)
├── bin/            → Essential user binaries (ls, cat, cp)
├── sbin/           → System binaries (fdisk, reboot)
├── etc/            → Configuration files
├── home/           → User home directories
│   └── kali/       → Your personal folder (~)
├── var/            → Variable data (logs, databases)
├── tmp/            → Temporary files (cleared on reboot)
├── usr/            → User system resources (programs)
├── opt/            → Optional software
├── proc/           → Virtual filesystem for processes
├── dev/            → Device files (hardware)
└── root/           → Root user's home directory
```

### Navigation Commands

```bash
# Go somewhere
cd /etc              # go to /etc
cd ~                 # go to your home directory
cd ..                # go up one directory
cd ../..             # go up two directories
cd -                 # go to previous directory

# See where you are
pwd                  # print working directory

# List contents
ls                   # basic listing
ls -l                # long format (permissions, size, date)
ls -a                # show hidden files (starting with .)
ls -la               # combined
ls -lh               # human-readable file sizes
ls -R                # recursive (show subdirectories)

# Paths
# Absolute: starts from root
cd /home/kali/Documents

# Relative: starts from current location
cd Documents          # if you're already in /home/kali
cd ../kali/Documents  # go up, then down
```

### Hidden Files

Any file starting with `.` is hidden:

```bash
ls -la ~  # you'll see .bashrc, .ssh, .profile, etc.
```

---

## 4. File Operations

### Creating & Viewing Files

```bash
# Create an empty file
touch file.txt

# Create a file with content
echo "Hello, Linux!" > greeting.txt
echo "Line 2" >> greeting.txt   # append

# View file contents
cat file.txt                    # print entire file
less file.txt                   # paginated view (q to quit)
head -n 5 file.txt              # first 5 lines
tail -n 10 file.txt             # last 10 lines
tail -f log.txt                 # follow file in real-time

# Count lines, words, characters
wc -l file.txt
wc -w file.txt
wc -c file.txt
```

### Copying, Moving, Deleting

```bash
# Copy
cp source.txt destination.txt
cp -r folder/ backup_folder/   # recursive (for directories)

# Move / Rename
mv oldname.txt newname.txt
mv file.txt /tmp/              # move to /tmp

# Delete (⚠️ NO recycle bin!)
rm file.txt                    # delete file
rm -r folder/                  # delete directory and contents
rm -f file.txt                 # force (no confirmation)
rm -rf folder/                 # ⚠️ DANGEROUS — deletes everything
```

### Creating Directories

```bash
mkdir new_folder
mkdir -p parent/child/grandchild   # create nested directories
```

### Finding Files

```bash
# Find by name
find /home -name "*.txt"
find / -name "flag.txt" 2>/dev/null   # search from root, hide errors

# Locate (faster, uses database)
sudo updatedb       # update database first
locate flag.txt

# Which command is being run?
which python
which ls
```

---

## 5. Permissions & Ownership

### Viewing Permissions

```bash
ls -l file.txt
# Output: -rwxr-xr-- 1 kali kali 1024 May 9 10:00 file.txt
#          ^^^^^^^^^  ^^^  ^^^^
#          permissions user group
```

### Permission Breakdown

```
-rwxr-xr--
│└┬┘└┬┘└┬┘
│ │  │  └── Others (everyone else): r--
│ │  └───── Group (same group):      r-x
│ └──────── Owner (user):           rwx
└────────── File type (-=file, d=directory, l=symlink)
```

- **r** = read (4)
- **w** = write (2)
- **x** = execute (1)
- **-** = no permission

### Changing Permissions

```bash
# Symbolic mode
chmod u+x file.sh       # add execute for owner
chmod g-w file.txt      # remove write for group
chmod o+r file.txt      # add read for others
chmod a+x script.sh     # add execute for everyone

# Numeric mode (octal)
chmod 755 script.sh     # rwxr-xr-x (owner:7, group:5, others:5)
chmod 644 file.txt      # rw-r--r--
chmod 600 secret.txt    # rw------- (only owner can read/write)
chmod 777 file           # ⚠️ rwxrwxrwx (DANGEROUS — everyone can do everything)

# Common permissions:
# 755 → executables, directories
# 644 → regular files
# 600 → sensitive files (SSH keys)
# 400 → read-only configs
```

### Changing Ownership

```bash
chown user:group file.txt
chown kali file.txt              # change owner only
chown :kali file.txt             # change group only
chown -R kali:kali /home/kali/   # recursive
```

### Special: SUID, SGID, Sticky Bit

```bash
chmod u+s binary     # SUID — runs with owner's permissions
chmod g+s directory  # SGID — new files inherit group
chmod +t /tmp        # Sticky bit — only owner can delete their files
```

---

## 6. Users & Groups

### User Information

```bash
whoami            # current username
id                # user ID, group IDs
id kali           # info for specific user
users             # who's logged in
who               # detailed login info
w                 # who's doing what
last              # login history
```

### Managing Users (requires sudo/root)

```bash
# Add user
sudo useradd -m newuser       # create user with home directory
sudo passwd newuser           # set password

# Modify user
sudo usermod -aG sudo newuser  # add to sudo group
sudo usermod -d /home/new newuser  # change home directory

# Delete user
sudo userdel -r newuser       # delete user and home directory
```

### Groups

```bash
groups              # list your groups
groups kali         # list groups for user kali
groupadd hackers    # create a group
groupdel hackers    # delete a group
```

### The Superuser (root)

```bash
sudo command          # run command as root
sudo -i               # open root shell (careful!)
sudo -u kali command  # run as user kali
```

---

## 7. Processes & Services

### Viewing Processes

```bash
ps                    # your processes
ps aux                # all processes (detailed)
ps -ef                # all processes (standard format)
top                   # live process viewer (q to quit)
htop                  # prettier version (install if needed)

# Find a specific process
ps aux | grep nginx
pgrep -a nginx        # find PID of process
```

### Managing Processes

```bash
# Background a process
sleep 30 &            # run in background

# Bring to foreground
fg                    # bring last background job to foreground

# List background jobs
jobs

# Kill processes
kill 1234             # graceful termination (SIGTERM)
kill -9 1234          # force kill (SIGKILL) — last resort
killall nginx         # kill all processes named nginx
pkill -f "pattern"    # kill by pattern
```

### Services (systemd)

```bash
systemctl status ssh       # check if SSH is running
systemctl start ssh        # start a service
systemctl stop ssh         # stop a service
systemctl restart ssh      # restart
systemctl enable ssh       # start on boot
systemctl disable ssh      # don't start on boot

service ssh status         # older syntax (still works)
```

---

## 8. Networking Basics

### Check Network Configuration

```bash
ip a                    # show all network interfaces (modern)
ifconfig                # older (may need net-tools)
ip r                    # routing table
```

### Test Connectivity

```bash
ping 8.8.8.8            # test internet connectivity (Ctrl+C to stop)
ping -c 4 google.com    # send 4 pings only

curl http://example.com         # fetch a URL
curl -I http://example.com      # fetch only headers
wget http://example.com/file    # download a file
```

### DNS & Name Resolution

```bash
nslookup google.com     # DNS lookup
dig google.com          # detailed DNS info
host google.com         # simple DNS lookup
```

### Ports & Connections

```bash
ss -tuln                # listening ports (modern)
netstat -tuln           # listening ports (older)
ss -tup                 # active connections with processes

# Check if a port is open
nc -zv 192.168.1.1 80   # netcat scan
nmap -p 22,80 192.168.1.1  # nmap (if installed)
```

### SSH (Secure Shell)

```bash
ssh kali@192.168.1.100              # connect to remote machine
ssh -i key.pem ubuntu@ec2.amazon.com  # connect with key file
scp file.txt kali@192.168.1.100:/tmp/  # copy file over SSH
```

---

## 9. Text Processing Power Tools

These are **essential** for CTFs and cybersecurity work.

### grep — Search Text

```bash
grep "flag" file.txt               # find lines containing "flag"
grep -i "flag" file.txt            # case-insensitive
grep -r "password" /etc/           # recursive search in directory
grep -v "error" log.txt            # invert match (lines WITHOUT error)
grep -c "failed" auth.log          # count matches
grep -E "flag|secret|key" file     # extended regex (OR)
```

### sed — Stream Editor

```bash
sed 's/old/new/g' file.txt         # replace all occurrences
sed -i 's/old/new/g' file.txt      # edit file in-place
sed '5d' file.txt                  # delete line 5
sed '1,10d' file.txt               # delete lines 1-10
```

### awk — Text Processing

```bash
awk '{print $1}' file.txt          # print first column
awk '{print $1, $3}' file.txt      # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd  # use : as delimiter
awk '$3 > 50' file.txt             # print lines where col3 > 50
```

### cut — Extract Columns

```bash
cut -d: -f1 /etc/passwd           # first field, delimiter :
cut -c1-10 file.txt                # first 10 characters of each line
```

### sort & uniq

```bash
sort file.txt                      # sort alphabetically
sort -n file.txt                  # sort numerically
sort -r file.txt                  # reverse sort
sort file.txt | uniq               # remove duplicates
sort file.txt | uniq -c            # count occurrences
```

### Pipes & Redirection

```bash
# Pipe: send output from one command to another
cat log.txt | grep "error" | head -10

# Redirect output to a file
echo "hello" > file.txt       # overwrite
echo "world" >> file.txt      # append

# Redirect stderr
command 2>/dev/null           # ignore errors
command 2>&1                  # merge stderr with stdout

# Tee: output to file AND screen
command | tee output.txt
```

---

## 10. Hands-On Challenges

These challenges are designed to be run **live during the workshop**. Each builds on the previous one.

### Challenge 1: Linux Orientation

**Goal:** Get comfortable with the terminal and navigation.

```bash
# Step 1: Find out who you are and where you are
whoami
pwd

# Step 2: Explore the file system
ls /
ls /etc
ls /var/log

# Step 3: Go to your home directory
cd ~
pwd
ls -la

# Step 4: Check what shell you're using
echo $SHELL

# 🔥 Flag: What is the absolute path to your home directory?
```

---

### Challenge 2: File Detective

**Goal:** Use `find`, `grep`, and file operations to locate hidden data.

```bash
# Step 1: Create a workspace
mkdir -p ~/challenge2
cd ~/challenge2

# Step 2: Create several files with different content
echo "The password is hidden elsewhere" > note.txt
echo "FLAG{Linux_File_Ops}" > secret.txt
echo "This is just a decoy" > readme.txt
mkdir hidden_folder
echo "FLAG{Hidden_Directory}" > hidden_folder/flag.txt

# Step 3: Practice finding files
find ~/challenge2 -name "*.txt"
grep -r "FLAG" ~/challenge2/

# 🔥 Challenge: Use find and grep to locate all flags
# Expected flags: FLAG{Linux_File_Ops} and FLAG{Hidden_Directory}
```

**Solution:**
```bash
find ~/challenge2 -type f -name "*.txt" -exec cat {} \;
# OR
grep -r "FLAG" ~/challenge2/
```

---

### Challenge 3: Permission Puzzle

**Goal:** Understand and manipulate file permissions.

```bash
# Step 1: Create a script
cd ~
echo '#!/bin/bash
echo "FLAG{Permissions_Master}"' > get_flag.sh

# Step 2: Try to run it
./get_flag.sh
# You'll get: Permission denied

# Step 3: Check permissions
ls -l get_flag.sh
# Output: -rw-r--r-- ... (no x)

# Step 4: Make it executable
chmod +x get_flag.sh

# Step 5: Run it again
./get_flag.sh
# Output: FLAG{Permissions_Master}

# 🔥 Challenge: Create a file that only YOU can read/write/execute
#              (everyone else should have zero access)
```

**Solution:**
```bash
chmod 700 private_script.sh
# OR
chmod u=rwx,g=,o= private_script.sh
```

---

### Challenge 4: The sudo Game

**Goal:** Understand privilege escalation basics.

```bash
# Step 1: What can you run with sudo?
sudo -l

# Step 2: Read a protected file
sudo cat /etc/shadow
# Wait — that's the password hashes! Let's use a safer target:

sudo cat /var/log/syslog | head -20

# Step 3: Write to a protected location
sudo touch /opt/i_was_here
ls -l /opt/i_was_here

# 🔥 Challenge: Create a file in /root/ (root's home) that says "FLAG{sudo_pwned}"
```

**Solution:**
```bash
echo "FLAG{sudo_pwned}" | sudo tee /root/flag.txt
# OR
sudo sh -c 'echo "FLAG{sudo_pwned}" > /root/flag.txt'
```

---

### Challenge 5: Process Hunter

**Goal:** Monitor and control processes.

```bash
# Step 1: Run a background process
sleep 300 &

# Step 2: Find it
ps aux | grep sleep

# Step 3: Kill it
kill <PID>
# Verify it's gone
ps aux | grep sleep

# Step 4: Use top/htop
top
# Press 'q' to quit

# 🔥 Challenge: Start a process, find its PID using pgrep, and kill it with SIGKILL
```

**Solution:**
```bash
sleep 999 &
pgrep -a sleep
kill -9 <PID>
```

---

### Challenge 6: Network Scout

**Goal:** Basic network reconnaissance.

```bash
# Step 1: Find your IP address
ip a
# Look for something like: inet 192.168.x.x

# Step 2: Test connectivity
ping -c 4 8.8.8.8
ping -c 4 google.com

# Step 3: DNS lookup
nslookup google.com
host ctf101.org

# Step 4: Check what ports are listening
ss -tuln
netstat -tuln

# Step 5: Make a web request
curl -I https://example.com

# 🔥 Challenge: Use curl to fetch the flag from a fake server
# echo "FLAG{Network_Pro}" | nc -l -p 9999 &
# In another terminal: curl http://localhost:9999
```

**Solution (requires two terminals or backgrounding):**
```bash
# Terminal 1 (background):
echo "FLAG{Network_Pro}" | nc -l -p 9999 &

# Terminal 2:
curl http://localhost:9999
```

---

### Challenge 7: Text Processing Pipeline

**Goal:** Combine tools to extract information.

```bash
# Step 1: Create a messy data file
cat > ~/challenge7_data.txt << 'EOF'
user:john:password123:admin
user:jane:qwerty:user
user:admin:supersecret:admin
user:bob:letmein:user
user:alice:hunter2:moderator
EOF

# Step 2: Extract all usernames (first column)
cut -d: -f2 ~/challenge7_data.txt

# Step 3: Find admin users
grep "admin" ~/challenge7_data.txt

# Step 4: Count users by role
cut -d: -f4 ~/challenge7_data.txt | sort | uniq -c

# Step 5: Extract admin passwords only
grep "admin" ~/challenge7_data.txt | cut -d: -f3

# 🔥 Challenge: Find the flag hidden in this output:
# FLAG{pipeline_master}
```

**Solution:**
```bash
grep "admin" ~/challenge7_data.txt | cut -d: -f3
# Output: password123
# Output: supersecret
```

---

### Challenge 8: The Ultimate Challenge — CTF-style

**Goal:** Combine everything you've learned.

You're investigating a compromised Linux server. Find all the flags.

```bash
# Create the challenge environment
cd ~
mkdir -p ctf_challenge
cd ctf_challenge

# Hidden in file metadata
echo "backup config" > config.txt
echo "FLAG{metadata_leak}" >> config.txt

# Hidden in a log file
echo "[ERROR] 10.0.0.1 - - [09/May/2026]" > access.log
echo "[INFO] FLAG{log_analysis} - admin logged in" >> access.log
echo "[ERROR] 10.0.0.2 - - [09/May/2026]" >> access.log

# Hidden in a subdirectory
mkdir -p .secret
echo "FLAG{dot_directory}" > .secret/hidden_flag

# Protected file (root-owned simulation)
echo "FLAG{privilege_escalation}" > protected.txt
chmod 600 protected.txt

# Hidden in process output
echo "FLAG{running_process}" > /tmp/flag_process
```

**Tasks:**

| # | Task | Command |
|---|------|---------|
| 1 | Find all text files in ctf_challenge | `find ~/ctf_challenge -type f -name "*.txt"` |
| 2 | Find the string "FLAG" in all files | `grep -r "FLAG" ~/ctf_challenge/` |
| 3 | List ALL files including hidden | `ls -la ~/ctf_challenge/` |
| 4 | Check the contents of .secret/ | `cat ~/ctf_challenge/.secret/hidden_flag` |
| 5 | Try to read protected.txt as root | `sudo cat ~/ctf_challenge/protected.txt` |
| 6 | Find the process flag | `cat /tmp/flag_process` |
| 7 | Search for IP addresses in access.log | `grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" ~/ctf_challenge/access.log` |

**Flags found:** `FLAG{metadata_leak}`, `FLAG{log_analysis}`, `FLAG{dot_directory}`, `FLAG{privilege_escalation}`, `FLAG{running_process}`

---

## Quick Reference Card

```bash
# NAVIGATION
pwd                     # where am I?
ls -la                  # list everything
cd ~                    # go home
cd -                    # go back

# FILE OPS
cat file                # read file
cp file1 file2          # copy
mv file1 file2          # move/rename
rm file                 # delete
mkdir dir               # create directory

# PERMISSIONS
chmod 755 file          # rwxr-xr-x
chown user:group file   # change owner

# FINDING THINGS
find / -name "file"     # search files
grep -r "text" .        # search content
which command           # locate a command

# PROCESSES
ps aux                  # all processes
kill PID                # kill process
Ctrl+C                  # interrupt

# NETWORK
ip a                    # IP addresses
ping host               # test connectivity
ss -tuln                # listening ports
curl URL                # HTTP request

# TEXT PROCESSING
|                       # pipe
grep                   # search
awk '{print $1}'       # column extraction
sed 's/a/b/g'          # replace
sort | uniq -c         # count occurrences

# HELP
man command             # manual
command --help          # quick help
```

## Where to Practice

| Platform | What to Do |
|----------|------------|
| **OverTheWire: Bandit** | SSH-based Linux challenges — start here! |
| **TryHackMe: Linux Fundamentals** | Guided Linux rooms |
| **PicoCTF** | General CTF with Linux-based challenges |
| **Your own Kali VM** | Play around, break things, fix them |

---

**Remember:** Linux is a superpower in cybersecurity. The terminal is your weapon. Practice until it feels natural.

Happy hacking!
