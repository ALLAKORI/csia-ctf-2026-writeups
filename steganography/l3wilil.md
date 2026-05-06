# L3WILIL

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Steganography |
| Challenge | L3WILIL |
| Points | 450 |

Hint:

```text
Tahya 1vice président mn 3nd lprésident
```

The challenge provided a PNG image:

```text
challenge.png
```

## Initial Enumeration

I started by identifying the file type:

```bash
file challenge.png
```

Output:

```text
challenge.png: PNG image data, 868 x 1156, 8-bit/color RGB, non-interlaced
```

## Metadata and Strings

I checked the metadata:

```bash
exiftool challenge.png
```

There were no suspicious comments or embedded metadata.

I also searched for obvious flag strings:

```bash
strings challenge.png | grep -i csia
```

No direct result appeared.

At this point, the image format and the challenge hint suggested LSB steganography.

## Understanding the Hint

The hint contained:

```text
1vice président
```

The `1` looked intentional. I interpreted it as a reference to bit 1, which pointed toward LSB extraction with a mode such as `b1`.

## Steganography Analysis

I ran a full `zsteg` scan:

```bash
zsteg -a challenge.png
```

The important result was:

```text
b1,rgb,lsb,xy .. text: "44:Q1NJQXtsQjRCNF9DaDRGZXFfSzR5U2xlbV80bGlLMG19"
```

This showed hidden data in:

- bit 1
- RGB channels
- LSB mode
- XY pixel order

The extracted content looked encoded.

## Extracting the Payload

I extracted the payload directly:

```bash
zsteg challenge.png -E b1,rgb,lsb,xy
```

Output:

```text
44:Q1NJQXtsQjRCNF9DaDRGZXFfSzR5U2xlbV80bGlLMG19
```

The `44:` prefix appeared to be a marker or noise. The remaining part looked like Base64.

## Base64 Decoding

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

## Key Takeaways

- The challenge used LSB steganography.
- The payload was hidden in RGB channels.
- The hint pointed to the correct bit plane.
- The extracted data required a final Base64 decoding step.

