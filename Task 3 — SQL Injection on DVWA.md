# Task 3 — SQL Injection on DVWA (Low Security)

## What is SQL injection?
SQL injection happens when an application takes user input and drops it straight into a database query without checking or escaping it. In that situation, the database can't tell the difference between "data" and "code." An attacker can craft input that changes the query logic — letting them see data they shouldn't, bypass login checks, or even modify/delete records.

---

## Environment setup
DVWA was installed locally on Kali Linux using the LAMP stack (Apache2, MariaDB, PHP 8.4.24). A dedicated MySQL user (`dvwa`) and database (`dvwa`) were created with DVWA's default settings. The security level was set to **Low**, meaning no input sanitisation or protection — deliberately insecure for training.

---

## What the payloads exposed
Two test payloads were run against the SQL Injection module (see `sql_injection_notes.md` for full details):

1. **`' OR '1'='1`** — bypassed the single‑user lookup and returned every user's first name and surname.
2. **`' UNION SELECT user, password FROM users -- -`** — extracted usernames and password hashes from the `users` table. This revealed that two accounts (`admin` and `smithy`) shared the same password, a well‑known MD5 hash of "password."

This shows how a simple login or search form, if built insecurely, can leak an entire user table including credentials.

---

## Why these payloads worked
DVWA built its SQL query by directly inserting raw input into a string. When the input contained SQL syntax (quotes, `OR`, `UNION SELECT`), the database treated it as part of the query itself. That's why both payloads succeeded — the application never separated "user data" from "SQL code."

---

## How to fix it
The reliable fix is to use **parameterised queries** (prepared statements). Instead of building queries by string concatenation, the query structure is defined first with placeholders, and user input is passed in separately as data. The database then treats input strictly as values, not executable SQL.

**PHP (PDO example):**
```php
// Vulnerable version:
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysqli_query($conn, $query);

// Fixed version:
$stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = :id");
$stmt->execute(['id' => $id]);
$result = $stmt->fetchAll();
```

Even if a user enters something like `' OR '1'='1`, the database treats the entire string as a literal value to search for, not as SQL syntax — so the injection simply fails to match any real user ID instead of altering the query logic.