# Cylab - 13

| Info         | Detalhe                          |
|--------------|-------------------------------------|
| Plataforma   | picoCTF                      |
| Categoria    | Cryptography                       |
| Dificuldade  | Easy                                |
| Status       | ✅ Resolvido                        |

## Sumário
- [Cylab - 13](#cylab---13)
  - [Sumário](#sumário)
  - [Desafio](#desafio)
  - [Análise](#análise)
  - [Resolução](#resolução)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Desafio

> Cryptography can be easy, do you know what ROT13 is?
>
> `cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}`

---

## Análise

A descrição do desafio já mencionava diretamente **ROT13**, uma cifra de substituição conhecida que desloca cada letra do alfabeto em 13 posições. Como o alfabeto tem 26 letras, aplicar ROT13 duas vezes retorna o texto original — o que também faz dela sua própria inversa, ou seja, codificar e decodificar usam exatamente a mesma operação.

Com essa dica, não foi necessária muita análise — bastava rodar a string cifrada em um decodificador ROT13.

---

## Resolução

Usei a ferramenta [ROT13 do dcode.fr](https://www.dcode.fr/rot-13-cipher) e colei o texto cifrado:

```
cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}
```

Aplicando o deslocamento ROT13 em cada letra, a flag em texto puro foi revelada.

---

## Flag

```
picoCTF{not_too_bad_of_a_problem}
```

---

## Lições aprendidas

- ROT13 é uma cifra simétrica (auto-inversa) — codificar e decodificar são exatamente a mesma operação, o que a torna trivial de quebrar uma vez identificada.
- Descrições de desafios em CTFs frequentemente já mencionam a técnica exata a ser usada — sempre vale ler com atenção antes de partir para análises mais complexas.
- Ferramentas online como o dCode são convenientes para resolver rapidamente cifras clássicas conhecidas, mas entender o mecanismo por trás (deslocar letras em uma quantidade fixa) é o que permite reconhecer cifras semelhantes (como ROT-N com outros valores) em desafios futuros.
