# Cylab - interencdec

| Info         | Detalhe        |
|--------------|-----------------|
| Plataforma   | picoCTF        |
| Categoria    | Cryptography   |
| Dificuldade  | Easy           |
| Status       | ✅ Resolvido   |

## Sumário
- [Desafio](#desafio)
- [Análise](#análise)
- [Resolução](#resolução)
- [Flag](#flag)
- [Lições aprendidas](#lições-aprendidas)

---

## Desafio

> Can you get the real meaning from this file.

O desafio disponibilizou um arquivo chamado `enc_flag`.

---

## Análise

Ao abrir o `enc_flag` como um arquivo de texto puro, obtive o seguinte conteúdo:

```
YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclgya3lNRFJvYTJvMmZRPT0nCg==
```

O `==` no final é um forte indicativo de codificação em **Base64**, então decidi começar por aí.

---

## Resolução

### Passo 1 — Decodificando Base64 (primeira camada)

Usei a ferramenta [Base64 do dcode.fr](https://www.dcode.fr/base-64-encoding) para decodificar a string, que retornou:

```
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrX2kyMDRoa2o2fQ=='
```

Essa saída tem o formato de um literal de bytes do Python (`b'...'`), o que é um resquício comum quando um script Python codifica uma string e o resultado de `repr()`/`str()` acaba sendo salvo assim mesmo no arquivo.

### Passo 2 — Limpando e decodificando Base64 (segunda camada)

Removi o `b` inicial e as aspas simples ao redor, restando:

```
d3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrX2kyMDRoa2o2fQ==
```

Rodando isso novamente no decodificador Base64, obtive:

```
wpjvJAM{jhlzhy_k3jy9wa3k_i204hkj6}
```

### Passo 3 — Decodificando a cifra de César

O resultado ainda seguia a estrutura padrão de flag (`algo{...}`), mas não era legível — o que sugeria uma cifra de substituição por cima da codificação dupla em Base64. Logo, tentei em seguida uma **cifra de César**.

Usei a ferramenta [Cifra de César do dcode.fr](https://www.dcode.fr/caesar-cipher) com detecção automática por força bruta, que revelou a flag final.

---

## Flag

```
picoCTF{caesar_d3cr9pt3d_b204adc6}
```

---

## Lições aprendidas

- Cadeias de codificação (Base64 → Base64 → César, nesse caso) são uma técnica comum em CTFs para fazer uma flag simples parecer mais ofuscada do que realmente é — o segredo é remover uma camada por vez e reavaliar o resultado após cada etapa.
- Reconhecer o padrão de saída do `repr()` do Python (como `b'...'`) ajuda a identificar resquícios do script original que precisam ser removidos antes de continuar a decodificação.
- Ferramentas automatizadas de identificação de cifras (como o solucionador por força bruta de César do dCode) são muito eficazes assim que as camadas de codificação são removidas e resta apenas uma cifra de substituição clássica.
