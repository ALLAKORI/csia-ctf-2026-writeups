# The Hidden Archive

## Challenge Information

| Field | Value |
| --- | --- |
| Category | OSINT |
| Challenge | The Hidden Archive |
| Points | 500 |

Description:

```text
We intercepted the data of a suspect known as CipherRaven.
They hinted that they split their secret into two parts to avoid being discovered.
```

The challenge provided a ZIP archive:

```text
CipherRaven_Data.zip
```

The description already hinted that the flag was split into two pieces.

## Extracting the Archive

```bash
unzip CipherRaven_Data.zip
```

Output:

```text
Archive:  CipherRaven_Data.zip
Here is the rest of what you are looking for: d4t4_l34ks_99}
inflating: tweets.json
```

This immediately revealed the second half of the flag:

```text
d4t4_l34ks_99}
```

The archive also extracted:

```text
tweets.json
```

## Investigating tweets.json

I searched for suspicious keywords:

```bash
grep -iE "flag|CSIA|CipherRaven|secret|part|archive|leak|data|d4t4" tweets.json
```

Interesting result:

```json
"text": "They are onto me. I split the key just in case. Part 1 is CSIA{Tr4ck1ng_ ... The second part is attached to the container itself. Good luck finding it."
```

This revealed the first part:

```text
CSIA{Tr4ck1ng_
```

## Reconstructing the Flag

Part 1:

```text
CSIA{Tr4ck1ng_
```

Part 2:

```text
d4t4_l34ks_99}
```

Combined:

```text
CSIA{Tr4ck1ng_d4t4_l34ks_99}
```

## Flag

```text
CSIA{Tr4ck1ng_d4t4_l34ks_99}
```

## Key Takeaways

- Archive output can contain useful hidden data.
- OSINT-style challenges often require correlating multiple small clues.
- Searching structured files for contextual keywords is a fast first step.

