# World of Sports

![Category](https://img.shields.io/badge/Category-OSINT-0891b2?style=flat-square)
![Points](https://img.shields.io/badge/Points-450-f59e0b?style=flat-square)
![Theme](https://img.shields.io/badge/Theme-The%20Expanse-7c3aed?style=flat-square)
![Technique](https://img.shields.io/badge/Technique-Lore%20Correlation-16a34a?style=flat-square)

> Objective: interpret the challenge phrases, identify the fictional universe, and map the clue to a real Martian location.

## Executive Summary

The challenge gave short phrases that looked strange at first:

```text
Beratna
Good place for Dusters
mi pensa
```

The words `Beratna` and `mi pensa` point to **Belter Creole**, the constructed language used in **The Expanse**. In that universe, people from Mars are called **Dusters**.

Once the challenge was connected to Mars, the required place was a famous Martian location: **Valles Marineris**.

## Challenge Information

| Field | Value |
| --- | --- |
| Category | OSINT |
| Challenge | World of Sports |
| Points | 450 |
| Flag format | `CSIA{Name_of_Place}` |

Challenge text:

```text
Where in the wide, wide world of sports is this, beratna?
Good place for Dusters, mi pensa.

FLAG FORMAT: CSIA{Name_of_Place}
Example: CSIA{Grand_Canyon}
```

## Investigation Path

| Step | Clue | Interpretation | Result |
| --- | --- | --- | --- |
| 1 | `Beratna` | Belter Creole word for brother | The Expanse reference |
| 2 | `mi pensa` | Belter Creole phrase meaning I think | Confirms The Expanse |
| 3 | `Dusters` | Nickname for Martians in The Expanse | Focus on Mars |
| 4 | Good place for Dusters | Famous place on Mars | Valles Marineris |

## 1. Identifying the Language

The word:

```text
Beratna
```

comes from Belter Creole in **The Expanse**.

Meaning:

```text
Brother
```

This was the first strong signal that the challenge was not about a real sports location, but about a fictional universe.

## 2. Confirming the Universe

The phrase:

```text
mi pensa
```

is also Belter Creole.

Meaning:

```text
I think
```

This confirmed the reference to **The Expanse**.

## 3. Understanding “Dusters”

In **The Expanse**, people from Mars are commonly called:

```text
Dusters
```

So the clue:

```text
Good place for Dusters
```

points directly toward **Mars**.

## 4. Finding the Place

The flag format required a place name:

```text
CSIA{Name_of_Place}
```

With Mars identified, the most iconic canyon-like Martian place matching the example style was:

```text
Valles Marineris
```

Formatted for the flag:

```text
Valles_Marineris
```

## Flag

```text
CSIA{Valles_Marineris}
```

## Lessons Learned

| Observation | Lesson |
| --- | --- |
| Strange words were not random | Identify language or fictional-lore references |
| Multiple clues pointed to the same universe | Correlate hints instead of solving them separately |
| `Dusters` narrowed the target to Mars | Nicknames can reveal location context |
| Flag format required a place | Convert the final answer into the expected format |

