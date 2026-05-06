# CSIA CTF 2026 Writeups

Writeups from challenges solved during **CSIA CTF 2026** with team **GHOSTSHELL**.

Team result: **1st place**

## Categories

| Category | Challenge | Main techniques |
| --- | --- | --- |
| Web | [JWT Confusion](web/jwt-confusion.md) | JWT `alg=none`, cookie manipulation, AES-CBC decryption |
| Web | [NodeJS Academy](web/nodejs-academy-prototype-pollution.md) | Prototype pollution, access control bypass |
| Web | [L'Enigme du Code Apogee](web/enigme-code-apogee.md) | Apache 2.4.49 CVE-2021-41773, RCE, backup leakage |
| Steganography | [L3WILIL](steganography/l3wilil.md) | LSB steganography, zsteg, Base64 |
| Misc | [QuadraSignals](misc/quadrasignals.md) | Signal decoding, binary extraction, Python scripting |
| OSINT | [The Hidden Archive](osint/the-hidden-archive.md) | Archive inspection, JSON search, flag reconstruction |

## Notes

These writeups focus on methodology:

- initial reconnaissance
- hypothesis building
- exploitation or extraction steps
- final flag recovery
- key takeaways

Some challenge infrastructure may no longer be online after the CTF.

