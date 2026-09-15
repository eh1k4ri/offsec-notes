# Cylab - Mini RSA

| Info         | Detalhe        |
|--------------|-----------------|
| Plataforma   | picoCTF 2021   |
| Categoria    | Cryptography   |
| Dificuldade  | Medium         |
| Autor        | Sara           |
| Status       | ✅ Resolvido   |

## Sumário
- [Cylab - Mini RSA](#cylab---mini-rsa)
  - [Sumário](#sumário)
  - [Desafio](#desafio)
  - [Análise](#análise)
  - [Resolução](#resolução)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Desafio

> What happens if you have a small exponent? There is a twist though, we padded the plaintext so that `(M ** e)` is just barely larger than N. Let's decrypt this:

O desafio disponibilizou um arquivo chamado `values` contendo os parâmetros públicos do RSA e o texto cifrado:

```
N: 1615765684321463054078226051959887884233678317734892901740763321135213636796075462401950274602405095138589898087428337758445013281488966866073355710771864671726991918706558071231266976427184673800225254531695928541272546385146495736420261815693810544589811104967829354461491178200126099661909654163542661541699404839644035177445092988952614918424317082380174383819025585076206641993479326576180793544321194357018916215113009742654408597083724508169216182008449693917227497813165444372201517541788989925461711067825681947947471001390843774746442699739386923285801022685451221261010798837646928092277556198145662924691803032880040492762442561497760689933601781401617086600593482127465655390841361154025890679757514060456103104199255917164678161972735858939464790960448345988941481499050248673128656508055285037090026439683847266536283160142071643015434813473463469733112182328678706702116054036618277506997666534567846763938692335069955755244438415377933440029498378955355877502743215305768814857864433151287

e: 3

ciphertext (c): 5709720175026317841944505166332182779419760031115432215563425901875620052791608163398750297304277886495537367793006288112991932717211066611899319963701838062828209254458935983141207100322897221021063116398621664612418813778554568264340430225832578491114653771121708580262264085968492732626559392615882541267670045570974444907542776294075571558085100930760760722416252378077610954367547852126659348372505030936460878940046808104816042562380177143194070773417030106158693436848897733795588156153439353374354378946281556140400056185498147808737766921652179963585037964078150136578302230878368862263000059181416416783313808950923297557514158129515879758935765781055731969330005473022370060498393105532201494698762244481915406880607127300210095672274400200661881861043562650754163743246330438392531433701024389754537178000821754581449881408420282753987132115940027280372196160429387747427183641264436940827271631231510105131701780747726012636466088768725514820344357055715523621362143648027363044170885031058505704
```

Abri o arquivo com um editor de texto puro (bloco de notas) para extrair os três valores: `N`, `e` e `c`.

---

## Análise

A descrição já dava uma dica direta: o expoente público `e` é incomumente pequeno (**e = 3**), e o texto plano foi "paddado" (preenchido) de forma que `M^e` fica apenas um pouco maior que `N`.

Esse é um cenário clássico para um **ataque de expoente público baixo em RSA**: quando `e` é pequeno e a mensagem não tem um padding forte o suficiente, `M^e` pode não "dar a volta" (wrap around) muitas vezes no módulo `N` — nesse caso, apenas uma vez —, o que torna possível recuperar o texto plano sem precisar fatorar `N` ou descobrir a chave privada; uma simples extração de raiz cúbica (ajustada para o módulo) costuma ser suficiente.

---

## Resolução

Em vez de implementar o ataque manualmente, usei a ferramenta [RSA Cipher do dCode](https://www.dcode.fr/rsa-cipher), colando os valores em seus respectivos campos:

- **N** → o módulo
- **e** → o expoente público (3)
- **c** → o texto cifrado

A ferramenta decifrou o texto diretamente, revelando a flag.

---

## Flag

```
picoCTF{e_sh0u1d_b3_lArg3r_92f4d5a5}
```

---

## Lições aprendidas

- Usar um expoente público pequeno como `e = 3` sem um padding adequado (ex: OAEP) é uma fraqueza conhecida do RSA — a própria flag faz essa referência (*"e should be larger"*).
- Quando `M^e` é apenas um pouco maior que `N`, o texto plano geralmente pode ser recuperado com técnicas simples de extração de raiz, em vez de um ataque completo de fatoração, já que a redução modular só "dá a volta" um pequeno número de vezes.
- Esse desafio é um bom lembrete de que a segurança do RSA não depende só do tamanho da chave (`N`) — escolhas de parâmetros como `e` e esquemas de padding adequados são igualmente críticos.
