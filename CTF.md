# CTF (Capture The Flag) — Beginner's Guide

## What is a CTF?

CTF stands for **Capture The Flag**. It is a cybersecurity competition where participants solve challenges to find hidden "flags" (text strings like `flag{...}` or `CTF{...}`). Flags are proof that you solved a challenge.

Think of it like a digital scavenger hunt — each challenge has a secret piece of text hidden somewhere, and your job is to find it using hacking techniques.

### Types of CTF Formats

| Format | How it works |
|---|---|
| **Jeopardy-style** | Challenges are grouped by category (Web, Crypto, Forensics, etc.). Solve them to earn points. Most beginner-friendly. |
| **Attack-Defense** | Each team runs their own server. Attack others while defending yours. Live, intense, for advanced players. |
| **King of the Hill** | Everyone attacks the same server. Whoever holds control the longest wins. |

Most beginners start with **Jeopardy-style** CTFs.

## Challenge Categories — Deep Dive

### 1. Web Exploitation

You are given a website URL that has a vulnerability. Your goal is to exploit it.

**Common vulnerabilities:**
- **SQL Injection** — Trick the database by injecting SQL into input fields
  - Example: Entering `' OR 1=1 --` in a login form to bypass authentication
- **Cross-Site Scripting (XSS)** — Inject JavaScript into a page to steal cookies or redirect users
- **Local File Inclusion (LFI)** — Read files on the server by manipulating URL parameters
  - Example: `?page=../../../etc/passwd`
- **Server-Side Template Injection (SSTI)** — Inject code into template engines like Jinja2 or Twig
- **IDOR (Insecure Direct Object Reference)** — Change an ID parameter to access someone else's data

**Your toolkit:**
- Burp Suite — intercept and modify HTTP requests
- Browser DevTools (F12) — inspect source, network, storage
- `curl` and `wget` — make requests from terminal
- `gobuster` / `dirb` — discover hidden directories

**Example challenge:** A login page that says "admin" in the source code comments. Try `admin:admin` — it works. The flag is on the dashboard.

### 2. Cryptography

You are given an encrypted message or file. Decrypt it to find the flag.

**Common ciphers and how to spot them:**

| Cipher | Looks like | How to break |
|---|---|---|
| **Base64** | Ends with `=` or `==`, only A-Za-z0-9+/ | `echo "dGVzdA==" | base64 -d` |
| **ROT13 / Caesar** | Letters shifted, preserves case | `tr 'A-Za-z' 'N-ZA-Mn-za-m'` or CyberChef |
| **XOR** | Binary data, repeated key | `xortool` or `pwn.xor()` in Python |
| **Vigenère** | Letters, no pattern | Use `vigenere-solver` or frequency analysis |
| **RSA** | Numbers n, e, c | Factor n (try factordb.com), compute d |
| **Morse Code** | Dots and dashes | Online decoder or CyberChef |
| **Hexadecimal** | 0-9a-f pairs | `xxd -r -p` or CyberChef |

**Your toolkit:**
- CyberChef — https://gchq.github.io/CyberChef
- Python with `pycryptodome`
- `hash-identifier` — identify hash types
- CrackStation / HashCat — crack hashes

**Example challenge:** The challenge gives you `R1FRR1FRR0dR0dB`. It's Base64. Decode it → `QGGQGGpGGpA`. Looks like ROT? Decode ROT13 → `DTTDTTcTTcN`. Still garbage. Actually it's Base64 encoded twice. Decode again → `flag{base64_all_the_way}`.

### 3. Reverse Engineering

You are given a binary file (compiled program). Figure out what it does and bypass its protections.

**Approach:**
- **Static analysis** — examine the code without running it
  1. Run `strings binary` — look for flag patterns and interesting strings
  2. `file binary` — check architecture and OS
  3. Open in Ghidra or IDA — decompile to readable code
- **Dynamic analysis** — run and observe behavior
  1. Run with `ltrace ./binary` — log library calls
  2. `strace ./binary` — log system calls
  3. `gdb ./binary` — debug step by step
  4. `objdump -d binary` — disassemble

**Example challenge:** A binary asks for a password. Using `strings`, you see the password in plaintext right in the binary. Type it in and get the flag.

### 4. Forensics

You are given a file (image, PCAP, memory dump, disk image). Find hidden evidence.

**Common scenarios:**
- **Packet capture (PCAP)** — Analyze network traffic with Wireshark
  - Follow TCP streams to see conversations
  - Export HTTP objects (files transferred)
  - Look for DNS exfiltration
- **Memory dump** — Analyze RAM with `volatility` or `strings`
  - Dump processes, clipboard, command history
