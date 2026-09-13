# HTB - Starting Point - Meow

🇧🇷 [Versão em Português](./README.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | Hack The Box  |
| Track        | Starting Point|
| Difficulty   | Very Easy     |
| OS           | Linux         |
| Status       | ✅ Completed  |

## Table of Contents
- [HTB - Starting Point - Meow](#htb---starting-point---meow)
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

![Target IP](./assets/01-target-ip.png)

**Target IP:** `10.129.118.109`

I then connected to the Pwnbox (HTB's attack machine) and, through it, connected to the target's network via VPN:

![Pwnbox connected](./assets/02-pwnbox-connected.png)

I copied the target IP (via clipboard) to use directly in the Pwnbox terminal.

### Connectivity check

To confirm the target was reachable, I ran a `ping`:

```bash
ping 10.129.118.109
```

![Ping target](./assets/03-ping-target.png)

The host responded normally, confirming it was online and reachable over the VPN.

---

## Enumeration

With connectivity confirmed, I ran an **Nmap** scan to identify open ports and services on the target:

```bash
nmap 10.129.118.109
```

![Nmap scan](./assets/04-nmap-scan.png)

Only port **23/tcp (telnet)** was open on the target.

Since I wasn't sure how to properly interact with the service, I checked the telnet command's usage:

```bash
telnet --usage
```

![Telnet usage](./assets/05-telnet-usage.png)

This showed that it's possible to connect directly by specifying the target's IP and port.

---

## Exploitation

I connected to the target using telnet on port 23:

```bash
telnet 10.129.118.109 23
```

![Telnet login](./assets/06-telnet-login.png)

I first tried a generic credential:

- **User:** `admin` / **Password:** `admin` → ❌ Login incorrect

I then tried the default Linux administrator account:

- **User:** `root` → ✅ Login successful (no password required)

Once logged in, I listed the files in the current directory and read the flag:

```bash
ls
cat flag.txt
```

![Flag](./assets/07-flag.png)

---

## Flag

```
b40abdfe23665f766f9c61ecba8a4c19
```

---

## Lessons Learned

- Telnet is a plaintext protocol (no encryption), which is already a security risk on its own.
- Allowing passwordless login for the `root` user is a critical misconfiguration — a strong reminder of why privileged accounts must always require strong authentication.
- Reinforces the importance of always enumerating ports and services before attempting any exploitation.
