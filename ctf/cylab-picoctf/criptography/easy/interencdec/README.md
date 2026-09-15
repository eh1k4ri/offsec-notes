# Cylab - interencdec

| Info         | Detail        |
|--------------|---------------|
| Platform     | picoCTF       |
| Category     | Cryptography  |
| Difficulty   | Easy          |
| Status       | ✅ Completed  |

## Table of Contents
- [Challenge](#challenge)
- [Analysis](#analysis)
- [Solving](#solving)
- [Flag](#flag)
- [Lessons Learned](#lessons-learned)

---

## Challenge

> Can you get the real meaning from this file.

The challenge provided a file called `enc_flag`.

---

## Analysis

Opening `enc_flag` as a plain text file revealed the following content:

```
YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclgya3lNRFJvYTJvMmZRPT0nCg==
```

The trailing `==` padding is a strong indicator of **Base64** encoding, so I decided to start there.

---

## Solving

### Step 1 — Decoding Base64 (first layer)

I used [dcode.fr's Base64 tool](https://www.dcode.fr/base-64-encoding) to decode the string, which returned:

```
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrX2kyMDRoa2o2fQ=='
```

This output is formatted like a Python bytes literal (`b'...'`), which is a common artifact when a Python script encodes a string and its `repr()`/`str()` output gets saved to the file as-is.

### Step 2 — Cleaning up and decoding Base64 (second layer)

I removed the leading `b` and the surrounding single quotes, leaving:

```
d3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrX2kyMDRoa2o2fQ==
```

Running this through the Base64 decoder again produced:

```
wpjvJAM{jhlzhy_k3jy9wa3k_i204hkj6}
```

### Step 3 — Decoding the Caesar cipher

The result still followed the standard flag structure (`something{...}`), but wasn't readable — this suggested a substitution cipher on top of the double Base64 encoding. Therefore, I tried a **Caesar cipher** next.

I used [dcode.fr's Caesar Cipher tool](https://www.dcode.fr/caesar-cipher) with automatic brute-force detection, which revealed the final flag.

---

## Flag

```
picoCTF{caesar_d3cr9pt3d_b204adc6}
```

---

## Lessons Learned

- Encoding chains (Base64 → Base64 → Caesar, in this case) are a common technique in CTFs to make a simple flag look more obfuscated than it actually is — the key is to peel back one layer at a time and re-evaluate the output after each step.
- Recognizing the pattern of a Python `repr()` output (like `b'...'`) helps identify leftover artifacts from the original script that need to be stripped before further decoding.
- Automated cipher-identification tools (like dCode's brute-force Caesar solver) are very effective once the encoding layers have been removed and only a classic substitution cipher remains.
