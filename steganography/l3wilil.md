# L3WILIL

![Category](https://img.shields.io/badge/Category-Steganography-7c3aed?style=flat-square)
![Points](https://img.shields.io/badge/Points-450-f59e0b?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-LSB%20RGB-2563eb?style=flat-square)
![Tool](https://img.shields.io/badge/Tool-zsteg-16a34a?style=flat-square)

> Objective: extract a hidden message from a PNG image using the hint to select the correct LSB bit plane.

## Executive Summary

The challenge provided a PNG image and a hint containing a suspicious `1`. Standard metadata and string checks did not reveal anything useful, so the next hypothesis was LSB steganography.

Using `zsteg`, the payload appeared in `b1,rgb,lsb,xy`. The extracted data contained a short marker followed by Base64, which decoded directly to the flag.

## Challenge Data

| Field | Value |
| --- | --- |
| File | `challenge.png` |
| Dimensions | `868 x 1156` |
| Color mode | RGB, 8-bit/color |
| Hint | `Tahya 1vice president mn 3nd lpresident` |

## Extraction Flow

| Step | Tool | Result |
| --- | --- | --- |
| File identification | `file` | Valid PNG image |
| Metadata review | `exiftool` | Nothing suspicious |
| String search | `strings` | No direct flag |
| LSB scan | `zsteg -a` | Hidden Base64 found |
| Payload extraction | `zsteg -E` | Encoded message recovered |
| Decode | `base64 -d` | Flag recovered |

## 1. File Identification

```bash
file challenge.png
```

Output:

```text
challenge.png: PNG image data, 868 x 1156, 8-bit/color RGB, non-interlaced
```

## 2. Metadata and Strings

```bash
exiftool challenge.png
```

No suspicious metadata or embedded comments were present.

```bash
strings challenge.png | grep -i csia
```

No direct flag appeared.

## 3. Hint Interpretation

The hint:

```text
Tahya 1vice president mn 3nd lpresident
```

The `1` looked intentional and suggested **bit 1**, which mapped naturally to a `zsteg` extraction mode such as `b1`.

## 4. LSB Discovery

```bash
zsteg -a challenge.png
```

Important output:

```text
b1,rgb,lsb,xy .. text: "44:Q1NJQXtsQjRCNF9DaDRGZXFfSzR5U2xlbV80bGlLMG19"
```

This gave the exact extraction path:

| Parameter | Meaning |
| --- | --- |
| `b1` | bit plane 1 |
| `rgb` | RGB channels |
| `lsb` | least significant bit order |
| `xy` | XY pixel traversal |

## 5. Payload Extraction

```bash
zsteg challenge.png -E b1,rgb,lsb,xy
```

Output:

```text
44:Q1NJQXtsQjRCNF9DaDRGZXFfSzR5U2xlbV80bGlLMG19
```

The `44:` prefix looked like a marker. The remaining string was Base64.

## 6. Decoding

```bash
echo 'Q1NJQXtsQjRCNF9DaDRGZXFfSzR5U2xlbV80bGlLMG19' | base64 -d
```

Output:

```text
CSIA{lB4B4_Ch4Feq_K4ySlem_4liK0m}
```

## Flag

```text
CSIA{lB4B4_Ch4Feq_K4ySlem_4liK0m}
```

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| Clean metadata | Do not stop at `exiftool` |
| Suspicious number in hint | Hints may point directly to bit planes |
| `zsteg -a` found structured data | Full scans are useful before manual extraction |
| Base64 after LSB | Stego challenges often stack encoding layers |