- **Disk image** — Mount and explore the file system
  - Find deleted files with `photorec` or `foremost`
  - Check file metadata (EXIF data)

**Your toolkit:**
- `strings` — extract all readable text
- `file` — identify file type
- `binwalk` — extract embedded files
- `photorec` / `foremost` — file recovery
- Wireshark / tshark — packet analysis
- Volatility 3 — memory forensics
- `exiftool` — metadata extraction

**Example challenge:** A `.png` image is given. `strings image.png | grep flag` reveals `flag{hidden_in_plain_sight}` appended at the end of the file.

### 5. Binary Exploitation (Pwn)

You are given a binary and its source code (or not). Exploit a memory corruption bug to gain code execution or read the flag.

**Common vulnerabilities:**
- **Buffer overflow** — Write past the end of a buffer to overwrite the return address
- **Format string** — Use `%x`, `%n` in printf to leak or write memory
- **Return-Oriented Programming (ROP)** — Chain small code snippets (gadgets) together
- **Heap exploitation** — Corrupt heap metadata

**Your toolkit:**
- `pwntools` — Python library for exploit development
- `gdb` with `pwndbg` or `gef` — debugging
- `checksec binary` — check security mitigations (ASLR, NX, PIE, RELRO)
- `one_gadget` — find execve gadgets in libc

**Example challenge:** A program reads input into a 64-byte buffer. Send 100 characters — it crashes. Overwrite the return address to jump to a `win()` function that prints the flag.

### 6. Steganography

A message is hidden inside an image, audio file, or video without being obvious.

**Techniques:**
- **LSB (Least Significant Bit)** — The least important bit of each pixel is modified to hide data
  - Use `zsteg` or `stegsolve` to extract
  - Write a Python script to read LSBs
- **Appended data** — Data added after the file's end marker
  - `strings file | tail` or `xxd file | tail`
  - `binwalk file` to find embedded files
- **Image metadata** — EXIF data, comments
  - `exiftool image.jpg`
- **Audio spectrograms** — Sound waves visually encoding text
  - Open in Audacity, switch to spectrogram view
- **Pixel value differences** — Compare two similar images (XOR them)

**Example challenge:** Download a `.wav` file. Open in Audacity, switch to spectrogram view. Read the flag written in the audio frequencies.

### 7. OSINT (Open Source Intelligence)

Find the flag using only publicly available information.

**What to look for:**
- Social media profiles (Twitter, LinkedIn, GitHub)
- Whois records of domains
- Google dorks (advanced search queries)
  - `site:example.com filetype:pdf`
  - `intitle:"index of" password`
- Archived web pages (Wayback Machine)
- Geolocation from photos (GPS coordinates in EXIF)
- Reverse image search (Google Images, TinEye)

**Your toolkit:**
- `theHarvester` — email/domain enumeration
- `sherlock` — find usernames across platforms
- `Google dorks` — advanced search operators
- `Wayback Machine` — https://web.archive.org
- `Shodan` — search internet-connected devices
- `haveibeenpwned` — check breached emails

**Example challenge:** "Find the flag hidden on my GitHub profile." You check their GitHub, look at repo descriptions, browse commits, find a deleted comment with the flag.

## How to Approach Any Challenge — Step by Step

When you see a new challenge, follow this checklist:

```text
1. READ THE DESCRIPTION TWICE — it always contains hints
2. What category is this? (Web / Crypto / Forensics / RE / Pwn / Stego / OSINT)
3. Download and examine the file:
   - What type is it? (file command)
   - Are there any readable strings? (strings command)
   - Is there anything unusual? (ls -la, check file size)
4. Search for writeups or similar challenges online
5. Try the basic technique for that category
6. If stuck, try another approach or ask a teammate
7. Found something that looks like a flag? Submit it!
8. If it doesn't work, look more carefully — flag format might be case-sensitive
```

## Setting Up Your Environment

### Option 1: Kali Linux (Recommended for Beginners)

Kali Linux comes with most tools pre-installed. Use it in a VM:
- Download from https://www.kali.org/get-kali/#kali-virtual-machines
- Import into VirtualBox or VMware
- Default credentials: `kali:kali`

### Option 2: Python + Essential Tools (Lightweight)

If you prefer to stay on your current OS:

```bash
# Install Python tools
pip install pwntools requests beautifulsoup4 pycryptodome

# Install system tools (Ubuntu/Debian)
sudo apt install steghide binwalk exiftool foremost xxd
```

### Option 3: Online Tools (No Installation)

