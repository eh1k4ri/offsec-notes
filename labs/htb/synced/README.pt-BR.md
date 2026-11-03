# HTB - Starting Point - Synced

🇺🇸 [English version](./README-synced.md)

| Info        | Detalhe             |
|-------------|----------------------|
| Plataforma  | Hack The Box         |
| Track       | Starting Point       |
| Dificuldade | Muito Fácil          |
| OS          | Linux                |
| Status      | ✅ Resolvido          |

## Sumário
- [HTB - Starting Point - Synced](#htb---starting-point---synced)
  - [Sumário](#sumário)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Recon

Iniciei a máquina alvo pela plataforma do HTB, obtendo o endereço de IP do target:

**IP do alvo:** `10.129.228.37`

Em seguida, conectei na Pwnbox (máquina de ataque fornecida pelo HTB) e, através dela, me conectei na rede do target via VPN.

Rodei um scan completo de portas com detecção de serviço/versão para identificar portas e serviços abertos no alvo:

```bash
nmap -sV -p- 10.129.228.37
```

```
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

Apenas a porta **873** estava aberta, rodando **rsync** — um daemon usado para sincronização e transferência eficiente de arquivos, que pode ser configurado com "módulos" (compartilhamentos) acessíveis publicamente.

---

## Enumeration

Verifiquei o menu de ajuda do cliente `rsync` para revisar as opções disponíveis:

```bash
rsync -h
```

Primeiro tentei conectar diretamente ao daemon sem especificar uma fonte, o que falhou por falta de argumentos obrigatórios:

```bash
rsync --address 10.129.228.37 --port 873
```

```
rsync error: syntax or usage error (code 1) at main.c(1837) [Receiver=3.4.1]
```

A forma correta de listar os módulos disponíveis em um daemon rsync é através da sintaxe de URL `rsync://`. Listei os módulos expostos pelo daemon:

```bash
rsync rsync://10.129.228.37/
```

```
public         	Anonymous Share
```

Um módulo chamado **`public`** foi listado, com a descrição "Anonymous Share". Tentei erroneamente conectar usando o texto da descrição em vez do nome real do módulo:

```bash
rsync rsync://10.129.228.37/Anonymous
```

```
@ERROR: Unknown module 'Anonymous'
```

Corrigi isso e listei o conteúdo do módulo real `public`:

```bash
rsync rsync://10.129.228.37/public/
```

```
drwxr-xr-x          4,096 2022/10/24 18:02:23 .
-rw-r--r--             33 2022/10/24 17:32:03 flag.txt
```

Um arquivo `flag.txt` estava presente dentro do compartilhamento `public`, acessível **sem exigir nenhuma autenticação** (módulo anônimo).

---

## Exploitation

Baixei o arquivo para minha máquina local, especificando um caminho de destino (`.` para o diretório atual):

```bash
rsync rsync://10.129.228.37/public/flag.txt .
```

Em seguida, li o arquivo localmente:

```bash
cat flag.txt
```

---

## Flag

```
72eaf5344ebb84908ae543a719830519
```

---

## Lições aprendidas

- Daemons rsync podem expor "módulos" (compartilhamentos) que não exigem autenticação — conectar via esquema de URL `rsync://` sem credenciais é suficiente quando um módulo está configurado para acesso anônimo.
- O nome de exibição/descrição do módulo (ex: "Anonymous Share") é apenas texto descritivo — o nome real usado para conectar é o nome do módulo exibido na coluna mais à esquerda (`public` neste caso).
- Listar um módulo (`rsync rsync://host/modulo/`) apenas mostra seu conteúdo; um caminho de destino precisa ser fornecido para realmente baixar arquivos (`rsync rsync://host/modulo/arquivo .`).
- Sempre rode um scan completo de portas (`-p-`), já que serviços como rsync (porta 873) não fazem parte das portas mais comuns do Nmap em todos os cenários, e podem ser o único ponto de entrada em um host.
