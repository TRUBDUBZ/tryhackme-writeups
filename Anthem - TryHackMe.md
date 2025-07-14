[[CTF Notes]]

---

## Target IP 

```
10.10.94.51
```

---
## Nmap

```bash
nmap -Pn 10.10.94.51
```

#### Results:

```pgsql
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
```

---

## Recon

Visit: `http://10.10.94.51/robots.txt`

```bash
UmbracoIsTheBest!

# Define the directories not to crawl
Disallow: /bin/
Disallow: /config/
Disallow: /umbraco/
Disallow: /umbraco_client/
```

At the top of the page, we find a **password**: `UmbracoIsTheBest!`

View source across various pages — find **3 flags** embedded in HTML.

Final flag via full site mirroring:

```bash
wget --recursive http://10.10.94.51
cd 10.10.94.51
grep -R 'THM'
```

We find the name of the administrator by googling a cryptic poem left on the site: `Solomon Grundy`

We can deduce his email based on another persons email on the site: 

```css
SG@anthem.com 
```

---

### RDP Access

Use `rdesktop` to connect:

```bash
rdesktop -u SG
```

Login using:

	Username: SG
	
	Password: `UmbracoIsTheBest!`

Once logged in:

	 Grab `user.txt` from Desktop
	 
	Explore `C:\backup` (hidden folder)
	
	Inside: `restore.txt`


---

### Escalation

`restore.txt` is unreadable at first — **edit file permissions** to give yourself read access.

Inside is the **administrator password**.

Run **PowerShell as Administrator**.

Navigate to Administrator’s Desktop:

```powershell
cd 'C:\Users\Administrator\Desktop'
```

```powershell
.\root.txt
```