| Tool | Use |
|---|---|
| CyberChef | Decode everything — base64, hex, ROT, XOR, etc. |
| dcode.fr | Cipher identifier + solver |
| factordb.com | Factor large numbers (RSA) |
| crackstation.net | Crack unsalted hashes |
| audacity online or spectrogram app | Audio stego |

## Practice Platforms Ranked by Difficulty

| Platform | Difficulty | Best For |
|---|---|---|
| **PicoCTF** | ★☆☆☆☆ | Absolute beginners |
| **CTFlearn** | ★★☆☆☆ | Beginners with some basics |
| **TryHackMe** | ★★☆☆☆ | Guided learning paths |
| **OverTheWire** | ★★☆☆☆ | Linux/command line skills |
| **HackTheBox Academy** | ★★★☆☆ | Structured learning |
| **pwnable.tw / pwnable.kr** | ★★★★☆ | Binary exploitation |
| **CryptoHack** | ★★★☆☆ | Cryptography only |
| **RingZer0 CTF** | ★★★☆☆ | Mixed difficulty |
| **Root-Me** | ★★★☆☆ | Wide variety of challenges |
| **HackTheBox (machines)** | ★★★★☆ | Real-world pentesting |
| **Google CTF / DEF CON quals** | ★★★★★ | Advanced, competitive |

## Mindset & Tips for Beginners

### Do This:

- **Google everything** — "how to decode base64", "what is SQL injection", "CTF writeup [challenge name]". Someone has documented it.
- **Take notes** — Create a personal cheatsheet. You'll reuse techniques across challenges.
- **Start small** — Don't jump into Pwn or advanced crypto. Do Web and Forensics first.
- **Read writeups** — After trying (or giving up), read how others solved it. You'll learn patterns.
- **Join a team** — CTFs are team events. Talk, share ideas, divide challenges.
- **Use the hint system** — On PicoCTF and similar platforms, hints cost points but are worth it when learning.

### Avoid This:

- **Don't brute force** — If you don't know what you're looking for, randomly trying tools won't help.
- **Don't skip the basics** — Understanding HTTP, binary, hex, and ASCII is essential.
- **Don't cheat** — Using someone else's solution without understanding it teaches you nothing.

### What to Do When You're Stuck

```
1. Re-read the challenge — you missed a hint
2. Check the file again with different tools
3. Search for "[challenge name] writeup"
4. Ask teammates — "hey, anyone working on this?"
5. Look at the challenge from a different category angle
   e.g., a "Forensics" challenge might use crypto inside
6. Try harder — the answer is usually simpler than you think
```

## Real CTF Example Walkthrough

Let's walk through a real beginner challenge from **PicoCTF**:

### Challenge: "Obedient Cat" (Forensics, 5 points)

**Description:** "This file has a flag in plain sight. Download it."

**Step 1:** Download the file. It has no extension.

**Step 2:** Run `file flag` → "ASCII text"

**Step 3:** Run `cat flag` → output: `picoCTF{something_something}`

**Step 4:** Submit the flag. Done.

The point? Sometimes the answer really is that simple. Don't overthink.

### Challenge: "First Grep" (Forensics, 5 points)

**Description:** "Can you find the flag in this file?"

**Step 1:** `file file` → "ASCII text" (but very large, 1000+ lines)

**Step 2:** `strings file | grep flag` → finds the flag line immediately

**Moral:** `strings` and `grep` are your best friends.

## Resources

### YouTube Channels
- **John Hammond** — CTF solve videos, beginner-friendly
- **LiveOverflow** — Deep technical explanations
- **IppSec** — HackTheBox walkthroughs
- **NetworkChuck** — Motivational cybersecurity content
- **PicoCTF walkthroughs** — Search for any past year

### Websites
- **CTFtime** (https://ctftime.org) — Upcoming CTF events, team rankings
- **CTFlearn** (https://ctflearn.com) — Practice challenges with writeups
- **PicoCTF** (https://picoctf.org) — The best place to start
- **CryptoHack** (https://cryptohack.org) — Learn cryptography through challenges
- **XSS Game** (https://xss-game.appspot.com) — Learn XSS step by step

### Books
- *The Web Application Hacker's Handbook* — Web security
- *Practical Binary Analysis* — Reverse engineering
- *Serious Cryptography* — Cryptography fundamentals
- *Hacking: The Art of Exploitation* — Classic introduction

### Cheatsheets
- **Reverse Engineering:** https://github.com/wtsxDev/reverse-engineering
- **SQL Injection:** https://portswigger.net/web-security/sql-injection/cheat-sheet
- **Linux Commands:** https://cheatsheet.dennyzhang.com/cheatsheet-linux-a4

---

**Remember:** The real flag is the knowledge you gain along the way. Happy hacking!
