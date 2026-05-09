# Linux Fundamentals — Challenge Solutions

## Connection

```bash
ssh cyberclub@38.247.148.233
# Password: CyberClub2026!
```

---

## Challenge 1: Linux Orientation

There's no flag to capture. Just explore.

```bash
whoami
pwd
ls /
ls -la ~
echo $SHELL
```

---

## Challenge 2: File Detective

**Location:** `~/linux_challenges/challenge1/`

```bash
cd ~/linux_challenges/challenge1
```

**Flag 1 — `ccd{file_detective}`:**
```bash
cat secret.txt
```

**Flag 2 — `ccd{hidden_directory}`:**
```bash
ls -la
cat hidden_folder/flag.txt
```

**Alternative one-liner:**
```bash
grep -r ccd .
find . -type f -name "*.txt" -exec cat {} \;
```

---

## Challenge 3: Permission Puzzle

**Location:** `~/linux_challenges/challenge2/`

```bash
cd ~/linux_challenges/challenge2
```

**Flag 1 — `ccd{permissions_master}`:**

The script `get_flag.sh` has no execute permission initially:
```bash
ls -l get_flag.sh
```
Output: `-rw-r--r--` (no `x`)

Fix it:
```bash
chmod +x get_flag.sh
bash get_flag.sh
# Output: ccd{permissions_master}
```

**Flag 2 — `ccd{root_protected}`:**

The file `root_only.txt` is owned by root with `-rw-------`. You can't read it as `cyberclub`.

Solution: The SUID binary `secret_reader` can read it for you:
```bash
secret_reader
# Output: ccd{suid_exploit}
```

Wait — that's actually flag 3. For flag 2, `root_only.txt` isn't directly accessible. The SUID binary is the intended path.

**Flag 2 — `ccd{root_protected}` (via SUID binary):**
```bash
secret_reader
# This reads admin_secret.txt which contains: ccd{suid_exploit}
```

Actually `root_only.txt` does contain `ccd{root_protected}`. To read it, you need to use the SUID binary approach or elevate. Since you're `cyberclub` with no sudo, the intended path is:

```bash
# Read root_only.txt using the SUID binary's approach
# The secret_reader binary runs as root (SUID) and reads admin_secret.txt
secret_reader
# Output: ccd{suid_exploit}

# For root_only.txt - compile your own reader if gcc were available,
# or use the fact that the SUID binary can read root files
# Actually secret_reader only reads admin_secret.txt
# So: ccd{root_protected} is in root_only.txt which you CAN'T read directly.
# The trick: both flags are accessible through the SUID binary concept.
```

**Correction — the flags in challenge2 are:**
- `ccd{permissions_master}` — fix chmod and run script
- `ccd{root_protected}` — in `root_only.txt` (root-owned, can't read without sudo/SUID)
- `ccd{suid_exploit}` — in `admin_secret.txt`, readable via `secret_reader` (SUID binary)

---

## Challenge 4: Process Hunter

```bash
# Find the hidden flag
find / -name "challenge3_flag.txt" 2>/dev/null
# The flag was moved to: /tmp/.hidden/challenge3_flag.txt
cat /tmp/.hidden/challenge3_flag.txt
# Output: ccd{process_hunter}
```

**Process management demo:**
```bash
sleep 300 &
pgrep -a sleep
kill <PID>
```

---

## Challenge 5: Network Scout

No flag to capture. Exploration only.

```bash
ip a
ping -c 4 8.8.8.8
ping -c 4 google.com
nslookup google.com
ss -tuln
curl -I http://localhost
```

---

## Challenge 6: Text Processing Pipeline

**Location:** `~/linux_challenges/challenge5/`

```bash
cd ~/linux_challenges/challenge5
```

**Flag — `ccd{pipeline_master}`:**
```bash
cat data.txt
# The flag is on the last line
tail -1 data.txt
# OR
grep "ccd" data.txt
```

**Bonus — data extraction:**
```bash
cut -d: -f2 data.txt            # usernames
grep "admin" data.txt           # admin users
cut -d: -f4 data.txt | sort | uniq -c  # count by role
```

---

## Challenge 7: Hidden Flag Hunt

**Location:** `~/linux_challenges/challenge6/`

```bash
cd ~/linux_challenges/challenge6
```

**Flag 1 — `ccd{dotfile_hunter}`:**
```bash
ls -la
cat .hidden_flag
```

**Flag 2 — `ccd{nested_discovery}`:**
```bash
find . -type f -name "*.txt"
cat a/b/c/d/e/f/g/flag.txt
```

**Flag 3 — `ccd{archive_extractor}`:**
```bash
tar -xf archive.tar
cat flag_archive.txt
```

---

## Challenge 8: Grep Master

**Location:** `~/linux_challenges/challenge7/`

```bash
cd ~/linux_challenges/challenge7
```

**Flag — `ccd{grep_master}`:**
```bash
grep "ccd" access.log
# Output: [ERROR] 192.168.1.1 - ccd{grep_master} - authentication failed
```

---

## Challenge 9: Final Showdown

**Location:** `~/linux_challenges/challenge8/`

```bash
cd ~/linux_challenges/challenge8
```

**Flag — `ccd{final_showdown}`:**
```bash
cat victory.txt
```

---

## All Flags Summary

| # | Flag | Location |
|---|------|----------|
| 1 | `ccd{file_detective}` | `challenge1/secret.txt` |
| 2 | `ccd{hidden_directory}` | `challenge1/hidden_folder/flag.txt` |
| 3 | `ccd{permissions_master}` | `challenge2/get_flag.sh` (after chmod) |
| 4 | `ccd{root_protected}` | `challenge2/root_only.txt` (root-owned) |
| 5 | `ccd{suid_exploit}` | `challenge2/admin_secret.txt` (via SUID binary) |
| 6 | `ccd{process_hunter}` | `/tmp/.hidden/challenge3_flag.txt` |
| 7 | `ccd{pipeline_master}` | `challenge5/data.txt` |
| 8 | `ccd{dotfile_hunter}` | `challenge6/.hidden_flag` |
| 9 | `ccd{nested_discovery}` | `challenge6/a/b/c/d/e/f/g/flag.txt` |
| 10 | `ccd{archive_extractor}` | `challenge6/flag_archive.txt` (in archive.tar) |
| 11 | `ccd{grep_master}` | `challenge7/access.log` |
| 12 | `ccd{final_showdown}` | `challenge8/victory.txt` |

**Total: 12 flags**
