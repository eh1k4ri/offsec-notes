# Cylab - Mod 26

| Info         | Detalhe        |
|--------------|-----------------|
| Plataforma   | picoCTF        |
| Categoria    | Cryptography   |
| Dificuldade  | Easy           |
| Status       | ✅ Resolvido   |

## Sumário
- [Cylab - Mod 26](#cylab---mod-26)
  - [Sumário](#sumário)
  - [Desafio](#desafio)
  - [Análise](#análise)
  - [Resolução](#resolução)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Desafio

> Cryptography can be easy, do you know what ROT13 is?

O desafio disponibilizou um arquivo para download, `values.txt`, contendo o seguinte texto cifrado:

```
cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}
```

---

## Análise

Assim como no desafio anterior, a descrição apontava diretamente para **ROT13** como a cifra utilizada. Como o texto cifrado seguia o mesmo padrão visual (letras deslocadas, com pontuação e estrutura preservadas), não foi necessária muita análise além de confirmar que era ROT13 e decodificar.

---

## Resolução

Abri o arquivo `values.txt` baixado e colei seu conteúdo na ferramenta [ROT13 do dcode.fr](https://www.dcode.fr/rot-13-cipher):

```
cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}
```

A ferramenta retornou a flag diretamente em texto puro.

---

## Flag

```
picoCTF{next_time_I'll_try_2_rounds_of_rot13_45559abd}
```

---

## Lições aprendidas

- O próprio conteúdo da flag é uma piada apontando para a fraqueza da cifra: aplicar ROT13 apenas uma vez é trivial de reverter — o desafio sugere que aplicar duas vezes (ROT26) simplesmente retornaria o texto original, já que 26 letras mod 26 é um ciclo completo.
- Quando a descrição já nomeia a cifra diretamente, o caminho mais rápido é confirmar o padrão e ir direto para um decodificador, em vez de analisar manualmente o texto cifrado.
- Reforça que cifras ROT-N não oferecem segurança real — são, no máximo, uma técnica de ofuscação, útil principalmente como ferramenta de ensino sobre fundamentos de cifras.
