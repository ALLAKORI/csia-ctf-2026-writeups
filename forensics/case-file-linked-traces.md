# Case File: Linked Traces

![Category](https://img.shields.io/badge/Category-Forensics-7c3aed?style=flat-square)
![Type](https://img.shields.io/badge/Type-OSINT%20Forensics-0891b2?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-Wayback%20Pivot-f59e0b?style=flat-square)
![Signal](https://img.shields.io/badge/Signal-Social%20Graph%20Tracing-16a34a?style=flat-square)

> Objective: investigate an old case file, recover archived social traces, pivot across accounts, and extract the flag from a high-resolution avatar.

## Executive Summary

The challenge image looked like a cold case investigation folder. It gave the target handle `justinccase2511` and hinted that the subject had changed all social media handles about three years earlier.

The key was to investigate old cached data instead of current profiles. Since X was still Twitter around that period, the Wayback Machine was queried with the old Twitter URL. The archived profile leaked a stable internal identifier, which allowed a pivot to the current X account. From there, a screenshot revealed a Tumblr presence. The Tumblr avatar, when fetched in high resolution through Tumblr's avatar endpoint, contained the flag hidden in very small text at the bottom.

## Challenge Evidence

| Clue | Interpretation |
| --- | --- |
| `TARGET: justinccase2511` | Original social handle |
| `Changed all social media handles approx 3 years ago` | Search archived pages, not current profiles |
| `Find the OLD cached data` | Use Wayback Machine |
| `Current profiles are clean decoys` | Pivot through stable identifiers |
| Hidden text in final avatar | Recover high-resolution image |

## Investigation Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Read the case file image | Target handle found |
| 2 | Query Wayback with old Twitter domain | Archived profile found |
| 3 | Inspect archived source | Stable internal ID extracted |
| 4 | Pivot from ID to current X account | New handle identified |
| 5 | Inspect timeline media | Tumblr clue found |
| 6 | Query Tumblr avatar endpoint | High-resolution image recovered |
| 7 | Zoom bottom of image | Flag found |

## 1. Case File Triage

The image provided the initial target:

```text
justinccase2511
```

It also included two important notes:

```text
Changed all social media handles approx 3 years ago
Find the OLD cached data. Current profiles are clean decoys.
```

This indicated that current accounts were intentionally misleading and that archived social data should be investigated.

## 2. Wayback Machine Pivot

Because the target changed handles about three years earlier, the platform name mattered. X was still commonly referenced as Twitter at that time.

The archived URL to test was:

```text
https://www.twitter.com/justinccase2511
```

The Wayback Machine archive of the old Twitter profile did not expose the flag directly, but it contained a more important pivot point: the stable internal user identifier.

## 3. Internal Identifier Extraction

When a user changes their visible handle, the platform's internal numeric identifier can remain stable.

Inspecting the archived page source revealed:

```json
"identifier": "1579193231652405248"
```

This ID became the bridge between the old account identity and the current account identity.

## 4. Pivot to the Current X Profile

Using the stable identifier, the account could be mapped to its current handle:

```text
@t0mmyx1a0mi
```

The current profile itself was mostly a decoy, but its timeline contained useful media.

## 5. Pivot from X to Tumblr

In the timeline of:

```text
@t0mmyx1a0mi
```

one tweet contained a screenshot. Careful inspection of the browser tabs and icons in the screenshot revealed a Tumblr clue.

The likely Tumblr blog was:

```text
https://www.tumblr.com/t0mmyx1a0mi
```

## 6. High-Resolution Avatar Extraction

The Tumblr profile image contained unreadable text when viewed normally because the image was compressed or displayed too small.

Tumblr exposes avatar images through a predictable endpoint. Requesting the 512x512 version revealed the text clearly:

```text
https://api.tumblr.com/v2/blog/t0mmyx1a0mi/avatar/512
```

The flag was hidden in very small text at the bottom of the image.

## Flag

Original flag format:

```text
MetaCTF{3v3ryth1ng_has_c0nnecti0n}
```

Submitted CSIA format:

```text
CSIA{3v3ryth1ng_has_c0nnecti0n}
```

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| Current profiles were decoys | Historical data can be more valuable than live profiles |
| Twitter became X | Use old domain names when searching archives |
| Handles can change | Stable internal IDs enable account correlation |
| Social screenshots leak context | Browser tabs, icons and UI details can reveal pivots |
| Avatar was compressed | Request higher-resolution assets directly |

