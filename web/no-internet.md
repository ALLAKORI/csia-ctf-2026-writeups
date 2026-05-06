# no_internet

![Category](https://img.shields.io/badge/Category-Web-2563eb?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-16a34a?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-Source%20Code%20Inspection-f59e0b?style=flat-square)
![Signal](https://img.shields.io/badge/Signal-HTML%20Comment%20%2F%20Hidden%20Text-7c3aed?style=flat-square)

> Objective: inspect the web page source and locate the flag directly inside the client-side content.

## Executive Summary

`no_internet` was a beginner-friendly web challenge based on a simple but important habit: always inspect the client-side source code.

No exploitation, fuzzing or authentication bypass was needed. Opening the page source and searching for the flag format `CSIA` revealed the flag directly.

## Challenge Information

| Field | Value |
| --- | --- |
| Category | Web |
| Challenge | no_internet |
| Difficulty | Easy |
| Main idea | Source-code inspection |

## Attack Path

| Step | Action | Result |
| --- | --- | --- |
| 1 | Open the challenge page | Static web page loaded |
| 2 | View page source | HTML source became visible |
| 3 | Search for `CSIA` | Flag location found |
| 4 | Copy the flag | Challenge solved |

## Walkthrough

The first step was simply to open the challenge page in the browser.

Since the challenge was a web task and there was no obvious interaction needed, I inspected the page source:

```text
Right click -> View Page Source
```

or with the keyboard shortcut:

```text
Ctrl + U
```

Then I searched inside the source code:

```text
Ctrl + F -> CSIA
```

The flag appeared directly in the page source.

## Alternative CLI Check

The same idea can be reproduced with `curl`:

```bash
curl -s "https://challenge-url-here/" | grep -oE 'CSIA\{[^}]+\}'
```

If the flag is stored in the HTML, this extracts it directly.

## Flag

```text
CSIA{flag_was_found_in_page_source}
```

Note: the exact flag value should be replaced if recovered from the original challenge page. The solving method was direct source inspection with `Ctrl + F` searching for `CSIA`.

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| Flag was client-side | Always inspect HTML source first |
| No exploit was required | Easy web challenges often test basics |
| `Ctrl + F` found `CSIA` | Search by flag format is fast and effective |
| Client-side secrets are exposed | Sensitive data should never be embedded in HTML |

