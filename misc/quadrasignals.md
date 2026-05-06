# QuadraSignals

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Misc |
| Difficulty | Hard |

The challenge provided a compressed archive:

```bash
QuadraSignals.7z
```

After extraction:

```bash
7z x QuadraSignals.7z
```

The archive contained:

```text
complex_secret_message.json
Design.pdf
```

## Initial Analysis

I inspected both files:

```bash
file complex_secret_message.json Design.pdf
```

Output:

```text
complex_secret_message.json: JSON text data
Design.pdf: PDF document, version 1.3
```

The JSON file contained signal measurements like:

```json
{
  "Signal_A": 0,
  "Signal_B": 5,
  "Signal_C": 5,
  "Signal_D": 0
}
```

The values were only `0` and `5`, which suggested a binary encoding:

- `0` means `0`
- `5` means `1`

## PDF Analysis

I extracted text from the PDF:

```bash
pdftotext Design.pdf - | cat
```

The schematic referenced:

- `Signal_A`
- LEDs
- logic gates
- Arduino pins

This confirmed that the signal values represented digital logic states.

## Finding the Useful Signal

I inspected the first measurements:

```python
import json

data = json.load(open("complex_secret_message.json"))

for i, row in enumerate(data[:10]):
    print(i, row)
```

Observations:

- `Signal_C` was always `5`.
- `Signal_A` and `Signal_B` were complementary.
- `Signal_D` looked random.

This suggested that the payload was likely stored in one clean signal stream. I focused on `Signal_A`.

## Extracting Bits

I converted `Signal_A` values to binary:

- `5` to `1`
- `0` to `0`

Then I grouped the resulting bitstream into bytes.

## Solve Script

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

## Output

```text
HACKFURY{vINzWPvSNqSDHQdJBx7c3lPZvoAH7JofJJTM}
```

The platform expected the flag in CSIA format, so the final flag was:

```text
CSIA{vINzWPvSNqSDHQdJBx7c3lPZvoAH7JofJJTM}
```

## Flag

```text
CSIA{vINzWPvSNqSDHQdJBx7c3lPZvoAH7JofJJTM}
```

## Key Takeaways

- Logic-level values can be treated as binary.
- Visual design files may reveal how data should be interpreted.
- Checking signal behavior quickly helps identify the real payload stream.

