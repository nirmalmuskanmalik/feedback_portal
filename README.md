# Student Feedback Portal — Vulnerable vs Safe
## Use case
A small student feedback portal: students log in, search other
students' reviews, and post a review/comment about a course.

## Tech stack
- Backend: Node.js + Express
- DB: MySQL (mysql2 driver)
- Frontend: plain HTML + vanilla JS (served by Express)
- Two folders in one repo: `vulnerable/` and `safe/` — same features,
  different security posture (toggle = which folder you run).

## How to run
```
cd vulnerable   # or: cd safe
npm install
mysql -u root -p < ../db/schema.sql
node app.js     # http://localhost:3000
```

## Vulnerabilities implemented (vulnerable/) and their fix (safe/)

| # | Vuln | Where | Payload that works (vulnerable) | Fix (safe) |
|---|------|-------|----------------------------------|------------|
| 1 | Reflected XSS | `GET /search?q=` echoes `q` straight into HTML | `<script>alert(document.cookie)</script>` in the search box | HTML-escape `q` with a templating/escape function before output |
| 2 | Stored XSS | `POST /review` saves `comment` raw, `GET /reviews` renders it raw | `<img src=x onerror=alert('stored-xss')>` as a review comment | Escape on output (or sanitize on input) before inserting into DOM |
| 3 | DOM-based XSS | client JS does `document.getElementById('welcome').innerHTML = location.hash.substring(1)` | visit `/#<img src=x onerror=alert(1)>` | use `textContent` instead of `innerHTML`, or sanitize hash value |
| 4 | CSRF | `POST /change-email` changes email with only a session cookie, no token | auto-submitting HTML form hosted anywhere, pointing at `/change-email` | add CSRF token (`csurf`-style) + `SameSite=Strict` cookie |
| 5 | SQLi – Easy | `POST /login` builds query by string concatenation | `' OR '1'='1' -- ` as username | parameterized query (`?` placeholders) |
| 6 | SQLi – Intermediate | `GET /profile?id=` boolean-based blind | `1 AND 1=1` vs `1 AND 1=2` → different response | parameterized query + strict input validation (must be integer) |
| 7 | SQLi – Advanced | `GET /search-user?name=` UNION-based | see staged payloads below | parameterized query everywhere |

## Advanced SQLi — staged payloads (document these with screenshots)
1. Find column count:
   `test' UNION SELECT NULL,NULL,NULL-- -` (increase NULLs until no error)
2. Find DB/table names:
   `test' UNION SELECT table_schema,table_name,NULL FROM information_schema.tables-- -`
3. Find column names of `users` table:
   `test' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'-- -`
4. Dump data:
   `test' UNION SELECT username,password,NULL FROM users-- -`

## Marking-aligned checklist
- [ ] Reflected / Stored / DOM XSS — working in vulnerable, blocked in safe (screenshots)
- [ ] CSRF — working in vulnerable, blocked in safe (screenshots)
- [ ] SQLi easy / intermediate / advanced (all 4 stages) — working in vulnerable, blocked in safe
- [ ] GitHub repo with full commit history
- [ ] seed-data.sql populated
- [ ] Word file with payload + screenshot per entry (use this README's table as the format)

### Authors: Nirmal & Abdullah
