# CSIA CTF 2026 Writeups

![Event](https://img.shields.io/badge/Event-CSIA%20CTF%202026-111827?style=for-the-badge)
![Team](https://img.shields.io/badge/Team-GHOSTSHELL-0f766e?style=for-the-badge)
![Rank](https://img.shields.io/badge/Rank-1st%20Place-f59e0b?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Web%20%7C%20Stego%20%7C%20Misc%20%7C%20OSINT-2563eb?style=for-the-badge)

Professional writeups for challenges solved during **CSIA CTF 2026** with team **GHOSTSHELL**.

This repository is written as a portfolio artifact: each writeup shows the reasoning path, the commands used, the vulnerability or encoding layer, and the final recovery step.

## Writeup Index

| Category | Challenge | Difficulty / Points | Core idea | Writeup |
| --- | --- | --- | --- | --- |
| Web | Developpeur mbw9: JWT Rookie | Rookie | JWT `alg=none` auth bypass and cookie discovery | [Read](web/jwt-rookie.md) |
| Web | JWT Confusion | Medium | JWT `alg=none` to admin, then AES-CBC decrypt | [Read](web/jwt-confusion.md) |
| Web | NodeJS Academy | Web | Prototype pollution to bypass admin checks | [Read](web/nodejs-academy-prototype-pollution.md) |
| Web | L'Enigme du Code Apogee | Medium / Hard | Apache 2.4.49 traversal + RCE chain | [Read](web/enigme-code-apogee.md) |
| Steganography | L3WILIL | 450 pts | LSB extraction from RGB bit plane + Base64 | [Read](steganography/l3wilil.md) |
| Misc | QuadraSignals | Hard | Binary signal reconstruction from JSON measurements | [Read](misc/quadrasignals.md) |
| OSINT | The Hidden Archive | 500 pts | Split flag recovery from archive output and tweets JSON | [Read](osint/the-hidden-archive.md) |

## Repository Structure

```text
.
|-- misc/
|   `-- quadrasignals.md
|-- osint/
|   `-- the-hidden-archive.md
|-- steganography/
|   `-- l3wilil.md
|-- web/
|   |-- enigme-code-apogee.md
|   |-- jwt-confusion.md
|   |-- jwt-rookie.md
|   `-- nodejs-academy-prototype-pollution.md
`-- README.md
```

## Methodology Style

Each writeup follows the same structure:

| Section | Purpose |
| --- | --- |
| Executive summary | What the challenge was about and what solved it |
| Attack path | High-level chain before the command details |
| Technical walkthrough | Reproducible commands, scripts and observations |
| Flag | Final recovered flag |
| Lessons learned | Practical security takeaways |

## Disclaimer

All content is from a CTF environment. Commands and techniques are documented for education, training and portfolio review only.
