---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - HTTP
  - CMS
  - WordPress
  - RDP
  - PrivilegeEscalation
  - AbuseUAC
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
		- [[#Escalada de Privilegios#CVE-2019-1388|CVE-2019-1388]]

---
# Blaster

![](Screenshots/07ad91ce2628efb8a2a3c5fddaeac710.png)

> En este **Write-Up** se documenta la resolución paso a paso de la máquina **Blaster** de [TryHackMe](https://tryhackme.com/room/blaster), clasificada como dificultad _Easy_. Se cubren las fases de **enumeración**, **explotación** y **escalada de privilegios**.
> Es la continuación de la máquina **Retro** de [TryHackMe](https://tryhackme.com/room/retro), dejo mi **Write-Up**: [Retro - Write-Up (TryHackMe)](https://medium.com/@manum0re/retro-write-up-tryhackme-d065b7c97649).

---
## Enumeración
### Ping

`ping -c 1 10.81.147.134`

![](Screenshots/Pasted%20image%2020251207182724.png)

Al darme error el ping, pruebo con *Nmap* para saber el **TTL** de la máquina.
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.81.147.134 -oG allPorts`

![](Screenshots/Pasted%20image%2020251207182927.png)

`nmap -p80,3389 -Pn -sCV 10.81.147.134 -oN targeted`

![](Screenshots/Pasted%20image%2020251207182958.png)
### HTTP

`http://10.81.147.134/`

![](Screenshots/Pasted%20image%2020251207183321.png)

Es curioso pero el **CMS** que nos indica es *WordPress*, pero la web que nos encontramos es un *Internet Information Services (ISS)*.
#### Fuzzing Web

`gobuster dir -u http://10.81.147.134 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251207183640.png)

`http://10.81.147.134/retro/`

![](Screenshots/Pasted%20image%2020251207183726.png)
#### WPScan

`wpscan --url http://10.81.147.134/retro/ --enumerate`

![](Screenshots/Pasted%20image%2020251207183844.png)
![](Screenshots/Pasted%20image%2020251207183911.png)

Se encuentra el usuario `wade`, se procede a realizar *fuerza bruta* para averiguar la contraseña.

`wpscan --url http://10.81.147.134/retro/ -U wade -P /usr/share/wordlists/rockyou.txt`

No se encuentra la contraseña, se procede a realizar una búsqueda por la web.

En la sección de comentarios del usuario `Wade`, se encuentra una contraseña.

![](Screenshots/Pasted%20image%2020251207184116.png)

---
## Explotación
### RDP

`xfreerdp3 /u:wade /p:parzival /v:10.81.147.134`

![](Screenshots/Pasted%20image%2020251208182143.png)

Encontramos la *flag* de usuario, además de ver un ejecutable llamado `hhupd`.
### Escalada de Privilegios
#### CVE-2019-1388

Buscando el archivo, se encuentra una vulnerabilidad [CVE-2019-1388](https://github.com/nobodyatall648/CVE-2019-1388) y como explotarla.

Se ejecuta el archivo con permisos de *administrador*.

![](Screenshots/Pasted%20image%2020251208182446.png)

Se muestra la información del certificado.

![](Screenshots/Pasted%20image%2020251208182516.png)

Se visita el link del certificado y se guarda el archivo siguiendo la siguiente ruta: `Tools/File/Save as`.

![](Screenshots/Pasted%20image%2020251208182622.png)

Y se pone lo siguiente: `C:\Windows\System32\cmd.exe`.

![](Screenshots/Pasted%20image%2020251208183200.png)

Se ejecuta y se nos abre un **cmd** con permisos de *Administrator*.

![](Screenshots/Pasted%20image%2020251208183244.png)

La *flag* de **Root** se encuentra en la ruta: `C://Users/Administrator/Desktop/root.txt`.

---