# Cylab - 13

| Info         | Detail                          |
|--------------|-----------------------------------|
| Platform     | picoCTF                     |
| Category     | Cryptography                     |
| Difficulty   | Easy                              |
| Status       | ✅ Completed                      |

## Table of Contents
- [Cylab - 13](#cylab---13)
  - [Table of Contents](#table-of-contents)
  - [Challenge](#challenge)
  - [Analysis](#analysis)
  - [Solving](#solving)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Challenge

> Cryptography can be easy, do you know what ROT13 is?
>
> `cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}`

---

## Analysis

The challenge description directly mentioned **ROT13**, a well-known substitution cipher that shifts each letter of the alphabet by 13 positions. Since the alphabet has 26 letters, applying ROT13 twice returns the original text — which also makes it its own inverse, so encoding and decoding use the exact same operation.

Given that hint, no extensive analysis was needed — the encrypted string just had to be run through a ROT13 decoder.

---

## Solving

I used [dcode.fr's ROT13 tool](https://www.dcode.fr/rot-13-cipher) and pasted the ciphertext:

```
cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}
```

Applying the ROT13 shift to every letter revealed the flag in plaintext.

---

## Flag

```
picoCTF{not_too_bad_of_a_problem}
```

---

## Lessons Learned

- ROT13 is a symmetric cipher (self-inverse) — encoding and decoding are the exact same operation, which makes it trivial to break once identified.
- Challenge descriptions in CTFs often name the exact technique to use — always read them carefully before jumping into more complex analysis.
- Online tools like dCode are convenient for quickly solving well-known classic ciphers, but understanding the underlying mechanism (shifting letters by a fixed amount) is what allows recognizing similar ciphers (like ROT-N with other values) in future challenges.
