---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]

---
# Resolviendo la máquina TakeOver

>En esta publicación, comparto cómo resolví la máquina **TakeOver** de [TryHackMe](https://tryhackme.com/room/takeover).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.141.85`

![[Pasted image 20250724090150.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.141.85 -oG allPorts`

![[Pasted image 20250724090234.png]]

`nmap -p22,80,443 -sCV 10.10.141.85`

![[Pasted image 20250724090353.png]]
### HTTP
#### Fuzzing Web

Se añade al `/etc/hosts`.

`echo "10.10.141.85 futurevera.thm" >> /etc/hosts`

`gobuster dir -u futurevera.thm -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250724090723.png]]

`dirsearch -u futurevera.thm -t 50 -i 200`

![[Pasted image 20250724090751.png]]

`dirb https://futurevera.thm/`

![[Pasted image 20250724090824.png]]

`wfuzz -c --hl=0 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.futurevera.thm" -u 10.10.141.85`

![[Pasted image 20250724090855.png]]

Se añaden al `/etc/hosts`.

`echo "10.10.141.85 portal.futurevera.thm" >> /etc/hosts`

`echo "10.10.141.85 payroll.futurevera.thm" >> /etc/hosts`

`gobuster dir -u portal.futurevera.thm -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250724091031.png]]

`dirsearch -u portal.futurevera.thm -t 50 -i 200`

![[Pasted image 20250724091146.png]]

`dirb https://portal.futurevera.thm/`

![[Pasted image 20250724091203.png]]

`gobuster dir -u payroll.futurevera.thm -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250724091244.png]]

`dirsearch -u payroll.futurevera.thm -t 50 -i 200`

![[Pasted image 20250724091316.png]]

`dirb https://payroll.futurevera.thm/`

![[Pasted image 20250724091340.png]]

Se procede a revisar todos los directorios encontrados:
- `futurevera.thm`.
	- `/index.html`.
	- `/assets`.
	- `/js`.
	- `/css`.
	- `/server-status`.
- `portal.futurevera.thm`.
	- `/index.html`.
	- `/assets`.
	- `/js`.
	- `/css`.
	- `/server-status`.
- `payroll.futurevera.thm/`.
	- `/index.html`.
	- `/assets`.
	- `/js`.
	- `/css`.
	- `/server-status`.

Al no encontrar nada, se procede a mirar información en el certificado.

![[Pasted image 20250724091752.png]]

No se encuentra nada, se procede a volver a realizar *fuzzing web* con la herramienta *ffuf*.

`ffuf -u https://10.10.141.85/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.futurevera.thm" -fs 4605 `

![[Pasted image 20250724091937.png]]

Se añaden al `/etc/hosts`.

`echo "10.10.141.85 blog.futurevera.thm" >> /etc/hosts`

`echo "10.10.141.85 support.futurevera.thm" >> /etc/hosts`

Se procede a mirar información en el certificado.

![[Pasted image 20250724092110.png]]

![[Pasted image 20250724092143.png]]

Se añade al `/etc/hosts`.

`echo "10.10.141.85 secrethelpdesk934752.support.futurevera.thm" >> /etc/hosts`

`http://secrethelpdesk934752.support.futurevera.thm`

![[Pasted image 20250724092324.png]]

---