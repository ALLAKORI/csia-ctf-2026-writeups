# The Hidden Archive

![Category](https://img.shields.io/badge/Category-OSINT-0891b2?style=flat-square)
![Points](https://img.shields.io/badge/Points-500-f59e0b?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-Archive%20Forensics-7c3aed?style=flat-square)
![Data](https://img.shields.io/badge/Data-JSON%20Search-16a34a?style=flat-square)

> Objective: reconstruct a split flag from archive output and social-data style JSON.

## Executive Summary

The challenge description said that `CipherRaven` split the secret into two parts. The ZIP extraction output immediately leaked the second half of the flag, while `tweets.json` contained a message revealing the first half.

The solve was based on correlating both pieces rather than exploiting a complex vulnerability.

## Challenge Brief

```text
We intercepted the data of a suspect known as CipherRaven.
They hinted that they split their secret into two parts to avoid being discovered.
```

Provided file:

```text
CipherRaven_Data.zip
```

## Investigation Path

| Step | Action | Evidence |
| --- | --- | --- |
| 1 | Extract ZIP | Archive output leaked part 2 |
| 2 | Inspect extracted file | `tweets.json` recovered |
| 3 | Search suspicious keywords | Tweet revealed part 1 |
| 4 | Combine both fragments | Full CSIA flag reconstructed |

## 1. Archive Extraction

```bash
unzip CipherRaven_Data.zip
```

Output:

```text
Archive:  CipherRaven_Data.zip
Here is the rest of what you are looking for: d4t4_l34ks_99}
inflating: tweets.json
```

Recovered second part:

```text
d4t4_l34ks_99}
```

## 2. JSON Investigation

I searched for terms related to the challenge description:

```bash
grep -iE "flag|CSIA|CipherRaven|secret|part|archive|leak|data|d4t4" tweets.json
```

Interesting hit:

```json
"text": "They are onto me. I split the key just in case. Part 1 is CSIA{Tr4ck1ng_ ... The second part is attached to the container itself. Good luck finding it."
```

Recovered first part:

```text
CSIA{Tr4ck1ng_
```

## 3. Flag Reconstruction

| Fragment | Value |
| --- | --- |
| Part 1 | `CSIA{Tr4ck1ng_` |
| Part 2 | `d4t4_l34ks_99}` |

Combined:

```text
CSIA{Tr4ck1ng_d4t4_l34ks_99}
```

## Flag

```text
CSIA{Tr4ck1ng_d4t4_l34ks_99}
```

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| Secret split into two pieces | Trust the challenge wording |
| Archive extraction printed text | Terminal output is evidence |
| JSON contained social messages | Search context words, not only `flag` |
| Fragments were plain text | OSINT challenges reward correlation |

