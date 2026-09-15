# Cylab - The Numbers

| Info         | Detail                |
|--------------|------------------------|
| Platform     | picoCTF           |
| Category     | Cryptography           |
| Difficulty   | Easy                   |
| Status       | ✅ Completed            |

## Table of Contents
- [Cylab - The Numbers](#cylab---the-numbers)
  - [Table of Contents](#table-of-contents)
  - [Challenge](#challenge)
  - [Analysis](#analysis)
  - [Solving](#solving)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Challenge

> The numbers... what do they mean?

The challenge provided the following sequence of numbers:

![Challenge](./assets/01-challenge.png)

```
16 9 3 15 3 20 6 { 20 8 5 14 21 13 2 5 18 19 13 1 19 15 14 }
```

---

## Analysis

The hint *"the numbers... what do they mean?"* combined with a sequence of numbers, curly braces, and the general format of `something{...}` (the standard picoCTF flag format) strongly suggested a simple **letter-to-number substitution cipher**, where each number represents the position of a letter in the alphabet:

```
a=1, b=2, c=3, d=4, e=5, f=6, g=7, h=8, i=9, j=10, k=11, l=12, m=13,
n=14, o=15, p=16, q=17, r=18, s=19, t=20, u=21, v=22, w=23, x=24, y=25, z=26
```

---

## Solving

Mapping each number back to its corresponding letter:

| Numbers | Letters |
|---------|---------|
| 16 9 3 15 3 20 6 | p i c o c t f |
| 20 8 5 14 21 13 2 5 18 19 13 1 19 15 14 | t h e n u m b e r s m a s o n |

Putting it all together (keeping the `{` and `}` as-is, since they're not letters):

```
picoctf{thenumbersmason}
```

---

## Flag

```
picoctf{thenumbersmason}
```

---

## Lessons Learned

- Whenever a challenge presents a sequence of small numbers (roughly in the 1–26 range) alongside a cryptography tag, a simple alphabet-position substitution is one of the first things worth trying.
- Recognizing the flag format (`something{...}`) can help confirm which parts of the sequence are literal characters (like `{` and `}`) versus which need decoding.
- Classic/basic ciphers like this are common as an easy entry point in CTFs to introduce pattern recognition before moving to more complex cryptographic schemes.
