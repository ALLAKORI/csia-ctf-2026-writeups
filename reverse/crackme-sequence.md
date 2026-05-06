# Crackme Sequence

![Category](https://img.shields.io/badge/Category-Reverse%20Engineering-7c3aed?style=flat-square)
![Points](https://img.shields.io/badge/Points-500-f59e0b?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-Binary%20Pattern%20Analysis-2563eb?style=flat-square)
![Script](https://img.shields.io/badge/Script-Python-16a34a?style=flat-square)

> Objective: recover a hidden flag from a raw binary file by identifying a numeric sequence and extracting the encoded deltas.

## Executive Summary

The provided `crackme` file was not recognized as a normal executable or known file format. `file`, `exiftool`, and `strings` gave no useful information.

Hex inspection revealed a regular 16-bit little-endian sequence. The expected values increased by `+1`, but some values were shifted. The difference between each actual value and the expected arithmetic sequence encoded printable ASCII characters.

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Reverse Engineering |
| Challenge | crackme |
| Points | 500 |
| Main idea | Decode hidden data from a custom numeric sequence |

## Investigation Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Run `file` | Unknown raw data |
| 2 | Run `exiftool` | No known metadata |
| 3 | Run `strings` | No printable strings |
| 4 | Inspect with `xxd` | 16-bit little-endian pattern found |
| 5 | Model expected sequence | Arithmetic `+1` progression |
| 6 | Extract deltas | Printable characters appeared |
| 7 | Assemble output | Flag recovered |

## 1. Basic Triage

```bash
file crackme
```

Output:

```text
crackme: data
```

The file had no recognized signature.

```bash
exiftool crackme
```

No known format or useful metadata was identified.

```bash
strings crackme
```

No useful strings appeared.

This suggested one of the following:

| Possibility | Reason |
| --- | --- |
| Custom binary structure | No standard file header |
| Numeric encoding | No ASCII strings |
| Obfuscation | Hidden data may be derived, not stored directly |

## 2. Hex Inspection

```bash
xxd -g 1 -l 128 crackme
```

Output excerpt:

```text
00000000: 2c 03 2d 03 2e 03 2f 03 30 03 31 03 ...
```

Reading the bytes as little-endian 16-bit integers gives:

```text
0x032c
0x032d
0x032e
0x032f
...
```

The values increased by `+1`, which revealed the underlying sequence.

## 3. Hypothesis

The file looked like a mostly regular arithmetic sequence:

```text
expected[i] = base + i
```

The key idea was that some values were modified, and the modification distance encoded the hidden message.

For each 16-bit value:

```text
diff = actual_value - expected_value
```

Then:

```text
char = diff % 256
```

If `char` was printable ASCII, it was likely part of the flag.

## 4. Extraction Script

```python
import struct

data = open("crackme", "rb").read()
vals = list(struct.unpack("<" + "H" * (len(data) // 2), data))

base = vals[0]

for i, v in enumerate(vals):
    expected = (base + i) % 0x400
    diff = (v - expected) % 0x400
    c = diff % 256

    if 32 <= c <= 126:
        print(f"[+] offset={i * 2:#x} diff={diff:#04x} char={chr(c)}")
```

## 5. Output Reconstruction

The script revealed the characters one by one:

```text
C
S
I
A
{
h
1
d
d
3
n
_
1
n
_
7
h
3
_
s
3
q
u
3
n
c
3
}
```

Assembled:

```text
CSIA{h1dd3n_1n_7h3_s3qu3nc3}
```

## Flag

```text
CSIA{h1dd3n_1n_7h3_s3qu3nc3}
```

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| `file` returned only `data` | Unknown files need manual structure analysis |
| `strings` found nothing | Flags can be encoded numerically |
| Hex values formed a sequence | Pattern recognition is critical in reverse challenges |
| Deltas were printable ASCII | Compare actual data to expected models |

