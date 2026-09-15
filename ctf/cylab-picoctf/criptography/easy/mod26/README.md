# Cylab - Mod 26

🇧🇷 [Versão em Português](./README.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | picoCTF       |
| Category     | Cryptography  |
| Difficulty   | Easy          |
| Status       | ✅ Completed  |

## Table of Contents
- [Cylab - Mod 26](#cylab---mod-26)
  - [Table of Contents](#table-of-contents)
  - [Challenge](#challenge)
  - [Analysis](#analysis)
  - [Solving](#solving)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Challenge

> Cryptography can be easy, do you know what ROT13 is?

The challenge provided a downloadable file, `values.txt`, containing the following ciphertext:

```
cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}
```

---

## Analysis

Just like in the previous challenge, the description directly pointed to **ROT13** as the cipher in use. Since the ciphertext followed the same visual pattern (letters shifted, punctuation and structure preserved), no further analysis was needed beyond confirming it was ROT13 and decoding it.

---

## Solving

I opened the downloaded `values.txt` file and copied its content into [dcode.fr's ROT13 tool](https://www.dcode.fr/rot-13-cipher):

```
cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}
```

The tool returned the flag directly in plaintext.

---

## Flag

```
picoCTF{next_time_I'll_try_2_rounds_of_rot13_45559abd}
```

---

## Lessons Learned

- The flag content itself is a bit of a joke pointing to the cipher's weakness: applying ROT13 only once is trivial to reverse — the challenge hints that even applying it twice (ROT26) would just return the original plaintext, since 26 letters mod 26 is a full cycle.
- When a description names the cipher outright, the fastest path is to confirm the pattern and go straight to a decoder rather than manually analyzing the ciphertext.
- Reinforces that ROT-N ciphers offer no real security — they're an obfuscation technique at best, useful mainly as a teaching tool for cipher basics.
