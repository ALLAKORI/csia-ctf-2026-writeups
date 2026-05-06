# QuadraSignals

![Category](https://img.shields.io/badge/Category-Misc-0f766e?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Hard-dc2626?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-Binary%20Signal%20Decoding-2563eb?style=flat-square)
![Language](https://img.shields.io/badge/Script-Python-f59e0b?style=flat-square)

> Objective: interpret logic-level signal measurements and reconstruct the hidden message as binary data.

## Executive Summary

The archive contained a JSON file with repeated signal measurements and a PDF schematic. The values were only `0` and `5`, strongly suggesting digital logic levels.

After reviewing the signal behavior, `Signal_A` was selected as the useful stream. Mapping `5 -> 1` and `0 -> 0`, then grouping bits into bytes, produced the flag text.

## Provided Files

| File | Role |
| --- | --- |
| `complex_secret_message.json` | Signal measurements |
| `Design.pdf` | Schematic / interpretation hint |

## Signal Analysis

Example measurement:

```json
{
  "Signal_A": 0,
  "Signal_B": 5,
  "Signal_C": 5,
  "Signal_D": 0
}
```

Digital interpretation:

| Value | Bit |
| --- | --- |
| `0` | `0` |
| `5` | `1` |

## Attack Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Extract archive | JSON + PDF recovered |
| 2 | Inspect file types | Confirmed JSON data and PDF schematic |
| 3 | Read PDF text | Signals, LEDs and logic gates referenced |
| 4 | Inspect signal patterns | `Signal_A` selected |
| 5 | Convert values to bits | Binary stream built |
| 6 | Group into bytes | Message decoded |
| 7 | Normalize flag format | CSIA flag accepted |

## 1. Archive Extraction

```bash
7z x QuadraSignals.7z
```

Extracted files:

```text
complex_secret_message.json
Design.pdf
```

## 2. Initial Inspection

```bash
file complex_secret_message.json Design.pdf
```

Output:

```text
complex_secret_message.json: JSON text data
Design.pdf: PDF document, version 1.3
```

## 3. PDF Clue

```bash
pdftotext Design.pdf - | cat
```

The schematic referenced:

- `Signal_A`
- LEDs
- logic gates
- Arduino pins

This confirmed that the values should be treated as digital states.

## 4. Finding the Payload Stream

```python
import json

data = json.load(open("complex_secret_message.json"))

for i, row in enumerate(data[:10]):
    print(i, row)
```

Observed behavior:

| Signal | Behavior |
| --- | --- |
| `Signal_A` | Clean changing bitstream |
| `Signal_B` | Complementary to `Signal_A` |
| `Signal_C` | Constant `5` |
| `Signal_D` | Random-looking noise |

`Signal_A` was the best candidate for the payload.

## 5. Solver

```python
import json

data = json.load(open("complex_secret_message.json"))

bits = ""

for row in data:
    bits += "1" if row["Signal_A"] == 5 else "0"

flag = ""

for i in range(0, len(bits), 8):
    byte = bits[i:i + 8]
    if len(byte) == 8:
        flag += chr(int(byte, 2))

print(flag)
```

Output:

```text
HACKFURY{vINzWPvSNqSDHQdJBx7c3lPZvoAH7JofJJTM}
```

The platform expected CSIA format:

```text
CSIA{vINzWPvSNqSDHQdJBx7c3lPZvoAH7JofJJTM}
```

## Flag

```text
CSIA{vINzWPvSNqSDHQdJBx7c3lPZvoAH7JofJJTM}
```

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| Values were `0` and `5` only | Think digital logic |
| PDF referenced signals and gates | Documentation files can define encoding rules |
| One signal was constant | Not every provided stream is useful |
| Complementary streams existed | Choose the cleanest stream and test byte grouping |

